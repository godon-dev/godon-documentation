---
description: "godon Guardrails concept — hard safety limits protecting production systems during optimization. Numeric hard limits, consecutive-violation rollback, strategies."
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

## Guardrails

**Guardrails** are safety limits that protect production systems during optimization. Unlike objectives which guide the search, guardrails enforce hard limits — trials that violate them are rejected.

### Guardrails vs Objectives

| Aspect | Objective | Guardrail |
|--------|-----------|-----------|
| **Purpose** | What to optimize | What to avoid exceeding |
| **Behavior** | Continuous guidance | Binary pass/fail |
| **Violation** | Suboptimal result | Trial rejected |
| **Example** | Minimize latency | Temperature must stay < 40 |

Objectives say "make this better." Guardrails say "don't break this."

---

### How Guardrails Work

```
┌─────────────────────────────────────────────────────────────┐
│                      Trial Execution                         │
│                                                              │
│    Parameters ──▶ Effectuate ──▶ Reconnoiter ──▶ Check      │
│                                                      │       │
│                                         ┌────────────┴───┐   │
│                                         │   Guardrails?   │   │
│                                         └────────────┬───┘   │
│                                                      │       │
│                              ┌───────────────────────┼─────┐ │
│                              ▼                       ▼     │ │
│                         Violated                  OK       │ │
│                            │                        │       │ │
│                            ▼                        ▼       │ │
│                    Mark FAILED            Accept trial       │ │
│                    Count failure          Update best        │ │
└─────────────────────────────────────────────────────────────┘
```

Guardrails check reconnaissance data after each trial.

---

### Defining Guardrails

A guardrail is a named metric with a numeric upper bound:

```yaml
guardrails:
  - name: cpu_usage
    hard_limit: 90.0
  - name: max_temp
    hard_limit: 40.0
```

| Field | Meaning |
|-------|---------|
| `name` | Which reconnaissance metric to check |
| `hard_limit` | Numeric upper bound — a value above it is a violation |

Plain by design: a number, not an expression. A metric that cannot be read or evaluates to infinity counts as a violation — the safe direction is the default.

The same limits bound the impulse protocol's probe pushes: characterization pushes stay inside the same guardrails as ordinary optimization trials. Same knobs, same bounds, no special permissions.

---

### Evaluation

All guardrails must pass for a trial to succeed:

```
  cpu_usage:    85%    ≤ 90%    ✓ PASS
  max_temp:     38°C   ≤ 40°C   ✓ PASS
                          → Trial accepted

  cpu_usage:    85%    ≤ 90%    ✓ PASS
  max_temp:     45°C   ≤ 40°C   ✗ FAIL
                          → Trial rejected
```

Any fail → trial rejected.

---

### On Violation: Fail, then Roll Back

A violating trial is marked failed and the failure is counted. Consecutive failures trip the rollback:

```
┌─────────────────────────────────────────────────────────────┐
│                     Rollback Flow                            │
│                                                              │
│  Trial 1: OK ──▶ Trial 2: VIOLATED ──▶ Trial 3: VIOLATED   │
│      │                │                       │              │
│      ▼                ▼                       ▼              │
│  last_good         failures=1             failures=2        │
│  = params_1                                   │              │
│                                               ▼              │
│                                    ┌──────────────────┐      │
│                                    │ Threshold hit?   │      │
│                                    │ (e.g., 10)       │      │
│                                    └────────┬─────────┘      │
│                                             │                │
│                              ┌──────────────┴──────────┐    │
│                              ▼                         ▼    │
│                            Yes                        No    │
│                              │                         │    │
│                              ▼                         ▼    │
│                    Apply last_good params       Continue     │
│                    Reset failure counter                    │
│                              │                              │
│                              ▼                              │
│                       After-policy: continue / pause / stop │
└─────────────────────────────────────────────────────────────┘
```

```yaml
rollback_strategies:
  standard:
    consecutive_failures: 10   # Rollback after N failed trials
    target_state: previous     # previous, best, or baseline
    max_attempts: 3
    on_failure: continue       # continue, stop, or skip_target
    timeout_seconds: 60
    after:                     # What to do after a rollback
      action: pause            # continue, pause, or stop
      duration: 300
```

---

### Rollback Strategies

| Strategy | What it restores | Use case |
|----------|------------------|----------|
| `previous` | Last successful trial's parameters | Conservative, safe |
| `best` | Best Pareto-optimal trial | Aggressive, assumes best is stable |
| `baseline` | Empty/original configuration | Fallback to known defaults |

---

### Guardrails in Multi-Objective Optimization

Guardrails constrain the Pareto front: trials that violate a guardrail are excluded from it. The effective front is the unguarded front cut at the limit — tradeoffs beyond the boundary are invisible to the optimizer, by design.

---

### Guardrail Design Guidelines

| Guideline | Why |
|-----------|-----|
| Set guardrails before objectives | Safety first |
| Start conservative | Loosen after proven safe |
| Monitor guardrail hit rate | Too many hits = search space issue |

**Too tight:** Algorithm can't explore, all trials fail.

**Too loose:** Risk of production incidents.

---

### Summary

| Aspect | What it means |
|--------|---------------|
| **Purpose** | Protect systems during optimization |
| **Shape** | Named metric + numeric hard limit |
| **Behavior** | Binary pass/fail, all must pass |
| **Violation** | Trial rejected; consecutive failures roll back |
| **Rollback** | previous / best / baseline, with an after-policy |

---

### See Also

- [Reconnaissance](concept_reconnaissance.md) — Provides metrics for guardrail checks
- [Effectuator](concept_effectuator.md) — Applies the restored parameters on rollback
- [Breeder](concept_breeder.md) — Configures and enforces guardrails
- [Configuration Guide](config_guide.md) — The shipped config, end to end
