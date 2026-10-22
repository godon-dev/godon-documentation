---
date: 2026-06-07
tags:
  - essay
authors:
  - matthias-tafelmeier
description: "Every territory is made of a medium, and the practice grows around the medium — ground had the cadastre, water the chart, spectrum its allocations, load the grid. This territory is made of relations: autonomous actors coupled through shared media, surveyed extent by extent, the precision local, the overlaps chosen."
---

# A Territory Made of Relations

*Rewritten October 2026. The original garden framing retired first; an interim version carried a gallery of near-miss analogies. This one runs a single line instead — every territory is made of a medium, and this medium is relations. The numbers are the same earned ones.*

On a floor that runs software, things adjust themselves. A cooling controller follows a temperature. A scheduler rebalances load. A search agent tries parameter values one after another and keeps what works. None of these actors was placed with the others in mind, and they share their mediums anyway: memory bandwidth, heat, power — and, one layer up, structure itself — schemas, queues, the values one service assumes about another. When the scheduler's move shifts the cooling controller's readings, the controller acts on a cause it cannot name.

Every operator of a complicated system lives with one quiet question: what affects what? Which knob, turned here, moves a number there? For most of engineering history the question had an answer on file — a schematic, a wiring diagram, an architecture drawing — because complicated systems were designed, and designers drew what they built. The systems that now run our infrastructure do not have that drawing, and cannot. The shift on the floor is real, and it appears on no diagram — not because the diagram is out of date, but because the coupling was never designed by anyone. It exists as a relation between actors, and relations file no paperwork.

<!-- more -->

One such relation is a nuisance. At population scale it is a condition. Millions of autonomous actors now share their mediums, coupled through everything they touch. Every part has an owner. The whole has none.

## What Territories Are Made Of

The reach for an old word is sound, because this is not the first territory people have had to hold and map. Every territory is made of a medium, and the practice grows around the medium.

The oldest are made of ground. The cadastral map holds parcels declared by people, parcels that hold still because the ground does; the same ground taught the honest experiment — block designs, invented on farm fields to isolate one treatment's effect from the endless variation of soil. Territories made of water carry the nautical chart, a map of the bottom corrected for centuries by the mariners' own reports: a ship grounding where the chart said clear water brings the resurvey of a whole neighborhood. The radio spectrum was the first invisible territory, and its operators gave the language its word for the weather of a shared medium: interference. Their descendants built the detector that finds a known return in noise nobody can model. And the electrical grid — a territory made of load — grew wide-area measurement for a body no one owns, after blackouts whose investigation reports named the blindness.

Each medium taught one move, and each practice stops where its medium stops. Ground holds still; its parcels and its blocks presume a terrain that waits between surveys — this one moves when the actors move. The chart's bottom stays put; it never measured what the mariners do to each other. The spectrum's boundaries are administered — frequencies are allocated, so no one had to discover who couples to whom; the shared media here, heat and buses and schemas, are assigned by no one. And the grid's wiring was drawn before its sensors ever watched: it observes a body it was handed, and discovery was never asked of it.

No medium's practice ever needed to discover coupling between autonomous actors, at extents that change with the question — because until now no territory was made of its actors' mutual influence. Maps follow territories. This one is made of relations: settings in motion pressing on other settings in motion, through wall, wire, or schema. Halt the actors and the territory goes dormant; the medium was the interacting itself. What each practice truly owned, this one took — the declared extent and the honest block from the ground, the self-correcting map from the water, the detector from the spectrum, the ownerless senses from the load. The old words are not wrong; each was built for another medium. The practice is the seam.

## The Survey

The work starts the way work on any territory starts: with a declaration. Someone draws an extent around the part they intend to tend — a power unit, a network, a floor — as small as the question at hand, overlapping its neighbors or not, as the operators configure. The extent is a province, not a fragment: worked at full depth, and never under obligation to join a wider map. Precision lives in the province.

Then comes a measurement no amount of watching could replace: one agent pushes — deliberately, within bounds it already respects — while its neighbor holds still, and everyone watches what moves. Push, release, compare. If the neighbor's objective shifts and recovers, a coupling path exists — wall, wire, or schema between them, the method does not care which. Run to depth, the probe does more than detect the path; it measures its shape: a response curve with error bars, and the bars are themselves measured, not decorative ([From Edges to Curves](https://doi.org/10.5281/zenodo.22127166)).

## The Noise Floor

Probing is not a preference; it is forced. Anything gathered while the agents merely run is dominated by their own trial-and-error noise, roughly five hundred times stronger than any coupling signal inside it. The engine's history keeps the honest number: an early attempt broadcast a faint continuous tone and listened for it downstream, and after six cascaded transforms the signal-to-noise ratio was 0.002. The lesson is the method's proverb: the signal has to be created at the scale of the channel, not the scale of the sensor. The concept pages carry the full story; the outcome is a method validated across linear, deeply nonlinear, and slowly drifting coupling, zero false positives on uncoupled controls, and limits published as a measured map rather than a promise ([The Impulse Protocol](https://doi.org/10.5281/zenodo.21962956)). And measured curves compose — through nonlinear paths, to a horizon of about two hops; beyond it, composing the pieces is the only remaining door ([From Curves to Cascades](https://doi.org/10.5281/zenodo.22401439)).

## The Living Map

Each measured edge is banked into a wiring map — the [connectome](https://godon-dev.github.io/godon-documentation/concept_connectome/) — that persists across campaigns, so every survey begins where the last one ended. The map is candid about its gaps: an absent edge means not yet probed, or too weak to measure, and the map does not claim to know which. And the map ages by nature, not by neglect — its terrain is parameter spaces, and parameter spaces keep changing; drift is not weather on the territory, it is the territory's substance moving. So the map is built to age: it predicts — if this node moves, that one answers — and when a prediction fails, the failure localizes the change and the neighborhood is measured again. A map that keeps itself honest by failing.

## The Practice

None of the measuring is for its own sake. An operator's intent — hold this throughput, keep this latency inside its band — can be planned against the measured map, applied, held while the world drifts, corrected in place when the bands are brushed. Holding intent inside a moving territory deserves its own essay. It is the next one.

Engineering finished its work at its layers. Identity, boundaries, policies — designed, verified, standing. What kept moving underneath was never going to be finished, because it is not that kind of thing. It is tended. Ground got the cadastre, water the chart, spectrum its allocations, load the grid; relations get the survey, the map, and the held intent. The territory is made of relations; the relations can be measured; the map answers for itself, one failed prediction at a time.

---

*This is the first in a series of articles on living systems, interference detection, and the future of tending. [godon](https://github.com/godon-dev/godon) is an open-source live systems tending and causal discernment engine.*
