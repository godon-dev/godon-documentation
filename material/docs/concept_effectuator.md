---
description: "godon Effectuator concept — applies configurations to target systems via SSH and HTTP channels. Idempotent operations, timing, and rollback support."
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

## Effectuator

An **effectuator** applies configurations to target systems — it's the bridge between abstract parameters and real-world changes. Effectuators take a parameter set and make it happen.

### Role in the Optimization Loop

```
┌─────────────────────────────────────────────────────────────┐
│                    Optimization Loop                         │
│                                                              │
│    Algorithm ──▶ Effectuator ──▶ Target System              │
│    (suggest)      (apply)         (changed)                 │
│                                                              │
│                                         │                    │
│                                         ▼                    │
│                                   Reconnaissance            │
│                                     (observe)               │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

The effectuator is responsible for making the algorithm's suggestions real.

---

### Effectuator Interface

All effectuators implement the same contract:

```
┌─────────────────────────────────────────────────────────────┐
│                     Effectuator                              │
│                                                              │
│  Input:  { param_a: 0.5, param_b: "foo", ... }             │
│                                                              │
│  Action: Apply configuration to target                      │
│                                                              │
│  Output: Success | Error                                     │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

| Phase | Responsibility |
|-------|----------------|
| **Validate** | Check parameters are applicable |
| **Apply** | Make changes to target system |
| **Verify** | Confirm changes took effect |
| **Report** | Return success or error details |

---

### Built-in Effectuators

Two channels ship today:

| Effectuator | Target | Mechanism |
|-------------|--------|-----------|
| **SSH** | Remote servers, VMs | Playbook execution via Windmill |
| **HTTP** | APIs, services | REST calls |

#### HTTP Effectuator

The bench path — parameters go to a target's apply endpoint:

```yaml
effectuation:
  type: http
  targetRefs: ["generic-node-1"]   # target created via the API
  endpoint_config:
    method: POST
    path: /apply
    timeout_seconds: 30
```

Use for: services with config APIs, bench nodes, control planes.

#### SSH Effectuator

The infrastructure path — a playbook applies the parameters:

```yaml
effectuation:
  type: ssh
  playbook_path: f/prod/sysctl    # Windmill flow that runs the playbook
```

Targets carry `address`, `username`, and `ssh_key_variable_path`; the
flow receives the parameters alongside them.

Use for: traditional servers, VMs, bare metal.

#### Adding a Channel

Effectuators are Windmill flows addressed as `f/effectuation/<type>` —
the breeder calls whatever flow the configured type names. A new channel
(a cloud API, a database, a hardware interface) is a new flow, not an
engine change.

---

### Idempotency

Effectuators should be **idempotent** — applying the same configuration twice has the same result as applying it once.

```
┌─────────────────────────────────────────────────────────────┐
│                    Idempotent Apply                          │
│                                                              │
│  Apply {pool_size: 10} ──▶ Success                          │
│  Apply {pool_size: 10} ──▶ Success (no-op)                  │
│  Apply {pool_size: 10} ──▶ Success (no-op)                  │
│                                                              │
│  Same result regardless of how many times applied           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Why it matters:**
- Retries don't cause corruption
- Rollbacks are predictable
- Parallel workers don't conflict

---

### Effectuation Timing

```
┌─────────────────────────────────────────────────────────────────┐
│                    Effectuation Timeline                        │
│                                                                  │
│  Start ──▶ Connect ──▶ Apply ──▶ Propagate ──▶ Steady ──▶ End  │
│    │          │          │           │           │              │
│    0s        1s         3s         10s         30s            │
│                                                                  │
│  Propagation delay: changes take time to reach full effect     │
└─────────────────────────────────────────────────────────────────┘
```

**Critical:** Reconnaissance samples only after a stabilization wait
(`stabilization_seconds`, default 2) — reading before the change has
propagated measures the old state.

---

### Rollback Support

Rollback is configured on the breeder side (`rollback_strategies` —
see [Guardrails](concept_guardrails.md)); the effectuator applies the
restored parameters like any other apply. Strategies restore the
previous successful trial, the best trial, or the baseline.

---

### Error Handling

Effectuation can fail for many reasons:

| Error Type | Cause | Recovery |
|------------|-------|----------|
| **Connection** | Network, auth | Failed trial; algorithm learns to avoid |
| **Validation** | Invalid params | Mark trial failed |
| **Timeout** | Slow target | Failed trial after `timeout_seconds` |

Failures are honest: a trial that could not be effectuated is failed,
counted, and — past the rollback threshold — restored.

---

### Safety Considerations

Effectuators modify real systems. The shipped safety line:

| Practice | How it ships |
|----------|--------------|
| Guardrail-bounded | Probe pushes use the same guardrails as ordinary trials |
| One change at a time | Turn-taking serializes senders during characterization |
| Audit trail | Every apply is a recorded trial with parameters and results |

---

### Summary

| Aspect | What it means |
|--------|---------------|
| **Role** | Apply parameters to target systems |
| **Channels** | SSH (playbook via Windmill), HTTP (apply endpoint) |
| **Contract** | Validate → Apply → Verify → Report |
| **Idempotency** | Same result on repeated applies |
| **Timing** | Stabilization wait before reconnaissance samples |
| **Extensibility** | New channel = new Windmill flow |

---

### See Also

- [Breeder](concept_breeder.md) — Orchestrates effectuators
- [Reconnaissance](concept_reconnaissance.md) — Observes what effectuators change
- [Guardrails](concept_guardrails.md) — Violations trigger rollback
- [Configuration Guide](config_guide.md) — The shipped config, end to end
