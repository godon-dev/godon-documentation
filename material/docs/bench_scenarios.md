---
description: "godon Bench Scenarios — available benchmark scenarios for interference detection, from linear to deeply nonlinear coupling channels."
---

## Bench Scenarios

Bench scenarios are controlled environments for developing and validating interference detection. Each bench models a different coupling channel type — from simple linear additive channels to deeply nonlinear cascaded systems. Together they span the complexity space that real-world multi-optimizer deployments encounter.

The benches live in `examples/bench/` in the godon repository. Each has its own directory with docker-compose configuration, target definitions, and breeder configs.

### Microgrid

A pair of coupled microgrid simulators sharing a power bus. Optimizer A adjusts load distribution on one microgrid; the coupling factor determines how much of A's load change leaks into B's grid.

| Property | Value |
|---|---|
| Directory | `examples/bench/scenario-microgrid` (workflow: `bench-scenario-microgrid.yml`) |
| Channel type | Linear additive |
| Coupling mechanism | Shared power bus — neighbor load adds directly to objectives |
| Detection status | Working reliably across all coupling strengths (0.0-0.9) |
| Trials needed | 100-300 |
| Configurable | `COUPLING_FACTOR` environment variable (0.0 to 0.9) |

The watermark signal passes through the coupling channel unchanged — same frequency content, proportional amplitude. FFT-based spectral detection with permutation testing reliably detects interference even at weak coupling (0.1). Zero false positives at coupling=0.0.

- See [Getting Started](getting_started.md) for a full walkthrough of this bench.

### Greenhouse

A pair of coupled greenhouse simulators sharing outside climate conditions. Optimizer A's waste heat, CO2 exhaust, and humidity drift affect the outside environment that Optimizer B's greenhouse is exposed to. The coupling signal then passes through zone thermal inertia, multiplicative growth models with dead zones, crop phase transitions with sensitivity jumps, irreversible damage thresholds, and sensor noise before reaching B's observed objectives.

| Property | Value |
|---|---|
| Directory | `examples/bench/scenario-4` (workflow: `bench-scenario-4.yml`, simulator image: `godon-bench-greenhouse`) |
| Channel type | Deeply nonlinear cascaded |
| Coupling mechanism | Waste heat, CO2 exhaust, humidity drift through thermal inertia, multiplicative growth, phase-dependent thresholds |
| Detection status | No reliable method yet — research phase |
| Signal path stages | Coupling delta → thermal inertia (×0.01) → zone physics → multiplicative growth (dead zones) → phase-dependent sensitivity (1x→3x) → irreversible damage → sensor noise → receiver exploration variance |
| SNR at objectives | ~0.002 (coupling signal 500× weaker than receiver's own exploration noise) |

The watermark signal is physically present in the system but heavily distorted by the time it reaches the observed objectives. All tested detection methods (FFT, mutual information, transfer entropy, Granger causality, cross-correlation, convergent cross mapping) have not produced reliable results at typical trial budgets (200-300 trials).

Promising research directions:

- **Higher trial counts** — thousands of trials may provide enough statistical power for methods like mutual information or distance correlation to detect the weak signal
- **Intermediate state detection** — measuring interference at raw sensor readings (zone temperatures, CO2 levels) before the growth model, where the signal is less distorted
- **Dedicated analysis phases** — holding the receiver optimizer still while the sender probes aggressively, eliminating the 500× exploration noise

### Detection Coverage

| Scenario | Channel Type | Status |
|---|---|---|
| Microgrid | Linear additive | Reliable |
| Greenhouse | Deeply nonlinear cascaded | Research phase |

As new benches are added, they should target specific gaps in this coverage — for example, a nonlinear channel with measurable intermediate state, or a non-stationary channel with phase transitions.

### Adding a New Bench

Each bench scenario follows a standard structure:

```
examples/bench/scenario-<name>/
├── docker-compose.yml          # Simulator containers with coupling config
├── targets/
│   ├── <target>-1.yaml         # Target definition for simulator 1
│   └── <target>-2.yaml         # Target definition for simulator 2
└── breeders/
    ├── breeder-1.yml            # Breeder config for optimizer 1
    └── breeder-2.yml            # Breeder config for optimizer 2
```

The simulator should expose an HTTP API compatible with godon's target interface. The coupling mechanism should be configurable via environment variables. See the microgrid bench as a reference implementation.

### Further Reading

- [Detection Capabilities](detection_capabilities.md) — channel taxonomy and detection status per channel type
- [Open Research](open_research.md) — active research directions
- See [Getting Started](getting_started.md) for a full walkthrough of this bench.
