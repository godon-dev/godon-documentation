---
date: 2026-06-07
tags:
  - essay
authors:
  - matthias-tafelmeier
description: "Autonomous agents coupled through shared media — physical and structural — form a territory whose terrain is the interaction of changing parameter spaces. Surveyed extent by extent, from a power unit to a floor, at overlaps operators draw."
---

# A Territory Made of Relations

*Rewritten October 2026. The garden framing retired with the original; this version is composed rather than assembled. The anchors are the same earned ones.*

Every operator of a complicated system lives with one quiet question: what affects what? Which knob, turned here, moves a number there? For most of engineering history the question had an answer on file — a schematic, a wiring diagram, an architecture drawing — because complicated systems were designed, and designers drew what they built.

The systems that now run our infrastructure do not have that drawing, and cannot.

<!-- more -->

On a floor that runs software, things adjust themselves. A cooling controller follows a temperature. A scheduler rebalances load. A search agent tries parameter values one after another and keeps what works. None of these actors was placed with the others in mind, and they share their mediums anyway: memory bandwidth, heat, power — and, one layer up, structure itself — schemas, queues, the values one service assumes about another. When the scheduler's move shifts the cooling controller's readings, the controller acts on a cause it cannot name. The shift is real, and it appears in no diagram — not because the diagram is out of date, but because the coupling was never designed by anyone. It exists as a relation between actors, and relations file no paperwork.

One such relation is a nuisance. At population scale it is a condition. Millions of autonomous actors now share their mediums, coupled through everything they touch, and — this is the second change, the quieter one — no one owns the whole. Every part has an owner. The whole has none.

The honest description of what they form is stranger than the usual words: a territory made of relations. Its terrain is neither steel nor silicon but interaction itself — dynamically changing parameter spaces acting on one another. Some of the couplings ride physics: heat through a shared wall, voltage on a shared bus. Others ride nothing heavier than structure: a shared schema, a queue, a chain of transforms that exists only in code. The method that surveys it is indifferent to the difference — it pushes parameters and reads objectives, and what sits between can be wall, wire, or schema. Halt the actors and the territory goes dormant; the terrain was the interacting, not the ground. The territory carries no boundaries of its own — carve it anywhere and it continues — so its boundaries are drawn by people, per question: this power unit, this network, this floor. A map of one extent can share edges with a map of its neighbor, or none. The overlaps are chosen, because the territory itself chooses nothing.

The work, then, starts with a declaration and proceeds like a survey. Someone draws an extent around the part they intend to tend. Then a measurement happens that the passive-monitoring tradition would not recognize: one agent pushes — deliberately, within bounds it already respects — while its neighbor holds still, and everyone watches what moves. Push, release, compare. If the neighbor's objective shifts and recovers, a coupling path exists; and the probe run to depth does more than detect it, it measures the path's shape, a response curve with error bars that mean something ([From Edges to Curves](https://doi.org/10.5281/zenodo.22127166)).

Probing is not a preference; it is forced. Anything gathered while the agents merely run is dominated by their own trial-and-error noise, roughly five hundred times stronger than any coupling signal inside it. The engine's history keeps the honest number: an early attempt broadcast a faint continuous tone and listened for it downstream, and after six cascaded transforms the signal-to-noise ratio was 0.002. The lesson that came back has the shape of a proverb — the signal must be created at the scale of the channel, not the scale of the sensor. The concept pages carry the full story; the outcome is a method validated across linear, deeply nonlinear, and slowly drifting coupling, zero false positives on uncoupled controls, limits published as a measured map rather than a promise ([The Impulse Protocol](https://doi.org/10.5281/zenodo.21962956)). And measured curves compose — through nonlinear paths, to a horizon of about two hops; beyond it, composing the pieces is the only remaining door ([From Curves to Cascades](https://doi.org/10.5281/zenodo.22401439)).

There is a strange comfort in where the method comes from. Its block structure — push, pause, compare — was invented for farm fields, where an agronomist had to isolate one treatment's effect from the endless variation of soil. Its detector, the part that refuses to cry wolf, comes from radar and sonar, where a known return must be found in noise nobody can model. Its guardrails come from control theory. Its drawn extents come from cartography. Four old disciplines that never needed each other, assembled now around a territory none of them ever had: one whose inhabitants are the terrain.

Each measured edge is banked into a wiring map — the [connectome](https://godon-dev.github.io/godon-documentation/concept_connectome/) — that persists across campaigns, so every survey begins where the last one ended. The map is honest about its gaps: an absent edge means not yet probed, or too weak to measure, and the map does not claim to know which. And the map ages by nature, not by neglect — its terrain is parameter spaces, and parameter spaces keep changing; drift is not weather on the territory, it is the territory's substance moving. So the map is built to age: it predicts — if this node moves, that one answers — and when a prediction fails, the failure localizes the change and the neighborhood is measured again. A map that keeps itself honest by failing.

None of the measuring is for its own sake. An operator's intent — hold this throughput, keep this latency inside its band — can be planned against the measured map, applied, held while the world drifts, corrected in place when the bands are brushed. Holding intent inside a moving territory deserves its own essay. It is the next one.

Engineering, it should be said plainly, finished its work at its layers. Identity, boundaries, policies — designed, verified, standing. What kept moving underneath was never going to be finished, because it is not that kind of thing. It is tended. The territory is made of relations; the relations can be measured; and the measurement stays honest by failing in public, one prediction at a time.

---

*This is the first in a series of articles on living systems, interference detection, and the future of tending. [godon](https://github.com/godon-dev/godon) is an open-source live systems tending and causal discernment engine.*
