---
description: "godon Getting Started — run your first interference detection bench locally with godon-cli, see the system detect optimizer interference in a microgrid scenario."
---

## Getting Started

This guide walks you through running a complete interference detection bench on your local godon stack. No scenario design needed — you'll start two coupled microgrid simulators, create two optimizers, and watch the observer detect interference between them.

The whole process takes about 30 minutes.

### Prerequisites

A running godon stack. If you haven't set it up yet, see [Setup](setup.md). You'll need:

- KinD cluster with the godon helm chart deployed
- `kubectl` configured with the KinD kubeconfig
- Docker (for the microgrid simulators and godon-cli)
- `jq` (for parsing API responses)

### Step 1: Start the Microgrid Simulators

The bench uses two coupled microgrid simulators — lightweight HTTP services that model a power grid with configurable coupling between instances.

From the godon repository root:

```bash
# Start with 90% coupling (strong interference)
export COUPLING_FACTOR=0.9
docker compose -f examples/bench/scenario-microgrid/docker-compose.yml -p bench-mg up -d
```

Wait for both simulators to be healthy:

```bash
for port in 8090 8091; do
  until curl -s -f http://127.0.0.1:${port}/health; do sleep 2; done
  echo "Port ${port}: healthy"
done
```

If your godon stack runs in KinD, connect the simulators to the kind network:

```bash
MG1_CID=$(docker compose -f examples/bench/scenario-microgrid/docker-compose.yml -p bench-mg ps -q microgrid-1)
MG2_CID=$(docker compose -f examples/bench/scenario-microgrid/docker-compose.yml -p bench-mg ps -q microgrid-2)

docker network connect kind $MG1_CID 2>/dev/null || true
docker network connect kind $MG2_CID 2>/dev/null || true

MG1_IP=$(docker inspect $MG1_CID -f '{{.NetworkSettings.Networks.kind.IPAddress}}')
MG2_IP=$(docker inspect $MG2_CID -f '{{.NetworkSettings.Networks.kind.IPAddress}}')
echo "MG1: ${MG1_IP}:8090, MG2: ${MG2_IP}:8090"
```

### Step 2: Port-Forward the Godon API

Make the godon API accessible from your host:

```bash
export KUBECONFIG=/tmp/kind_kubeconfig.yaml
API_POD=$(kubectl get pods -n godon -l component=api -o jsonpath='{.items[0].metadata.name}')
kubectl port-forward -n godon --address 127.0.0.3 "${API_POD}" 9090:8080 &
sleep 3
```

Verify:

```bash
curl -s http://127.0.0.3:9090/health
```

### Step 3: Create Targets

Targets represent the systems being optimized. Create one for each microgrid:

```bash
CLI_IMAGE="ghcr.io/godon-dev/godon-cli:latest"
API_HOST="127.0.0.3"
API_PORT="9090"

# Create target for microgrid-1
cat examples/bench/scenario-microgrid/targets/microgrid-1.yaml | \
  sed -e 's/TARGET_NAME/bench-mg-1/' -e "s|TARGET_URL|http://${MG1_IP}:8090|" > /tmp/target-mg1.yaml

docker run --rm --network host \
  -v /tmp:/work:ro -w /work \
  "${CLI_IMAGE}" --hostname=${API_HOST} --port=${API_PORT} --insecure \
  target create --file=/work/target-mg1.yaml

# Create target for microgrid-2
cat examples/bench/scenario-microgrid/targets/microgrid-2.yaml | \
  sed -e 's/TARGET_NAME/bench-mg-2/' -e "s|TARGET_URL|http://${MG2_IP}:8090|" > /tmp/target-mg2.yaml

docker run --rm --network host \
  -v /tmp:/work:ro -w /work \
  "${CLI_IMAGE}" --hostname=${API_HOST} --port=${API_PORT} --insecure \
  target create --file=/work/target-mg2.yaml
```

Get the target IDs:

```bash
TARGET_1_ID=$(curl -s http://${API_HOST}:${API_PORT}/targets | jq -r '.[] | select(.name=="bench-mg-1") | .id')
TARGET_2_ID=$(curl -s http://${API_HOST}:${API_PORT}/targets | jq -r '.[] | select(.name=="bench-mg-2") | .id')
echo "Target 1: ${TARGET_1_ID}"
echo "Target 2: ${TARGET_2_ID}"
```

### Step 4: Create Breeders

Breeders are the optimization drivers. Each one runs a search against one microgrid, with watermark injection enabled for interference detection:

```bash
# Prepare breeder-1 config (points to microgrid-1)
cat examples/bench/scenario-microgrid/breeders/breeder-1.yml | \
  sed -e "s/\"microgrid-1\"/\"${TARGET_1_ID}\"/" \
      -e "s/\"microgrid-2\"/\"${TARGET_1_ID}\"/" \
      -e "s|http://microgrid-1:8090|http://${MG1_IP}:8090|" \
      -e "s|http://microgrid-2:8090|http://${MG1_IP}:8090|" > /tmp/breeder-mg1.yaml

docker run --rm --network host \
  -v /tmp:/work -w /work \
  "${CLI_IMAGE}" --hostname=${API_HOST} --port=${API_PORT} --insecure \
  breeder create --name=bench-mg-1 --file=/work/breeder-mg1.yaml

# Prepare breeder-2 config (points to microgrid-2)
cat examples/bench/scenario-microgrid/breeders/breeder-2.yml | \
  sed -e "s/\"microgrid-1\"/\"${TARGET_2_ID}\"/" \
      -e "s/\"microgrid-2\"/\"${TARGET_2_ID}\"/" \
      -e "s|http://microgrid-1:8090|http://${MG2_IP}:8090|" \
      -e "s|http://microgrid-2:8090|http://${MG2_IP}:8090|" > /tmp/breeder-mg2.yaml

docker run --rm --network host \
  -v /tmp:/work -w /work \
  "${CLI_IMAGE}" --hostname=${API_HOST} --port=${API_PORT} --insecure \
  breeder create --name=bench-mg-2 --file=/work/breeder-mg2.yaml
```

Get the breeder IDs:

```bash
BREEDER_1_UUID=$(curl -s http://${API_HOST}:${API_PORT}/breeders | jq -r '.[] | select(.name=="bench-mg-1") | .id')
BREEDER_2_UUID=$(curl -s http://${API_HOST}:${API_PORT}/breeders | jq -r '.[] | select(.name=="bench-mg-2") | .id')
echo "Breeder 1: ${BREEDER_1_UUID}"
echo "Breeder 2: ${BREEDER_2_UUID}"
```

### Step 5: Watch Optimization Progress

The breeders are now running. Check progress through the observer:

```bash
kubectl exec -n godon deploy/godon-godon-observer -- \
  wget -qO- 'http://localhost:8089/api/breeders'
```

You'll see something like:

```json
[
  {"name": "bench-mg-1", "status": "active", "total_trials": 145},
  {"name": "bench-mg-2", "status": "active", "total_trials": 138}
]
```

Wait until both breeders have 100+ trials before running detection. Each trial takes a few seconds — expect 15-20 minutes to reach 200+ trials.

### Step 6: Run Interference Detection

Query the observer to detect interference between the two breeders:

```bash
kubectl exec -n godon deploy/godon-godon-observer -- \
  wget -qO- "http://localhost:8089/api/watermark-detection/${BREEDER_1_UUID}/${BREEDER_2_UUID}"
```

The response shows detection results per objective:

```json
{
  "detected": true,
  "per_objective": [
    {"objective_index": 0, "detected": true,  "fft": {"p_value": 0.0004}},
    {"objective_index": 1, "detected": true,  "fft": {"p_value": 0.0002}},
    {"objective_index": 2, "detected": true,  "fft": {"p_value": 0.0002}},
    {"objective_index": 3, "detected": false, "fft": {"p_value": 0.88}}
  ]
}
```

Objectives 0-2 show interference (p < 0.05). Objective 3 (energy cost) is clean — it's not affected by the coupling channel in this scenario. The observer detected Breeder 1's watermark in Breeder 2's outcomes, proving interference through the shared microgrid.

### Step 7: Verify with No Coupling

Confirm the detection is real by running with zero coupling. First, clean up the existing breeders:

```bash
docker run --rm --network host "${CLI_IMAGE}" \
  --hostname=${API_HOST} --port=${API_PORT} --insecure \
  breeder purge --force --id="${BREEDER_1_UUID}"

docker run --rm --network host "${CLI_IMAGE}" \
  --hostname=${API_HOST} --port=${API_PORT} --insecure \
  breeder purge --force --id="${BREEDER_2_UUID}"
```

Restart the microgrids without coupling:

```bash
docker compose -f examples/bench/scenario-microgrid/docker-compose.yml -p bench-mg down -v
export COUPLING_FACTOR=0.0
docker compose -f examples/bench/scenario-microgrid/docker-compose.yml -p bench-mg up -d
```

Then repeat Steps 1 (network reconnect), 4 (create breeders), 5 (wait for trials), and 6 (run detection). This time you should see all objectives clean — no interference detected.

### Step 8: Clean Up

When done, tear everything down:

```bash
# Delete breeders and targets
docker run --rm --network host "${CLI_IMAGE}" \
  --hostname=${API_HOST} --port=${API_PORT} --insecure \
  breeder purge --force --id="${BREEDER_1_UUID}"
docker run --rm --network host "${CLI_IMAGE}" \
  --hostname=${API_HOST} --port=${API_PORT} --insecure \
  breeder purge --force --id="${BREEDER_2_UUID}"

docker run --rm --network host "${CLI_IMAGE}" \
  --hostname=${API_HOST} --port=${API_PORT} --insecure \
  target delete --id="${TARGET_1_ID}"
docker run --rm --network host "${CLI_IMAGE}" \
  --hostname=${API_HOST} --port=${API_PORT} --insecure \
  target delete --id="${TARGET_2_ID}"

# Stop simulators
docker compose -f examples/bench/scenario-microgrid/docker-compose.yml -p bench-mg down -v
```

### Understanding the Results

| Field | Meaning |
|-------|---------|
| `detected` | Overall: true if any objective shows interference |
| `per_objective[].detected` | Per-objective detection result |
| `fft.p_value` | Permutation test p-value (p < 0.05 = significant) |
| `fft.combined_power` | Spectral power at watermark frequencies |
| `aligned_trials` | Number of timestamp-aligned trials used |

The detection uses FFT to measure spectral power at the sender's known watermark frequencies, then compares against 5000 random shuffles. See [Interference Detection](concept_interference_detection.md) for the full methodology.

### Varying Coupling Strength

The `COUPLING_FACTOR` environment variable controls interference strength:

| Coupling | What to expect |
|----------|---------------|
| 0.0 | No detection — clean baseline |
| 0.1 | Partial detection — some objectives may trigger |
| 0.5 | Most coupled objectives detected |
| 0.9 | Strong detection on all coupled objectives |

Try different values and observe how p-values change.

### Next Steps

- Read the [Interference Detection](concept_interference_detection.md) concept page for methodology and validation data
- Explore the [Breeder](concept_breeder.md) configuration for watermark and optimization parameters
- Check [Architecture](architecture.md) for system component details
