---
description: "godon Reconnaissance concept — observes target systems after effectuation using Prometheus and HTTP sources. Stabilization, sampling, and aggregation."
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
│  Output: { metric_a: 123.4, metric_b: 42.0, ... }          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

| Phase | Responsibility |
|-------|----------------|
| **Stabilize** | Wait for the change to propagate |
| **Collect** | Gather samples from sources |
| **Aggregate** | Combine samples into one value |
| **Report** | Return structured data |

---

### Reconnaissance Sources

| Source | What it provides | Protocol |
|--------|------------------|----------|
| **HTTP** | Stats and metrics endpoints | REST/HTTP |
| **Prometheus** | Time-series metrics | PromQL |

Two sources ship today — a known limit, not the design's boundary; more are in development. The per-objective block (path/key/samples/aggregation) is the stable interface each source fills.

#### HTTP Reconnaissance

How a breeder reads its target — per objective (and per observation),
sampled and aggregated:

```yaml
reconnaissance:                 # where the target lives
  type: http
  http:
    url: "http://bench-generic:8090/node-1"

objectives:                     # what gets read, how
  - name: objective_0
    direction: maximize
    reconnaissance:
      service: http
      path: /metrics/json
      key: objective_0
      samples: 3
      aggregation: median
```

Use for: services with stats endpoints, bench nodes, health APIs.

#### Prometheus Reconnaissance

Per-objective PromQL queries against a Prometheus endpoint — a global
default with per-objective override — sampled and aggregated the same
way as HTTP reads.

Use for: cloud-native systems, Kubernetes, microservices.

---

### Timing and Sampling

```
┌─────────────────────────────────────────────────────────────────┐
│                    Reconnaissance Timeline                       │
│                                                                  │
│  Effectuation        Stabilization      Sampling                │
│       │               ├────────┤        ● ● ●                   │
│       │               │        │        │                       │
│       ▼               ▼        ▼        ▼                       │
│  ─────●───────────────●────────●────────●──────────────────▶    │
│       │                                                  trials │
│                                                              │
│  Wait for the change to land, then take N samples             │
└─────────────────────────────────────────────────────────────────┘
```

| Parameter | Purpose |
|-----------|---------|
| `stabilization_seconds` | Wait before first sample (default 2s — propagation time) |
| `samples` | How many samples to take per read |
| `aggregation` | How the samples collapse to one value |

---

### Aggregation

Multiple samples → single observation:

| Aggregation | Use case |
|-------------|----------|
| **median** | Default — robust to outliers |
| **mean** | Typical value when noise is symmetric |
| **min** / **max** | Bounds |

```yaml
reconnaissance:
  service: http
  path: /metrics/json
  key: latency
  samples: 5
  aggregation: median
```

---

### Multiple Channels

Every objective and every observation is its own reconnaissance block —
one target read through as many channels as you care to name:

```yaml
objectives:                     # optimized
  - name: objective_0
    direction: maximize
    reconnaissance: {service: http, path: /metrics/json, key: objective_0, samples: 3, aggregation: median}

observations:                   # read, never optimized — detection channels
  - name: objective_1
    reconnaissance: {service: http, path: /metrics/json, key: objective_1, samples: 3, aggregation: median}
```

Observations feed the detector, not the optimizer — a channel you
suspect is coupled but don't want an agent chasing belongs here.

---

### Handling Missing Data

A metric that cannot be read does not silently pass:

| Scenario | Handling |
|----------|----------|
| No valid samples | The read fails — the trial fails |
| Read error | No value; counts as a guardrail violation if a limit watches it |

The safe direction is the default: absence is a failure, not a zero.

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
| More samples | Less variance in the aggregate |
| Median aggregation | Resists outliers |
| The ABA block design | Push/pause contrast separates signal from drift |

---

### Reconnaissance vs Objectives

| Reconnaissance | Objectives |
|-----------------|------------|
| Collects raw metrics | Computes fitness |
| System-specific | Problem-specific |
| Multiple values | Single/directional |
| Descriptive | Evaluative |

```
Reconnaissance: { latency: 45ms, throughput: 8000, error_rate: 0.001 }
                                        │
                                        ▼
Objective: minimize latency  ──▶  fitness = 45
```

---

### Summary

| Aspect | What it means |
|--------|---------------|
| **Role** | Observe system after effectuation |
| **Sources** | HTTP endpoints, Prometheus queries |
| **Timing** | Stabilization wait, then N samples |
| **Aggregation** | median (default), mean, min, max |
| **Output** | Metrics dict used for fitness, guardrails, and detection |

---

### See Also

- [Effectuator](concept_effectuator.md) — What reconnaissance observes
- [Guardrails](concept_guardrails.md) — Check reconnaissance data against limits
- [Breeder](concept_breeder.md) — Orchestrates the optimization loop
- [Configuration Guide](config_guide.md) — The shipped config, end to end
