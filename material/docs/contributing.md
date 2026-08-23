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



### Who Fits Where

Roles the project genuinely needs — none require prior background in the specific stack:

| Role | What you'd do | Background that maps well |
|---|---|---|
| **Statistician** | Attack the stopping rule, harden the uncertainty bars, design the boundary-grid cells, critique the sweep methodology | Experimental design, robust statistics, sequential analysis |
| **Rust engineer** | The causal service (detection, curves), benches, API — a small, well-tested codebase where measurement correctness is the product | Systems Rust, Axum/tokio, a taste for numerical code |
| **SRE / infrastructure engineer** | Deploy against real substrate, report where the instrument breaks, improve the Helm/CI/observability plumbing | Running production systems, Kubernetes, the pain of not knowing why things interact |
| **Controls / system-identification researcher** | The composition keystone: when do measured response functions chain, what correction machinery does nonlinearity need | Nonlinear system ID, response-surface methods, causal composition |
| **Optimization engineer** | The breeder engine: multi-objective search, parallel campaigns, algorithm diversity | Optuna/metaheuristics, production optimization loops |
| **Technical writer / communicator** | The story is unusual (measure-first infrastructure epistemology) and under-told; docs site, blog, papers | Can explain measurement discipline without hype |
| **Domain owner** | You operate coupled infrastructure (datacenter, grid, HVAC, industrial) and want to know what actually influences what | The substrate itself — the scarcest contribution |
| **Student** | Reproduce a published cell, extend the boundary grid by one row — bounded, real, citable work | Curiosity; the bench and data are public |

The core loop crosses statistics, systems engineering, and measurement physics — most useful contributions come from one of those angles, not from knowing godon.

### Concrete First Contributions

The fastest contributions are one workflow dispatch away — the assets are public:

**Reproduce a validation cell.** [`scenario-composition-gate`](https://github.com/godon-dev/godon/tree/main/examples/bench/scenario-composition-gate) (three agents, chained coupling: do measured one-hop curves compose to the two-hop response?) and `scenario-verification-star` (per-receiver curve separation with an uncoupled witness) each run via a single `bench-characterization.yml` dispatch on your own cluster. A reproduction report — confirmed, diverged, or surprising — is a genuine contribution.

**Extend the boundary map.** The detection boundary is an open grid: coupling strength × noise × shape. The published cells (21 so far, data in [`papers/detection/experiments/`](https://github.com/godon-dev/godon/tree/main/papers/detection/experiments)) each cost one bench run. Pick an untested cell, run it, append the result.

**Analyze the open data.** The sweep data and characterization curve exports (raw points with uncertainty bars) are in the repository. Independent statistical treatment — alternative estimators, stopping-rule critique, visualizations — strengthens the instrument.

**Port a bench.** The generic bench is one Rust container; a scenario is one topology YAML. A bench for a substrate you operate (with its real coupling physics) is the highest-value contribution there is.

**Attack the stopping rule.** Curve retirement uses a deliberately simple information-price argument (remaining ignorance = gap jump × width, vs the local measurement bar). The tests and data to attack it are public; a rigorous replacement would harden every measurement the engine makes.

The method reference: [The Impulse Protocol](https://doi.org/10.5281/zenodo.21962957) (Zenodo DOI) — the protocol, its validation, and its honest boundaries.

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
