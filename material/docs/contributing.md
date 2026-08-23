---
description: "Contributing to godon — what we're looking for and how to get started."
---

## Contributing to godon

godon is young and the problem space is wide. We welcome contributions across the full stack — from theoretical work on coupling characterization to infrastructure engineering to documentation.

### What We're Looking For

**Real Substrate Deployments**

The engine is validated on synthetic benches with planted ground truth. The most valuable contribution is pointing it at reality: a system you operate where multiple autonomous optimizers share substrate — data centers, building automation, industrial process control, energy grids. A deployment report (worked, failed, or surprising) is worth more than another bench.

**Bench Scenarios**

Every new coupling channel type extends the boundary map. See [Bench Scenarios](bench_scenarios.md) for the current set and how to add new ones — the generic bench makes arbitrary topologies a single YAML file.

**Nonlinear Composition**

Edges are measured; whether measured edges compose to predict multi-hop response on nonlinear channels is the open keystone (additive composition is validated). If you work in nonlinear system identification, response-surface methods, or causal composition — this is the sharpest open problem in the project.

**Statistics of Priced Stopping**

Termination is decided by an information-price argument (remaining ignorance vs measurement cost). The current arithmetic is deliberately the simplest defensible version; rigorous treatments (optimal stopping, experimental design under budget) would harden it.

**Infrastructure and Platform Engineering**

godon runs on Kubernetes with Helm charts, container images, and GitHub Actions workflows. Contributions that improve deployment, observability, scaling, or add support for new platforms are practical and welcome.

**Optimization and AI Operations**

Multi-objective search, parallel campaigns, heterogeneous strategies. LLM integration as operations copilot — interpreting measured coupling structure, drafting probe configurations, flagging anomalies.

**Documentation and Communication**

Real-world use cases, deployment guides, architecture explanations, blog posts. If you've deployed godon or built something with it, writing about it helps more people than code contributions.

### How to Contribute

1. **Open an issue first** — describe what you want to do before investing time in code. We'll discuss scope and approach.
2. **Fork and branch** — create a branch from main. Use descriptive branch names.
3. **Rebase merges only** — we use rebase merges on all godon-dev repositories. Keep your branch up to date with main.
4. **Open a pull request** — describe the change, reference the issue, list what was tested.

### Repository Overview

| Repository | Purpose |
|---|---|
| [godon](https://github.com/godon-dev/godon) | Core — workflows, bench scenarios, papers |
| [godon-images](https://github.com/godon-dev/godon-images) | Container images — api, causal, observer, cli, seeder, mcp, benches |
| [godon-charts](https://github.com/godon-dev/godon-charts) | Helm charts for Kubernetes deployment |
| [godon-breeders](https://github.com/godon-dev/godon-breeders) | Breeder engine — optimization and characterization agents |
| [godon-controller](https://github.com/godon-dev/godon-controller) | Lifecycle logic — breeder coordination, cleanup cascades |
| [godon-documentation](https://github.com/godon-dev/godon-documentation) | Documentation site and source |
