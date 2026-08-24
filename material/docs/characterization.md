---
description: "godon characterization — measuring coupling response curves with uncertainty bars, priced stopping, multi-receiver walks, and composition of measured edges."
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

# Characterization

Detection answers *is there an edge?* Characterization measures its **shape**: for every (sender, receiver, parameter, channel), a response curve — parameter level versus measured objective shift, point by point, each point with an uncertainty bar. The result is a measured coupling map, not a yes/no graph.

## What a Curve Is

Each curve point is a triple `(level, shift, bar)`:

- **level** — where the sender's parameter was pushed (the walk's current probe level)
- **shift** — the receiver's median objective shift, push window versus pause window
- **bar** — the measurement uncertainty, from the raw sample scatter of those windows

The ABA structure is built into every point: the shift is only counted when it appears during the push and recovers during the pause. A shift that does not recover is flagged as drift, not measured as structure.

## How the Walk Runs

Breeders in a shared interference group take turns through DB-backed leases. One is the sender; the others hold still.

- **The walk is deterministic and complete by contract.** Levels are visited in farthest-point order — midpoint, then extremes, then quarters — a low-discrepancy sequence computed, not sampled. No level is skipped while the walk runs; smartness only accelerates (early retirement of converged parameters), it never licenses gaps.
- **Every listener is measured at once.** One sender's walk measures the response curves of *all* holding receivers simultaneously — in a three-agent chain run, a single walk produced all 36 curves, of which exactly the three planted coupling paths were non-flat.
- **Re-measurement blends.** A repeat probe within bars tightens the existing point (inverse-variance blend); beyond bars the point relocates and the curve is flagged as drifted.
- **Dead ground is cheap.** A parameter with no coupling retires after ~3 flat probes; a full curve on a 100-level grid cost 6-9 probes in validated runs. The budget goes to structure, not silence.

## Priced Stopping

A curve retires when **converged** (repeated measurement agrees) **and** every remaining gap is *priced*: the ignorance left in an unresolved interval is

```
ignorance = jump × width / range
```

and retirement requires it below the local measurement bar — remaining uncertainty cheaper than one more probe. In the deepest validated run (a 9-level tent curve, every point within 0.55σ of planted truth), the walk retired by price with three shape-honest gaps still open on the steep flank, each priced below its local bar. The bracket between measured points is part of the artifact: it says where the edge could still be, and what that uncertainty costs.

## Calibration Against Planted Truth

On the generic bench the ground truth is known, so calibration is exact comparison:

| Run | What it measured | Result |
|---|---|---|
| One-way edge (0.7, saturation) | carrier level 100 | −0.118 ± 0.022 vs truth −0.117 |
| Full tent curve | 9 levels | every point ≤ 0.55σ |
| Two channels | objective_0 vs objective_1 | tent on channel 0 (≤ 0.49σ); channel 1 flat (≤ 0.73σ) |
| Three agents, one uncoupled | per-receiver separation | edge measured within ~0.011 of truth; uncoupled agent flat |

The two-channel row is the param→channel mapping measured rather than assumed: a parameter influences exactly the channels where its curve is non-flat, and the flat channel is honest flat, not missing.

## Composition: What the Map Predicts

Measured edges are only useful if they compose. On a chain topology (node-1 —0.7→ node-3 —0.5→ node-2), with all curves measured by the engine:

- **Identity (in-sample):** the two-hop curve composed from chained one-hop curves matches the directly measured two-hop response — 9 of 9 levels within 2σ, median deviation 0.18σ.
- **Prediction (out-of-sample):** levels never probed before, predicted from the composed map, then measured: 0.13σ and 0.35σ deviation.
- **Steering (the loop's first closing):** given a target value at the chain's end, the measured map was inverted to a sender parameter, the parameter applied, and the result measured: target −0.100, landed −0.1038 ± 0.038.

Scope, stated plainly: these composition results are on the quiet bench (no opposing optimizers acting during the act), with additive-linear coupling physics, one seed per scenario. Composition across *nonlinear* coupling paths is open — see [Open Research](open_research.md).

## Reading Curves from a Running Engine

The causal service owns the curves and serves them at any time:

```bash
GET /curves           → every curve: sender, receiver, param, channel, points, gaps
GET /detect/{a}/{b}   → detection verdict per channel
```

Curves persist across service restarts and are removed with their breeder's lifecycle. The [Getting Started](getting_started.md) walkthrough runs this end to end against a planted topology; the scenario library ships the calibration cells above (see [Bench Scenarios](bench_scenarios.md)).

## See Also

- [Interference Detection](concept_interference_detection.md) — the detection method underneath
- [Detection Capabilities](detection_capabilities.md) — the validated boundary map
- [Open Research](open_research.md) — what is deliberately not claimed yet
- [Publications](publications.md) — the detection paper with full validation data
