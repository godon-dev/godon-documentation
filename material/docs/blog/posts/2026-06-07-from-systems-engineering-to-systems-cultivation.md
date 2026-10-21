---
date: 2026-06-07
tags:
  - essay
authors:
  - matthias-tafelmeier
description: "Below engineering's finished layers lies a territory made of relations — autonomous actors coupled through shared physics. The practice that tends it: declare an extent, measure its couplings, keep the map, correct it when its predictions fail."
---

# A Territory Made of Relations

*Rewritten October 2026 — the original essay argued a shift from engineering to cultivation and carried a garden metaphor. Both retired. The point about layers survives unadorned; the argument here is new; the anchors are the same earned ones.*

Engineering finished its layers. Identity settings, boundaries, policies — designed, verified, standing. At those layers the work is done, and done well; nothing here argues with it.

<!-- more -->

Below them are the layers that keep moving, and they were never engineering's to finish.

Autonomous actors have lived inside our systems for more than a century — control loops since the ship autopilot, later autoscalers, schedulers, search agents. Each is a small mind with one purpose: hold this, maximize that, keep this within bounds. Millions of them, awake around the clock, acting on shared systems.

They are coupled, and not by design. Two actors on one machine share memory bandwidth; on one rack, heat; on one power bus, voltage. When one pushes, another's objective shifts. Neither knows.

Together these moving layers form a territory — and the territory is made of relations. Not ground, not water: coupling. Its geology is workload interference, its rivers are thermal flows, its weather is contention. A territory whose terrain is what the inhabitants do to each other.

## The Companions, and Where Each Breaks

Maps follow territories, so it is tempting to reach for the old mapmakers' words. Each one breaks.

The word **body** breaks first. Bodies have skin, one owner, edges of their own — this territory has none of those; carve it anywhere and it continues. The **holobiont**, the symbiotic collective, at least gathers many into one, but presumes a host to organize around. There is no host here. The **garden** fails differently: a garden is a population with nothing wired inside it — nothing to measure between the beds.

The old physical world's instruments fail at the same place from the other side. The cadastral map holds declared boundaries — over ground that never moves. The living nautical chart, corrected for centuries by its own users' reports, measures a bottom, passively; it never measured what the mariners do to each other. Traffic systems share the actors' positions, never their coupling. The power grid — the one ownerless body that did grow wide-area senses, after blackouts whose reports named the blindness — never needed to discover its wiring: the grid was designed, its diagram drawn. Its sensors watch a known body.

Each companion holds one face. None holds the conjunction — discovered coupling between autonomous actors, at extents that change with the question — because until now no territory of relations existed to need one. This is not a claim about anyone's cleverness. It is a claim about the territory: it is new, and maps follow territories.

## Declare

A territory with no joints of its own leaves the joints to us. That is not a lack; it is the practice, and it begins with a declaration.

An extent is declared: a power distribution unit, a network, a floor. As small as the question at hand, overlapping its neighbors or not, as the operators configure. The declared extent is a hypothesis — this is the inside that matters for what we intend — and like any hypothesis it can be redrawn when the question moves.

## Measure

Then the inside is measured, and in a territory of relations the anatomy cannot be watched; it has to be probed. One actor applies a guarded push to parameters it already controls, inside bounds it already respects, while the others hold still. Push, release, compare. If a neighbor's objective shifts and recovers, a coupling path exists — and the same probe, run to depth, measures its shape: a response curve with honest error bars ([From Edges to Curves](https://doi.org/10.5281/zenodo.22127166)).

The statistics are radar's family: a known-shape response detected in unknown noise, at a constant false-alarm rate. Passive watching cannot substitute — the actors' own exploration noise sits roughly five hundred times above any coupling signal in the gathered data. The method is validated across coupling shapes — linear, deeply nonlinear cascaded, slowly drifting — with zero false positives on uncoupled controls, and its boundaries are published as a measured map ([The Impulse Protocol](https://doi.org/10.5281/zenodo.21962956)). Curves compose through nonlinear paths to a horizon of about two hops; beyond it, composing the measured pieces is the only remaining door ([From Curves to Cascades](https://doi.org/10.5281/zenodo.22401439)).

## Keep

The measured edges accumulate into a local body schema — the [connectome](https://godon-dev.github.io/godon-documentation/concept_connectome/): every edge the extent currently believes in, each with its curve and its error. It is partial by design — an absent edge means not probed here, or below the noise floor, and the map does not pretend to know which. Every honest map is weighted this way. It persists across campaigns: each one banks its edges, and the next begins where the last ended.

## Correct

Maps of relations age faster than maps of ground, because relations move when the actors do. A frozen connectome claiming an edge that no longer exists is a phantom limb. Prediction error sheds the phantoms: a measured map enables prediction — if this node moves, that one answers — and when the prediction fails, the failure points to where the territory changed. The neighborhood is re-measured. The first rung of that loop runs today; what stands open at any moment is kept on the [open research](https://godon-dev.github.io/godon-documentation/open_research/) page.

## The Practice

Declare, measure, keep, correct. One move is missing from that list — holding an intent against the measured map while the world drifts — and it is the subject of an essay to come.

Engineering finished its layers. The layers that keep moving were never finished, because they are not the kind of thing that finishes. They are tended. The territory is made of relations; the relations are measured; the measurement is kept honest by its own failures.

---

*This is the first in a series of articles on living systems, interference detection, and the future of tending. [godon](https://github.com/godon-dev/godon) is an open-source live systems tending and causal discernment engine.*
