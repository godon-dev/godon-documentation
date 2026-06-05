---
description: "godon Reconnaissance concept — observes target systems after effectuation using Prometheus, HTTP, and script sources. Timing, aggregation, and noise handling."
---

## Reconnaissance

**Reconnaissance** observes the target system after effectuation — collecting metrics, checking health, and gathering the data needed to evaluate fitness. It's the eyes of the optimization loop.

### Role in the Optimization Loop

```
┌─────────────────────────────────────────────────────────────┐
│                    Optimization Loop                         │
│                                                              │
│    Effectuator ──▶ Target System ──▶ Reconnaissance        │
│    (apply)          (changed)        (observe)              │
│                                                              │
│                                           │                  │
│                                           ▼                  │
│                                      Metrics &              │
│                                      Observations           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

Reconnaissance answers: "What happened after we made that change?"

---

### Reconnaissance Interface

```
┌─────────────────────────────────────────────────────────────┐
│                     Reconnaissance                           │
│                                                              │
│  Input:  Trial context (what was applied)                   │
│                                                              │
│  Action: Collect observations from target                   │
│                                                              │
│  Output: { metric_a: 123.4, metric_b: "ok", ... }          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

| Phase | Responsibility |
|-------|----------------|
| **Wait** | Allow system to reach steady state |
| **Collect** | Gather metrics from sources |
| **Aggregate** | Combine multiple observations |
| **Report** | Return structured data |

---

### Built-in Reconnaissance Sources

| Source | What it provides | Protocol |
|--------|------------------|----------|
| **Prometheus** | Time-series metrics | PromQL |
| **HTTP** | Health checks, stats | REST/HTTP |
| **Custom scripts** | Arbitrary observations | Exec |

#### Prometheus Reconnaissance

```yaml
reconnaissance:
  type: prometheus
  endpoint: http://prometheus:9090
  
  metrics:
    - name: latency_p99
      query: histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[1m]))
      
    - name: throughput
      query: sum(rate(http_requests_total[1m]))
      
    - name: error_rate
      query: sum(rate(http_errors_total[1m])) / sum(rate(http_requests_total[1m]))
```

Use for: Cloud-native systems, Kubernetes, microservices

#### HTTP Reconnaissance

```yaml
reconnaissance:
  type: http
  endpoint: https://api.example.com/stats
  method: GET
  
  extract:
    latency_p99: ".metrics.latency.p99"
    active_connections: ".connections.active"
```

Use for: Services with stats endpoints, health APIs

#### Script Reconnaissance

```yaml
reconnaissance:
  type: script
  command: /opt/scripts/collect_metrics.sh
  
  # Script outputs JSON to stdout
  parse: json
```

Use for: Legacy systems, databases, custom metrics

---

### Timing and Sampling

Reconnaissance timing affects data quality:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Reconnaissance Timeline                       │
│                                                                  │
│  Effectuation        Reconnaissance Window                      │
│       │                   ├─────────────────┤                   │
│       │                   │                 │                   │
│       ▼                   ▼                 ▼                   │
│  ─────●───────────────────●─────────────────●──────────────▶    │
│       │                   │                 │                   │
│       0s                wait: 30s        duration: 60s         │
│                                                                  │
│  Wait for steady state, then observe for duration              │
└─────────────────────────────────────────────────────────────────┘
```

| Parameter | Purpose |
|-----------|---------|
| **wait** | Delay before first observation (propagation time) |
| **duration** | How long to observe |
| **interval** | How often to sample within duration |

```yaml
reconnaissance:
  timing:
    wait: 30s        # Let changes propagate
    duration: 60s    # Observe for 1 minute
    interval: 5s     # Sample every 5 seconds
```

---

### Aggregation

Multiple samples → single observation:

| Aggregation | Use Case |
|-------------|----------|
| **mean** | Typical value |
| **median** | Robust to outliers |
| **p95/p99** | Tail behavior |
| **max** | Worst case |
| **min** | Best case |

```yaml
reconnaissance:
  metrics:
    - name: latency
      query: ...
      aggregation: p99  # Report 99th percentile
```

---

### Multi-Source Reconnaissance

Combine observations from multiple sources:

```yaml
reconnaissance:
  sources:
    - name: prometheus
      type: prometheus
      endpoint: http://prometheus:9090
      metrics:
        - latency_p99
        - throughput
        
    - name: app_stats
      type: http
      endpoint: http://app:8080/stats
      extract:
        - active_threads
        - queue_depth
```

All sources contribute to the trial's observation data.

---

### Handling Missing Data

Reconnaissance may fail to collect some metrics:

| Scenario | Handling |
|----------|----------|
| Metric unavailable | Mark trial failed |
| Partial data | Continue with available metrics |
| Timeout | Fail or use partial data |

```yaml
reconnaissance:
  on_missing: fail  # or: ignore, default_value
  timeout: 30s
```

---

### Noise and Stability

Real metrics have noise:

```
┌─────────────────────────────────────────────────────────────┐
│                    Noisy Observations                        │
│                                                              │
│  True value: 50ms                                           │
│  Observed:   [48, 52, 47, 55, 49, 51, 48, 53, ...]         │
│                                                              │
│  Aggregation smooths noise for fitness evaluation           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Mitigation strategies:**

| Strategy | How it helps |
|----------|--------------|
| Longer duration | More samples = less variance |
| Robust aggregation | Median resists outliers |
| Multiple trials | Average over repeated evaluations |
| Replication | Run same config multiple times |

---

### Reconnaissance vs Objectives

| Reconnaissance | Objectives |
|----------------|------------|
| Collects raw metrics | Computes fitness |
| System-specific | Problem-specific |
| Multiple values | Single/directional |
| Descriptive | Evaluative |

```
Reconnaissance: { latency_p99: 45ms, throughput: 8000, error_rate: 0.001 }
                                        │
                                        ▼
Objective: minimize latency_p99  ──▶  fitness = 45
```

---

### Summary

| Aspect | What it means |
|--------|---------------|
| **Role** | Observe system after effectuation |
| **Sources** | Prometheus, HTTP, scripts |
| **Timing** | Wait for steady state, then sample |
| **Aggregation** | Combine samples into observations |
| **Output** | Metrics dict used for fitness and guardrails |

---

### See Also

- [Effectuator](concept_effectuator.md) — What reconnaissance observes
- [Guardrails](concept_guardrails.md) — Check reconnaissance data against limits
- [Breeder](concept_breeder.md) — Orchestrates the optimization loop
