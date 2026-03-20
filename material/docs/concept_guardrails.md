
## Guardrails

**Guardrails** are safety limits that protect production systems during optimization. Unlike objectives which guide the search, guardrails enforce hard constraints — trials that violate them are rejected.

### Guardrails vs Objectives

| Aspect | Objective | Guardrail |
|--------|-----------|-----------|
| **Purpose** | What to optimize | What to avoid |
| **Behavior** | Continuous guidance | Binary pass/fail |
| **Violation** | Suboptimal result | Trial rejected |
| **Example** | Minimize latency | Latency must stay < 100ms |

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
│                    Trigger rollback       Update best        │ │
└─────────────────────────────────────────────────────────────┘
```

Guardrails check reconnaissance data after each trial.

---

### Defining Guardrails

```yaml
guardrails:
  - name: max_latency
    metric: latency_p99
    condition: "< 100ms"
    on_violation: fail_trial
    
  - name: error_budget
    metric: error_rate
    condition: "< 0.01"
    on_violation: fail_trial
    
  - name: cpu_limit
    metric: cpu_usage_percent
    condition: "< 90"
    on_violation: fail_trial
```

| Field | Meaning |
|-------|---------|
| `metric` | Which reconnaissance value to check |
| `condition` | Threshold expression |
| `on_violation` | What to do when violated |

---

### Condition Syntax

| Operator | Meaning | Example |
|----------|---------|---------|
| `<` | Less than | `latency < 100` |
| `<=` | Less than or equal | `errors <= 10` |
| `>` | Greater than | `throughput > 1000` |
| `>=` | Greater than or equal | `uptime >= 0.99` |
| `==` | Equals | `status == "healthy"` |
| `!=` | Not equals | `state != "degraded"` |

---

### Guardrail States

```
┌─────────────────────────────────────────────────────────────┐
│                    Guardrail Evaluation                      │
│                                                              │
│  Metric value ──▶ Compare ──▶ [PASS] or [FAIL]             │
│                                                              │
│  latency_p99 = 45ms                                         │
│  condition: < 100ms                                         │
│  result: PASS                                               │
│                                                              │
│  error_rate = 0.05                                          │
│  condition: < 0.01                                          │
│  result: FAIL                                               │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

| State | Meaning |
|-------|---------|
| **PASS** | Metric within bounds, trial continues |
| **FAIL** | Metric exceeds limit, trial rejected |

---

### Multiple Guardrails

All guardrails must pass for a trial to succeed:

```
┌─────────────────────────────────────────────────────────────┐
│                   Guardrail Checks                           │
│                                                              │
│  latency_p99: 45ms    < 100ms    ✓ PASS                     │
│  error_rate:   0.001  < 0.01     ✓ PASS                     │
│  cpu_usage:    85%    < 90%      ✓ PASS                     │
│                                                              │
│  All pass → Trial accepted                                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                   Guardrail Checks                           │
│                                                              │
│  latency_p99: 45ms    < 100ms    ✓ PASS                     │
│  error_rate:   0.05   < 0.01     ✗ FAIL                     │
│  cpu_usage:    85%    < 90%      (skipped)                  │
│                                                              │
│  Any fail → Trial rejected                                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

### On Violation

When a guardrail fails, several actions are possible:

| Action | What happens |
|--------|--------------|
| `fail_trial` | Mark trial failed, continue optimization |
| `fail_and_rollback` | Mark failed, restore previous config |
| `stop_breeder` | Halt optimization entirely |

```yaml
guardrails:
  - name: critical_error_rate
    metric: error_rate
    condition: "< 0.1"
    on_violation: fail_and_rollback
    
  - name: catastrophic_failure
    metric: system_down
    condition: "== false"
    on_violation: stop_breeder
```

---

### Rollback on Violation

Consecutive guardrail violations can trigger automatic rollback:

```
┌─────────────────────────────────────────────────────────────┐
│                    Rollback Flow                             │
│                                                              │
│  Trial 1: OK ──▶ Trial 2: VIOLATED ──▶ Trial 3: VIOLATED   │
│      │                │                       │              │
│      ▼                ▼                       ▼              │
│  last_good         failures=1             failures=2        │
│  = params_1                                   │              │
│                                               ▼              │
│                                    ┌──────────────────┐      │
│                                    │ Threshold hit?   │      │
│                                    │ (e.g., 3)        │      │
│                                    └────────┬─────────┘      │
│                                             │                │
│                              ┌──────────────┴──────────┐    │
│                              ▼                         ▼    │
│                            Yes                        No    │
│                              │                         │    │
│                              ▼                         │    │
│                    Apply last_good params              │    │
│                    Reset failure counter               │    │
│                              │                         │    │
│                              └────────────┬────────────┘    │
│                                           ▼                  │
│                                    Continue trials            │
└─────────────────────────────────────────────────────────────┘
```

```yaml
rollback:
  enabled: true
  consecutive_failures: 3   # Trigger after N violations
  target_state: previous    # previous, best, or baseline
  on_failure: stop          # stop, continue, or skip_target
```

---

### Rollback Strategies

| Strategy | What it restores | Use Case |
|----------|------------------|----------|
| `previous` | Last successful trial | Conservative, safe |
| `best` | Best Pareto-optimal trial | Aggressive, assumes best is stable |
| `baseline` | Original configuration | Fallback to known defaults |

---

### Soft vs Hard Guardrails

**Soft guardrails** — Warning threshold, continue optimization

**Hard guardrails** — Hard limit, fail trial immediately

```yaml
guardrails:
  - name: latency_warning
    metric: latency_p99
    condition: "< 80ms"
    severity: soft      # Log warning, continue
    
  - name: latency_limit
    metric: latency_p99
    condition: "< 100ms"
    severity: hard      # Fail trial
```

Soft guardrails inform without blocking. Hard guardrails protect.

---

### Guardrail Design Guidelines

| Guideline | Why |
|-----------|-----|
| Set guardrails before objectives | Safety first |
| Start conservative | Loosen after proven safe |
| Monitor guardrail hit rate | Too many hits = search space issue |
| Use soft before hard | Graduated response |

**Too tight:** Algorithm can't explore, all trials fail

**Too loose:** Risk of production incidents

---

### Guardrails in Multi-Objective Optimization

Guardrails constrain the Pareto front:

```
              Throughput
                    ▲
                    │
                    │    · ·  ·   (outside guardrail)
                    │  ·   · ·    
                    │    · X      ← guardrail boundary
                    │  ·   ·
                    │    · · ·   (inside guardrail)
                    │
                    └─────────────────────▶
                       Latency

Only trials inside guardrail boundary are considered
```

The effective Pareto front excludes guardrail-violating trials.

---

### Summary

| Aspect | What it means |
|--------|---------------|
| **Purpose** | Protect systems during optimization |
| **Behavior** | Binary pass/fail on metric thresholds |
| **Violation** | Trial rejected, possible rollback |
| **Types** | Soft (warning) vs hard (fail) |
| **Rollback** | Auto-restore on consecutive failures |

---

### See Also

- [Reconnaissance](concept_reconnaissance.md) — Provides metrics for guardrail checks
- [Effectuator](concept_effectuator.md) — May execute rollback
- [Breeder](concept_breeder.md) — Configures and enforces guardrails
