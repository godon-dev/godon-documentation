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

Breeder configuration is YAML with three concepts only: **objectives** (what to optimize), **guardrails** (safety), and **observations** (extra channels to read). Parameters live under `settings` with per-parameter constraints. Interference characterization is a section, not a mode — every breeder in a group characterizes and optimizes concurrently.

---

### The Shape of a Breeder Config

```yaml
meta:
  configVersion: "0.3"
  strict_validation: false

breeder:
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

reconnaissance:                 # how this breeder reads its target
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

Any breeders sharing a `group` characterize each other. `convergence_threshold` is the single tuning knob: smaller means more re-measurement before a curve retires. Block sizes set the push/pause trial counts per probe. `hold_params` is the neutral position every breeder returns to when holding or pausing.

#### `objectives` vs `observations`

The separation is deliberate and load-bearing: objectives feed the optimizer's search; observations are collected per trial but never optimized — the detector reads both. A channel you suspect is coupled but don't want an agent chasing should be an observation, not an objective.

#### `reconnaissance`

How a breeder reads its target: HTTP endpoints (per-service `path`/`key`, sampling, aggregation), or Prometheus queries. See [Reconnaissance](concept_reconnaissance.md).

#### Guardrails

Safety limits with automatic response (fail trial, rollback, or skip target) are configured per strain and effectuator — see [Guardrails](concept_guardrails.md). The impulse protocol's pushes stay inside the same guardrails as ordinary optimization trials: same knobs, same bounds, no special permissions.

---

### Working Examples

Runnable, maintained examples live in the scenario library:
[`examples/bench/`](https://github.com/godon-dev/godon/tree/main/examples/bench) in the godon repository — each scenario ships breeder configs, target definitions, and the planted ground truth. The characterization scenarios are the reference implementations for the config above.

---

### See Also

- [Breeder](concept_breeder.md) — how configurations are executed
- [Interference Detection](concept_interference_detection.md) — what the interference section triggers
- [Getting Started](getting_started.md) — a full walkthrough against the generic bench
