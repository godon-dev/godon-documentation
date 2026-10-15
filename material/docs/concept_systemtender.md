---
description: "godon Systemtender concept — the agent that tends a live system: walking its dials, holding for peers, serving wishes, under guardrails with rollback."
---

## Systemtender

A **systemtender** is the pluggable agent in godon — the active component that propels effectuation and reconnaissance forward. Systemtenders walk live systems' parameters, driving the cycle of applying configurations and observing results.

They also serve [wishes](concept_wish.md): when a hold is planned against the measured map, a systemtender applies the planned setting and keeps it — and when the wish closes, it releases the setting back to neutral. A commissioned walk runs on the same machinery: the tender walks the declared region, spends its budget, and banks the return. Serving rides the same machinery as the walk: same guardrails, same effectuation channels, no special permissions.

### Core Responsibilities

A systemtender coordinates three concerns:

```
                    ┌──────────────────────────────┐
                    │   Systemtender               │
                    │     (agent)                  │
                    └──────────────────────────────┘
                                                   │
          ┌───────────────┼────────────────────────┐
          │               │                        │
          ▼               ▼               ▼
    ┌──────────┐    ┌──────────┐    ┌──────────────┐
    │   Walk   │    │Effectuate│    │Reconnoiter   │
    │ (Engine) │    │ (Apply)  │    │ (Observe)    │
    └──────────┘    └──────────┘    └──────────────┘
```

| Concern | Role | Examples |
|---------|------|----------|
| **Walk** | Which parameter and level to probe next | Coverage walk, notebook policy |
| **Effectuation** | How to apply configurations | SSH, HTTP, Kubernetes API |
| **Reconnaissance** | How to observe results | Prometheus, custom metrics |

The systemtender plugs these together into a coherent trial loop.

---

### The Systemtender Cycle

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│    ┌──────────┐    ┌──────────────┐    ┌──────────────┐      │
│    │  Decide  │───▶│  Effectuate  │───▶│  Reconnoiter │      │
│    │          │    │              │    │              │      │
│    │   Walk   │    │  Effectuator │    │   Observer   │      │
│    └──────────┘    └──────────────┘    └──────────────┘      │
│          ▲                                   │               │
│          │                                   ▼               │
│          │           ┌──────────────────────────────┐        │
│          └───────────│        Trial Record          │        │
│                      │     (told to the ledger)     │        │
│                      └──────────────────────────────┘        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

Each iteration:

1. **Decide** — the coordinator picks the next trial: a walk level, a hold, a park
2. **Effectuate** — Apply configuration to target system
3. **Reconnoiter** — Observe system behavior, collect metrics
4. **Record** — Tell the trial to the ledger; the measured ground grows

---

### Systemtender as Coordination Hub

The systemtender doesn't implement the walk, effectuation, or reconnaissance directly — it **coordinates** pluggable components:

#### The Walk

Which parameter to probe, and at which level, is the engine's own decision — the adaptive walk. The notebook policy reads the causal service's banked curves and probes the highest-priced unresolved gap; a dial retires only when its curve is converged and every remaining gap is priced below the local measurement bar — and drift un-retires it. A geometric farthest-point ladder exists as the explicit offline fallback.

#### Pluggable Effectuation

How configurations reach the target system:

| Mechanism | Target |
|-----------|--------|
| SSH | Remote servers, VMs |
| HTTP/REST | APIs, services |
| Kubernetes API | Pods, deployments, configmaps |

The effectuator is a plugin — the same walk can tend different targets.
#### Pluggable Reconnaissance

How the systemtender observes results:

| Source | What it provides |
|--------|------------------|
| Prometheus | Time-series metrics |
| Custom scripts | Arbitrary observations |
| HTTP endpoints | Health checks, stats |

The observer is a plugin — measure what matters for your objective.

---

### Systemtender Lifecycle
Systemtenders are managed through the Control API:
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
| `active` | Running trial loop |
| `stopped` | Gracefully paused; the current trial finishes |
| `error` | Unrecoverable failure occurred |
| `deleted` | Removed from system |

**Graceful shutdown:** When stopped, the current trial completes before the tender stops. This prevents half-applied configurations.

---

### The Instruments

The worker's state carries named facts, not more status words: one
status word, and beside it the field-sets — `strategies_run` (which
strategies have run), `wish` (the held wish's summary), `seek_spend`
(the commissioned walk's bill against its budget). They are stamped
into the tender's own state row on every state refresh and ride every
read of `GET /systemtenders/{id}` as `instruments` — present once the
worker has stamped, `null` before. Windmill's job state keeps its own
copy for its own face; the API field is the door's.

---

### Where Systemtenders Fit
In the godon architecture, systemtenders sit between the control plane and the target systems:
```
┌─────────────────────────────────────────────────────────────┐
│                      Control Plane                          │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│  │ Controller  │    │     API     │    │  Scheduler  │      │
│  └─────────────┘    └─────────────┘    └─────────────┘      │
└─────────────────────────────────────────────────────────────┘
                                                              │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                        Systemtender                         │
│                                                             │
│     ┌──────────┐          ┌──────────────┐                  │
│     │   Walk   │◀────────▶│ Coordinator  │                  │
│     │  (engine)│          │ (room/lease) │                  │
│     └──────────┘          └──────────────┘                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
          │                                                   │
          ▼                               ▼
┌─────────────────┐               ┌───────────────────────────┐
│   Effectuation  │               │  Reconnaissance           │
│    (apply)      │               │    (observe)              │
└─────────────────┘               └───────────────────────────┘
          │                                                   │
          └───────────────┬───────────────────────────────────┘
                          ▼
                   ┌──────────────────────────────────────────┐
                   │    Target                                │
                   │    System                                │
                   └──────────────────────────────────────────┘
```

---

### Guardrails
Guardrails are **safety limits** that protect production systems while trials run. Unlike objectives (which are measured), guardrails are binary constraints that must not be exceeded.
```
┌─────────────────────────────────────────────────────────────┐
│                      Trial Execution                        │
│                                                             │
│   Settings ──▶ Effectuate ──▶ Reconnoiter ──▶ Guardrails?   │
│                                                │            │
│                          ┌─────────────────────┴────┐       │
│                          ▼                          ▼       │
│                     Violated                        OK      │
│                          │                           │      │
│                          ▼                           ▼      │
│                   Mark FAILED                Accept trial   │
│                   Trigger rollback          Record values   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```
**How guardrails work:**

| Stage | Action |
|-------|--------|
| **Configure** | Define hard limits for critical metrics (e.g., `cpu_usage < 90%`) |
| **Check** | After each reconnaissance, compare collected metrics against limits |
| **Violated** | Mark trial as failed, increment failure counter, potentially trigger rollback |
| **OK** | Accept trial results, continue trials |
**Guardrails vs Objectives:**

| Aspect | Objectives | Guardrails |
|--------|------------|------------|
| **Purpose** | What is measured | What to avoid |
| **Example** | Minimize latency | CPU must not exceed 90% |
| **Behavior** | Recorded per trial, banked as measured ground | Binary pass/fail |
| **Failure** | — | Trial rejected |

---

### Rollback
When consecutive guardrail violations occur, systemtenders can automatically **rollback** to a known-good configuration.
```
┌──────────────────────────────────────────────────────────────┐
│                      Rollback Flow                           │
│                                                              │
│  Trial 1: OK ──▶ Trial 2: VIOLATED ──▶ Trial 3: VIOLATED     │
│      │                │                       │              │
│      ▼                ▼                       ▼              │
│  last_good         failures=1             failures=2         │
│  = params_1                   │                              │
│                               ▼                              │
│                    ┌───────────────────────┐                 │
│                    │     Threshold hit?    │                 │
│                    │      (e.g., 3)        │                 │
│                    └───────────┬───────────┘                 │
│                                │                             │
│                 ┌──────────────┴──────────────┐              │
│                 ▼                             ▼              │
│                Yes                            No             │
│                 │                              │             │
│                 ▼                              ▼             │
│      Apply last_good params          Continue trials         │
│      Reset failure counter                    │              │
│                 │                              │             │
│                 └──────────────┬───────────────┘             │
│                                ▼                             │
│             after-policy: continue / pause / stop            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
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

### Summary
| Aspect | What it means |
|--------|---------------|
| **Agent role** | Systemtender drives the walk, effectuation, and reconnaissance forward |
| **Pluggable** | Swap effectors and observers independently; the walk is the engine's own |
| **Guardrails** | Safety limits that reject trials exceeding thresholds |
| **Rollback** | Automatic restoration to known-good state after failures |
| **Managed** | Lifecycle controlled via API with graceful shutdown |

---

### See Also
- [Architecture](architecture.md) — Full system overview
- [API Reference](api.md) — Systemtender management endpoints
- [Comparison](comparison.md) — How systemtenders differ from optimization libraries
