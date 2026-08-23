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

A running godon stack. If you haven't set it up yet, see [Setup](setup.md). You'll need:

- KinD cluster with the godon helm chart deployed
- `kubectl` configured with the KinD kubeconfig
- Docker (for the bench simulator and godon-cli)
- `jq` (for parsing API responses)

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

### Step 2: Port-Forward the Godon API

```bash
export KUBECONFIG=/tmp/kind_kubeconfig.yaml
API_POD=$(kubectl get pods -n godon -l component=api -o jsonpath='{.items[0].metadata.name}')
kubectl port-forward -n godon --address 127.0.0.3 "${API_POD}" 9090:8080 &
sleep 3
curl -s http://127.0.0.3:9090/health
```

### Step 3: Create Targets and Breeders

The easiest path is the characterization workflow, which creates targets and breeders from a scenario directory, runs the protocol, and exports results:

```bash
gh workflow run bench-characterization.yml --repo godon-dev/godon \
  -f scenario=characterization-saturation \
  -f min_trials=350 -f max_wait_minutes=75
```

Scenarios live in [`examples/bench/`](https://github.com/godon-dev/godon/tree/main/examples/bench) — each one ships a `topology.yaml` (ground truth), target definitions, and breeder configs. The workflow:

1. starts the bench container with the scenario's topology,
2. creates one target and one breeder per node,
3. lets the breeders run the interference protocol,
4. exports response curves and coordinator traces to the run logs,
5. cleans up.

To drive it manually instead, create targets and breeders through the API or `godon-cli` exactly as the workflow does (see the workflow source for the exact calls).

### Step 4: What the Breeders Are Doing

Each breeder is an optimization agent. Beyond optimizing, they coordinate through lease-based turn-taking to characterize their coupling:

1. One breeder acquires the sender role, the other holds still (receiver).
2. The sender runs a **coverage walk** — pushing one parameter at a time across its levels (midpoint, extremes, quarters — a deterministic, complete exploration order).
3. For every push, the causal service measures the receiver's objective shift between push and pause windows, with an uncertainty bar from the raw sample scatter.
4. Re-measurements within bars blend (the point tightens); beyond bars relocate and flag drift.
5. A parameter retires when it is converged **and** every remaining gap in its response curve is priced below the local measurement bar — remaining ignorance cheaper than one more probe.

The result is a **response curve per (sender, parameter, channel)** — the measured shape of the coupling, not just a yes/no edge.

### Step 5: Read the Results

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

The `/detect` response reports per-channel rising/falling edges with confidence; `/curves` returns every measured (level, shift, bar) triple. For the planted topology above, expect: `param_1` of the sender showing a saturation-shaped curve on the receiver's `objective_0`, `param_0`/`param_2` flat (they carry no weight), and `objective_1` flat (the edge feeds channel 0 only).

### Step 6: Verify with No Coupling

Run the same scenario with the edge strength set to `0.0` in `topology.yaml` (or use a scenario without edges). Expect flat curves everywhere and no detections — the honest blank. Zero false positives across the sweep is a core validated property of the method.

### Step 7: Clean Up

The workflow purges breeders, targets, and the bench container automatically. Manually:

```bash
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
