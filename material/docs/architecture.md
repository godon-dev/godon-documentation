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

godon is a distributed system for live system optimization. It coordinates metaheuristic search algorithms with real-world effectuation and observation, running continuously against production systems.

---

### High-Level Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           Control Plane                                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                  │
│  │  Godon API  │───▶│  Windmill   │───▶│   Workers   │                  │
│  │  (extern)   │    │ (scheduler) │    │ (execute)   │                  │
│  └─────────────┘    └─────────────┘    └─────────────┘                  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           Storage Layer                                  │
│  ┌──────────────────┐              ┌──────────────────┐                 │
│  │   Metadata DB    │              │    Archive DB    │                 │
│  │   (PostgreSQL)   │              │   (YugabyteDB)   │                 │
│  │                  │              │                  │                 │
│  │  Component state │              │  Trial history   │                 │
│  │  Job tracking    │              │  Cooperation     │                 │
│  └──────────────────┘              └──────────────────┘                 │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         Execution Layer                                  │
│                                                                          │
│    ┌──────────┐         ┌──────────────┐         ┌──────────────┐       │
│    │ Breeder  │────────▶│  Effectuator │────────▶│    Target    │       │
│    │ (driver) │         │   (apply)    │         │   System     │       │
│    └──────────┘         └──────────────┘         └──────────────┘       │
│         │                                                │               │
│         │                                                ▼               │
│         │         ┌──────────────┐         ┌──────────────┐             │
│         └─────────│Reconnaissance│◀────────│   Metrics    │             │
│                   │  (observe)   │         │   Sources    │             │
│                   └──────────────┘         └──────────────┘             │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### Components

#### Godon API

The external interface for managing optimization runs.

| Responsibility | Description |
|----------------|-------------|
| Breeder lifecycle | Create, start, stop, delete breeders |
| Status queries | Check breeder and trial status |
| Configuration | Submit optimization configs |
| Results | Retrieve best configurations |

The API is stateless — it delegates to Windmill for orchestration.

#### Windmill

Workflow orchestration engine that schedules and executes godon jobs.

| Responsibility | Description |
|----------------|-------------|
| Job scheduling | Queue and dispatch work to workers |
| Worker management | Maintain worker pools by group |
| Retry handling | Recover from transient failures |
| Dependency resolution | Coordinate multi-step workflows |

Windmill provides the execution backbone without godon needing to implement scheduling logic.

#### Worker Groups

Workers are organized by job type:

| Group | Replicas | Timeout | Purpose |
|-------|----------|---------|---------|
| **controller** | 3 | 2 minutes | Fast operations: preflight, breeder create, status checks |
| **breeder** | 5 | None | Long-running optimization loops |
| **default** | 2 | Default | General operations, dependency resolution |

**Why separate groups:**
- Controller jobs are fast but frequent — need quick response
- Breeder jobs run continuously — no timeout, crash recovery via Optuna DB
- Default handles everything else without blocking specialized groups

#### Metadata DB (PostgreSQL)

Stores godon's operational state.

| Data | Purpose |
|------|---------|
| Breeder definitions | Configurations submitted via API |
| Job state | Windmill job tracking |
| Component metadata | Internal godon state |

PostgreSQL is sufficient here — moderate write volume, strong consistency needs.

#### Archive DB (YugabyteDB)

Stores trial history for optimization and cooperation.

| Data | Purpose |
|------|---------|
| Trial records | Parameters, metrics, fitness |
| Pareto fronts | Best configurations found |
| Cooperation data | Shared trials between breeders |

**Why YugabyteDB:**
- **Horizontal scalability** — Many concurrent breeders writing trials
- **PostgreSQL compatibility** — Uses YSQL, same queries as Optuna expects
- **Distribution** — Cooperative breeders need shared storage

#### Metrics Exporter

Exposes godon metrics for observability.

| Metric Type | Examples |
|-------------|----------|
| Trials | Total, successful, failed |
| Duration | Effectuation time, reconnaissance time |
| Breeder | Active count, worker utilization |

Pushes to Prometheus Push Gateway for aggregation.

---

### Data Flow

#### Creating a Breeder

```
User                API              Windmill          Controller Worker
  │                  │                   │                    │
  │ POST /breeders   │                   │                    │
  │─────────────────▶│                   │                    │
  │                  │ Create job        │                    │
  │                  │──────────────────▶│                    │
  │                  │                   │ Dispatch           │
  │                  │                   │───────────────────▶│
  │                  │                   │                    │ Init breeder
  │                  │                   │                    │ Write to Metadata DB
  │                  │                   │                    │ Spawn breeder workers
  │                  │                   │◀───────────────────│
  │                  │◀──────────────────│                    │
  │◀─────────────────│                   │                    │
  │ 201 Created      │                   │                    │
```

#### Optimization Loop

```
Breeder Worker      Effectuator        Target System      Reconnaissance
      │                 │                   │                    │
      │ Sample params   │                   │                    │
      │ from algorithm  │                   │                    │
      │                 │                   │                    │
      │ Apply config    │                   │                    │
      │────────────────▶│                   │                    │
      │                 │ SSH/HTTP/K8s      │                    │
      │                 │──────────────────▶│                    │
      │                 │                   │ Config applied     │
      │                 │◀──────────────────│                    │
      │◀────────────────│                   │                    │
      │                 │                   │                    │
      │ Wait for steady state               │                    │
      │                 │                   │                    │
      │ Collect metrics │                   │                    │
      │─────────────────────────────────────│───────────────────▶│
      │                 │                   │     Prometheus/HTTP│
      │◀────────────────────────────────────│────────────────────│
      │ Metrics         │                   │                    │
      │                 │                   │                    │
      │ Check guardrails│                   │                    │
      │ Compute fitness │                   │                    │
      │ Update algorithm│                   │                    │
      │ Share to Archive DB                 │                    │
      │                 │                   │                    │
      │ ◀── Loop ──▶    │                   │                    │
```

---

### Technology Choices

| Technology | Role | Why |
|------------|------|-----|
| **Windmill** | Workflow orchestration | Mature scheduler, Python-native, handles retry/dependency logic |
| **PostgreSQL** | Metadata storage | Reliable, well-understood, sufficient for component state |
| **YugabyteDB** | Trial archive | PostgreSQL-compatible, horizontally scalable, enables cooperation |
| **Kubernetes** | Deployment platform | Container orchestration, Helm for config, standard in cloud-native |
| **Prometheus** | Metrics | Industry standard, Push Gateway for batch job metrics |

**Design principles:**

- **Buy, don't build** — Windmill handles scheduling so godon doesn't have to
- **Separate concerns** — Metadata (operational) vs Archive (optimization) have different scaling needs
- **PostgreSQL ecosystem** — Both databases speak PostgreSQL, reducing cognitive load
- **Kubernetes-native** — Helm charts, Pod Disruption Budgets, standard deployment patterns

---

### Deployment

godon is deployed via Helm chart to Kubernetes.

```
┌─────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                        │
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │  godon-api  │  │  windmill   │  │  workers    │          │
│  │  (pod)      │  │  (pods)     │  │  (pods)     │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ metadata-db │  │ archive-db  │  │ pushgateway │          │
│  │ (postgres)  │  │ (yugabyte)  │  │ (prometheus)│          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
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
| Worker dies | In-flight trial lost | Optuna DB enables resume, algorithm continues |
| Metadata DB down | No new breeders | Existing breeders continue (state already dispatched) |
| Archive DB down | No cooperation, no persistence | Breeders continue locally, no cross-learning |
| Target system unreachable | Trial fails | Marked failed, algorithm learns to avoid |

**Crash safety:**
- Breeder workers have no timeout — they run until completion or crash
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

**Cooperation scaling:**
- Multiple breeders share Archive DB
- Each learns from others' trials
- Diminishing returns after ~10 cooperating breeders (search space coverage)

---

### See Also

- [Core Concepts](concept_breeder.md) — Breeder, Effectuator, Reconnaissance, Guardrails
- [Configuration Guide](config_guide.md) — How to configure optimization runs
- [Setup](setup.md) — Installation instructions
