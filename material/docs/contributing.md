---
description: "Contributing to godon — what we're looking for and how to get started."
---

## Contributing to godon

godon is young and the problem space is wide. We welcome contributions across the full stack — from theoretical work on nonlinear detection to infrastructure engineering to documentation.

### What We're Looking For

**Bench Scenarios**

Every new coupling channel type extends our understanding of where interference detection works and where it doesn't. If you work with a system where multiple autonomous optimizers share infrastructure — data centers, building automation, industrial process control, energy grids — a bench scenario for it would be a valuable contribution. See [Bench Scenarios](bench_scenarios.md) for the current set and how to add new ones.

**Detection Methods**

The [Methods Evaluated](open_research.md#methods-evaluated) section lists approaches that were tested with early signaling and haven't been re-evaluated with the current watermarking system. If you have expertise in time series analysis, causal inference, spectral methods, or information theory — implementing or adapting a detection method for the observer is a concrete, well-scoped contribution.

**Nonlinear Dynamics and Signal Processing**

The greenhouse bench represents a deeply nonlinear cascaded channel where no detection method has produced reliable results yet. If this sounds like your research area, the bench is a well-defined experimental setup with known signal characteristics. The signal death diagram in [Open Research](open_research.md) shows exactly where and why the signal degrades.

**Infrastructure and Platform Engineering**

godon runs on Kubernetes with Helm charts, Docker images, and GitHub Actions workflows. Contributions that improve deployment, observability, scaling, or add support for new platforms are practical and welcome.

**Optimization and AI Operations**

Multi-objective metaheuristic search, parallel optimization campaigns, LLM integration for configuration suggestion — if you work in optimization or AI operations and see gaps or improvements, we'd like to hear about it.

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
| [godon](https://github.com/godon-dev/godon) | Core — workflows, examples, bench scenarios |
| [godon-images](https://github.com/godon-dev/godon-images) | Container images — observer, breeder, simulators |
| [godon-charts](https://github.com/godon-dev/godon-charts) | Helm charts for Kubernetes deployment |
| [godon-breeders](https://github.com/godon-dev/godon-breeders) | Breeder engine — optimization worker |
| [godon-documentation](https://github.com/godon-dev/godon-documentation) | Documentation site and source |
