---
description: "godon Detection Capabilities — channel taxonomy and honest boundaries of interference detection across linear and nonlinear coupling channels."
---

## Detection Capabilities

Not all coupling channels are equal. The detectability of interference depends on how the coupling signal propagates from one optimizer's parameters through the shared infrastructure to another optimizer's observed outcomes. Based on empirical validation and analysis, four categories emerge:

| Channel Type | Detection | Method | Data Requirement |
|---|---|---|---|
| Linear additive | Reliable | FFT + permutation | 100-300 trials |
| Nonlinear, intermediate state measurable | Promising | Spectral analysis on raw sensors | 300-1000 trials (estimated) |
| Deeply nonlinear cascaded | No reliable method yet | Research ongoing | Requires intermediate state measurement or higher trial counts |
| Non-stationary | Research | Phase-aware methods | Depends on phase transition frequency |

### Linear Additive Channels

The coupling signal is added directly to the receiver's objectives. The watermark passes through unchanged — same frequency, proportional amplitude.

```
    Sender parameter → [ + coupling_delta ] → Receiver objective
```

Reliably detected with FFT + permutation test. Even weak coupling (0.1) produces detectable signal with sufficient trials.

Real-world examples: shared power bus (load affects voltage), shared network link (bandwidth contention), shared memory bus (bandwidth allocation).

### Nonlinear Channels with Measurable Intermediate State

The coupling signal passes through nonlinear dynamics before reaching the objectives, but intermediate state variables (temperatures, pressures, flow rates) are measurable and less distorted than the final objectives.

```
    Sender parameter → [ nonlinear physics ] → Intermediate state (measurable) → [ nonlinear response ] → Objective
                                       ↑ detection possible here                         ↑ signal may be heavily distorted here
```

Detection on intermediate state variables (raw sensor readings) is more promising than detection on derived objectives, because the signal hasn't yet passed through the response function's dead zones and multiplicative interactions. This requires targets to expose raw sensor channels in addition to optimization objectives — a natural extension of the target contract.

Real-world examples: shared heating network (zone temperatures before growth model), shared cooling loop (inlet temperature before server response), shared compressed air (line pressure before actuator response).

### Deeply Nonlinear Cascaded Channels

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

### Non-Stationary Channels

The coupling characteristics change over time. The same interference signal may be detectable in one operating phase and invisible in another.

```
    Phase 1: coupling visible (high sensitivity)
    Phase 2: coupling hidden (low sensitivity)
    Phase 3: coupling reversed (negative sensitivity)
```

Methods that assume a stationary relationship between sender and receiver will produce inconsistent results — detecting interference in some windows and missing it in others. Temporal segmentation and phase-aware detection are potential approaches.

Real-world examples: crop growth phases (CO2 sensitivity jumps 3× between seedling and flowering), day/night cycles (solar generation changes grid coupling characteristics), seasonal demand shifts (heating vs cooling mode changes infrastructure behavior).

### Further Reading

- [Interference Detection](concept_interference_detection.md) — full methodology and validation results
- [Bench Scenarios](bench_scenarios.md) — available benches and their channel types
- [Open Research](open_research.md) — active research directions for unsolved channel types
