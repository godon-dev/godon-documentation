---
description: "godon Interference Detection — autonomous discovery of coupling between independent optimizers through active probing and CFAR detection."
---

## Interference Detection

When multiple optimizers operate on shared infrastructure — a power grid, a data center, a factory floor — their decisions can interfere with each other. Each optimizer acts independently, making locally optimal choices, but through the shared physical or computational medium, one optimizer's actions affect another's outcomes.

**Neither optimizer knows this is happening.**

godon's interference detection makes this invisible problem visible. The current method is **active probing with CFAR detection** — one optimizer pushes extreme parameters while the other holds still and measures the response. A Constant False Alarm Rate detector discriminates the coupling step from the system's intrinsic noise.

### The Problem

Consider two independent optimization loops:

```
    ┌─────────────┐              ┌─────────────┐
    │  Optimizer A │              │  Optimizer B │
    │  (grid load) │              │  (grid cost) │
    └──────┬───────┘              └──────┬───────┘
           │                             │
           ▼                             ▼
    ┌──────────────────────────────────────────┐
    │           Shared Power Grid               │
    │    (the coupling channel)                 │
    └──────────────────────────────────────────┘
```

Optimizer A adjusts load distribution. Optimizer B adjusts cost parameters. Both affect the same grid. When A shifts load, it changes the conditions B is optimizing against. B's "optimal" decisions are now based on a landscape A has altered. Both optimizers degrade each other's performance, but neither can see the cause.

This isn't theoretical — it happens in any system where multiple optimization workloads share resources.

### Why Not Passive Detection?

Earlier work explored spectral watermarking — injecting a continuous sinusoidal signal into optimizer parameters and detecting it via FFT in another optimizer's objectives. This worked reliably on **linear additive channels** (e.g. microgrid power bus) but failed completely on **non-linear cascaded channels** (e.g. greenhouse climate control).

The greenhouse coupling signal passes through 6+ non-linear transforms — thermal inertia, multiplicative growth models, dead zones, phase-dependent sensitivity — before reaching the objectives. Each stage distorts or attenuates frequency content. By the time the signal reaches the observable objectives, its SNR is approximately 0.002. No spectral method can recover it.

All passive statistical methods were evaluated and failed: cross-correlation, Granger causality, mutual information, transfer entropy, convergent cross mapping. The fundamental barrier was not the detection algorithm but the **exploration noise** — the receiver's own optimization produces objective variance 500× larger than any coupling signal. No statistical method can extract a signal buried 500× below the noise floor.

### Active Probing Protocol

The solution is not a better passive detector. It is to stop being passive.

Instead of injecting a continuous signal and hoping it survives the channel, the protocol actively creates a perturbation large enough to survive any non-linear distortion, then measures the response while eliminating the dominant noise source.

#### Roles: Sender and Receiver

Two breeders participate. One becomes the **sender**, the other the **receiver**. They coordinate through a shared lease table:

1. **OPTIMIZE** — Both breeders optimize normally until enough trials accumulate.
2. **HOLD_CALIB** — Sender acquires the lease, holds at neutral parameters, waits for receiver readiness.
3. **IMPULSE_CALIB** — Sender probes at decreasing amplitude (1.0 → 0.5 → 0.25 → 0.125) to find the largest impulse that doesn't violate guardrails.
4. **PUSH** — Sender applies extreme parameters for N trials.
5. **PAUSE** — Sender returns to neutral parameters for N trials.
6. **COOLDOWN** — Sender releases lease. Receiver can now acquire lease.
7. **HOLD** — Receiver holds neutral parameters throughout the sender's push/pause, recording observations.

Turn-taking is automatic: after cooldown, roles swap. Both directions are tested.

#### Why This Works Where Passive Methods Failed

Two key differences:

1. **The perturbation is large enough to survive the channel.** Extreme parameters push the system far from its operating point. The coupling delta is proportional to the parameter delta — a large push creates a large thermal transfer that survives non-linear transforms.

2. **The receiver holds still, eliminating exploration noise.** The 500× noise barrier vanishes. The only remaining variance is the system's intrinsic stochasticity, which CFAR handles.

### CFAR Detection

The detector uses a Constant False Alarm Rate (CFAR) approach — the same principle used in active sonar, radar, and seismology for detecting signals in unknown noise environments.

#### Reference Window

The noise floor is estimated from the receiver's hold_calib trials — the trials where the receiver sits at neutral parameters and the sender is also holding (before push starts). These are pure noise samples.

Critical: the reference must exclude the **sender's own hold_calib trials** (`coord_state=hold_calib`), which include unstable early readings as the sender's system stabilizes. Including them inflates the noise estimate by an order of magnitude.

#### Adaptive Threshold

```
k = N × (Pfa^(-1/N) - 1)
threshold = k × MAD
```

Where N = number of reference cells, Pfa = false alarm probability (1 - detection_confidence), MAD = median absolute deviation of reference values.

More reference data → lower k → more sensitive. The `detection_confidence` parameter (default 0.95) is configurable.

#### Detection Criterion

For each channel, both **rising edge** (baseline → push) and **falling edge** (push → pause) must exceed the threshold. This ensures the ABA pattern is present — not just a random spike.

### Observations: Separate from Objectives

Detection channels are explicitly separated from optimization targets:

| Section | Purpose | Example |
|---|---|---|
| `objectives` | What optuna optimizes | growth_rate, energy, water |
| `observations` | What the detector reads | max_temp, max_co2, max_humidity |
| `guardrails` | Safety limits | max_temp (hard_limit=40) |

Observations are collected per trial but never fed into the optuna Pareto front. The detector merges them at query time — all channels checked independently.

### Validation Results

Original bidirectional validation on greenhouse scenario 4 (two coupled greenhouses, coupling factor 0.9):

#### Coupling = 0.9 (Coupled)

| Direction | Channel | Detected | Baseline | Push | Pause | MAD | Edge |
|---|---|---|---|---|---|---|---|
| B2→B1 | growth_rate | **Yes** | 0.720 | 0.626 | 0.851 | 0.011 | 0.226 |
| B2→B1 | max_temp | **Yes** | 28.0°C | 29.7°C | 25.9°C | 0.140 | 3.8°C |
| B1→B2 | growth_rate | **Yes** | 0.849 | 0.703 | 0.851 | 0.003 | 0.148 |
| B1→B2 | max_temp | **Yes** | 25.9°C | 28.5°C | 19.3°C | 0.351 | 9.2°C |
| B1→B2 | max_humidity | **Yes** | 0.660 | 0.698 | 0.593 | 0.001 | 0.104 |

Bidirectional detection. Growth_rate and temperature are the strongest coupling channels.

#### Coupling = 0.0 (Control — Uncoupled)

| Direction | Detected |
|---|---|
| B2→B1 | **No** |
| B1→B2 | **No** |

**Zero false positives.** The CFAR detector stays silent when no coupling exists. A large temperature swing (7.7°C) was observed in the control run, but the rising edge was within the CFAR band — the ABA pattern was not present, so detection correctly rejected it.

### Key Insights

1. **Active probing beats passive observation.** The impulse survives non-linear channels where continuous signals die. Not smarter algorithms — stronger signal.
2. **CFAR handles unknown noise.** Adaptive threshold from local reference statistics. No Gaussian assumption, no fixed threshold.
3. **The reference window must be clean.** Sender's own hold_calib contaminated the noise floor (MAD 0.18 vs 0.02). Filtering to receiver-only reduced it 18×.
4. **Temperature is the primary coupling channel.** Thermal transfer is immediate and proportional. Growth_rate works because it's downstream of temperature.
5. **Turn-taking gives both directions automatically.** The lease-based coordination means each breeder takes turns. Both directions tested without configuration.

### Current Limitations

**Calibration is bypassed, not solved.** When `hold_params` are specified in config, the flatness search is skipped. The generic case — discovering stable hold parameters autonomously — remains open.

**Non-stationarity partially handled.** The greenhouse IS non-stationary (crop model drifts over time). Detection succeeded because the CFAR reference window is local (5-8 trials), limiting exposure to drift. The remaining open case is non-stationarity with phase transitions faster than the detection window — e.g., a crop suddenly entering flowering mid-detection.

**Lease timing is fragile.** The 90-second lease with 15-second trials requires tight coordination. Untested at scale.

### Channel Taxonomy

The active probing approach resolves the channel type problem that blocked spectral methods:

| Channel Type | Spectral Method | Active Probing + CFAR | Status |
|---|---|---|---|
| Linear additive | Reliable (FFT + permutation) | Works (overkill) | **Solved** |
| Non-linear, intermediate state measurable | Failed at objective level | **Validated** | **Solved** |
| Deeply non-linear cascaded | Failed (SNR ~0.002) | **Validated on greenhouse** | **Solved** |
| Non-stationary (slow drift) | Failed (assumption violated) | **Validated on greenhouse** | **Solved** |
| Non-stationary (phase transitions) | Failed | Untested | **Open** |

One method for all channel types. The greenhouse bench is simultaneously deeply non-linear AND non-stationary (crop model drifts over time). Detection succeeded because the CFAR reference window is local — 5-8 trials — limiting exposure to drift. The only remaining open case is non-stationarity with phase transitions faster than the detection window.

For detailed channel descriptions with real-world examples, see [Detection Capabilities](detection_capabilities.md).

### Beyond Detection (Shipped and Next)

Detection is the entry point; the protocol runs deeper:

**Response curves (shipped).** Detection answers "is there an edge?" The same protocol, run to depth, measures the edge's full response curve — per (sender, parameter, channel), the level→shift shape with uncertainty bars, convergence by blended re-measurement, retirement by priced stopping. Multi-agent groups: one sender's walk measures every holding receiver simultaneously.

**Topology recovery (shipped).** Pairwise measurements across a group assemble the coupling graph — validated on chain topologies with exact topology recovery and ~5-10% edge-strength error.

**Composition (validated additive).** Measured edges compose: the two-hop response predicted from chained one-hop curves matches the directly measured response within bars. Nonlinear composition is the open frontier (see [Open Research](open_research.md)).

**Coupling-aware behavior (next).** Agents adapting to known coupling — the tending direction. Not yet built.

### Further Reading

- [Detection Capabilities](detection_capabilities.md) — channel taxonomy and detection boundaries
- [Open Research](open_research.md) — active research directions
- [Breeder](concept_breeder.md) — how breeders run optimization
- [Reconnaissance](concept_reconnaissance.md) — how the observer collects data
