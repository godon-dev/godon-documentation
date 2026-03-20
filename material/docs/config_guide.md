
## Configuration Guide

This guide walks through godon's configuration structure using examples from the [godon repository](https://github.com/godon-dev/godon/tree/master/examples).

### Configuration Overview

A godon configuration defines an optimization run — what to tune, how to apply changes, what to observe, and what constraints to enforce.

```
┌─────────────────────────────────────────────────────────────┐
│                    Configuration                             │
│                                                              │
│  ├── Search Space     (what parameters to tune)             │
│  ├── Objectives       (what to optimize)                    │
│  ├── Effectuator      (how to apply changes)                │
│  ├── Reconnaissance   (how to observe results)              │
│  └── Guardrails       (safety constraints)                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

### Example Configurations

Example configurations are available in the [godon examples directory](https://github.com/godon-dev/godon/tree/master/examples).

These examples cover common scenarios:

| Example | Description |
|---------|-------------|
| Database tuning | PostgreSQL, MySQL configuration optimization |
| Application tuning | Web server, cache configuration |
| Kubernetes tuning | Resource limits, replica counts |

---

### Configuration Sections

#### Search Space

Define parameters and their ranges:

```yaml
parameters:
  pool_size:
    type: integer
    bounds: [5, 50]
  
  timeout_ms:
    type: continuous
    bounds: [100.0, 5000.0]
  
  algorithm:
    type: categorical
    choices: [lru, lfu, fifo]
```

#### Objectives

Define what to optimize:

```yaml
objectives:
  - name: latency_p99
    direction: minimize
    
  - name: throughput
    direction: maximize
```

#### Effectuator

Define how to apply configurations:

```yaml
effectuator:
  type: ssh
  host: 10.0.0.50
  user: admin
  apply:
    command: |
      sed -i 's/pool_size=.*/pool_size={{ pool_size }}/' /etc/app/config.ini
      systemctl restart app
```

#### Reconnaissance

Define how to observe results:

```yaml
reconnaissance:
  type: prometheus
  endpoint: http://prometheus:9090
  metrics:
    - name: latency_p99
      query: histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[1m]))
```

#### Guardrails

Define safety limits:

```yaml
guardrails:
  - name: max_latency
    metric: latency_p99
    condition: "< 100ms"
    on_violation: fail_trial
```

---

### Full Example

See the [examples directory](https://github.com/godon-dev/godon/tree/master/examples) for complete, runnable configurations.

---

### See Also

- [Breeder](concept_breeder.md) — How configurations are executed
- [Effectuator](concept_effectuator.md) — Applying configurations
- [Reconnaissance](concept_reconnaissance.md) — Observing results
- [Guardrails](concept_guardrails.md) — Safety constraints
