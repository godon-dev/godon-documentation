---
description: "godon architecture — distributed system design with control plane, storage layer, execution layer. Trial loop, the walk, the steering loop, technology choices, failure modes, and scaling."
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

## Architecture

godon is a distributed system for tending live systems: it measures the coupling structure of the system it runs on, and it steers that system toward operating points a holder chooses. It coordinates autonomous agents (systemtenders) — walking the system's dials, and serving declared steerwishes — with real-world effectuation and observation, and a causal service that computes coupling detection and response-curve characterization from the agents' shared trial data.

---

### High-Level Overview

```
┌────────────────────────────────────────────────────────────────────────────┐
│                               Control Plane                                │
│                                                                            │
│    ┌─────────────┐       ┌─────────────┐       ┌─────────────┐             │
│    │  Godon API  │─────▶ │  Windmill   │─────▶ │   Workers   │             │
│    │  (extern)   │       │ (scheduler) │       │  (execute)  │             │
│    └─────────────┘       └─────────────┘       └─────────────┘             │
└────────────────────────────────────────────────────────────────────────────┘
                                                                             │
                                      ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                               Storage Layer                                │
│                                                                            │
│      ┌──────────────────┐                  ┌──────────────────┐            │
│      │   Metadata DB    │                  │    Archive DB    │            │
│      │   (PostgreSQL)   │                  │   (YugabyteDB)   │            │
│      │                  │                  │                  │            │
│      │  Component state │                  │  Trial history   │            │
│      │  Job tracking    │                  │  Leases, wishes  │            │
│      └──────────────────┘                  └──────────────────┘            │
└────────────────────────────────────────────────────────────────────────────┘
                                                                             │
                                      ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                              Execution Layer                               │
│                                                                            │
│    ┌─────────────┐       ┌──────────────┐        ┌──────────────┐          │
│    │ Systemtender│──────▶│  Effectuator │───────▶│    Target    │          │
│    │   (tender)  │       │   (apply)    │        │    System    │          │
│    └─────────────┘       └──────────────┘        └──────────────┘          │
│          │                                           │                     │
│          │                                           ▼                     │
│          │            ┌──────────────┐       ┌──────────────┐              │
│          └───────────▶│Reconnaissance│◀──────│   Metrics    │              │
│                       │  (observe)   │       │   Sources    │              │
│                       └──────────────┘       └──────────────┘              │
└────────────────────────────────────────────────────────────────────────────┘
```

---

### Components

#### Godon API

The external interface for managing systemtenders.

| Responsibility | Description |
|----------------|-------------|
| Systemtender lifecycle | Create, start, stop, delete systemtenders |
| Status queries | Check systemtender and trial status |
| Configuration | Submit systemtender configs |
| Results | Retrieve best configurations |

The API is stateless — it delegates to Windmill for orchestration.

#### MCP Server

The agent-facing external interface beside the REST API ([MCP Interface](mcp.md)): management and steering tools proxy the godon API; the map-reading tools query the causal service directly, so LLM clients see the same measured structure the engine holds.

#### Windmill

Workflow orchestration engine that schedules and executes godon jobs.

| Responsibility | Description |
|----------------|-------------|
| Job scheduling | Queue and dispatch work to workers |
| Worker management | Maintain worker pools by group |
| Retry handling | Recover from transient failures |
| Dependency resolution | Coordinate multi-step workflows |

Windmill provides the execution backbone without godon needing to implement scheduling logic.

#### Controller

Lifecycle logic between the API and the workers, executed as Windmill scripts: validation of steerwish declarations and corrections at the door (the grammar is checked before any planning; a refusal names its binding constraint), systemtender coordination, and cleanup cascades.

#### Godon Causal

The measurement computation service. Systemtenders push parameters and observe objectives; causal owns everything computed FROM those trials:

| Responsibility | Description |
|----------------|-------------|
| Coupling detection | CFAR on push/pause block contrasts — per (sender, receiver, channel), on demand |
| Response curves | Per (sender, receiver, parameter, channel): measured level→shift shape with uncertainty bars |
| Priced stopping | Per-curve gap analysis — a curve retires when remaining ignorance is cheaper than one more probe |
| Persistence | Curves survive restarts (write-through + replay) and follow systemtender lifecycle (purge cascade) |
| Graph artifact | The measured coupling structure, exportable as a versioned artifact |

Rust service, port 8091. Key endpoints: `/detect/{sender}/{receiver}`, `/characterize` (probe results in, shift/delta/convergence out), `/curves`, `/predict` and `/predict/multihop`, `/graph` and `/artifact` (the measured coupling map, exportable), `/impact/{systemtender_id}`, `/causes/{systemtender_id}`.

#### Godon Observer

Observability: Prometheus metrics, trial history, the dashboard, and detection proxies to causal (port 8089).

#### Worker Groups

Workers are organized by job type:

| Group | Timeout | Purpose |
|-------|---------|---------|
| **controller-fast** | 60s | Controller verbs and reads: preflight, status, stop, the wish door |
| **controller-slow** | 600s | Controller executors: create table builds, delete quiesce |
| **systemtender** | None by design — crash recovery via the Optuna DB | Long-running trial loops |
| **default** | Default | General operations, dependency resolution |

Replica counts are deployment values, not architecture — they live in the chart's `values.yaml`.

**Why separate groups:**
- Controller jobs are fast but frequent — need quick response
- Systemtender jobs run continuously — no timeout, crash recovery via Optuna DB
- Default handles everything else without blocking specialized groups

#### Metadata DB (PostgreSQL)

Stores godon's operational state.

| Data | Purpose |
|------|---------|
| Systemtender definitions | Configurations submitted via API |
| Job state | Windmill job tracking |
| Component metadata | Internal godon state |

PostgreSQL is sufficient here — moderate write volume, strong consistency needs.

#### Archive DB (YugabyteDB)

Stores the systemtenders' shared trial history.

| Data | Purpose |
|------|---------|
| Trial records | Parameters, metrics, values |
| Pareto fronts | Best trials found |

**Why YugabyteDB:**
- **Horizontal scalability** — Many concurrent systemtenders writing trials
- **PostgreSQL compatibility** — Uses YSQL, same queries as Optuna expects
- **Distribution** — Cooperative systemtenders need shared storage

#### Metrics Exporter

Exposes godon metrics for observability.

| Metric Type | Examples |
|-------------|----------|
| Trials | Total, successful, failed |
| Duration | Effectuation time, reconnaissance time |
| Systemtender | Active count, worker utilization |

Pushes to Prometheus Push Gateway for aggregation.

---

### Trial Loop

The core cycle that each systemtender worker executes:

```
┌──────────────────────────────────────────────────────────────────────────┐
│                          Systemtender Trial Loop                         │
│                                                                          │
│   ┌──────────┐      ┌───────────┐      ┌───────────┐      ┌──────────┐   │
│   │  Decide  │─────▶│ Effectuate│─────▶│Reconnoiter│─────▶│   Tell   │   │
│   │  (walk)  │      │  (apply)  │      │ (observe) │      │ (ledger) │   │
│   └──────────┘      └───────────┘      └───────────┘      └──────────┘   │
│        │                                                        │        │
│        │           config applied, metrics read                 │        │
│        │                                                        │        │
│        └────────────────── next trial ◀─────────────────────────┘        │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

| Phase | Action | Duration |
|-------|--------|----------|
| **Decide** | Coordinator picks the next trial — a walk level, a hold, a park | Milliseconds |
| **Effectuate** | Apply config to target system | Seconds to minutes |
| **Reconnoiter** | Wait for steady state, collect metrics | Seconds |
| **Tell** | Check guardrails, record the trial's values in the ledger | Milliseconds |

**Key properties:**
- Effectuation is idempotent — safe to retry
- Reconnaissance waits for steady state before collecting
- Guardrail violations short-circuit the loop, mark trial failed
- Every trial is told to the ledger; curves and fronts read from it

### The Walk (characterization)

Systemtenders in the same interference group coordinate through DB-backed leases (turn-taking: one sender, the rest hold):

```
  Sender: coverage walk — pick (parameter, level), push within guardrails,
          pause, return to hold
  Receivers: hold still, write observations with lease phase tags
  Causal: per probe — median shift push vs pause, uncertainty bar (MAD),
          curve update, convergence + gap pricing
  Retirement: converged AND every gap priced below the local bar
```

The adaptive walk steers by the measured map: the notebook policy picks the next (parameter, level) — the highest-priced unresolved gap in the banked curves — and a dial retires only when converged with every remaining gap priced below its local measurement bar. Re-measurement within bars blends instead of accumulating noise.

### Steering Loop (the action half)

Basic steering is live — the loop that turns a declared steerwish into a held state on the live system:

```
  Holder:  declare a wish — claims (readings to keep in bands), optional
           terms (protected readings), limits, budget
  Door:    validate before any planning — a malformed or unkeepable wish
           is refused by name
  Plan:    the measured map compiles the claims into an input setting;
           inputs shared between claims are reconciled, terms bound the move
  Serve:   a systemtender applies the setting and holds it
  Judge:   every claim and term read against its band — one verdict, the
           conjunction; a miss is stamped with its evidence
  Drift:   the reading leaves band — the wish re-acts within budget, or
           the holder corrects the terms in place
  Close:   the wish ends — the serving systemtender releases its setting
           back to neutral
```

The declaration is a durable, withdrawable record — the safety device for everything that acts on it ([Steerwish](concept_steerwish.md)). Serving rides the same machinery as the walk: same guardrails, same effectuation channels, no special permissions.

---

### Technology Choices

| Technology | Role | Why |
|------------|------|-----|
| **Windmill** | Workflow orchestration | Most mature and best performing open source workflow engine, abstracts Kubernetes complexity |
| **PostgreSQL** | Metadata storage | Reliable, well-understood, sufficient for component state |
| **YugabyteDB** | Trial archive | PostgreSQL-compatible, horizontally scalable, shared by the whole group |
| **Kubernetes** | Deployment platform | Container orchestration, Helm for config, standard in cloud-native |
| **Prometheus** | Metrics | Industry standard, Push Gateway for batch job metrics |

**Design principles:**

- **Open source stack** — Built entirely on open source components, no vendor lock-in
- **Separate concerns** — Metadata (operational) vs Archive (trials) have different scaling needs
- **PostgreSQL ecosystem** — Both databases speak PostgreSQL, reducing cognitive load
- **Kubernetes-native** — Helm charts, Pod Disruption Budgets, standard deployment patterns

---

### Deployment

godon is deployed via Helm chart to Kubernetes.

```
┌──────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                        │
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│  │  godon-api  │  │  windmill   │  │  workers    │           │
│  │  (pod)      │  │  (pods)     │  │  (pods)     │           │
│  └─────────────┘  └─────────────┘  └─────────────┘           │
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│  │ metadata-db │  │ archive-db  │  │ pushgateway │           │
│  │ (postgres)  │  │ (yugabyte)  │  │ (prometheus)│           │
│  └─────────────┘  └─────────────┘  └─────────────┘           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Deployment characteristics:**

- **Stateless API** — Can scale horizontally, rolling updates without downtime
- **Stateful databases** — YugabyteDB handles its own replication
- **Worker pools** — Scale independently based on load
- **Helm-managed** — Single chart installs the full stack

---

### Failure Modes

| Failure | Impact | Recovery |
|---------|--------|----------|
| API pod dies | No new requests | Kubernetes restarts, stateless |
| Worker dies | In-flight trial lost | Optuna DB enables resume, the walk continues |
| Metadata DB down | No new systemtenders | Existing systemtenders continue (state already dispatched) |
| Archive DB down | Ledger and lease coordination unavailable | Worker exits; resumes from the ledger on recovery |
| Target system unreachable | Trial fails | Marked failed, counted toward rollback |

**Crash safety:**
- Systemtender workers end when the characterization is complete, when a stop is requested, or on crash
- Optuna stores trial state in Archive DB — restart resumes from last known state
- No half-applied configs — effectuation is idempotent

---

### Scaling

| Component | Scale by | Limit |
|-----------|----------|-------|
| API | Replicas | Stateless, scale freely |
| Workers | Group replicas | More workers = more parallel trials |
| Metadata DB | Vertical | Single PostgreSQL instance |
| Archive DB | Horizontal | YugabyteDB distributes across nodes |

**Worker scaling:** each systemtender runs a single worker. Multiple systemtenders share the Archive DB, where all trials are stored.

---

### See Also

- [Core Concepts](concept_systemtender.md) — Systemtender, Effectuator, Reconnaissance, Guardrails
- [Steerwish](concept_steerwish.md) — the steering loop's intent surface
- [Configuration Guide](config_guide.md) — How to configure systemtenders
- [Setup](setup.md) — Installation instructions
