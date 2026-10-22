---
date: 2026-06-07
tags:
  - essay
authors:
  - matthias-tafelmeier
description: "Autonomous agents coupled through shared media — physical and structural — form a territory whose terrain is what the actors do to one another. Surveyed extent by extent: a power unit, a network, a floor, at overlaps the operators choose."
---

# A Territory Made of Relations

*Rewritten October 2026. The original essay told this shift as engineering giving way to cultivation, under a garden metaphor; both retired. The argument here is new; the numbers are the same.*

On a floor that runs software, things adjust themselves. A cooling controller follows a temperature. A scheduler rebalances load. A search agent tries parameter values one after another and keeps what works. None of these actors was placed with the others in mind, and they share their mediums anyway: memory bandwidth, heat, power — and, one layer up, structure itself — schemas, queues, the values one service assumes about another. When the scheduler's move shifts the cooling controller's readings, the controller acts on a cause it cannot name.

Every operator of a complicated system lives with one quiet question: what affects what? Which knob, turned here, moves a number there? For most of engineering history the question had an answer on file — a schematic, a wiring diagram, an architecture drawing — because complicated systems were designed, and designers drew what they built. The systems that now run our infrastructure do not have that drawing, and cannot. The shift on the floor is real, and it appears on no diagram — not because the diagram is out of date, but because the coupling was never designed by anyone. It exists as a relation between actors, and relations file no paperwork.

<!-- more -->

## The Territory

One such relation is a nuisance. At population scale it is a condition. Millions of autonomous actors now share their mediums, coupled through everything they touch. Every part has an owner. The whole has none.

What they form is a territory made of relations. Its terrain is neither steel nor silicon but interaction itself: settings in motion, pressing on other settings in motion. Some of the couplings ride physics: heat through a shared wall, voltage on a shared bus. Others ride nothing heavier than structure: a shared schema, a queue, a chain of transforms that exists only in code. Halt the actors and the territory goes dormant; the terrain was the interacting, not the ground. The territory carries no boundaries of its own — carve it anywhere and it continues — so its boundaries are drawn by people, per question: this power unit, this network, this floor. A map of one extent can share edges with a map of its neighbor, or none. The overlaps are chosen, because the territory itself chooses nothing.

## One Face Each

Maps follow territories, so the reach for an existing word is sound — and every word that gets close earned its face honestly. The cadastral map holds the declared extent: parcels drawn by people, meaningful because everyone agrees to them. The nautical chart holds the self-correcting map: centuries of mariners' reports revising the survey, a ship grounding where the chart said clear water bringing the resurvey of a whole neighborhood. The agricultural field holds the honest experiment: block designs invented to isolate one treatment's effect from the endless variation of soil. Radar and sonar hold the detector: a known return found in noise nobody can model, without crying wolf. Control theory holds the guardrail: action that stays inside bounds the actor already respects. And the power grid holds the precedent of ownerless senses — after blackouts whose investigation reports named the blindness, it grew wide-area measurement for a whole no one owns.

Each face is real, and each stops at a named edge. The cadastral parcels sit on ground that never moves; this terrain is nothing but movement. The chart corrects a map of a bottom — it never measured what the mariners do to each other. The field had one owner, and nothing inside it adjusts itself. The control loop couples one controller to one plant, a pairing someone designed; the couplings here were designed by no one. The grid's wiring was drawn before its sensors ever watched: it observes a body it was handed, and discovery was never asked of it. And the oldest words — a body, a garden — come nearest of all: an extent drawn tight around one power unit can be tended by one hand and never meshed into anything larger. They still break, at any scale: their boundaries are given, and a declaration is not — it is a hypothesis, moved when the question moves, drawn so that what crosses it can be measured. And what sits inside them is organs or plants, not autonomous actors pressing on one another.

No one holds the conjunction: discovered coupling between autonomous actors, at extents that change with the question — because until now there was no such territory to map, and maps follow territories. What each word truly owned, the practice took: the declared extent, the self-correcting map, the honest block, the detector, the guardrail, the ownerless senses. What each presumed, it left behind. The words are not wrong; they are one face wide. The practice is the seam.

## The Survey

The work, then, starts with a declaration and proceeds like a survey. Someone draws an extent around the part they intend to tend. Then comes a measurement no amount of watching could replace: one agent pushes — deliberately, within bounds it already respects — while its neighbor holds still, and everyone watches what moves. Push, release, compare. If the neighbor's objective shifts and recovers, a coupling path exists — wall, wire, or schema between them, the method does not care which. Run to depth, the probe does more than detect the path; it measures the path's shape: a response curve with error bars, and the bars are themselves measured, not decorative ([From Edges to Curves](https://doi.org/10.5281/zenodo.22127166)).

## The Noise Floor

Probing is not a preference; it is forced. Anything gathered while the agents merely run is dominated by their own trial-and-error noise, roughly five hundred times stronger than any coupling signal inside it. The engine's history keeps the honest number: an early attempt broadcast a faint continuous tone and listened for it downstream, and after six cascaded transforms the signal-to-noise ratio was 0.002. The lesson is the method's proverb: the signal has to be created at the scale of the channel, not the scale of the sensor. The concept pages carry the full story; the outcome is a method validated across linear, deeply nonlinear, and slowly drifting coupling, zero false positives on uncoupled controls, and limits published as a measured map rather than a promise ([The Impulse Protocol](https://doi.org/10.5281/zenodo.21962956)). And measured curves compose — through nonlinear paths, to a horizon of about two hops; beyond it, composing the pieces is the only remaining door ([From Curves to Cascades](https://doi.org/10.5281/zenodo.22401439)).

## The Living Map

Each measured edge is banked into a wiring map — the [connectome](https://godon-dev.github.io/godon-documentation/concept_connectome/) — that persists across campaigns, so every survey begins where the last one ended. The map is candid about its gaps: an absent edge means not yet probed, or too weak to measure, and the map does not claim to know which. And the map ages by nature, not by neglect — its terrain is parameter spaces, and parameter spaces keep changing; drift is not weather on the territory, it is the territory's substance moving. So the map is built to age: it predicts — if this node moves, that one answers — and when a prediction fails, the failure localizes the change and the neighborhood is measured again. A map that keeps itself honest by failing.

## The Practice

None of the measuring is for its own sake. An operator's intent — hold this throughput, keep this latency inside its band — can be planned against the measured map, applied, held while the world drifts, corrected in place when the bands are brushed. Holding intent inside a moving territory deserves its own essay. It is the next one.

Engineering finished its work at its layers. Identity, boundaries, policies — designed, verified, standing. What kept moving underneath was never going to be finished, because it is not that kind of thing. It is tended. The territory is made of relations; the relations can be measured; the map answers for itself, one failed prediction at a time.

---

*This is the first in a series of articles on living systems, interference detection, and the future of tending. [godon](https://github.com/godon-dev/godon) is an open-source live systems tending and causal discernment engine.*
