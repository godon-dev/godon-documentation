---
description: "godon Getting Started — run your first interference detection bench and see the system detect optimizer interference in a microgrid scenario."
---

## Getting Started

This guide walks you through running a complete interference detection bench — no scenario design needed. You'll trigger two optimizers on a shared microgrid simulation, watch them collect trials, and see the observer detect interference between them.

The whole process takes about 30 minutes.

### Prerequisites

A running godon stack with:

- Observer (collects trial data, runs detection)
- Two or more breeders (optimization drivers)
- Access to the bench workflow (GitHub Actions)

If you haven't set up the stack yet, see [Setup](setup.md).

### Step 1: Run the Coupled Bench

Trigger the microgrid bench with strong coupling. From the godon repository, run:

```bash
gh workflow run bench-scenario-microgrid.yml \
  --ref main \
  -f min_trials=300 \
  -f max_wait_minutes=90 \
  -f coupling_factor=0.9
```

This creates two breeders optimizing a shared microgrid simulation with 90% coupling — one optimizer's decisions strongly affect the other's outcomes.

Monitor progress:

```bash
gh run watch --exit-status
```

### Step 2: Watch the Breeders

While the bench runs, check the breeders' progress through the observer API:

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

Both breeders are running and collecting trials. Each trial is one optimization step — parameter values proposed, simulator evaluated, objectives recorded.

### Step 3: Run Detection

Once both breeders have 100+ trials, run interference detection. You need the breeder UUIDs from the previous step:

```bash
kubectl exec -n godon deploy/godon-godon-observer -- \
  wget -qO- 'http://localhost:8089/api/watermark-detection/{SENDER_UUID}/{RECEIVER_UUID}'
```

The response shows detection results per objective:

```json
{
  "detected": true,
  "per_objective": [
    {"objective_index": 0, "detected": true, "fft": {"p_value": 0.0004}},
    {"objective_index": 1, "detected": true, "fft": {"p_value": 0.0002}},
    {"objective_index": 2, "detected": true, "fft": {"p_value": 0.0002}},
    {"objective_index": 3, "detected": false, "fft": {"p_value": 0.88}}
  ]
}
```

**What this means:** Objectives 0-2 show detected interference (p < 0.05). Objective 3 (energy cost) is clean — this objective is not affected by the coupling channel. The observer detected that Optimizer A's watermark appears in Optimizer B's objectives 0-2, proving interference through the shared microgrid.

### Step 4: Verify with No Coupling

To confirm the detection is real (not false positives), run the same bench with zero coupling:

```bash
gh workflow run bench-scenario-microgrid.yml \
  --ref main \
  -f min_trials=300 \
  -f max_wait_minutes=90 \
  -f coupling_factor=0.0
```

After 100+ trials, run detection again. This time you should see:

```json
{
  "detected": false,
  "per_objective": [
    {"objective_index": 0, "detected": false, "fft": {"p_value": 0.90}},
    {"objective_index": 1, "detected": false, "fft": {"p_value": 1.00}},
    {"objective_index": 2, "detected": false, "fft": {"p_value": 0.27}},
    {"objective_index": 3, "detected": false, "fft": {"p_value": 0.99}}
  ]
}
```

**No interference detected.** All p-values well above 0.05. This confirms the detection is discriminating — it fires when coupling exists and stays quiet when it doesn't.

### Understanding the Results

| Field | Meaning |
|-------|---------|
| `detected` | Overall: true if any objective shows interference |
| `per_objective[].detected` | Per-objective detection result |
| `fft.p_value` | Permutation test p-value (p < 0.05 = significant) |
| `fft.combined_power` | Spectral power at watermark frequencies |
| `aligned_trials` | Number of timestamp-aligned trials used |

The detection uses FFT (Fast Fourier Transform) to measure spectral power at the sender's known watermark frequencies, then compares against 5000 random shuffles to determine statistical significance. See [Interference Detection](concept_interference_detection.md) for the full methodology.

### Varying Coupling Strength

The coupling_factor parameter lets you explore the detection's sensitivity:

| Coupling | What to expect |
|----------|---------------|
| 0.0 | No detection — clean baseline |
| 0.1 | Partial detection — some objectives may trigger |
| 0.5 | Most coupled objectives detected |
| 0.9 | Strong detection on all coupled objectives |

Try different values and observe how the p-values and detection results change.

### Next Steps

- Read the [Interference Detection](concept_interference_detection.md) concept page for the methodology
- Explore the [Breeder](concept_breeder.md) configuration for watermark parameters
- Check [Architecture](architecture.md) for system component details
