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

## Configuration Guide

Two surfaces carry intent into a running godon. The **systemtender config** is YAML — what a tender measures and moves. A **steerwish declaration** is made at runtime — what a holder asks the system to keep.

Systemtender configuration is YAML with three concepts only: **objectives** (what to optimize), **guardrails** (safety), and **observations** (extra channels to read). Parameters live under `settings` with per-parameter constraints. Interference characterization is a section, not a mode — every systemtender in a group characterizes and optimizes concurrently.

---

### The Shape of a Systemtender Config

```yaml
meta:
  configVersion: "0.3"
  strict_validation: false

systemtender:
  type: bench_generic

settings:                       # parameter search space
  generic:
    param_0:
      constraints:
        - {step: 20.0, lower: 0.0, upper: 100.0}
    param_1:
      constraints:
        - {step: 20.0, lower: 0.0, upper: 100.0}
    param_2:
      constraints:
        - {step: 20.0, lower: 0.0, upper: 100.0}

run:
  parallel: 1
  completion_criteria:
    iterations: {min: 10, max: 500}
    timing: {end: "75m"}

cooperation:
  active: false

interference_detection:         # characterization group membership
  group: bench-characterization
  mode: active
  convergence_threshold: 0.02   # the ONE tuning knob
  refinement_depth: 3
  push_block_size: 10
  pause_block_size: 10
  cooldown_trials: 5
  hold_params:                  # neutral position for hold/pause blocks
    param_0: 50.0
    param_1: 50.0
    param_2: 50.0

reconnaissance:                 # how this systemtender reads its target
  type: http
  http:
    url: "http://bench-generic:8090/node-1"

objectives:                     # what the optimizer optimizes
  - name: objective_0
    direction: maximize
    reconnaissance:
      service: http
      path: /metrics/json
      key: objective_0
      samples: 3
      aggregation: median

observations:                   # read but NOT optimized — detection channels
  - name: objective_1
    reconnaissance:
      service: http
      path: /metrics/json
      key: objective_1
```

---

### Sections

#### `settings` — the search space

Each parameter carries constraints. `step` sets the grid resolution; the characterization walk derives its level set from lower/upper (and refines between grid points when curves stay unresolved).

#### `interference_detection` — characterization

Any systemtenders sharing a `group` characterize each other. `convergence_threshold` is the single tuning knob: smaller means more re-measurement before a curve retires. Block sizes set the push/pause trial counts per probe. `hold_params` is the neutral position every systemtender returns to when holding or pausing.

#### `objectives` vs `observations`

The separation is deliberate and load-bearing: objectives feed the optimizer's search; observations are collected per trial but never optimized — the detector reads both. A channel you suspect is coupled but don't want an agent chasing should be an observation, not an objective.

#### `reconnaissance`

How a systemtender reads its target: HTTP endpoints (per-service `path`/`key`, sampling, aggregation), or Prometheus queries. See [Reconnaissance](concept_reconnaissance.md).

#### Guardrails

Safety limits with automatic response (fail trial, rollback, or skip target) are configured per strain and effectuator — see [Guardrails](concept_guardrails.md). The impulse protocol's pushes stay inside the same guardrails as ordinary optimization trials: same knobs, same bounds, no special permissions.

---

### The Shape of a Steerwish Declaration

Steering is declared at runtime, not written into a config file: a steerwish is a payload sent to the REST API (`/steerwishes`) or the `steerwish_declare` MCP tool. The grammar is a conjunction — one wish = one or more **claims** and optional **terms**, one verdict. Kept means every claim in its band and every term honored.

```yaml
claims:                        # the aims — N >= 1
  - outcome: chainend.shift
    band: {lo: -0.14, hi: -0.06, target: -0.10}
terms:                         # the protected readings — M >= 0
  - outcome: chainend.offset
    band: {lo: -0.02, hi: 0.02}
limits:
  exclude: [param_2]
  maxChange: 0.5
budget: 2
regime: standing
```

#### `claims` vs `terms`

Claims and terms share one shape — a named measured value plus a band — and mirror the objectives/observations split above: claims are what the wish keeps; terms are protected readings it must never push out of band while keeping them. A broken term is a miss with the same standing as a broken claim. Bands carry the outcome's own measurement units — the reading, never a movement. The `target` inside a band is for receipts and reporting, never judged.

#### The serving fields

- `limits` — parameters excluded from movement, and a maximum change per act; checked at plan time, a refusal names the binding one
- `budget` — the re-act allowance after drift events; omitted means upkeep indefinitely
- `regime` — the closing rule; `standing` today (held until closed)

An unkeepable wish is refused by name before anything acts. A declared wish is a durable, withdrawable record — lifecycle, event history, and the holder's correction of a standing wish are covered in [Steerwish](concept_steerwish.md).

---

### Working Examples

Runnable, maintained examples live in the scenario library:
[`examples/bench/`](https://github.com/godon-dev/godon/tree/main/examples/bench) in the godon repository — each scenario ships systemtender configs, target definitions, and the planted ground truth. The characterization scenarios are the reference implementations for the config above. For steerwish declarations, [`examples/bench/scenario-wish-stack/`](https://github.com/godon-dev/godon/tree/main/examples/bench/scenario-wish-stack) is the reference — a bench whose corridors couple the aims and the protected readings, so both claims and terms bind.

---

### See Also

- [Systemtender](concept_systemtender.md) — how configurations are executed
- [Steerwish](concept_steerwish.md) — what a declaration triggers, and its lifecycle
- [Interference Detection](concept_interference_detection.md) — what the interference section triggers
- [Getting Started](getting_started.md) — a full walkthrough against the generic bench
