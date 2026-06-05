---
description: "godon Interference Detection — detect when independent optimizers degrade each other through shared infrastructure using watermark-based spectral analysis."
---

## Interference Detection

When multiple optimizers operate on shared infrastructure — a power grid, a data center, a factory floor — their decisions can interfere with each other. Each optimizer acts independently, making locally optimal choices, but through the shared physical or computational medium, one optimizer's actions affect another's outcomes.

**Neither optimizer knows this is happening.**

godon's interference detection makes this invisible problem visible. It works by injecting a known mathematical signal (a *watermark*) into each optimizer's parameters, then observing whether that signal appears in another optimizer's objective values. If it does, the optimizers are interfering.

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

### How Detection Works

The detection pipeline has three stages:

#### 1. Watermark Injection

Each breeder (optimizer driver) injects a unique mathematical fingerprint into its trial parameters. The fingerprint is a multi-frequency signal — a sum of sinusoidal perturbations at specific periods, applied to selected parameters.

```
    Breeder A watermark: periods [29, 37]
    Breeder B watermark: periods [67, 71]

    (Non-overlapping periods — no shared frequencies between breeders)
```

The periods are drawn from a pool of prime numbers, with each breeder assigned a unique, non-overlapping set. This ensures that any spectral power at Breeder A's frequencies in Breeder B's observations can *only* come from interference — not from Breeder B's own watermark.

#### 2. Observation and Alignment

The observer collects trial data from both breeders, aligning them by timestamp. After self-subtraction (removing each breeder's own watermark contribution from its objective values), the residual signal contains only external influences.

#### 3. Spectral Detection

The observer performs an FFT (Fast Fourier Transform) on the aligned residual signal, measuring spectral power at the sender's known watermark frequencies. A permutation test (5000 random shuffles) determines whether the observed spectral power is statistically significant:

- **Null hypothesis**: the residual signal has no structure at the sender's frequencies
- **Test statistic**: combined spectral power at the sender's periods
- **p-value**: fraction of random shuffles that produce equal or greater power
- **Detection threshold**: p < 0.05

The permutation test is non-parametric — no assumptions about the noise distribution, the optimizer's dynamics, or the coupling channel's characteristics.

### Validation Results

The detection was validated on a microgrid optimization scenario with two breeders optimizing four objectives (load balance, renewable utilization, grid stability, and energy cost). The coupling factor was varied from 0.0 (no coupling) to 0.9 (strong coupling).

| Coupling | obj0 | obj1 | obj2 | obj3 (energy cost) |
|----------|------|------|------|-------------------|
| 0.0 | clean (p=0.90) | clean (p=1.00) | clean (p=0.27) | clean (p=0.99) |
| 0.1 | detected (p=0.005) | clean (p=0.61) | detected (p=0.0002) | clean (p=0.73) |
| 0.5 | detected (p=0.005) | detected (p=0.034) | detected (p=0.004) | clean (p=0.74) |
| 0.9 | detected (p=0.0004) | detected (p=0.0002) | detected (p=0.0002) | clean (p=0.88) |

**Key observations:**

- **Zero false positives**: at coupling=0.0, no objectives are falsely flagged. The non-overlapping period assignment eliminates spectral contamination between breeders.
- **Monotonic detection**: as coupling strength increases, more objectives show detectable interference, and detection confidence (lower p-values) increases.
- **Objective-specific sensitivity**: energy cost (obj3) shows no interference at any coupling level — this objective is not affected by the coupling channel in the microgrid scenario.
- **Spectral power scales with coupling**: the actual measured power at watermark frequencies increases proportionally with coupling strength, providing a path toward intensity measurement.

### Design Principles

**Non-overlapping frequency assignment**: Each breeder uses unique prime periods from a shared pool. The current implementation uses 12 primes supporting up to 6 breeders with 2 periods each. The pool can be extended — primes are infinite — but practical scaling is limited by FFT frequency resolution: with N trials, frequencies closer than 1/N apart become indistinguishable. Small primes are well-separated, so 20-30 primes are feasible with a few hundred trials. Beyond that, adjacent primes converge spectrally and require proportionally more data, at which point alternative encodings (see below) become more efficient.

**No distributional assumptions**: The permutation test compares the observed spectral power against random shuffles of the same data. It works regardless of the optimizer's dynamics, the noise characteristics, or the coupling channel's properties.

**Scenario-agnostic**: The watermark encoding (`MultiFrequencyMultiParam`) accepts arbitrary parameters and periods. The detection reads watermark metadata from trial records. No scenario-specific code in the detection pipeline.

**Non-invasive**: The watermark amplitudes are small perturbations within the optimizer's parameter space. They don't alter the optimization's convergence behavior — they're embedded in the existing search process.

### Channel Taxonomy

Not all coupling channels are equal. The detectability of interference depends on how the coupling signal propagates from one optimizer's parameters through the shared infrastructure to another optimizer's observed outcomes. Based on empirical validation and analysis, we identify four categories:

#### Linear Additive Channels

The coupling signal is added directly to the receiver's objectives. The watermark passes through unchanged — same frequency, proportional amplitude.

```
    Sender parameter → [ + coupling_delta ] → Receiver objective
```

**Detection status:** Reliably detected with FFT + permutation test. The microgrid bench falls in this category. Even weak coupling (0.1) produces detectable signal with sufficient trials.

**Real-world examples:** Shared power bus (load affects voltage), shared network link (bandwidth contention), shared memory bus (bandwidth allocation).

#### Nonlinear Channels with Measurable Intermediate State

The coupling signal passes through nonlinear dynamics before reaching the objectives, but intermediate state variables (temperatures, pressures, flow rates) are measurable and less distorted than the final objectives.

```
    Sender parameter → [ nonlinear physics ] → Intermediate state (measurable) → [ nonlinear response ] → Objective
                                       ↑ detection possible here                         ↑ signal may die here
```

**Detection status:** Research phase. Detection on intermediate state variables (raw sensor readings) is more promising than detection on derived objectives, because the signal hasn't yet passed through the response function's dead zones and multiplicative interactions. This requires targets to expose raw sensor channels in addition to optimization objectives — a natural extension of the target contract.

**Real-world examples:** Shared heating network (zone temperatures before growth model), shared cooling loop (inlet temperature before server response), shared compressed air (line pressure before actuator response).

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

**Detection status:** No reliable method yet at the objective level. All tested approaches (FFT, mutual information, transfer entropy, Granger causality, cross-correlation, convergent cross mapping) have not produced reliable results on this channel type at typical trial budgets (200-300 trials). Promising directions include: higher trial counts (thousands), dedicated analysis phases where the receiver holds still, and detection on intermediate state rather than objectives.

**Real-world examples:** Greenhouse climate control (waste heat → thermal inertia → zone physics → multiplicative growth → crop yield), chemical process control (catalyst temperature → reaction kinetics → yield with saturation), building HVAC (shared chilled water → zone mixing → comfort index with dead band).

#### Non-Stationary Channels

The coupling characteristics change over time. The same interference signal may be detectable in one operating phase and invisible in another.

```
    Phase 1: coupling visible (high sensitivity)
    Phase 2: coupling hidden (low sensitivity)
    Phase 3: coupling reversed (negative sensitivity)
```

**Detection status:** Research phase. Methods that assume a stationary relationship between sender and receiver will produce inconsistent results — detecting interference in some windows and missing it in others. Temporal segmentation and phase-aware detection are potential approaches.

**Real-world examples:** Crop growth phases (CO2 sensitivity jumps 3× between seedling and flowering), day/night cycles (solar generation changes grid coupling characteristics), seasonal demand shifts (heating vs cooling mode changes infrastructure behavior).

#### Summary

| Channel Type | Detection | Method | Data Requirement |
|---|---|---|---|
| Linear additive | Reliable | FFT + permutation | 100-300 trials |
| Nonlinear, intermediate state measurable | Promising | Spectral analysis on raw sensors | 300-1000 trials (estimated) |
| Deeply nonlinear cascaded | No reliable method yet | Research ongoing | Requires intermediate state measurement or higher trial counts |
| Non-stationary | Research | Phase-aware methods | Depends on phase transition frequency |

This taxonomy is intentionally honest. Interference detection is not a universal capability — it depends on the nature of the coupling channel. Documenting these boundaries is as important as documenting successes: it tells operators when they can trust the detection result and when they need additional measurement points or dedicated analysis phases.

### Outlook

Interference detection is the sensing layer — the first step toward making multi-optimizer systems work reliably. The detection result (present/absent) opens the door to a much richer understanding of how optimizers interact through shared systems.

#### Intensity Measurement

Detection answers "is there interference?" The next question is "how much?" The spectral power at watermark frequencies scales with coupling strength — this relationship can be calibrated into an interference intensity metric. An optimizer running against a power grid at 10% interference and one at 90% interference face fundamentally different problems. Intensity measurement gives operators the information to decide whether to act.

#### Interference Topology

With three or more optimizers, the question becomes: who interferes with whom, through what? The pairwise detection results between all breeders form an interference graph — a directed, weighted graph where nodes are optimizers, edges are interference channels, and weights are intensity. This topology reveals the structure of the system: clusters of tightly-coupled optimizers, isolated components, and bottleneck resources that concentrate interference.

```
    ┌──────────┐  0.7  ┌──────────┐
    │ Breeder A │──────▶│ Breeder B │
    └──────────┘       └──────────┘
         │                  │
      0.2│               0.1│
         ▼                  ▼
    ┌──────────┐  0.0  ┌──────────┐
    │ Breeder C │──────▶│ Breeder D │
    └──────────┘       └──────────┘
```

In this example, A heavily interferes with B through shared infrastructure, while C and D are essentially independent. The topology tells you where to focus decoupling efforts.

#### Per-Parameter Tracing

Not all parameters contribute equally to interference. By watermarking individual parameters and analyzing which ones propagate through the coupling channel, the detection can trace interference to specific configuration seams — the exact points where one optimizer's decisions leak into another's domain. This enables targeted decoupling: instead of isolating entire optimizers, you can adjust specific parameters or add constraints at the seams.

#### Permeating Configuration

As interference understanding deepens, the configuration of the optimization system itself can be adapted. If the interference topology shows that two optimizers conflict through a specific resource, that resource's configuration can be partitioned, scheduled, or isolated. The detection doesn't just observe the problem — it informs the system's architecture. Optimization becomes aware of its own multi-agent context and reconfigures at the seams.

#### Temporal Dynamics

Interference isn't static. As optimizers converge, their interference patterns shift. A resource that was heavily contested early in optimization may become stable later. Temporal tracking of interference intensity reveals these dynamics, enabling adaptive strategies: coordinate during high-interference phases, run independently during low-interference phases.

#### Alternative Encoding Schemes

The current approach uses sinusoidal watermarks detected via FFT — a frequency-division multiplexing (FDMA) scheme where each breeder occupies unique prime-numbered periods. This is a deliberate choice for the data regime optimization operates in: typically 200-300 discrete trial samples, not continuous analog signals with millions of data points.

Spread-spectrum approaches like direct-sequence CDMA (DSSS) with orthogonal Walsh-Hadamard codes would scale to more breeders and offer better noise resilience, but they require chip sequences of length 50-100+ per parameter to achieve reliable orthogonality. With only 200-300 total trials and 2-3 watermarked parameters, the codes would be too short to correlate reliably. FDMA with prime periods works because frequency resolution is achievable with short data windows — a single period of 37 samples is enough to distinguish that frequency from its neighbors.

As optimization campaigns grow longer (thousands of trials in production deployments) or the number of concurrent optimizers increases significantly, CDMA-style encodings become viable. The transition would be natural — same watermark injection architecture, different encoding and detection kernels. Studying this transition, along with chirp signals for time-varying channels and wavelet-based encodings for non-stationary interference, is an open research direction.

#### Cross-Scenario Generalization

The detection framework is scenario-agnostic — the watermark encoding and spectral analysis make no assumptions about what's being optimized. Validating across diverse scenarios (power grids, data centers, supply chains, chemical plants) establishes the generality of the approach and reveals which interference patterns are universal vs. domain-specific.

### Further Reading

- [Breeder](concept_breeder.md) — how breeders run optimization and inject watermarks
- [Architecture](architecture.md) — the overall system architecture
- [Reconnaissance](concept_reconnaissance.md) — how the observer collects and analyzes trial data
