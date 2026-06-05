---
description: "godon Open Research — active research directions for interference detection across nonlinear channels, intensity measurement, and topology mapping."
---

## Open Research

Interference detection works reliably on linear coupling channels. The open research frontier is extending detection to nonlinear, cascaded, and non-stationary channels — and moving from binary detection to quantitative understanding.

### Methods Evaluated

Over the course of development, several detection approaches were evaluated. The early methods were tested before the watermarking system matured — the original approach added noise to the sampler output rather than producing a clean sinusoidal signal by modulating the parameter directly. This means early failures are confounded with immature signaling. The methods have not been re-evaluated with the current watermarking.

#### ABA (Pause One Breeder)

Stop one optimizer and observe whether the other's behavior changes. Conceptually simple but operationally expensive — it requires pausing optimization campaigns, which defeats the purpose of continuous online detection. Each ABA cycle produces one data point. Compared to watermark-based detection, which produces a signal every trial without interrupting optimization, ABA is sluggish and impractical for production use.

#### Statistical Methods (Cross-Correlation, Granger Causality, Mutual Information, Transfer Entropy, Convergent Cross Mapping, HSIC)

A range of statistical and information-theoretic methods were evaluated for detecting coupling between optimizer time series. These are passive approaches — they don't require an injected signal. None produced reliable results on the greenhouse bench.

Important caveat: these were evaluated with early signaling that added noise to the sampler rather than modulating parameters with a clean sinusoidal. The current watermarking produces a well-defined spectral signature. Whether these passive methods would perform better with the improved signal — or whether they could detect coupling even without watermarking — remains an open question.

#### Lock-In Detection

Amplitude-phase detection at the known watermark frequency. Requires the optimizer to converge so that noise_std drops below the signal amplitude. Multi-objective Pareto optimization never converges — the optimizer keeps exploring indefinitely, so noise_std never stabilizes. Lock-in detection is effective for single-objective optimization where convergence occurs, but fundamentally incompatible with Pareto front exploration.

#### FFT Spectral Detection

The current method. Goertzel algorithm at known watermark frequencies with Hann windowing, noise floor estimation from neighboring bins, and permutation testing for significance. Works reliably on linear additive channels (microgrid bench, all coupling strengths 0.0-0.9). On deeply nonlinear channels, the signal is too heavily distorted by the time it reaches the objectives for FFT to extract it at 200-300 trial budgets.

### Greenhouse Nonlinear Channels

The greenhouse bench represents the hardest detection case. The coupling signal traverses 6+ nonlinear stages before reaching the objectives, with an SNR of ~0.002. Solving this would demonstrate that interference detection works across the full complexity spectrum, not just on linear channels.

The coupling signal passes through:

1. Coupling delta (tiny perturbation)
2. Thermal inertia (×0.01 per tick)
3. Zone physics (mixing, diffusion)
4. Multiplicative growth model with dead zones (zero derivative in optimal range)
5. Phase-dependent sensitivity (1× → 3× jump between growth phases)
6. Irreversible damage (permanent attenuation past thresholds)
7. Sensor noise
8. Receiver's own exploration variance (500× larger than coupling signal)

Every method tested so far has not produced reliable results at 200-300 trials. The question is whether higher trial counts or different detection strategies can overcome the signal death.

For a visual comparison of signal propagation through the microgrid (linear) vs greenhouse (nonlinear) channels, see the [Signal Death Diagram](assets/signal-death-diagram.html).

### Intermediate State Detection

Measuring interference at raw sensor readings (zone temperatures, CO2 levels) before the growth model, where the signal is less distorted. The coupling signal may be detectable at intermediate points in the channel even when it's invisible at the final objectives.

This requires targets to expose raw sensor channels in addition to optimization objectives — a natural extension of the target contract. The observer would analyze these raw channels using the same spectral pipeline, but on data that hasn't passed through the growth model's dead zones.

### Dedicated Analysis Phases

Holding the receiver optimizer still while the sender probes aggressively. This eliminates the 500× exploration noise that currently overwhelms the coupling signal. The trade-off is paused optimization on the receiver side during analysis.

A dedicated analysis phase could be triggered automatically when the observer needs more signal power — the receiver pauses its optimization, the sender injects stronger watermarks, and the observer collects a clean signal. After analysis completes, both optimizers resume.

### CDMA Encoding for Scale

The current FDMA approach (prime-numbered periods) scales to ~20-30 breeders with a few hundred trials. For significantly larger deployments, spread-spectrum approaches like direct-sequence CDMA with orthogonal Walsh-Hadamard codes would offer better noise resilience.

CDMA requires chip sequences of length 50-100+ per parameter to achieve reliable orthogonality. With only 200-300 total trials, codes are too short to correlate reliably. This becomes viable with thousands of trials in production deployments — a natural transition as optimization campaigns grow longer.

### Interference Intensity Measurement

Detection answers "is there interference?" The next question is "how much?" The spectral power at watermark frequencies scales with coupling strength — this relationship can be calibrated into an interference intensity metric.

An optimizer running against a power grid at 10% interference and one at 90% interference face fundamentally different problems. Intensity measurement gives operators the information to decide whether to act.

### Interference Topology

With three or more optimizers, pairwise detection results form an interference graph — a directed, weighted graph where nodes are optimizers, edges are interference channels, and weights are intensity.

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

This topology reveals clusters of tightly-coupled optimizers, isolated components, and bottleneck resources that concentrate interference. The detection doesn't just observe the problem — it maps the structure.

### Further Reading

- [Detection Capabilities](detection_capabilities.md) — channel taxonomy and current detection status
- [Bench Scenarios](bench_scenarios.md) — available benches for validation
- [Interference Detection](concept_interference_detection.md) — full methodology and detection pipeline
