---
description: "godon Steerwish concept — a declared wish with coordinates: a named measured outcome brought into its band and held. The record ships today (declare, list, get, close); the serving loop is the active development rung."
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

## Steerwish

A **steerwish** is a declared wish with coordinates: a named measured value is to be brought into its band and held there. A wish is a chosen state, not a maximum — it asks for a condition to keep, not a score to push upward. Wishes are declared, never invented by the engine: someone states the outcome; the engine's work starts from that statement.

The connectome is what a wish is held on — the outcome must resolve to a measured entry in the map ([Connectome](concept_connectome.md)).

### The Record Is the Safety Device

The declaration is a durable, withdrawable record — the safety device for everything that acts on it. Validation happens at the door: a malformed wish is rejected before any planning.

Closing is equally explicit. A closed wish binds nothing — history, not law. On close, the serving side releases its setting back to neutral.

### Coordinates

| Field | Meaning |
|-------|---------|
| **outcome** | Plain name of the measured value — must resolve to exactly one entry in the connectome's outcome registry |
| **band** | `lo` / `hi` / `target` — the acceptable band, in the outcome's own measurement units |
| **limits** | How the wish may be served: parameters excluded from movement, a maximum change per act — checked at plan time; a refusal names the binding one |
| **budget** | Re-act allowance after drift events; omitted means upkeep indefinitely |
| **regime** | The closing rule — `standing` today (held until closed); time-bound and event-bound close rules are planned |

### Lifecycle

```
declared ──▶ planned ──▶ acted ──▶ landed
    │           │                   │
    │           └──▶ refused        ├──▶ missed ──▶ re_opened
    │                               │
    └───────────────────────────────┴──▶ closed
```

The events — `declared`, `planned`, `refused`, `acted`, `landed`, `missed`, `re_opened`, `closed` — are the truth: the full history is kept, and the state is derived from it on read, never stored. Each event may carry its evidence; a refusal names its binding constraint ("target outside measured range").

Judging is in/out of band only. The target inside the band is receipt and reporting — not a grade, not a maximization score.

### What Ships Today

The record surface: declare, list, get, close — through the REST API (`/steerwishes`) and the `steerwish_*` MCP tools, with validation at the door and the full event history on every read.

The serving loop — the map planning the input setting, acting on it, judging against the band, and re-acting within budget as the system drifts — is in active development, the project's current research rung; see [Open Research](open_research.md). What stays stable is the record's contract above.

---

### See Also

- [Connectome](concept_connectome.md) — The map a wish is held on
- [Guardrails](concept_guardrails.md) — Limits are guardrails on serving
- [Interference Detection](concept_interference_detection.md) — The coupling a wish moves through
- [Open Research](open_research.md) — The steering rung
