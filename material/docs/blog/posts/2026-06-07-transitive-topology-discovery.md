---
date: 2026-06-07
tags:
  - essay
  - technical
authors:
  - matthias-tafelmeier
description: "Each detection reveals one edge in a coupling graph. Accumulated edges form a topology nobody designed and nobody mapped. Transitive signal tracing discovers the hidden structure of live systems."
---

# How Live Systems Reveal Their Own Structure — Transitive Topology Discovery

*Revised October 2026 — the sections on where the work stands now state what has since shipped, and the wording follows the engine's current terms: tending, not optimizing. The argument is unchanged.*

When godon detects interference between two systemtenders, it reveals one edge. Systemtender A's guarded pushes shift systemtender B's objectives. That's one coupling path — one connection you didn't know existed.

A single edge is useful. It tells you these two agents interfere. You can act on that — adjust parameters, add constraints, isolate them.

But a single edge is also a fragment of something larger. A topology.

<!-- more -->

## From Edges to Graphs

Consider a system with multiple autonomous agents. A resource agent on one subsystem. A thermal controller on another. A throughput agent on the shared bus. A quality controller on the output.

Pairwise detection between any two of them reveals whether they're coupled. Run detection across all pairs and you get a graph — who affects whom, through which coupling paths.

This graph wasn't designed. Nobody drew it. Nobody specified "agent A couples to agent C through shared memory bandwidth." It emerged from the system's dynamics — the shared state, the parameter interactions, the coupling channels that exist because the subsystems share physical or logical resources.

The topology is a property of the system itself. godon doesn't create it. It reveals it. One edge at a time.

## The Transitive Insight

The key insight is that detection is transitive. If systemtender A's pushes reach systemtender B, and systemtender B's pushes reach systemtender C, then there's a coupling chain A → B → C. The influence propagates through intermediate agents.

This matters because the operators of A and C might not know about B's role. A's configuration change affects B, which affects C. Without B in the picture, A → C looks like unexplained variance. With B, it becomes a traceable path.

Each detection is one observation. Accumulated observations form a graph. The graph is the topology of interference in the system — the hidden structure that explains why certain parameter changes have unpredictable side effects.

## What Makes This Different

Traditional system mapping is top-down. You document the architecture: these services talk to these databases over these network paths. The map is designed, maintained, and usually out of date.

Transitive topology discovery is bottom-up. You don't start with a map. You start with probes. Each probe reveals one connection that actually exists in the running system, right now. The map emerges from observation, not documentation.

This has several advantages:

- **It's empirical**, not theoretical. The edges are measured, not assumed. If two agents don't couple in practice, they don't appear in the graph — regardless of whether the architecture diagram says they share resources.
- **It's current**. The topology reflects the system as it is right now, not as it was when someone last updated the documentation.
- **It reveals unexpected connections**. Coupling can exist through paths nobody designed — shared thermal budgets, shared power delivery, indirect resource contention. These don't appear in any architecture diagram.

## Where It Stands

The mechanism is validated across coupling shapes — linear, deeply nonlinear cascaded, slowly drifting — with zero false positives on uncoupled controls, and the boundaries of the method are published as a measured map ([The Impulse Protocol](https://doi.org/10.5281/zenodo.21962956)). When systemtender A's push reaches systemtender B, we detect it — and measure it: the characterized edge carries a fitted response curve with honest error bars ([From Edges to Curves](https://doi.org/10.5281/zenodo.22127166)).

Topology assembly is no longer the next step — it exists. The engine keeps the [connectome](https://godon-dev.github.io/godon-documentation/concept_connectome/): every node and every characterized edge it currently believes in, accumulated across campaigns, durable, exportable as a certified artifact. The map this post argues for is now the engine's kept object.

What remains open — and what has closed since — is kept where it can stay current: the [open research](https://godon-dev.github.io/godon-documentation/open_research/) page.

## Why It Matters

If you're running two autonomous agents on a system, detecting interference between them is sufficient. You know they conflict. You act on it.

If you're running five, or ten, or fifty — pairwise detection isn't enough. You need the graph. You need to know which agents form clusters of mutual interference, which are isolated, and where the critical coupling paths are. The topology tells you where to focus your decoupling efforts, where coordination would have the most impact, and where the system is most fragile.

## The Topology as Data

The topology is knowledge about the system itself — independent of any single campaign. It describes the hidden structure that affects every agent operating on the system.

This knowledge is no longer ephemeral. The connectome persists: each campaign banks its measured edges in the map, and the next starts from where the last one left off. The topology — once discovered — is durable. It changes slowly because the physical and logical coupling paths change slowly. Shared memory bandwidth doesn't rearrange itself overnight. Thermal coupling between subsystems persists across software updates.

Each detection refines the graph. Each campaign adds edges. The topology accumulates. Over time, the map becomes more complete than any single campaign could produce.

This accumulated topology is a dataset in its own right. It can be shared across teams, across deployments, across organizations running similar systems. The topology of coupling in one microgrid informs expectations about another. The interference patterns in one cluster reveal likely coupling paths in a similar configuration.

The map is the product. Not the campaign results — those belong to a single run. The topology is structural knowledge about how the system actually behaves.

## The Deeper Implication

Each detection is one edge. Accumulated edges form a topology. The topology describes the system's hidden coupling structure.

But there's something deeper: the topology is a property the system reveals about itself, through its own dynamics. You don't need to model the system. You don't need a digital twin or a simulation. You embed probes in the agents that are already running, and the system tells you its own structure.

This is perception without modeling. Observation without assumption. The system reveals what it is, one edge at a time, because you gave it a way to speak.

That's the core thesis of godon. Not detection for its own sake, but detection as a method for understanding live systems through their own behavior. The system is the model. You just need to listen.

---

*This is the third in a series of articles on living systems, interference detection, and the future of tending. [godon](https://github.com/godon-dev/godon) is an open-source live systems tending and causal discernment engine.*
