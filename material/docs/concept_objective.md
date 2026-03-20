
## Objective

An **objective** defines what you're optimizing — the goal that guides the search. Objectives translate raw metrics into a fitness signal that algorithms can optimize.

### Single-Objective Optimization

The simplest case: one goal to minimize or maximize.

```
┌─────────────────────────────────────────────────────────────┐
│                 Single Objective                            │
│                                                              │
│  Minimize: latency_p99                                      │
│                                                              │
│  Trial A: 45ms  ──▶  fitness = 45                          │
│  Trial B: 32ms  ──▶  fitness = 32  ◀── better              │
│  Trial C: 78ms  ──▶  fitness = 78                          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

| Direction | Example |
|-----------|---------|
| **Minimize** | Latency, error rate, cost |
| **Maximize** | Throughput, accuracy, efficiency |

---

### Defining an Objective

```yaml
objectives:
  - name: latency_p99
    direction: minimize
    source: reconnaissance.latency_p99
```

The objective pulls a value from reconnaissance metrics and assigns a direction.

---

### Multi-Objective Optimization

Real systems often have competing goals:

```
┌─────────────────────────────────────────────────────────────┐
│                 Multi-Objective                             │
│                                                              │
│  Minimize: latency                                          │
│  Maximize: throughput                                       │
│                                                              │
│  Trial A: latency=30ms, throughput=5000 rps                 │
│  Trial B: latency=50ms, throughput=8000 rps                 │
│  Trial C: latency=40ms, throughput=7000 rps                 │
│                                                              │
│  Which is best? It depends...                              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Trade-offs are inevitable:**
- Lower latency often means lower throughput
- Higher accuracy may require more resources
- Better quality may increase cost

---

### Pareto Optimality

When objectives conflict, there's no single best — only **Pareto-optimal** solutions:

```
              Throughput (maximize)
                    ▲
                    │
              8000 ┤       · B
                    │    ·   C
              6000 ┤
                    │
              4000 ┤  · A
                    │     · D (dominated)
              2000 ┤
                    └─────────────────────▶
                       20    40    60
                    Latency (minimize)

Pareto front: A, B, C (none dominates another)
Dominated: D (worse than C on both objectives)
```

**Pareto-optimal** — No other trial is better on all objectives

**Dominated** — Some other trial is better on all objectives

---

### Pareto Front

The set of all Pareto-optimal solutions:

```
┌─────────────────────────────────────────────────────────────┐
│                                                              │
│   Objective 1 ──▶  [Trial A, Trial B, Trial C, ...]        │
│   Objective 2 ──▶                                          │
│                    └──▶ Pareto Front                       │
│                                                              │
│   The frontier of best trade-offs                          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

The breeder maintains the Pareto front across all trials, using algorithms like NSGA-II or NSGA-III.

---

### Scalarization

For single-objective algorithms, combine multiple objectives into one:

```yaml
scalarization:
  type: weighted_sum
  weights:
    latency: 0.7
    throughput: 0.3
```

| Method | Formula | Trade-off |
|--------|---------|-----------|
| Weighted sum | `w1·obj1 + w2·obj2` | Requires normalization |
| Weighted product | `obj1^w1 × obj2^w2` | Handles different scales |
| Chebyshev | `max(w1·obj1, w2·obj2)` | Finds extreme points |

**Limitation:** Scalarization cannot find all Pareto-optimal solutions for non-convex fronts.

---

### Choosing Objectives

| Good Objective | Bad Objective |
|----------------|---------------|
| Measurable | Subjective |
| Responsive to parameters | Unaffected by changes |
| Low noise | High variance |
| Clear direction | Ambiguous goal |

**Tips:**

| Tip | Why |
|-----|-----|
| Start with one objective | Simpler to understand and debug |
| Add objectives sparingly | Each adds dimensionality |
| Proxy metrics are OK | If they correlate with real goals |
| Avoid redundant objectives | They don't add information |

---

### Objective vs Guardrail

| Aspect | Objective | Guardrail |
|--------|-----------|-----------|
| **Purpose** | Optimize toward | Enforce limit |
| **Behavior** | Continuous guidance | Binary pass/fail |
| **Violation** | Suboptimal result | Trial rejected |
| **Example** | Minimize latency | Latency must be < 100ms |

Objectives guide the search. Guardrails protect the system.

---

### Multi-Objective Algorithm Selection

| Algorithm | Objectives | Best For |
|-----------|------------|----------|
| TPE | 1 | Single-objective, sample-efficient |
| NSGA-II | 2-3 | Few objectives, good convergence |
| NSGA-III | 4+ | Many objectives, reference-point based |

---

### Summary

| Aspect | What it means |
|--------|---------------|
| **Definition** | What to minimize or maximize |
| **Direction** | Minimize (costs) or maximize (benefits) |
| **Single** | One goal, clear best solution |
| **Multi** | Competing goals, Pareto front |
| **Scalarization** | Combining objectives for single-objective algorithms |

---

### See Also

- [Search Space](concept_search_space.md) — Parameters being optimized
- [Guardrails](concept_guardrails.md) — Safety limits vs optimization goals
- [Breeder](concept_breeder.md) — Algorithm selection for objectives
