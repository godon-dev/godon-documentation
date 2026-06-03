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

**Non-overlapping frequency assignment**: Each breeder uses unique prime periods from a shared pool. With 12 primes, up to 6 breeders can run simultaneously with 2 periods each, guaranteed zero frequency overlap.

**No distributional assumptions**: The permutation test compares the observed spectral power against random shuffles of the same data. It works regardless of the optimizer's dynamics, the noise characteristics, or the coupling channel's properties.

**Scenario-agnostic**: The watermark encoding (`MultiFrequencyMultiParam`) accepts arbitrary parameters and periods. The detection reads watermark metadata from trial records. No scenario-specific code in the detection pipeline.

**Non-invasive**: The watermark amplitudes are small perturbations within the optimizer's parameter space. They don't alter the optimization's convergence behavior — they're embedded in the existing search process.

### What Comes Next

Interference detection is the sensing layer. The next steps build toward making multi-optimizer systems work reliably:

- **Intensity measurement**: quantifying interference strength from spectral power, not just detecting presence/absence
- **Topology mapping**: identifying which optimizers interfere, through which shared resources, and along which parameter paths
- **Automatic decoupling**: using interference information to guide optimizer decisions away from interfering regions of the search space

### Further Reading

- [Breeder](concept_breeder.md) — how breeders run optimization and inject watermarks
- [Architecture](architecture.md) — the overall system architecture
- [Reconnaissance](concept_reconnaissance.md) — how the observer collects and analyzes trial data
