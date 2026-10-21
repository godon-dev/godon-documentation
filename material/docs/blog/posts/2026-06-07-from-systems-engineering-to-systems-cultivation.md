---
date: 2026-06-07
tags:
  - essay
authors:
  - matthias-tafelmeier
description: "Autonomous agents coupled through shared memory, heat, and power form a territory whose terrain is relations — measured extent by extent, from a power unit to a floor, at overlaps the operators draw. The practice that tends it, and the disciplines that never shared a table until it existed."
---

# A Territory Made of Relations

*Rewritten October 2026. The original argued a shift from engineering to cultivation and carried a garden metaphor; both retired. This version states what the territory is, what tends it, and what it draws from fields that never met. The anchors are the same earned ones.*

Agricultural statistics and radar detection never met. One was invented to find a treatment's effect amid the variation of farm soils; the other, to find a known return amid unknown noise in the sky. They meet in this essay, in a place neither discipline imagined: a territory of autonomous agents coupled through shared memory, heat, and power. The block structure of the method below comes from the first parent; the honest detector comes from the second. What neither supplies is the territory. That is new, and it is the subject.

<!-- more -->

## The Territory

On any floor that runs software, agents now adjust things on their own. A cooling controller follows a temperature. A scheduler rebalances load. A search agent tunes its own parameters, trial by trial. When the scheduler's move shifts the cooling controller's readings, the controller reacts to a cause it cannot name — nothing in its metrics says where the shift came from.

The terrain here is not the machines. It is what happens between them — relations. Two agents sharing memory bandwidth are coupled whether or not anyone drew the line; heat crosses racks; voltage crosses a bus. None of this coupling is on any diagram, because none of it was designed. And the terrain has no joints of its own: it does not come divided into parts.

## The Dimensions

The divisions are drawn. An operator declares an extent — this power distribution unit, or this network, or this floor — and the extent is a hypothesis about where the inside ends. Extents run from local to further out: a map of one unit's couplings, a map of a network's, a map of a floor's, each at its own distance. They overlap or not, exactly as the operators configure — the unit's map and the network's may share edges, or none. From a single unit to a whole floor, the same practice applies at every scale, and each extent keeps its own map. Nothing about the territory forces the choice; that is what makes it a practice rather than a discovery.

## What Meets Here

Nothing in the practice is unprecedented; what is unprecedented is the table. Agricultural field trials contributed the block design — push, pause, compare — the structure that isolates one effect amid variation. Radar and sonar contributed the detector that holds its false-alarm rate constant when the noise is unknown. Control theory contributed the held state and the guardrail. Cartography contributed the declared extent, drawn over ground that never drew itself. Each field gave one organ. None had a territory of autonomous, reconfigurable agents to assemble the whole for, because until recently there was none.

## The Practice

Then the inside of an extent is measured — and in a territory of relations, the anatomy cannot be watched; it has to be probed. One agent applies a guarded push to parameters it already controls, inside bounds it already respects, while the other holds still. Push, release, compare. If the neighbor's objective shifts and recovers, a coupling path exists, and the same probe run to depth measures its shape: a response curve with honest error bars ([From Edges to Curves](https://doi.org/10.5281/zenodo.22127166)).

Watching cannot substitute. The agents' own trial-and-error wiggling — the noise any search makes — sits roughly five hundred times above the coupling signal in everything gathered without probing. An earlier, gentler attempt in the engine's history learned this at a number: a faint continuous tone, listened for downstream, that six cascaded transforms ate down to a signal-to-noise of 0.002 — the concept pages carry the story, and the lesson that came back: the signal has to be created at the scale of the channel, not the sensor. The method that answers is validated across coupling shapes — linear, deeply nonlinear cascaded, slowly drifting — with zero false positives on uncoupled controls, and its boundaries are published as a measured map ([The Impulse Protocol](https://doi.org/10.5281/zenodo.21962956)). Measured curves compose through nonlinear paths to a horizon of about two hops; beyond it, composing the pieces is the only remaining door ([From Curves to Cascades](https://doi.org/10.5281/zenodo.22401439)).

Each measured edge is banked into a wiring map — the [connectome](https://godon-dev.github.io/godon-documentation/concept_connectome/): every edge the extent currently believes in, each with its curve and its error. It is partial by design. An absent edge means not probed here, or below the noise floor, and the map does not pretend to know which. It persists across campaigns; each one begins where the last ended.

And the maps correct themselves, because relations move when the actors do. A frozen map that still claims a dead edge is a phantom limb. The cure is prediction: a measured map predicts — if this node moves, that one answers — and when the prediction fails, the failure points to where the territory changed. The neighborhood is re-measured. That loop runs today in its first rung; what stands open at any moment lives on the [open research](https://godon-dev.github.io/godon-documentation/open_research/) page.

## The Steering in It

The map is for something. An operator's intent — hold this throughput, keep this latency inside its band — is planned against the measured map, applied, and held while the world drifts; judged against declared bands; corrected in place; released when done. Holding intent inside the territory deserves its own essay. It is the next one.

## The Close

Engineering finished its layers. Identity, boundaries, policies — designed, verified, standing; nothing here argues with them. The layers that keep moving were never engineering's to finish, because they are not the kind of thing that finishes. They are tended. The territory is made of relations; the relations are measured; the measurement is kept honest by its own failures.

---

*This is the first in a series of articles on living systems, interference detection, and the future of tending. [godon](https://github.com/godon-dev/godon) is an open-source live systems tending and causal discernment engine.*
