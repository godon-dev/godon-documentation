
## Search Space

The **search space** defines what configurations the breeder can explore — the boundaries and structure of parameters being optimized. It constrains what's possible and guides how algorithms sample.

### Defining a Search Space

```
┌─────────────────────────────────────────────────────────────┐
│                      Search Space                            │
│                                                              │
│     ┌─────────────────────────────────────────────┐         │
│     │  param_a: [0.0, 1.0]  (continuous)          │         │
│     │  param_b: [10, 100]   (integer)             │         │
│     │  param_c: {a, b, c}   (categorical)         │         │
│     └─────────────────────────────────────────────┘         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

A search space is a collection of parameters, each with:
- **Name** — Identifier
- **Type** — Continuous, integer, or categorical
- **Domain** — Valid range or set of values

---

### Parameter Types

| Type | Domain | Example | Use Case |
|------|--------|---------|----------|
| **Continuous** | `[min, max]` | `[0.0, 1.0]` | Learning rates, ratios |
| **Integer** | `{min...max}` | `{1, 2, 3, ..., 10}` | Thread counts, batch sizes |
| **Categorical** | `{choice1, choice2, ...}` | `{sgd, adam, rmsprop}` | Algorithm selection |

**Continuous parameters** offer infinite choices within bounds.

**Integer parameters** discretize the space — useful for counts.

**Categorical parameters** represent qualitative choices without ordering.

---

### Example Search Space

```yaml
parameters:
  # Connection pool sizing
  pool_size:
    type: integer
    bounds: [5, 50]
  
  # Timeout configuration
  timeout_ms:
    type: continuous
    bounds: [100.0, 5000.0]
  
  # Algorithm selection
  algorithm:
    type: categorical
    choices: [lru, lfu, fifo]
  
  # Feature flag
  cache_enabled:
    type: categorical
    choices: [true, false]
```

---

### Search Space Geometry

The structure of the search space affects optimization:

```
2D Continuous Space          Categorical + Continuous

     param_b                     algorithm
       ▲                           │
  max ┤     ·  ·  ·                ├── a ── param_x: [0,1]
       │   ·  ·  ·                 │
       │   ·  ·  ·                 ├── b ── param_x: [0,1]
  min ┤     ·  ·  ·                │
       └─────────────▶             └── c ── param_x: [0,1]
         min    max
         param_a
```

**Dimensionality** grows with parameter count — more dimensions means harder search.

**Categorical choices** create separate subspaces — algorithms must explore each.

---

### Constraints

Parameters may have relationships that constrain valid combinations:

```yaml
constraints:
  # pool_size must be >= connection_count
  - pool_size >= connection_count
  
  # If cache_enabled is false, cache_size is irrelevant
  - implies: 
      if: cache_enabled == false
      then: ignore: cache_size
```

| Constraint Type | Example |
|-----------------|---------|
| **Bound** | `x >= 0` |
| **Relational** | `threads <= cpu_count` |
| **Conditional** | If feature disabled, skip related params |
| **Mutual exclusion** | Only one of {a, b} can be set |

Constraints prune invalid regions, but complex constraints can fragment the search space.

---

### Logarithmic Scales

Some parameters span orders of magnitude:

```yaml
learning_rate:
  type: continuous
  bounds: [0.0001, 1.0]
  scale: log  # Sample uniformly in log space
```

| Scale | Sampling | Use Case |
|-------|----------|----------|
| Linear | Uniform in value | Most parameters |
| Log | Uniform in log(value) | Learning rates, timeouts |

Log scale prevents over-sampling at one end of a wide range.

---

### Search Space vs Solution Space

```
┌───────────────────┐         ┌───────────────────┐
│   Search Space    │         │  Solution Space   │
│                   │         │                   │
│  Parameters       │  ────▶  │  Outcomes         │
│  (what we try)    │         │  (what we get)    │
│                   │         │                   │
│  [0, 1] × [0, 1]  │         │  Fitness landscape│
└───────────────────┘         └───────────────────┘
```

**Search space** — What we directly explore

**Solution space** — The mapping from parameters to fitness

A simple search space can have a complex solution space (rugged fitness landscape).

---

### Choosing a Search Space

**Too narrow:**
- Misses optimal configurations
- Algorithm has nothing to explore

**Too broad:**
- Includes invalid or irrelevant regions
- Takes longer to find good solutions

**Guidelines:**

| Guideline | Why |
|-----------|-----|
| Start broad, then narrow | Don't assume too much upfront |
| Include known-good as baseline | Ensures search includes working configs |
| Use constraints, not tight bounds | Let the algorithm find limits |
| Prefer continuous over categorical | Smoother optimization landscape |

---

### Search Space and Algorithm Choice

The search space structure influences which algorithm works best:

| Space Characteristic | Suitable Algorithms |
|---------------------|---------------------|
| Low-dimensional, continuous | TPE, QMC |
| High-dimensional | QMC, Random (first pass) |
| Many categoricals | TPE |
| Multi-objective | NSGA-II, NSGA-III |

---

### Summary

| Aspect | What it means |
|--------|---------------|
| **Definition** | Set of all valid parameter configurations |
| **Types** | Continuous, integer, categorical |
| **Constraints** | Relationships between parameters |
| **Scale** | Linear vs logarithmic sampling |
| **Impact** | Determines algorithm choice and convergence speed |

---

### See Also

- [Objective](concept_objective.md) — What we're optimizing toward
- [Breeder](concept_breeder.md) — Explores the search space
