---
description: "godon Steerwish concept — intent made keepable: declared claims (named measured values with bands) under terms, brought into their bands and held. Record and serving loop are live — declare, list, get, close, update; the full drift arc is the demonstration now closing out."
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

A **steerwish** is intent made keepable: a holder — human or AI mind — states a chosen point for a live system, and the engine holds it there. The intent is declared, never invented by the engine; the engine's work starts from that statement.

A wish carries one or more **claims** — each a named measured value to be brought into its band and held there — under optional **terms**. A chosen point may sit at an optimum, but need not: a steerwish asks for conditions to keep, not scores to push.

Holding a chosen point within bounds is an old idea — thermostats keep temperature, SLOs keep latency, control loops keep setpoints. What we know of no open counterpart for is the conjunction: the band is declared on a measured map rather than an assumed model; the keeping runs continuously against a drifting system; an unkeepable wish is refused by name rather than silently degraded; and the keeping is priced, visibly, in trials. The declarer is a holder — human or AI mind; what never happens is the serving layer inventing its own intents. Fragments of this live in control, in optimization, in observability. The whole is the steerwish.

The connectome is what a wish is held on — the outcome must resolve to a measured entry in the map ([Connectome](concept_connectome.md)).

### The Record Is the Safety Device

The declaration is a durable, withdrawable record — the safety device for everything that acts on it. Validation happens at the door: a malformed wish is rejected before any planning.

Closing is equally explicit. A closed wish binds nothing — history, not law. On close, the serving side releases its setting back to neutral — the un-steered baseline.

A standing wish the world has moved beyond is neither silently abandoned nor silently rewritten. The holder decides: close it, or **correct** it in place — the same wish re-aimed to new terms, stamped with the previous band, identity and trail intact. Correcting, not replacing, is the primitive: a replacement wish for the same intent loses the trail.

### Coordinates

A wish carries one or more **claims**; each claim names one measured value and the band to keep it in:

| Field | Meaning |
|-------|---------|
| **outcome** | Plain name of the measured value — must resolve to exactly one entry in the connectome's outcome registry |
| **band** | `lo` / `hi` / `target` — the acceptable band, in the outcome's own measurement units |

The remaining fields are the **terms** — the fences around how the wish may be served, not what is kept:

| Field | Meaning |
|-------|---------|
| **limits** | Parameters excluded from movement, a maximum change per act — checked at plan time; a refusal names the binding one |
| **budget** | Re-act allowance after drift events; omitted means upkeep indefinitely |
| **regime** | The closing rule — `standing` today (held until closed); time-bound and event-bound close rules are planned |

One judge rules the whole conjunction, claims first, then terms: every claim inside its band, every term respected. A broken term counts as a miss. What cannot be kept is refused — by name, never silently.

### Lifecycle

```
declared ──▶ planned ──▶ acted ──▶ landed
    │           │                   │
    │           └──▶ refused        ├──▶ missed ──▶ re_opened
    │                               │
    └───────────┬───────────────────┴──▶ closed
                └──▶ corrected ──▶ planned   (same wish, new terms)
```

The events — `declared`, `planned`, `refused`, `acted`, `landed`, `missed`, `re_opened`, `corrected`, `closed` — are the truth: the full history is kept, and the state is derived from it on read, never stored. Each event may carry its evidence; a refusal names its binding constraint ("target outside measured range").

Judging is in/out of band only. The target inside the band is for receipts and reporting — not a grade, not a score to optimize.

### What Ships Today

The record and the serving loop are live. The surface — declare, list, get, close, and **update** (the holder's correction of a standing wish) — is served through the REST API (`/steerwishes`) and the `steerwish_*` MCP tools, with validation at the door and the full event history on every read.

The serving loop — the map planning the input setting, acting on it, judging against the band, and re-acting within budget as the system drifts — has flown end-to-end: a wish landed and held its band through a live run. The full drift arc — hold, world moves, the holder corrects, re-land — is the demonstration now being closed out; see [Open Research](open_research.md). What stays stable is the record's contract above.

---

### See Also

- [Connectome](concept_connectome.md) — The map a wish is held on
- [Guardrails](concept_guardrails.md) — Limits are guardrails on serving
- [Interference Detection](concept_interference_detection.md) — The coupling a wish moves through
- [Open Research](open_research.md) — The steering rung
