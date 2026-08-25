---
description: "godon roadmap — the full arc from coupling discovery to self-modeling regulation: seven steps, each marked validated, designed, or speculative. Ordered by construction, not calendar."
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

# Roadmap

Ordered by construction sequence, not calendar. Each step opens when the one below it holds — the project is gated by results, and promising dates ahead of results is exactly what this roadmap refuses to do.

**The destination:** an engine that measures the coupling structure of the live system it is part of, predicts interventions through that measured map, keeps the map current as the system drifts, and tends the whole — multiple autonomous agents, one shared substrate — toward operating points people actually want. A self-modeling regulator for live coupled systems, built from measurement rather than specification.

## The Arc

| # | Step | What it is | Status |
|---|---|---|---|
| 1 | **Detect** | Edges discovered: guarded pushes, held receivers, CFAR — any shape, physical or logical substrate, zero false positives on controls | **Validated** — [paper](publications.md) |
| 2 | **Characterize** | Response curves per (sender, parameter, channel) with uncertainty bars; termination by priced ignorance, not fixed budget | **Validated** — [Characterization](characterization.md) |
| 3 | **Predict** | Measured edges compose: multi-hop response predicted from the map and confirmed out-of-sample | **Additive validated** — nonlinear is the open gate |
| 4 | **Maintain** | The map stays alive: prediction error triggers rescans where the map went stale; drift re-arms retired cells | **Designed** — not yet an engine loop |
| 5 | **Transfer** | The connectome as a versioned, queryable artifact — robots embed it, controllers consume it, LLMs reason from it | **Designed** — the artifact exists per-run; the consumer ecosystem does not |
| 6 | **Accumulate** | Many measured systems over time: the first cross-domain collection of empirical coupling maps | **Speculative** — cannot be forced; starts with substrate one |
| 7 | **Generalize** | Cross-domain coupling laws — universal patterns, if they exist, visible only in the accumulated collection | **Speculative** |

The acting half grows on top of the seeing half: agents that account for known coupling (don't trample), then coordinated moves between coupled agents, then destination-directed steering through the inverted map. A first steering precursor has landed once, on the quiet bench — the map inverted itself, the act executed, the target met within a bar ([Characterization](characterization.md)). A precursor is not a rung; steering repeats under real conditions or it does not count.

## You Are Here

Steps 1-2 shipped and evidenced; step 3 holds for additive coupling; step 4 is the next thing built. The honest boundary on everything standing: quiet bench (no opposing agents during measurement), additive-linear physics, single seed per scenario — and the two gates ahead are named: a **nonlinear-edge bench capability** (without it, step 3's open half cannot even be tested) and the **live-agents regime** (measurement while the system's own optimizers work).

What the map already enables, today, with none of the above: understanding structure nobody designed, foreseeing cascade paths before anything breaks, attributing regressions through measured paths, right-sizing isolation spend, discovering coupling nobody suspected. The seeing half is the product of the present.

## Not on the Roadmap

- **Scale past ~6 agents** — a different coordination regime; open research, not engineering, until the arc holds.
- **A substrate catalog** — the interface contract (parameters pushable, objectives readable, turns takable) is the published statement. The list worth keeping is *measured* substrates; it starts with the first one.

## See Also

- [Open Research](open_research.md) — the frontier, unsequenced
- [Characterization](characterization.md) — the current top of the validated stack
- [Publications](publications.md) — evidence for steps 1-2
