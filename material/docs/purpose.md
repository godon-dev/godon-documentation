---
description: "godon purpose and positioning — Open Source Live Systems Tending and Causal Discernment Engine. Revealing hidden causal structure through driven pressure, for human and AI co-pilots."
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

## Summary

Godon is an open source engine for **discerning and tending** live systems.

Every system has two configurations. The documented one — architecture diagrams, IaC, config files. And the real one — the emergent causal structure where changing parameter X on component A silently shifts the behavior of component B through a shared physical channel.

The first is solved. The second doesn't exist as a discipline. Godon reveals it.

## The Hidden Structure Problem

When autonomous agents — optimizers, controllers, AI — share physical substrate, they silently interfere through coupling channels nobody designed and nobody monitors.

No observability tool can reveal this, because the coupling signal is never collected. No AI can reason about it, because the data doesn't exist. Passive monitoring fundamentally cannot distinguish "improving together because coupled" from "improving together because both converging."

**Active perturbation is the only path.** Probe the system with known signals, hold other parts still, measure who responds. The coupling topology emerges from the system's own behavior — not modeled, not assumed, discovered.

## What Flows From Discovery

Revealing hidden coupling structure is the enabling step. What you do with that knowledge is where the value compounds.

### Diagnostics

"Your latency regression traces back to a parameter change in a neighboring optimizer 20 minutes ago through a thermal coupling path." The topology doesn't just tell you what's connected — it tells you where the problem came from and through what channel.

### Coordination

Once breeders know who they're coupled to, they can adapt. Instead of blindly corrupting each other through shared substrate, they account for the coupling — constraining parameters that propagate, scheduling around active edges, avoiding interference.

### Tending

Steer the whole system jointly, accounting for coupling, toward better operating points. Not each optimizer independently chasing its own optimum at the expense of neighbors, but coordinated improvement that respects the real topology.

### Isolation Certification

The same signal that discovers coupling can prove its absence. Certify that two workloads are genuinely isolated — not by policy, but empirically verified at runtime. Critical for multi-tenant environments and regulated industries.

### System Understanding

The discovered topology is knowledge about the system itself — independent of any single optimization campaign. It reveals bottleneck resources that concentrate coupling, critical paths that nobody designed, and clusters of tightly-coupled agents that need coordination.

## Human + AI Co-pilots

Human operators and LLM agents work alongside each other throughout the
entire loop — not stacked in a pipeline, but as collaborating co-pilots
with different strengths. Multiple LLM agents can work in parallel on the
same godon cluster, each focusing on a different aspect.

Humans contribute:

- **Domain expertise** — deep knowledge of the system's physics, constraints,
  and operational realities that no model carries
- **Operational judgment** — when to push, when to hold, when something
  looks wrong, when to intervene
- **Charter design** — translate goals into structured optimization
  campaigns: parameters, objectives, guardrails, constraints
- **Architecture decisions** — given a discovered coupling topology, decide
  what to do: isolate, schedule, constrain, restructure

LLMs contribute:

- **Charter generation** — draft optimization charters from natural language
  intent, propose parameter spaces and objectives
- **System comprehension** — interpret discovered coupling topology and
  explain what it means: "Your GPU job's batch size is silently degrading
  training throughput through L3 cache contention"
- **Operational awareness** — continuous commentary on what godon is finding,
  flagging anomalies, suggesting when to probe deeper or when to act
- **Strategy proposal** — given a discovered topology, propose decoupling
  strategies and parameter adjustments. Godon tests them empirically

They work in parallel, not in sequence. An LLM agent drafts a charter while
the human reviews a coupling topology from a previous run. A second LLM
agent monitors ongoing trials while the human decides whether to adjust
guardrails. Godon discerns and tends underneath all of them, feeding
structured trial data back to whoever needs it.

Co-pilots guide and comprehend. The engine discerns and acts. Nobody
touches the system directly except godon — human and LLM suggestions flow
through the breeder, get tested against reality, and are kept or discarded
based on measured outcomes.

## Scope

Anything reachable via network that has tunable parameters and measurable outcomes:

- **Infrastructure** — GPU clusters, HPC, databases, networking, storage
- **Platforms** — application runtimes, message queues, caches
- **Physical systems** — power grids, industrial control, HVAC, microgrids
- **Multi-tenant environments** — cloud isolation verification, regulated industries

Not limited to Kubernetes or cloud-native. The coupling problem exists wherever multiple processes share a physical medium.

## Capabilities

| Capability | Description |
|------------|-------------|
| **Topology Discovery** | Discover internal live system interplay topology from behavior — not assumed from diagrams or models |
| **Diagnostics** | Trace regressions and anomalies to their actual source through discovered coupling paths |
| **Coordination** | Breeders adapt their behavior given known coupling — constraining, scheduling, avoiding interference |
| **Live System Tending** | Continuously steer toward better operating points with guardrails and rollback |
| **Isolation Certification** | Empirically verify that workloads are genuinely decoupled — proving absence |
| **Co-pilot Integration** | Human and LLM domain knowledge validated in the loop — charter design, comprehension, strategy |
| **Multi-Algorithm Exploration** | Metaheuristics (TPE, NSGA-II/III, QMC) explore parameter spaces, with optional ML integration |
| **Training-Free** | No training data or model training required — learns from live system behavior |
| **Self-Hosted** | Runs on your infrastructure. AGPLv3. No vendor lock-in |

## Caveats

| Caveat | Description |
|--------|-------------|
| **Not a Monitoring Tool** | Monitoring observes. Godon experiments. Different epistemological category |
| **Not a Reasoning Engine** | LLMs reason about what is represented. Godon discerns what isn't — the coupling signal that was never collected |
| **Not a Digital Twin** | No simulation, no upfront modeling. The live system is the model |
| **Human-AI Supervision Required** | Not fully hands-off — requires human and/or LLM setup, supervision, and planning |
| **No Guaranteed Best Solution** | Approximates better-than-untouched states rather than finding global optimum |
| **ML-Light by Default** | Open to lightweight ML/RL where it enhances search, but ML is not the core paradigm |
| **Application, Not Framework** | Applies metaheuristics to live systems; not a metaheuristics library |

## Positioning

Humans and LLMs work alongside godon, not above it:

| Layer | What Happens | Who |
|-------|-------------|-----|
| **Discernment** | Probe, measure, discover hidden causal structure | **Godon** |
| **Comprehension** | Interpret topology, explain findings, draft charters, propose strategies | LLM agents + Human operators |
| **Tending** | Steer the system toward better operating points | **Godon** |
| **Reality** | Execute, measure, provide feedback | Live systems |

Multiple LLM agents and humans work in parallel on the comprehension layer.
Godon handles discernment and tending underneath — the engine that makes
their work grounded in measured reality.

## Principles

- Active perturbation over passive observation
- Empirical discovery over assumed models
- Cultivation over one-shot optimization
- Open source. AGPLv3.
