---
description: "godon comparison vs Optuna, Hyperopt, Nevergrad, Ray Tune, Ax, Akamas, StormForge, Datadog, KEDA — positioning as optimization application for live operational systems."
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

Godon is an **optimization application** for live operational systems. It sits at a different layer than most tools in this space:

**Libraries** (Optuna, Hyperopt, Nevergrad) provide algorithm primitives — you build the application around them.

**Frameworks** (Ray Tune, Ax) provide orchestration for ML workloads — you adapt to their model.

**SaaS Platforms** (Akamas, StormForge) provide managed optimization — you subscribe and cede control.

Godon provides a complete, self-hosted optimization application: algorithms wrapped in plumbing (effectuation, reconnaissance, coordination) and ops safety (guardrails, rollback, failure handling). It leverages open optimization frameworks internally — currently Optuna, with potential for custom samplers.

The primary differentiator: Godon optimizes **live systems**, not simulations or offline models. This requires safety mechanisms that pure optimization libraries don't provide.

```
┌─────────────────────────────────────────────────────────────────┐
│                         GODON                                    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Plumbing Layer                        │    │
│  │  Effectuation (SSH, HTTP, APIs)  │  Reconnaissance      │    │
│  │  Trial coordination              │  Worker management    │    │
│  └─────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Ops Safety Layer                      │    │
│  │  Guardrails  │  Rollback  │  Consecutive failure handling│   │
│  └─────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              Optimization Core                           │    │
│  │  (Optuna: TPE, NSGA-II/III, QMC  →  custom samplers)    │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

## Overview

| Category | Tools |
|----------|-------|
| **Optimization Libraries** | Optuna, Hyperopt, Nevergrad, Scikit-Optimize |
| **ML Frameworks** | Ray Tune, Ax/BoTorch, Weights & Biases |
| **Infrastructure Platforms** | Akamas, StormForge, Turbonomic |
| **AIOps / Observability** | Datadog, Dynatrace, New Relic |
| **Kubernetes Autoscaling** | KEDA, HPA/VPA, Predictive HPA |

## Optimization Libraries

### Optuna

| Dimension | Godon | Optuna |
|-----------|-------|--------|
| **What it provides** | Complete application | Algorithm library |
| **Plumbing - Effectuation** | Built-in (SSH, HTTP, APIs) | None — you build it |
| **Plumbing - Reconnaissance** | Built-in (Prometheus) | None — you build it |
| **Plumbing - Coordination** | Controller API, trial sharing | Study management only |
| **Ops Safety - Guardrails** | Hard limits with automatic response | No |
| **Ops Safety - Rollback** | Previous/best/baseline restoration | No |
| **Ops Safety - Failure handling** | Consecutive failure thresholds, skip_target | No |
| **Deployment** | Kubernetes-native Helm chart | Python library |
| **Scope** | Production systems | Any optimization problem |

### Hyperopt

| Dimension | Godon | Hyperopt |
|------------|-------|----------|
| **Plumbing - Effectuation** | Built-in (SSH, HTTP, APIs) | None |
| **Plumbing - Reconnaissance** | Built-in (Prometheus) | None |
| **Plumbing - Coordination** | Controller API, trial sharing | MongoDB-based trials |
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
| **Ops Safety - Guardrails** | Yes | No |
| **Ops Safety - Rollback** | Yes | No |
| **Algorithms** | Meta-heuristics (TPE, EA) | Derivative-free (Evolution, Bandits) |
| **Multi-objective** | Yes | Yes |
| **Domain** | Infrastructure + generic | Generic functions |
| **Deployment** | Kubernetes | Python library |

## ML-Focused Frameworks

### Ray Tune

| Aspect | Godon | Ray Tune |
|--------|-------|----------|
| Primary Domain | Infrastructure, systems | ML training |
| Live System Integration | Native | Manual |
| Effectuation Layer | Yes (SSH, HTTP, APIs) | No |
| Guardrails | Yes | No |
| Rollback | Yes | No |
| Deployment | Kubernetes-native | Ray cluster |
| Overhead | Lightweight | Heavy (Ray runtime) |
| Training Required | No | No |

### Ax / BoTorch

| Aspect | Godon | Ax |
|--------|-------|-----|
| Algorithm | Meta-heuristics | Bayesian optimization |
| Live System Integration | Native | Manual |
| Constraints | Guardrails + Rollback | Parameter constraints |
| Deployment | Self-hosted | Hosted service or self-hosted |
| Multi-objective | Yes | Yes |
| ML Dependency | None | BoTorch (Gaussian processes) |

### Weights & Biases Sweeps

| Aspect | Godon | W&B Sweeps |
|--------|-------|------------|
| Type | Self-hosted application | SaaS + library |
| Live System Integration | Native | Manual |
| Data Ownership | Full | Vendor-hosted |
| Cost | Free | Subscription tiers |
| Offline Support | Yes | Limited |

## Infrastructure Optimization Platforms

### Akamas

| Dimension | Godon | Akamas |
|------------|-------|--------|
| **License** | AGPL (open source) | Proprietary |
| **Deployment** | Self-hosted (Helm) | SaaS |
| **Kubernetes-bound** | No | Yes |
| **Algorithm Transparency** | Full | Black-box |
| **Extensibility** | Custom breeders | Vendor-defined |
| **Cost** | Free | Subscription |
| **Vendor Lock-in** | None | High |

## Infrastructure Optimization Platforms

### Akamas

| Dimension | Godon | Akamas |
|------------|-------|--------|
| **License** | AGPL (open source) | Proprietary |
| **Deployment** | Self-hosted (Helm) | SaaS |
| **Kubernetes-bound** | No | Yes |
| **Algorithm Transparency** | Full | Black-box |
| **Extensibility** | Custom breeders | Vendor-defined |
| **Cost** | Free | Subscription |
| **Vendor Lock-in** | None | High |

### StormForge

### Turbonomic (IBM)

| Aspect | Godon | Turbonomic |
|--------|-------|------------|
| Approach | Search-based optimization | Resource management + placement |
| License | Open source | Proprietary |
| Scope | Configuration tuning | Full resource orchestration |
| Real-time | Continuous optimization | Real-time decisions |
| Integration | Network-accessible systems | VMware, cloud providers |

## AIOps Platforms

### Datadog Watchdog

| Aspect | Godon | Datadog Watchdog |
|--------|-------|------------------|
| Approach | Proactive optimization | Reactive anomaly detection |
| Action | Configuration changes | Alerts, some recommendations |
| ML Required | No | Yes |
| Optimization | Search-based | Pattern recognition |
| Cost | Free | Part of Datadog subscription |

### Dynatrace Davis

| Aspect | Godon | Dynatrace Davis |
|--------|-------|-----------------|
| Approach | Optimization search | AI-powered root cause |
| Proactive/Reactive | Proactive | Reactive |
| Training | None | Proprietary ML models |
| Config Changes | Automated effectuation | Recommendations only |

## Kubernetes Autoscaling

### KEDA / HPA / VPA

| Aspect | Godon | KEDA/HPA/VPA |
|--------|-------|--------------|
| Optimization Target | Configuration parameters | Replica counts, resource limits |
| Approach | Search-based | Threshold-based rules |
| Proactive/Reactive | Proactive | Reactive |
| Relationship | **Complementary** | Complementary |

Godon can optimize autoscaler parameters (thresholds, scaling factors) while KEDA/HPA handles scaling.

## Feature Summary

| Feature | Godon | Optuna | Ray Tune | Ax | Akamas | StormForge |
|---------|-------|--------|----------|-----|--------|------------|
| **Live System Integration** | Yes | No | No | No | Yes | Yes |
| **Effectuation Layer** | Yes | No | No | No | Yes | Yes |
| **Reconnaissance** | Yes | No | No | No | Yes | Yes |
| **Guardrails** | Yes | No | No | Limited | Yes | Limited |
| **Rollback** | Yes | No | No | No | Yes | No |
| **Multi-objective** | Yes | Yes | Yes | Yes | Yes | Yes |
| **Algorithm Diversity** | Yes | Manual | Manual | Manual | Yes | Unknown |
| **Worker Cooperation** | Yes | No | No | No | Unknown | Unknown |
| **Open Source** | Yes | Yes | Yes | Partial | No | No |
| **Self-hosted** | Yes | N/A | Yes | Yes | No | No |
| **Kubernetes Native** | Yes | Optional | Optional | No | Yes | Yes |
