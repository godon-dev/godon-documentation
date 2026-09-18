---
description: "godon Connectome concept — the measured wiring map of a live system: nodes, directed edges, and response curves with honest error bars. Living partial map and citable artifact; asking it predicts without touching the system."
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

## Connectome

The **connectome** is the system's measured wiring map: every node and every characterized edge the system currently believes in — each edge carrying a fitted response, a confidence, and a noise floor. It is the formal name for the map the engine keeps.

The map is not inferred from observation. It is measured by driven pressure: one parameter is pushed, the responses elsewhere are read, and coupling that answers becomes an edge with a curve. The live system is the model — the connectome is its measured part.

### Partial by Design

Curves exist only where probed. The connectome is an honest map of what has been measured, not a guessed model of everything — an absent edge means *not probed here* or *below the noise floor*, and the map does not pretend to know which.

### Two Forms

| Form | Where | Nature |
|------|-------|--------|
| **Live connectome** | `/connectome` | The living partial map — every node and characterized edge currently believed in |
| **Artifact** | `/connectome/artifact` | The certified export, persisted at last build — complete at build time, the citable object |

The distinction is deliberate: the live map grows with every probe; the artifact freezes a build of it for citation and comparison.

```
┌──────────────────────────────────────────────────────────────┐
│                    Building and Asking                        │
│                                                               │
│   probe ──▶ measure ──▶ live connectome ──▶ artifact          │
│  (push)     (read)       (grows, partial)    (frozen, citable)│
│                                                               │
│   predict / impact / causes ──▶ answers, never touches        │
│                                the running system             │
└──────────────────────────────────────────────────────────────┘
```

### What an Edge Carries

Per edge, the connectome holds **response curves**: probe levels with measured shifts and honest error bars (`/connectome/curves`). A curve is the coupling's measured shape — not a coefficient asserted by a model, and not more precise than the noise floor allows.

### Asking the Connectome

Reads never touch the system — the map answers; the running system is not disturbed.

| Question | Operation | Answer |
|----------|-----------|--------|
| If this node moves by X, what does its direct neighbor do? | `predict` | One-hop shift, linearized |
| What happens at the far end of a measured path? | `predict/multihop` | Composed cascade shift along measured edges |
| What has one systemtender's probing moved? | `impact/{id}` | Measured impact across the map |
| What upstream causes feed one systemtender's patch? | `causes/{id}` | Measured upstream causes |

Refusal instead of extrapolation: an ask outside the measured range is refused — the map says so, it does not guess. Composition along measured paths is validated at its boundary, not assumed: 95/97 referee points within propagated 2σ bars, with a composition horizon of ≈2 nonlinear hops at σ=0.02 ([Publications](publications.md)).

### How It Connects

[Interference detection](concept_interference_detection.md) finds candidate coupling; characterization walks measure the curves; the connectome accumulates both. [Systemtenders](concept_systemtender.md) produce it by probing under [guardrails](concept_guardrails.md) — and [steerwishes](concept_steerwish.md) are held on it: a wish is a declared outcome with coordinates in the map.

### Surface

The connectome is readable through the REST API's `/connectome` family (relayed to the causal service), through the `connectome_*` MCP tools, and in the OpenAPI `connectomes` group — see [API](api.md) and [MCP Interface](mcp.md).

---

### See Also

- [Interference Detection](concept_interference_detection.md) — Finds the candidate edges
- [Characterization](characterization.md) — The walks that measure the curves
- [Systemtender](concept_systemtender.md) — Produces the map by probing
- [Guardrails](concept_guardrails.md) — Every probe is bounded
- [Steerwish](concept_steerwish.md) — Outcomes held on the map
