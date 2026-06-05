---
description: "godon architecture — distributed system design with control plane, storage layer, execution layer. Optimization loop, technology choices, failure modes, and scaling."
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

### Optimization Loop

The core cycle that each breeder worker executes:

```
┌──────────────────────────────────────────────────────────────────────────┐
│                        Breeder Worker Loop                                   │
│                                                                              │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌──────────┐  │
│  │   Sample   │───▶│  Effectuate │───▶│Reconnoiter │───▶│  Update  │  │
│  │  (algorithm)│    │ (apply)     │    │ (observe)  │    │ (fitness) │  │
│  └─────────────┘    └─────────────┘    └─────────────┘    └──────────┘  │
│         │                  │                  │                  │           │
│         │                  ▼                  │                  │           │
│         │         ┌──────────────────────────────────┐        │           │
│         │         │         Target System              │        │           │
│         │         │  ┌────────┐  ┌────────────┐     │        │           │
│         └─────────▶│  SSH   │  │ Kubernetes │─────▶        │           │
│                   │  HTTP   │  │   API      │     │        │           │
│                   └────────┘  └────────────┘     │        │           │
│                                            │                  │           │
│                                            ▼                  │           │
│                              ┌──────────────────────────┐        │           │
│                              │   Prometheus / Metrics    │        │           │
│                              └────────────┬─────────────┘        │           │
│                                           │                                │           │
│                                           ▼                                │           │
│                              ┌──────────────────────────┐        │           │
│                              │  Guardrails? Fitness?    │        │           │
│                              └────────────┬─────────────┘        │           │
│                                           │                                │           │
│                              ┌────────────┴─────────────┐        │           │
│                              ▼                           ▼        │           │
│                         ┌──────────┐              ┌──────────┐  │           │
│                         │  Share   │              │  Next    │  │           │
│                         │  (opt)   │              │  Sample  │  │           │
│                         └──────────┘              └──────────┘  │           │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────┘
```

| Phase | Action | Duration |
|-------|--------|----------|
| **Sample** | Algorithm suggests next parameters | Milliseconds |
| **Effectuate** | Apply config to target system | Seconds to minutes |
| **Reconnoiter** | Wait for steady state, collect metrics | Seconds |
| **Update** | Check guardrails, compute fitness, update algorithm | Milliseconds |
| **Communicate** (optional) | Publish trial to Archive DB for cooperation | Milliseconds |

**Key properties:**
- Effectuation is idempotent — safe to retry
- Reconnaissance waits for steady state before collecting
- Guardrail violations short-circuit the loop, mark trial failed
- Archive DB write is async, doesn't block next sample

---

### Technology Choices

| Technology | Role | Why |
|------------|------|-----|
| **Windmill** | Workflow orchestration | Most mature and best performing open source workflow engine, abstracts Kubernetes complexity |
| **PostgreSQL** | Metadata storage | Reliable, well-understood, sufficient for component state |
| **YugabyteDB** | Trial archive | PostgreSQL-compatible, horizontally scalable, enables cooperation |
| **Kubernetes** | Deployment platform | Container orchestration, Helm for config, standard in cloud-native |
| **Prometheus** | Metrics | Industry standard, Push Gateway for batch job metrics |

**Design principles:**

- **Open source stack** — Built entirely on open source components, no vendor lock-in
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
