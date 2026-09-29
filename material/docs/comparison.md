---
description: "godon comparison vs Optuna, Hyperopt, Nevergrad, Ray Tune, Ax, Akamas, StormForge, Datadog, KEDA — every tool below finds settings; none keeps one. Finding is a crowded field. Keeping is godon's category."
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

# Comparison & Competitors

## Positioning

Every tool below finds settings. None of them keeps one.

Optimizers (Optuna, Ray Tune, Ax, Akamas, StormForge) push scores: they search, report a best configuration, and stop. Autoscalers (KEDA, HPA/VPA) hold setpoints on assumed models. Observability (Datadog, Dynatrace) watches and alerts. Model-based RL learns a model of its environment but cannot refuse, keeps no record of why it acts, and re-learns from scratch when that environment drifts.

Godon keeps — and the steering is intent-driven: a holder — human or AI mind — declares a steerwish, and the engine serves that intent. A wish holds chosen values on chosen axes: one or several measured outcomes, each kept in its band, under terms that respect the rest of the system. It measures, plans against the measured map, holds while the world drifts, and refuses by name when a promise cannot be kept. Several steerwishes can live on one complex system — kept together where that is possible, collisions surfaced where it is not. Keeping is priced: the trials it costs are counted, not hidden.

Finding is well served by the tools below; several are excellent at it. Keeping — declared intent, held continuously on a measured map, honestly refused, visibly priced — none of them offers. That conjunction is godon's category.

Godon is a self-hosted engine that discerns hidden system structure through active perturbation, and tends what it finds toward better operating points.

```
┌─────────────────────────────────────────────────────────────────┐
│                         GODON                                    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              Discernment Layer                           │    │
│  │  Coupling detection  │  Topology discovery               │    │
│  │  Active perturbation │  Isolation certification          │    │
│  └─────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Tending Layer                         │    │
│  │  Live system steering  │  Guardrails  │  Rollback        │    │
│  └─────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Plumbing Layer                        │    │
│  │  Effectuation (SSH, HTTP, APIs)  │  Reconnaissance      │    │
│  │  Trial coordination              │  Worker management    │    │
│  └─────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              Algorithm Core                              │    │
│  │  (TPE, NSGA-II/III, QMC  →  custom samplers, ML)        │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

## Optimization Libraries

### Optuna

| Dimension | Godon | Optuna |
|-----------|-------|--------|
| **What it provides** | Complete engine (discern + tend) | Algorithm library |
| **Plumbing - Effectuation** | Built-in (SSH, HTTP) | None — you build it |
| **Plumbing - Reconnaissance** | Built-in (Prometheus, HTTP) | None — you build it |
| **Plumbing - Coordination** | Controller API, trial sharing | Study management only |
| **Coupling Detection** | Active perturbation, topology discovery | None |
| **Ops Safety - Guardrails** | Hard limits with automatic response | No |
| **Ops Safety - Rollback** | Previous/best/baseline restoration | No |
| **Deployment** | Kubernetes-native Helm chart | Python library |
| **Scope** | Production live systems | Any optimization problem |

### Hyperopt

| Dimension | Godon | Hyperopt |
|------------|-------|----------|
| **Plumbing - Effectuation** | Built-in (SSH, HTTP, APIs) | None |
| **Plumbing - Reconnaissance** | Built-in (Prometheus) | None |
| **Coupling Detection** | Active perturbation, topology discovery | None |
| **Ops Safety - Guardrails** | Yes | No |
| **Ops Safety - Rollback** | Yes | No |
| **Multi-objective** | Yes | No |
| **Algorithm** | TPE, NSGA-II/III, QMC, Random | TPE, Random, Atpe |
| **API** | REST + CLI | Python only |
| **Status** | Active | Maintenance mode |

### Nevergrad

| Dimension | Godon | Nevergrad |
|------------|-------|-----------|
| **Plumbing - Effectuation** | Built-in | None |
| **Plumbing - Reconnaissance** | Built-in | None |
| **Coupling Detection** | Active perturbation, topology discovery | None |
| **Ops Safety - Guardrails** | Yes | No |
| **Ops Safety - Rollback** | Yes | No |
| **Algorithms** | TPE, NSGA-II/III, QMC | Derivative-free (Evolution, Bandits) |
| **Multi-objective** | Yes | Yes |
| **Domain** | Live systems + generic | Generic functions |
| **Deployment** | Kubernetes | Python library |

## ML-Focused Frameworks

### Ray Tune

| Aspect | Godon | Ray Tune |
|--------|-------|----------|
| Primary Domain | Live systems, infrastructure | ML training |
| Live System Integration | Native | Manual |
| Coupling Detection | Yes — multi-agent interference topology | None |
| Effectuation Layer | Yes (SSH, HTTP) | No |
| Guardrails | Yes | No |
| Rollback | Yes | No |
| Deployment | Kubernetes-native | Ray cluster |
| Overhead | Lightweight | Heavy (Ray runtime) |

### Ax / BoTorch

| Aspect | Godon | Ax |
|--------|-------|-----|
| Algorithm | Multi-strategy (TPE, NSGA, QMC) | Bayesian optimization |
| Coupling Detection | Yes — multi-agent interference topology | None |
| Live System Integration | Native | Manual |
| Constraints | Guardrails + Rollback | Parameter constraints |
| Deployment | Self-hosted | Hosted service or self-hosted |
| ML Dependency | None | BoTorch (Gaussian processes) |

### Weights & Biases Sweeps

| Aspect | Godon | W&B Sweeps |
|--------|-------|------------|
| Type | Self-hosted engine | SaaS + library |
| Coupling Detection | Yes | None |
| Live System Integration | Native | Manual |
| Data Ownership | Full | Vendor-hosted |
| Cost | Free | Subscription tiers |

## Model-Based RL

The closest cousin: like godon, model-based reinforcement learning learns a model of its environment and plans against it. The fair representatives are PILCO-class (Gaussian-process world models) and PETS-class (ensemble neural world models); SAC as the model-free regime exhibit.

| Aspect | Godon | Model-Based RL (PILCO / PETS class) |
|--------|-------|--------------------------------------|
| World model | Measured curves with error bars, partial by design | Learned model, full-coverage assumption |
| Keeping | Holds a declared band, with terms | Optimizes a return; no keep semantics |
| Refusal | Named refusal when a promise cannot be kept | None — it keeps sampling |
| Drift | Demotes curves to priors, re-anchors on the map | Discards the model, re-learns from scratch |
| Records | Full event trail — every verdict re-derivable | Trajectories only |
| Home regime | Costly, drifting trials (production) | Dense, cheap interaction (simulation) |

Different home turf, honestly stated: dense cheap interaction is where that family shines, and its behavior under capped budget in a costly drifting regime is regime evidence, not a defeat. The planned referee grid (see [Open Research](open_research.md)) gives both families equal budget and equal observation access.

## Infrastructure Optimization Platforms

### Akamas

| Dimension | Godon | Akamas |
|------------|-------|--------|
| **License** | AGPL (open source) | Proprietary |
| **Deployment** | Self-hosted (Helm) | SaaS |
| **Coupling Detection** | Active perturbation, topology discovery | None |
| **Kubernetes-bound** | No | Yes |
| **Algorithm Transparency** | Full | Black-box |
| **Extensibility** | Custom systemtenders | Vendor-defined |
| **Cost** | Free | Subscription |
| **Vendor Lock-in** | None | High |

### StormForge

| Dimension | Godon | StormForge |
|------------|-------|------------|
| **License** | AGPL (open source) | Proprietary |
| **Coupling Detection** | Active perturbation, topology discovery | None |
| **Deployment** | Self-hosted | SaaS |
| **Scope** | Any network-accessible system | Kubernetes only |
| **Vendor Lock-in** | None | High |

### Turbonomic (IBM)

| Aspect | Godon | Turbonomic |
|--------|-------|------------|
| Approach | Discernment + tending | Resource management + placement |
| Coupling Detection | Active perturbation, topology discovery | None |
| License | Open source | Proprietary |
| Scope | Configuration + structure discovery | Full resource orchestration |
| Integration | Network-accessible systems | VMware, cloud providers |

## AIOps Platforms

### Datadog Watchdog

| Aspect | Godon | Datadog Watchdog |
|--------|-------|------------------|
| Approach | Active perturbation, causal discernment | Passive anomaly detection |
| Coupling Detection | Discovers hidden causal structure | No — sees symptoms, not causes |
| Action | Configuration changes, tending | Alerts, some recommendations |
| Epistemology | Experimentation (counterfactuals) | Observation (correlations) |
| Cost | Free | Part of Datadog subscription |

### Dynatrace Davis

| Aspect | Godon | Dynatrace Davis |
|--------|-------|-----------------|
| Approach | Active causal discernment | AI-powered root cause (passive) |
| Coupling Detection | Discovers hidden causal structure | No — infers from observability data |
| Proactive/Reactive | Proactive | Reactive |
| Training | None | Proprietary ML models |
| Config Changes | Automated effectuation | Recommendations only |

## Kubernetes Autoscaling

### KEDA / HPA / VPA

| Aspect | Godon | KEDA/HPA/VPA |
|--------|-------|--------------|
| Optimization Target | Configuration parameters + coupling structure | Replica counts, resource limits |
| Approach | Driven pressure, discernment | Threshold-based rules |
| Coupling Detection | Yes — discovers HPA/VPA interference | No |
| Proactive/Reactive | Proactive | Reactive |
| Relationship | **Complementary** | Complementary |

Godon can discern interference between HPA and VPA decisions, and tend autoscaler parameters accordingly.

## Feature Summary

| Feature | Godon | Optuna | Ray Tune | Ax | Akamas | StormForge | Datadog |
|---------|-------|--------|----------|-----|--------|------------|---------|
| **Holds declared intents (steerwishes)** | **Yes** | No | No | No | No | No | No |
| **Refusal with named reason** | **Yes** | No | No | No | No | No | No |
| **Measured map (connectome)** | **Yes** | No | No | No | No | No | No |
| **Coupling Detection** | **Yes** | No | No | No | No | No | No |
| **Topology Discovery** | **Yes** | No | No | No | No | No | No |
| **Isolation Certification** | **Yes** | No | No | No | No | No | No |
| **Live System Integration** | Yes | No | No | No | Yes | Yes | N/A |
| **Effectuation Layer** | Yes | No | No | No | Yes | Yes | No |
| **Guardrails** | Yes | No | No | Limited | Yes | Limited | No |
| **Rollback** | Yes | No | No | No | Yes | No | No |
| **Multi-objective** | Yes | Yes | Yes | Yes | Yes | Yes | N/A |
| **Algorithm Diversity** | Yes | Manual | Manual | Manual | Yes | Unknown | N/A |
| **Worker Cooperation** | Yes | No | No | No | Unknown | Unknown | N/A |
| **Open Source** | Yes | Yes | Yes | Partial | No | No | No |
| **Self-hosted** | Yes | N/A | Yes | Yes | No | No | No |

Coupling detection, topology discovery, and isolation certification appear in **no other tool**. And keeping — declared intent held on a measured map, honestly refused, visibly priced — appears in none either. Finding is a crowded field; keeping, in the open tooling landscape, is godon's category.
