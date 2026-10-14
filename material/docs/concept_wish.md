---
description: "godon Wish concept — declared intent on a live system. A hold (steer) keeps chosen outcomes in their bands under terms; a commissioned walk (seek) searches a region of dials under a budget and returns the best found with its error bars and the trials spent. Declare, list, get, close, correct — live end to end."
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

## Wish

A **wish** is intent made keepable: a holder — human or AI mind — states what they want from a live system, and the engine serves that statement. The intent is declared, never invented by the engine; the engine's work starts from that declaration.

A wish states its **type**. A **steer** wish holds: one or more **claims**, each a named measured value to be brought into its band and kept there, under optional **terms** — the fences around how the wish may be served. A **seek** wish commissions a measured walk: a **region** of dials to search, and either a **criterion** with a **budget** of measured trials (**find_best**) or no budget at all (**cover_now**, which walks the region to extend the map's coverage). A chosen point may sit at an optimum, but need not: a steer wish asks for conditions to keep, not scores to push; a seek asks for the best the system can show within a declared budget — and comes home with the bill.

Holding a chosen point within bounds is an old idea — thermostats keep temperature, SLOs keep latency, control loops keep setpoints. What we know of no open counterpart for is the conjunction: the band is declared on a measured map rather than an assumed model; the keeping runs continuously against a drifting system; an unkeepable wish is refused by name rather than silently degraded; and the serving is priced, visibly, in measured trials. The declarer is a holder — human or AI mind; what never happens is the serving layer inventing its own intents. Fragments of this live in control, in optimization, in observability. The whole is the wish.

The connectome is what a wish is served on — a steer claim must resolve to a measured entry in the map ([Connectome](concept_connectome.md)); a seek's criterion names a reading the map knows. And when the map already answers a seek's question, the wish is refused with the reading inside the refusal — the refusal is the answer.

### The Record Is the Safety Device

The declaration is a durable, withdrawable record — the safety device for everything that acts on it. Validation happens at declaration: a malformed wish is rejected before any planning, with the reason.

Closing is equally explicit. A closed wish binds nothing — history, not law. On close, the serving side releases its setting back to neutral — the un-steered baseline.

A standing wish the world has moved beyond is neither silently abandoned nor silently rewritten. The holder decides: close it, or **correct** it in place — the same wish re-aimed to new terms, stamped with the previous band, identity and trail intact. Correcting, not replacing, is the primitive: a replacement wish for the same intent loses the trail.

### Coordinates — the Hold's Grammar

A steer wish carries one or more **claims**; each claim names one measured value and the band to keep it in:

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

### The Walk's Grammar

A seek wish carries its own fields:

| Field | Meaning |
|-------|---------|
| **form** | `find_best` — search the region under a budget — or `cover_now` — walk the region now to extend map coverage; carries no budget |
| **region** | The dials to walk, each with its bounds |
| **criterion** | find_best: what counts — the reading and the direction (`maximize` / `minimize`) |
| **budget** | find_best: the measured trials the walk may spend |
| **candidates** | Optional measured launch points; they count against the same budget |

A find_best returns the best setting found, with its error bars and the number of trials spent. Budget exhaustion still returns the best so far — there is no missed verdict for a walk, only a return. Refusals happen at declaration: a question the map already answers is refused with the reading; ground the map has not measured is named in the refusal.

### Lifecycle

```
steer:  declared ──▶ planned ──▶ assigned ──▶ acted ──▶ landed
            │           │                       │
            │           └──▶ refused            ├──▶ missed ──▶ re_opened
            │                                   │
            └───────────┬───────────────────────┴──▶ closed
                        └──▶ corrected ──▶ planned   (same wish, new terms)

seek:   declared ──▶ planned ──▶ assigned ──▶ returned
            │           │
            │           └──▶ refused        (may carry the answer)
            └──▶ plan_error
```

The events — `declared`, `planned`, `assigned`, `refused`, `acted`, `landed`, `missed`, `re_opened`, `corrected`, `returned`, `closed`, `plan_error` — are the truth: the full history is kept, and the state is derived from it on read, never stored. Each event may carry its evidence; a refusal names its binding constraint ("target outside measured range"). `plan_error` marks a planning failure on either grammar; the reason rides the event.

Judging a hold is in/out of band only. The target inside the band is for evidence and reporting — not a grade, not a score to optimize. A walk is not judged at all: it returns the best found, with its bars and its spend.

### What Ships Today

The record and the serving loop are live. The surface — declare, list, get, close, and **update** (the holder's correction of a standing wish) — is served through the REST API (`/wishes`) and the `wish_*` MCP tools, with validation at declaration and the full event history on every read. Both grammars are served: holds planned against the measured map, acted, judged against their bands; commissioned walks executing find_best and cover_now over their regions, their returns mirrored into the event trail. The seek walk's live end-to-end validation is the current work.

The serving loop for holds has flown end-to-end: a wish landed and held its band through a live run. The full drift arc — hold, world moves, the holder corrects, re-land — is the demonstration now being closed out; see [Open Research](open_research.md). What stays stable is the record's contract above.

---

### See Also
- [Connectome](concept_connectome.md) — The map a wish is served on
- [Systemtender](concept_systemtender.md) — The agent that serves wishes and runs the walks
- [Guardrails](concept_guardrails.md) — Limits are guardrails on serving
- [Interference Detection](concept_interference_detection.md) — The coupling a wish moves through
- [API](api.md) — the `/wishes` surface
- [MCP Interface](mcp.md) — wish tools for LLM clients
- [Open Research](open_research.md) — The steering rung
