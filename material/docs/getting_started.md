<!--
Copyright (c) 2019 Matthias Tafelmeier.

This file is part of godon

godon is free software: you can redistribute it and/or modify
it under the terms of the GNU Affero General Public License as
published by the Free Software Foundation, either version 3 of the
License, or (at your option) any later version.

godon is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU Affero General Public License for more details.

You should have received a copy of the GNU Affero General Public License
along with this godon. If not, see <http://www.gnu.org/licenses/>.
-->

## Getting Started

Everything in this walkthrough runs on **your machine**: your cluster, your bench simulator, your agents. No godon-dev credentials, no CI. When it's done you will have planted a coupling edge into a simulator with known ground truth, let two autonomous optimizers discover and measure it through the shared substrate, and read the measured response curves back out.

Budget **45-90 minutes**, most of it waiting while the agents work.

A running godon stack. If you haven't set it up yet, see [Setup](setup.md). You'll need:

- Linux host with the godon helm chart deployed on a kind cluster ([Setup](setup.md))
- `kubectl` configured with the kind kubeconfig
- Docker (for the bench simulator and the godon CLI)
- `jq` and `curl`

The walkthrough uses the **generic bench** — a configurable synthetic coupling simulator with known ground truth. You plant the coupling (topology, strength, shape, noise), the engine discovers it, and you compare. Two autonomous optimizers run against two coupled bench nodes; each one discovers the other through the shared substrate.

### Step 1: Start the Bench Simulator

The bench runs as a single container exposing two virtual nodes with a planted coupling edge:

```bash
docker create --name bench-generic \
  --user 0:0 -p 8090:8090 \
  -e CONFIG_PATH=/topology.yaml -e PORT=8090 \
  -e GENERIC_SEED=42 -e RUST_LOG=info \
  ghcr.io/godon-dev/godon-bench-generic:0.2.1

# Plant the coupling: node-1 -> node-2, strength 0.7, saturation shape
cat > /tmp/topology.yaml <<'EOF'
nodes:
  - id: node-1
    params: 3
    objectives: 2
    base: saturation
    param_lower: 0.0
    param_upper: 100.0
    weights: [[0.0, 1.0, 0.0], [0.3, 0.2, 0.5]]
  - id: node-2
    params: 3
    objectives: 2
    base: linear
    param_lower: 0.0
    param_upper: 100.0
    weights: [[0.4, 0.3, 0.3], [0.2, 0.5, 0.3]]
edges:
  - from: node-1
    from_channel: 0
    to: node-2
    to_channel: 0
    strength: 0.7
noise:
  gaussian_sigma: 0.02
  colored_sigma: 0.0
  drift_rate: 0.0
EOF

docker cp /tmp/topology.yaml bench-generic:/topology.yaml
docker start bench-generic
docker network connect kind bench-generic 2>/dev/null || true
BENCH_IP=$(docker inspect bench-generic -f '{{.NetworkSettings.Networks.kind.IPAddress}}')
echo "Bench at: ${BENCH_IP}:8090"
curl -s http://127.0.0.1:8090/health | jq .
```

The `/health` response lists the nodes and edge count — your ground truth for what the engine should find.

The `docker network connect kind` line matters: it puts the bench on the same docker network as your kind nodes, so the breeders (which run inside the cluster) can reach it at `BENCH_IP`. Keep that variable — the configs in step 3 need it.

### Step 2: Port-Forward the Godon API

```bash
export KUBECONFIG=<path to your kind kubeconfig>
API_POD=$(kubectl get pods -n godon -l component=api -o jsonpath='{.items[0].metadata.name}')
kubectl port-forward -n godon --address 127.0.0.3 "${API_POD}" 9090:8080 &
sleep 3
curl -s http://127.0.0.3:9090/health
```

`127.0.0.3` instead of `127.0.0.1` avoids colliding with anything already bound to localhost — the loopback range gives you free aliases.

### Step 3: Create Targets and Breeders

Two kinds of object make an agent run: a **target** (the HTTP endpoint it optimizes) and a **breeder config** (its parameter space, objectives, and characterization group). You write both yourself — four small files.

The targets just point at the bench nodes you started in step 1:

```bash
mkdir -p /tmp/godon-scenario && cd /tmp/godon-scenario

cat > target-node-1.yaml <<EOF
name: generic-node-1
targetType: http
spec:
  url: "http://${BENCH_IP}:8090/node-1"
  auth_type: none
EOF

cat > target-node-2.yaml <<EOF
name: generic-node-2
targetType: http
spec:
  url: "http://${BENCH_IP}:8090/node-2"
  auth_type: none
EOF
```

The breeder config is where the experiment lives. Three parts matter: the **search space** (`settings` — three parameters on a 0-100 grid), the **objectives vs observations split** (`objective_0` is optimized, `objective_1` is only watched — the detector reads both), and the **`interference_detection` group** — any two breeders sharing a group name begin coordinating: turn-taking, probing, measuring their coupling.

```bash
cat > breeder-1.yml <<EOF
meta:
  configVersion: "0.3"
  strict_validation: false

breeder:
  type: bench_generic

settings:                       # parameter search space
  generic:
    param_0:
      constraints:
        - {step: 20.0, lower: 0.0, upper: 100.0}
    param_1:
      constraints:
        - {step: 20.0, lower: 0.0, upper: 100.0}
    param_2:
      constraints:
        - {step: 20.0, lower: 0.0, upper: 100.0}

run:
  parallel: 1
  completion_criteria:
    iterations: {min: 10, max: 500}
    timing: {end: "75m"}

cooperation:
  active: false

interference_detection:         # shared group name = these agents measure each other
  group: bench-characterization
  mode: active
  convergence_threshold: 0.02
  refinement_depth: 3
  push_block_size: 10
  pause_block_size: 10
  cooldown_trials: 5
  hold_params: {param_0: 50.0, param_1: 50.0, param_2: 50.0}

reconnaissance:                 # how this breeder reads its target
  type: http
  http:
    url: "http://${BENCH_IP}:8090/node-1"

objectives:
  - name: objective_0
    direction: maximize
    reconnaissance:
      service: http
      path: /metrics/json
      key: objective_0
      samples: 3
      aggregation: median

observations:                   # read but NOT optimized — an extra detection channel
  - name: objective_1
    reconnaissance:
      service: http
      path: /metrics/json
      key: objective_1
      samples: 3
      aggregation: median

rollback_strategies:
  standard:
    consecutive_failures: 10
    target_state: previous
    max_attempts: 3
    on_failure: continue
    timeout_seconds: 60

effectuation:                   # how this breeder writes parameters
  type: http
  targetRefs: ["generic-node-1"]
  endpoint_config:
    method: POST
    path: /apply
    timeout_seconds: 30
EOF

# breeder-2 is the same file pointed at node-2
sed 's/node-1/node-2/g' breeder-1.yml > breeder-2.yml
```

Note how the pieces connect: `targetRefs: ["generic-node-1"]` refers to the target by the `name` you gave it above, and the reconnaissance URL is the same node endpoint. Every knob is documented in the [Configuration Guide](config_guide.md) — the defaults above are the validated characterization setup.

Now create the objects through the API, using the godon CLI (public image from ghcr; it converts the YAML into API calls):

```bash
CLI="docker run --rm --network host -v /tmp/godon-scenario:/work:ro -w /work ghcr.io/godon-dev/godon-cli:latest"
API="--hostname 127.0.0.3 --port 9090 --insecure"

$CLI $API target create --file=target-node-1.yaml
$CLI $API target create --file=target-node-2.yaml

$CLI $API breeder create --name=bench-char-1 --file=breeder-1.yml
$CLI $API breeder create --name=bench-char-2 --file=breeder-2.yml

# Capture the breeder UUIDs — the results in step 5 are keyed by them
BREEDER_1_UUID=$(curl -s http://127.0.0.3:9090/breeders | jq -r '.[] | select(.name=="bench-char-1") | .id')
BREEDER_2_UUID=$(curl -s http://127.0.0.3:9090/breeders | jq -r '.[] | select(.name=="bench-char-2") | .id')
echo "sender: ${BREEDER_1_UUID}, receiver: ${BREEDER_2_UUID}"
```

(The [scenario library](bench_scenarios.md) ships these same files pre-written for many topologies — useful once you know the shape. Typing them once teaches the shape.)

### Step 4: What the Breeders Are Doing

Each breeder is an optimization agent. Beyond optimizing, they coordinate through lease-based turn-taking to characterize their coupling:

1. One breeder acquires the sender role, the other holds still (receiver).
2. The sender runs a **coverage walk** — pushing one parameter at a time across its levels (midpoint, extremes, quarters — a deterministic, complete exploration order).
3. For every push, the causal service measures the receiver's objective shift between push and pause windows, with an uncertainty bar from the raw sample scatter.
4. Re-measurements within bars blend (the point tightens); beyond bars relocate and flag drift.
5. A parameter retires when it is converged **and** every remaining gap in its response curve is priced below the local measurement bar — remaining ignorance cheaper than one more probe.

The result is a **response curve per (sender, parameter, channel)** — the measured shape of the coupling, not just a yes/no edge.

### Step 5: Watch Progress, Read the Results

Progress: check the breeder states and give it time. First probe blocks typically appear within ~15 minutes; a complete carrier curve at these block sizes takes 30-75 minutes.

```bash
curl -s http://127.0.0.3:9090/breeders/${BREEDER_1_UUID} | jq '{status, name}'
```

While the run is in progress (or after), query the causal service directly:

```bash
CAUSAL_POD=$(kubectl get pods -n godon -l component=godon-causal -o jsonpath='{.items[0].metadata.name}')
kubectl port-forward -n godon --address 127.0.0.3 "${CAUSAL_POD}" 9091:8091 &
sleep 3

# Detection verdict for one direction (sender -> receiver)
curl -s http://127.0.0.3:9091/detect/${BREEDER_1_UUID}/${BREEDER_2_UUID} | jq .

# All measured response curves — the coupling map
curl -s http://127.0.0.3:9091/curves | jq '.curves[] | {sender_id, receiver_id, param, channel, points: .state.points}'
```

The `/detect` response reports per-channel rising/falling edges with confidence; `/curves` returns every measured `(level, shift, bar)` triple — level on the sender's parameter grid, measured shift on the receiver's objective, and the uncertainty bar from the raw samples.

For the planted topology above, expect: `param_1` of the sender showing a saturation-shaped curve on the receiver's `objective_0`, `param_0`/`param_2` flat (they carry no weight), and `objective_1` flat (the edge feeds channel 0 only). Curves keyed the other direction (node-2 as sender) stay flat — nothing is planted that way.

### Step 6: Verify with No Coupling

Run the same walkthrough with the edge strength set to `0.0` in `topology.yaml` (or a topology without edges). Expect flat curves everywhere and no detections — the honest blank. Zero false positives across the sweep is a core validated property of the method.

### Step 7: Clean Up

```bash
# Purge the breeders — their measured curves and observation rows are removed with them
$CLI $API breeder purge --force --id=${BREEDER_1_UUID}
$CLI $API breeder purge --force --id=${BREEDER_2_UUID}

# Remove the targets (ids from: curl -s http://127.0.0.3:9090/targets)
curl -s -X DELETE http://127.0.0.3:9090/targets/<target-id>

docker rm -f bench-generic
```

### Understanding the Protocol

The method is the **impulse protocol**: non-destructive perturbation within guardrail bounds, ABA block design (push → pause → compare), and CFAR detection with adaptive thresholds — the same statistical family as radar and sonar. Receiver exploration noise — the dominant interference signal blocker — is eliminated by the receiver holding still during measurement rather than by stronger statistics.

Full methodology and validation: [Interference Detection](concept_interference_detection.md).

### Varying the Planted Reality

The topology file is the experiment design:

| Knob | What it changes |
|------|-----------------|
| `strength` | Edge coupling strength (0.1 floor to 0.9, validated sweep) |
| `base` | Per-node shape: `linear`, `threshold`, `saturation`, `polynomial` |
| `gaussian_sigma` | Measurement noise (0.02 validated; boundary characterized 0.05-0.10) |
| `edges` | Any DAG: chains, stars, multi-agent groups |

### Next Steps

- [Interference Detection](concept_interference_detection.md) — the methodology and its validation boundaries
- [Architecture](architecture.md) — the causal service and the components around it
- [Bench Scenarios](bench_scenarios.md) — the scenario library
