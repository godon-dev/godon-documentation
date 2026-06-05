---
description: "godon Breeder concept — pluggable optimization driver coordinating algorithms, effectuation, and reconnaissance. Worker collaboration, guardrails, rollback, and cooperation."
---

## Breeder

A **breeder** is the pluggable optimization driver in godon — the active component that propels algorithms, effectuation, and reconnaissance forward. Breeders run meta-heuristic searches against live systems, driving the cycle of applying configurations and observing results.

### Core Responsibilities

A breeder coordinates three concerns:

```
                    ┌─────────────┐
                    │   Breeder   │
                    │  (Driver)   │
                    └─────────────┘
                          │
          ┌───────────────┼───────────────┐
          │               │               │
          ▼               ▼               ▼
    ┌──────────┐    ┌──────────┐    ┌──────────────┐
    │ Algorithm│    │Effectuate│    │Reconnoiter   │
    │ (Search) │    │ (Apply)  │    │ (Observe)    │
    └──────────┘    └──────────┘    └──────────────┘
```

| Concern | Role | Examples |
|---------|------|----------|
| **Algorithm** | What search strategy to use | TPE, NSGA-II, QMC, Random |
| **Effectuation** | How to apply configurations | SSH, HTTP, Kubernetes API |
| **Reconnaissance** | How to observe results | Prometheus, custom metrics |

The breeder plugs these together into a coherent optimization loop.

---

### The Breeder Cycle

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│    ┌──────────┐    ┌──────────────┐    ┌──────────────┐     │
│    │  Sample  │───▶│  Effectuate  │───▶│  Reconnoiter │     │
│    │          │    │              │    │              │     │
│    │ Algorithm│    │  Effectuator │    │   Observer   │     │
│    └──────────┘    └──────────────┘    └──────────────┘     │
│          ▲                                   │               │
│          │                                   ▼               │
│          │            ┌──────────────────────────────┐      │
│          └────────────│       Fitness Update         │      │
│                       │                              │──────┼──▶ Share Trial
│                       └──────────────────────────────┘      │   (optional)
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

Each iteration:

1. **Sample** — Algorithm suggests next configuration to try
2. **Effectuate** — Apply configuration to target system
3. **Reconnoiter** — Observe system behavior, collect metrics
4. **Fitness Update** — Score the trial, inform algorithm for next sample

---

### Breeder as Coordination Hub

The breeder doesn't implement algorithms, effectuation, or reconnaissance directly — it **coordinates** pluggable components:

#### Pluggable Algorithms

Breeders can use different search strategies:

| Sampler | Use Case |
|---------|----------|
| TPE | Single-objective, sample-efficient search |
| NSGA-II | Multi-objective optimization (2-3 objectives) |
| NSGA-III | Multi-objective optimization (4+ objectives) |
| QMC | High-dimensional spaces |
| Random | Baseline, initial exploration |

The algorithm is a plugin — swap it without changing effectuation or reconnaissance.

#### Pluggable Effectuation

How configurations reach the target system:

| Mechanism | Target |
|-----------|--------|
| SSH | Remote servers, VMs |
| HTTP/REST | APIs, services |
| Kubernetes API | Pods, deployments, configmaps |

The effectuator is a plugin — same algorithm can tune different targets.
#### Pluggable Reconnaissance

How the breeder observes results:

| Source | What it provides |
|--------|------------------|
| Prometheus | Time-series metrics |
| Custom scripts | Arbitrary observations |
| HTTP endpoints | Health checks, stats |

The observer is a plugin — measure what matters for your objective.

---

### Worker Collaboration

Breeders can run with multiple parallel workers:
```
                    ┌─────────────┐
                    │  Controller │
                    │   (API)     │
                    └─────────────┘
                          │
          ┌───────────────┼───────────────┼───────────────┐
          │               │               │
          ▼               ▼               ▼
    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │ Worker 1 │    │ Worker 2 │    │ Worker 3 │
    └──────────┘    └──────────┘    └──────────┘
          │               │               │
          └───────────────┴───────────────┘
                          │
                    Shared Trial History
```

**How collaboration works:**
- Workers share a common trial history
- Each worker samples from the algorithm, avoiding redundant evaluations
- The algorithm learns from all workers' trials
- Controller coordinates worker lifecycle (start, stop, delete)

**Benefits:**
- Faster convergence through parallel evaluation
- Better coverage of the search space
- Resilience: other workers continue if one fails

---

### Breeder Lifecycle
Breeders are managed through the Control API:
```
created ──▶ active ──┐
    │           │    │
    │           ▼    ▼
    │       stopped ──▶ deleted
    │           │
    └───────────┴──▶ error
```

| State | Meaning |
|-------|---------|
| `active` | Running optimization loop |
| `stopped` | Gracefully paused, workers finished current trials |
| `error` | Unrecoverable failure occurred |
| `deleted` | Removed from system |

**Graceful shutdown:** When stopped, workers complete their current trials before terminating. This prevents half-applied configurations.

---

### Where Breeders Fit
In the godon architecture, breeders sit between the control plane and the target systems:
```
┌─────────────────────────────────────────────────────────────┐
│                      Control Plane                          │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │ Controller  │    │   API       │    │  Scheduler  │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                       Breeder                               │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │ Algorithm│◀──▶│ Coordinator  │◀──▶│   Workers    │      │
│  └──────────┘    └──────────────┘    └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                          │
          ┌───────────────┴───────────────┐
          ▼                               ▼
┌─────────────────┐               ┌─────────────────┐
│   Effectuation  │               │  Reconnaissance │
│   (Apply)       │               │  (Observe)      │
└─────────────────┘               └─────────────────┘
          │                               │
          └───────────────┬───────────────┘
                          ▼
                   ┌─────────────┐
                   │   Target    │
                   │   System    │
                   └─────────────┘
```

---

### Guardrails
Guardrails are **safety limits** that protect production systems during optimization. Unlike objectives (which we optimize), guardrails are binary constraints that must not be exceeded.
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
**How guardrails work:**

| Stage | Action |
|-------|--------|
| **Configure** | Define hard limits for critical metrics (e.g., `cpu_usage < 90%`) |
| **Check** | After each reconnaissance, compare collected metrics against limits |
| **Violated** | Mark trial as failed, increment failure counter, potentially trigger rollback |
| **OK** | Accept trial results, continue optimization |
**Guardrails vs Objectives:**

| Aspect | Objectives | Guardrails |
|--------|------------|------------|
| **Purpose** | What to optimize | What to avoid |
| **Example** | Minimize latency | CPU must not exceed 90% |
| **Behavior** | Guides search direction | Binary pass/fail |
| **Failure** | Suboptimal result | Trial rejected |

---

### Rollback
When consecutive guardrail violations occur, breeders can automatically **rollback** to a known-good configuration.
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
**Rollback strategies:**

| Strategy | What it restores | Use case |
|----------|------------------|----------|
| `previous` | Last successful trial's params | Conservative, safe |
| `best` | Best Pareto-optimal trial | Aggressive, assumes best is stable |
| `baseline` | Original/untuned configuration | Fallback to known defaults |
**Configuration:**
```yaml
rollback:
  enabled: true
  strategy: standard

rollback_strategies:
  standard:
    consecutive_failures: 3      # Trigger after N violations
    target_state: previous       # previous, best, or baseline
    on_failure: stop             # stop, continue, or skip_target
    after:
      action: pause              # continue, pause, or stop
      duration: 300              # Pause duration in seconds
```
---

### Worker Cooperation
Multiple workers can collaborate by **sharing successful trials** across the optimization process.
```
┌─────────────────────────────────────────────────────────────────────┐
│                      Shared Trial History                            │
│                    (YugabyteDB / PostgreSQL)                         │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
           ┌───────────────────┼───────────────────┐
           │                   │                   │
           ▼                   ▼                   ▼
     ┌──────────┐        ┌──────────┐        ┌──────────┐
     │ Breeder A│        │ Breeder B│        │ Breeder C│
     │ Worker 1 │        │ Worker 1 │        │ Worker 1 │
     └──────────┘        └──────────┘        └──────────┘
           │                   │                   │
           │  Shares trial #5  │                   │
           ├──────────────────▶│                   │
           │                   │  Shares trial #12 │
           │                   ├──────────────────▶│
           │                   │                   │
           │         All breeders learn from shared trials          │
           └───────────────────────────────────────────────────────────────┘
```
**Sharing strategies:**

| Strategy | What gets shared | Benefit |
|----------|------------------|---------|
| `probabilistic` | Random trials with probability P | Simple, fast convergence |
| `best` | Top percentile performers | Focus on good solutions |
| `worst` | Bottom percentile performers | Learn what to avoid |
| `extremes` | Both best and worst | Diverse learning signal |
**How it works:**
1. Worker completes a successful trial
2. Strategy determines if trial should be shared
3. If yes, trial is added to cooperating studies via shared database
4. Other workers' algorithms learn from shared trials
5. Faster convergence through collective intelligence
---

### Algorithm Diversity
When running multiple parallel workers, breeders can assign **different algorithms** to each worker for better search space coverage.
```
┌─────────────────────────────────────────────────────────────┐
│                    Parallel Workers                          │
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │  Worker 1   │  │  Worker 2   │  │  Worker 3   │         │
│  │   TPE       │  │  NSGA-II    │  │  Random     │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│        │                │                │                   │
│        └────────────────┴────────────────┘                   │
│                         │                                    │
│                         ▼                                    │
│              ┌─────────────────────┐                        │
│              │  Shared Trial Store │                        │
│              │  (cross-pollination)│                        │
│              └─────────────────────┘                        │
│                                                              │
│  Each sampler explores differently, sharing discoveries      │
└─────────────────────────────────────────────────────────────┘
```
**Assigned samplers:**

| Workers | Samplers assigned |
|---------|-------------------|
| 1 | TPE (default) |
| 2 | TPE, NSGA-II |
| 3 | TPE, NSGA-II, Random |
| 4 | TPE, NSGA-II, Random, NSGA-III |
| 5+ | TPE, NSGA-II, Random, NSGA-III, QMC |
**Benefits:**
- **Diverse exploration**: Different algorithms search differently
- **Robustness**: If one algorithm gets stuck, others continue
- **Cross-pollination**: Workers share findings via cooperation
- **Hyperheuristic potential**: Future automatic algorithm selection
---

### Summary
| Aspect | What it means |
|--------|---------------|
| **Driver role** | Breeder drives algorithm, effectuation, and reconnaissance forward |
| **Pluggable** | Swap algorithms, effectors, and observers independently |
| **Guardrails** | Safety limits that reject trials exceeding thresholds |
| **Rollback** | Automatic restoration to known-good state after failures |
| **Cooperation** | Workers share successful trials for faster convergence |
| **Diversity** | Parallel workers use different algorithms for better coverage |
| **Managed** | Lifecycle controlled via API with graceful shutdown |

---

### See Also
- [Architecture](architecture.md) — Full system overview
- [API Reference](api.md) — Breeder management endpoints
- [Comparison](comparison.md) — How breeders differ from optimization libraries
