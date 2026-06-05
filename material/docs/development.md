---
description: "godon Development — current status, bench scenarios, detection capabilities, and open research directions for contributors."
---

## Development

This page covers the current development status of godon's interference detection capabilities, available bench scenarios, and open research directions. It is intended for contributors and researchers working on the godon project itself.

### Current Status

Interference detection between autonomous optimizers is the primary active development area. The detection pipeline — watermark injection, spectral analysis, permutation testing — is implemented and validated on linear coupling channels. Nonlinear channels remain an active research frontier.

### Bench Scenarios

Bench scenarios are controlled environments for developing and validating interference detection. Each bench models a different coupling channel type — from simple linear additive channels to deeply nonlinear cascaded systems. Together they span the complexity space that real-world multi-optimizer deployments encounter.

The benches live in `examples/bench/` in the godon repository. Each has its own directory with docker-compose configuration, target definitions, and breeder configs.

#### Microgrid

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

See [Getting Started](getting_started.md) for a full walkthrough of this bench.

#### Greenhouse

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

### Detection Capabilities

Not all coupling channels are equal. The detectability of interference depends on how the coupling signal propagates from one optimizer's parameters through the shared infrastructure to another optimizer's observed outcomes. Based on empirical validation and analysis, four categories emerge:

| Channel Type | Detection | Method | Data Requirement |
|---|---|---|---|
| Linear additive | Reliable | FFT + permutation | 100-300 trials |
| Nonlinear, intermediate state measurable | Promising | Spectral analysis on raw sensors | 300-1000 trials (estimated) |
| Deeply nonlinear cascaded | No reliable method yet | Research ongoing | Requires intermediate state measurement or higher trial counts |
| Non-stationary | Research | Phase-aware methods | Depends on phase transition frequency |

#### Linear Additive Channels

The coupling signal is added directly to the receiver's objectives. The watermark passes through unchanged — same frequency, proportional amplitude.

```
    Sender parameter → [ + coupling_delta ] → Receiver objective
```

Reliably detected with FFT + permutation test. Even weak coupling (0.1) produces detectable signal with sufficient trials.

Real-world examples: shared power bus (load affects voltage), shared network link (bandwidth contention), shared memory bus (bandwidth allocation).

#### Nonlinear Channels with Measurable Intermediate State

The coupling signal passes through nonlinear dynamics before reaching the objectives, but intermediate state variables (temperatures, pressures, flow rates) are measurable and less distorted than the final objectives.

```
    Sender parameter → [ nonlinear physics ] → Intermediate state (measurable) → [ nonlinear response ] → Objective
                                       ↑ detection possible here                         ↑ signal may be heavily distorted here
```

Detection on intermediate state variables (raw sensor readings) is more promising than detection on derived objectives, because the signal hasn't yet passed through the response function's dead zones and multiplicative interactions. This requires targets to expose raw sensor channels in addition to optimization objectives — a natural extension of the target contract.

Real-world examples: shared heating network (zone temperatures before growth model), shared cooling loop (inlet temperature before server response), shared compressed air (line pressure before actuator response).

#### Deeply Nonlinear Cascaded Channels

The coupling signal passes through multiple cascaded nonlinear transformations — multiplicative interactions, piecewise-linear dead zones, state-dependent thresholds, irreversible damage — before reaching the observed objectives. At each stage, the signal is distorted, attenuated, or eliminated.

```
    Sender parameter → [ coupling delta ]
                      → [ thermal inertia (×0.01) ]
                      → [ multiplicative growth model (dead zones, zero derivative in optimal range) ]
                      → [ phase-dependent sensitivity (1× → 3× jump) ]
                      → [ irreversible damage (permanent attenuation) ]
                      → [ sensor noise ]
                      → [ receiver's own exploration variance (500× larger than coupling) ]
                      → Objective
```

The signal-to-noise ratio at the objective level is approximately 0.002 — the coupling effect is 500 times weaker than the noise from the receiver's own optimization. The watermark signal is physically present but heavily distorted before it reaches the output.

No reliable method yet at the objective level. All tested approaches (FFT, mutual information, transfer entropy, Granger causality, cross-correlation, convergent cross mapping) have not produced reliable results on this channel type at typical trial budgets (200-300 trials).

Real-world examples: greenhouse climate control (waste heat → thermal inertia → zone physics → multiplicative growth → crop yield), chemical process control (catalyst temperature → reaction kinetics → yield with saturation), building HVAC (shared chilled water → zone mixing → comfort index with dead band).

#### Non-Stationary Channels

The coupling characteristics change over time. The same interference signal may be detectable in one operating phase and invisible in another.

```
    Phase 1: coupling visible (high sensitivity)
    Phase 2: coupling hidden (low sensitivity)
    Phase 3: coupling reversed (negative sensitivity)
```

Methods that assume a stationary relationship between sender and receiver will produce inconsistent results — detecting interference in some windows and missing it in others. Temporal segmentation and phase-aware detection are potential approaches.

Real-world examples: crop growth phases (CO2 sensitivity jumps 3× between seedling and flowering), day/night cycles (solar generation changes grid coupling characteristics), seasonal demand shifts (heating vs cooling mode changes infrastructure behavior).

### Open Research Directions

#### Greenhouse Nonlinear Channels

The greenhouse bench represents the hardest detection case. The coupling signal traverses 6+ nonlinear stages before reaching the objectives, with an SNR of ~0.002. Solving this would demonstrate that interference detection works across the full complexity spectrum, not just on linear channels.

#### Intermediate State Detection

Measuring interference at raw sensor readings (zone temperatures, CO2 levels) before the growth model, where the signal is less distorted. This requires targets to expose raw sensor channels — a natural extension of the target contract.

#### Dedicated Analysis Phases

Holding the receiver optimizer still while the sender probes aggressively. This eliminates the 500× exploration noise that currently overwhelms the coupling signal. The trade-off is paused optimization on the receiver side during analysis.

#### CDMA Encoding for Scale

The current FDMA approach (prime-numbered periods) scales to ~20-30 breeders with a few hundred trials. For significantly larger deployments, spread-spectrum approaches like direct-sequence CDMA with orthogonal Walsh-Hadamard codes would offer better noise resilience. This becomes viable with thousands of trials in production deployments.

#### Interference Intensity Measurement

Detection answers "is there interference?" The next question is "how much?" The spectral power at watermark frequencies scales with coupling strength — this relationship can be calibrated into an interference intensity metric.

#### Interference Topology

With three or more optimizers, pairwise detection results form an interference graph — a directed, weighted graph where nodes are optimizers, edges are interference channels, and weights are intensity. This topology reveals clusters of tightly-coupled optimizers, isolated components, and bottleneck resources.

### Further Reading

- [Interference Detection](concept_interference_detection.md) — methodology, detection pipeline, and validation results
- [Getting Started](getting_started.md) — walkthrough of the microgrid bench
- [Architecture](architecture.md) — system component overview
