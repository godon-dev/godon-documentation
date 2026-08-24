---
description: "godon Interference Detection — autonomous discovery of coupling between independent optimizers through active probing and CFAR detection."
---

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

## Interference Detection

The system answers a knock. The answer is a measurement.

Autonomous optimizers sharing a substrate corrupt each other through channels nobody configured and nobody monitors. Interference detection makes those channels visible — not by watching the systems, but by having the agents already embedded in the substrate perturb it, in turn, within guardrails, and read who answers.

This document follows the benches: the generic bench (nodes, parameters, planted edges — the same one [Getting Started](getting_started.md) runs) and the greenhouse bench (deeply nonlinear, the channel class that broke every passive method). Numbers quoted are measured engine output, not illustrations.

### The Problem

A group of autonomous optimizers, each locked to its own system — and substrates between the systems that belong to none of them. Two nets, same shape — one physical, one logical:

**The physical net** — the greenhouse bench's own coupling channels:

```
┌─────────────────────────┐   ┌─────────────────────────┐   ┌─────────────────────────┐
│ Breeder A  (node-1)     │   │ Breeder B  (node-2)     │   │ Breeder C  (node-3)     │
│ pushes p0 · p1 · p2     │   │ pushes p0 · p1 · p2     │   │ pushes p0 · p1 · p2     │
│ wants   growth_rate     │   │ wants   growth_rate     │   │ wants   growth_rate     │
└───────────┬─────────────┘   └───────────┬─────────────┘   └───────────┬─────────────┘
             │ tunes                       │ tunes                       │ tunes
             ▼                             ▼                             ▼
     ┌──────────────┐              ┌──────────────┐              ┌──────────────┐
     │ Greenhouse A │              │ Greenhouse B │              │ Greenhouse C │
     └──────┬───────┘              └──────┬───────┘              └──────┬───────┘
             │                             │                             │
             ╞═════════shared wall═════════╡                             │
             │                             │                             │
             │                             ╞════════air exchange═════════╡
             │                                                           │
             ╞════════shared water tank════╪═════════════════════════════╡
```

Not illustrations — the validation tables at the end of this page detect through exactly these channels.

**The logical net** — the same shape without physics:

```
┌─────────────────────────┐   ┌─────────────────────────┐   ┌─────────────────────────┐
│ Breeder A  (node-1)     │   │ Breeder B  (node-2)     │   │ Breeder C  (node-3)     │
│ pushes p0 · p1 · p2     │   │ pushes p0 · p1 · p2     │   │ pushes p0 · p1 · p2     │
│ wants   latency         │   │ wants   cost            │   │ wants   uptime          │
└───────────┬─────────────┘   └───────────┬─────────────┘   └───────────┬─────────────┘
             │ tunes                       │ tunes                       │ tunes
             ▼                             ▼                             ▼
       ┌───────────┐                 ┌───────────┐                 ┌───────────┐
       │ Service A  │                │ Service B  │                │ Service C  │
       └─────┬─────┘                 └─────┬─────┘                 └─────┬─────┘
             │                             │                             │
             ╞═══════shared database═══════╡                             │
             │                             │                             │
             │                             ╞════════message queue════════╡
             │                                                           │
             ╞═══════════shared schema═════╪═════════════════════════════╡
```

Each breeder tunes its own parameters against its own objective, reading only its own system's outputs. Influence crosses through substrates that appear in **no one's** configuration, **nor** anyone's telemetry. The net — physical or logical — is real in every reading and absent from every diagram.

That it works across kinds is not an edge case; it is the point. The protocol contains no physics: it needs parameters it can push, objectives it can read, and turns it can take. What sits between can be wall, wire, or schema. And both kinds are validated territory — the greenhouse's channels are physical; the generic bench's edges are pure mathematics (saturation and threshold transforms, no physics at all), and detection holds across the entire sweep on both.

On the generic bench this net is planted deliberately: nodes, parameters, edges, shapes, noise — the topology file *is* the ground truth, and the breeders optimizing those nodes know none of it. On the greenhouse bench it is physical: when A raises its heating, heat conducts through the shared wall and moves B's temperature. When A opens its vents, the air exchange carries CO2 and humidity to its neighbor. When A irrigates, the shared water tank drops and everyone's scarcity factor moves. B's optimizer receives each change as unexplained objective movement and does the only thing available to it: attributes the shift to its own parameters. Trials wasted chasing ghosts, convergence corrupted, every measurement quietly wrong. The coupling exists physically, in every reading — and in no model.

### Why Passive Detection Fails

The obvious move is statistics: watch both optimizers' metrics, look for correlation. It fails structurally, not implementationally.

While a receiver optimizes, its own search moves its objectives — trial by trial, deliberately. That self-inflicted variance sits roughly 500× above any coupling signal in passive data. Every passive estimator in the toolbox was tried on this channel class — cross-correlation, Granger causality, mutual information, transfer entropy, convergent cross mapping — with the same outcome: you cannot subtract a conversation you never heard. The noise is not in the sensor. The noise *is* the listener, thinking.

An earlier attempt made each optimizer broadcast a faint continuous tone (a watermark) and listened for it downstream with spectral methods. It worked on linear channels and died on the greenhouse: six cascaded transforms — thermal inertia, multiplicative growth, dead zones, phase-dependent sensitivity — ate the frequency content until the SNR was ≈ 0.002. The lesson was not a better detector. It was that the signal has to be created at the scale of the channel, not the scale of the sensor.

### The Method: Knock, Listen, Repeat

The protocol is what any engineer does to an unknown system: perturb it, watch what answers. Under discipline:

**Turn-taking.** Breeders in a shared group pass a lease — fencing tokens, one sender at a time, crash recovery via heartbeat staleness. One speaks; the others hold at neutral. Not politeness: a listener shifting in their chair is indistinguishable from an answer. Holding still deletes the 500× self-noise; what remains is the substrate's own voice.

**Scheduled pushes.** The sender walks one parameter through scheduled probe levels — midpoint, extremes, quarters; a deterministic order that visits every level by contract. Each push stays inside the same guardrails as ordinary optimization trials. Non-destructive by construction, not by hope.

**Blocks, not blips.** Every level gets a push block and a pause block of N trials each; the comparison is between block medians, not single samples.

The sender's cycle: `OPTIMIZE → PROBE_PUSH → PROBE_PAUSE → DONE → COOLDOWN`, receivers `HOLD` throughout. Roles rotate; both directions get measured.

### The Dynamics

What the instrument actually records — sender's parameter above, receiver's objective below, same time axis:

```
         push        pause       push        pause
       ┌───────┐   ┌───────┐   ┌───────┐
 param │  100  │   │  50   │   │   0   │    the walk: midpoint,
 level │       │   │(hold) │   │       │    extremes, quarters —
       └───────┘   └───────┘   └───────┘    every level, by contract
 ───────────────────────────────────────────────▶ trials

       ┌───────┐   ┌───────┐   ┌───────┐
 shift │ S(100)│   │  ~0   │   │ S(0)  │    the answer: same shape,
       └───────┘   └───────┘   └───────┘    scaled by the channel
```

Read the lower trace against the upper: each push produces a shift `S(level)`; each pause lets the receiver return. On the saturation bench cell the measured values behind those labels: `S(0) = −0.352 ± 0.020`, `S(100) = −0.113 ± 0.022` — planted truth `−0.350` and `−0.117`. Collect `S(level)` across the whole walk and the points *are* the response curve: detection and [characterization](characterization.md) are the same instrument at different depths.

Three things can move the receiver's objective, and the block design tells them apart:

| Movement | Verdict |
|---|---|
| Rises with the push, falls with the pause | **Coupling** — the edge is real |
| Rises — and stays | **Drift** — flagged, not measured as structure |
| Moves with no push at all | **Noise** — sizes the band everything else is judged against |

Both edges must clear the band. That reversibility criterion is why the method's false-positive count on uncoupled controls is zero: a coincidence can fake an arrival, a regime change can fake a level — faking an arrival *and* a timely departure is what noise does not do.

### Every Parameter Gets the Walk

The walk repeats per parameter — the map emerges per (parameter, channel), whatever the parameter's dynamics:

- A **carrier** (like `param_1` above) shows its shape: tent, threshold, saturation — the ABA contrast catches any repeatable level-dependence, fast or slow, saturating or stepped. Detection is shape-agnostic because the signature is *arrives with push, leaves with pause*, not any particular curve form.
- A **dead parameter** reads flat at every level and retires after ~3 probes — on a 100-level grid, the full cost of proving silence was 3 pushes. In one validated run, all three parameters converged on 16 probes out of 303 grid cells; the budget goes to structure, not silence.
- The **channel split** is measured, not assumed: the same walk that draws `param_1`'s tent on `objective_0` reads honest flat on `objective_1` — seven levels of nothing, bar-carrying. A parameter influences exactly the channels where its curve is non-flat.

### Several Breeders at Once

Turn-taking does not mean pairwise tedium. One sender's walk measures **every** holding receiver simultaneously, through however many substrates the influence must cross. The chain scenario (`node-1 → node-3 → node-2`, edges 0.7 and 0.5):

```
      breeder A — the walk: param_1 through its levels
        (50 → 0 → 100 → 25 → 75 → … — push block / pause block each)
                         │
        substrate 1      ▼      edge 0.7
                    ┌────────┐
                    │   C    │   answers: one-hop curve — 9 levels
                    └───┬────┘
        substrate 2      ▼      edge 0.5
                    ┌────────┐
                    │   B    │   answers: two-hop curve — the same
                    └────────┘   walk measures through both substrates

  everything else flat: reverse directions, second channels —
  36 curves, exactly the 3 planted signal paths non-flat
```

Both curves — the direct answer at C and the through-C answer at B — fell out of a single walk.

The witness is the control running inside the experiment: in the verification-star scenario (`node-1` sends, `node-2` coupled at 0.7, `node-3` uncoupled) the same windows and the same push schedule give node-2 a tent curve with every point ≤ 0.011 from planted truth (~0.5σ) — and node-3 flat, ±0.015. The honest blank is what certifies the tent as measurement, not artifact.

### Calling the Step: CFAR

How large must a step be before it is believed? In units of the local noise:

```
threshold = k × MAD,    k = N · (Pfa^(−1/N) − 1)
```

MAD is the median absolute deviation of the receiver's own neutral-hold window; N is the reference window size; Pfa is the false-alarm probability (`detection_confidence`, default 0.95). More reference data lowers k; a cleaner window lowers MAD. This is CFAR — constant false-alarm rate, the radar and sonar family — because the detector calibrates itself to the local noise, per channel, per round: a known-shape return in unknown noise.

One hygiene rule carries a hard-won invariant: only the receivers' observation rows enter the reference window. Sender self-reads poison the median (a real defect once, caught by live checks, fixed by a lease-phase write gate plus receiver-rows-only SQL — pinned by tests).

### Objectives Are Optimized; Observations Are Watched

Detection channels are explicitly separate from optimization targets:

| Section | Purpose | Example |
|---|---|---|
| `objectives` | What the optimizer optimizes | growth_rate, energy |
| `observations` | Collected per trial, never optimized — extra detector channels | max_temp, humidity |
| `guardrails` | Hard safety limits | max_temp (hard_limit 40) |

The detector reads objectives and observations alike; the optimizer only chases the former. A channel you suspect is coupled but do not want an agent chasing belongs in `observations`.

### Validation

On the greenhouse bench — deeply nonlinear cascaded coupling with crop-phase drift — bidirectional detection at coupling 0.9, zero false positives at coupling 0.0:

| Direction | Channel | Detected | Baseline | Push | Pause | MAD | Edge |
|---|---|---|---|---|---|---|---|
| B2→B1 | growth_rate | **Yes** | 0.720 | 0.626 | 0.851 | 0.011 | 0.226 |
| B2→B1 | max_temp | **Yes** | 28.0°C | 29.7°C | 25.9°C | 0.140 | 3.8°C |
| B1→B2 | growth_rate | **Yes** | 0.849 | 0.703 | 0.851 | 0.003 | 0.148 |
| B1→B2 | max_temp | **Yes** | 25.9°C | 28.5°C | 19.3°C | 0.351 | 9.2°C |
| B1→B2 | max_humidity | **Yes** | 0.660 | 0.698 | 0.593 | 0.001 | 0.104 |

The uncoupled control stayed silent in both directions — including through a genuine 7.7°C temperature swing whose ABA pattern was absent, so the detector correctly rejected it. On the generic bench, the 21-cell sensitivity sweep put the detection floor between 0.1 and 0.2 coupling at noise σ=0.02 with zero false positives across all control cells. Full sweep data: the [paper](publications.md); boundary map in brief: [Detection Capabilities](detection_capabilities.md).

### Limits

- **Calibration is bypassed, not solved.** With `hold_params` configured, the flatness search is skipped; discovering stable hold parameters autonomously remains open.
- **Fast phase transitions** — regime changes quicker than the detection window — remain open.
- **Scale.** Group-scoped fencing-token coordination is validated at 2-6 agents; the many-agent regime is open (see [Open Research](open_research.md)).

### Channel Taxonomy

One method covers the channel types that defeated each passive approach:

| Channel Type | Spectral Method | Active Probing + CFAR | Status |
|---|---|---|---|
| Linear additive | Reliable (FFT + permutation) | Works (overkill) | **Solved** |
| Non-linear, intermediate state measurable | Failed at objective level | **Validated** | **Solved** |
| Deeply non-linear cascaded | Failed (SNR ~0.002) | **Validated on greenhouse** | **Solved** |
| Non-stationary (slow drift) | Failed (assumption violated) | **Validated on greenhouse** | **Solved** |
| Non-stationary (phase transitions) | Failed | Untested | **Open** |

### Beyond Detection

Detection is the entry point; each deeper layer has its own page:

- **Response curves** — the edge's measured shape, with uncertainty bars and priced stopping: [Characterization](characterization.md)
- **Topology recovery** — pairwise measurements assemble the coupling graph (validated on chain topologies): [Detection Capabilities](detection_capabilities.md)
- **Composition** — measured edges compose to predict multi-hop response (validated additive): [Characterization](characterization.md)
- **Coupling-aware behavior** — agents adapting to known coupling; the tending direction, not yet built: [Open Research](open_research.md)

### Further Reading

- [Detection Capabilities](detection_capabilities.md) — the validated boundary map
- [Characterization](characterization.md) — from edges to measured curves
- [Publications](publications.md) — the method paper with full validation data
- [Breeder](concept_breeder.md) — the agents that run this protocol
