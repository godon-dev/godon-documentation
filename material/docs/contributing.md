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

**The Composition Horizon**

Nonlinear composition is measured: curves chain through nonlinear switching elements and sum at converging junctions to predict far-end response, with the propagated error bars validated as honest metrology — published as [From Curves to Cascades](https://doi.org/10.5281/zenodo.22401439). The open end is the horizon: about two nonlinear hops at noise σ=0.02, beyond which the far end falls below the detection floor. If you work in nonlinear system identification, uncertainty propagation, or error-in-variables methods — extending that horizon is the open end of the composition story.

**Response Dynamics**

The probe data the engine already collects contains more than the curves report: propagation delay, settling time, channel covariance, step-response shape. Extracting them is signal processing on existing data — designed, not built, and a self-contained contribution. It moves the map from statics toward dynamics: edges form cycles, feedback is the second blind spot, and loop gain / cycle tracing are open. If you are a signal-processing person, the quantities are named and the machinery exists.

**Empirical Live Models**

The measured map — curves with per-point bars, topology, priced gaps — is an empirical model of a live system: explicit, causal, discovered rather than specified, served live (`/curves`, `/predict`) and exportable as a versioned artifact. On the bench it already supports the seeing paths: attribute a regression to its source path, foresee a cascade before it fires, certify isolation empirically, right-size isolation spend. If you build models of systems — system identification, simulation, uncertainty quantification — the map is a new input class: structure you did not have to assume. Consuming it, hardening it, and finding where it breaks are all contributions.

**Live Systems Tending**

Detection, curves, and composition are the perception half. The action half — agents adapting to known coupling, joint moves toward chosen targets, simulate-before-execute — is the project's direction and not yet built. One receipt exists: a quiet-bench chain steered to a target at its far end (target −0.100, landed −0.1038 ± 0.038). If you work in control under coupling, multi-agent coordination, or scheduling, the measured map is the substrate and the loop is the open problem.

**Statistics of Priced Stopping**

Termination is decided by an information-price argument (remaining ignorance vs measurement cost). The current arithmetic is deliberately the simplest defensible version; rigorous treatments (optimal stopping, experimental design under budget) would harden it.

**Detection Statistics & Experiment Design**

Two hard statistical questions sit inside the protocol. CFAR's constant-false-alarm property is empirically untested across conditions, and the threshold arithmetic compares block medians against per-sample scatter — conservative, not literal; a median-aware threshold needs its false-alarm statistics re-derived. And a push that changes the receiver's variance instead of its median is invisible to the current detector — a second statistic on the same trial data (push-scatter vs baseline-scatter) would open variance coupling: jitter, oscillation, instability, the effects that matter most for tending. Fractional-factorial probe schedules are the design-of-experiments side. The fields most vital here: design of experiments and statistical calibration theory.

**Scale & Scan Scheduling**

2-6 agents validated. At N≫6 the pairwise scan is O(N²), turn-taking serializes, and the system drifts while being scanned — scan-rate vs drift-rate is an observability limit to characterize, not a bug to fix. Parallel probe groups, prediction-error-prioritized rescans, topology-aware scheduling (skip pairs the map says are quiet). The bench is N-generic today; the coordination regime is the open part. If you work in graph algorithms or online scheduling, this is a scheduling problem only this instrument creates.

**Infrastructure and Platform Engineering**

godon runs on Kubernetes with Helm charts, container images, and GitHub Actions workflows. Contributions that improve deployment, observability, scaling, or add support for new platforms are practical and welcome.

**Optimization and AI Operations**

Multi-objective search, parallel campaigns, heterogeneous strategies. LLM integration as operations copilot — interpreting measured coupling structure, drafting probe configurations, flagging anomalies. The MCP interface exposes the measured map to LLM agents, grounding their reasoning in measured structure rather than training data.

**Documentation and Communication**

Real-world use cases, deployment guides, architecture explanations, blog posts. If you've deployed godon or built something with it, writing about it helps more people than code contributions.



### Concrete First Contributions

godon is an empirical instrument for complex coupled systems — benches with planted truth, measured response curves, priced stopping, honest boundary maps. The program today is one independent scientist and AI collaborators; the roles below describe work that needs doing, not headcount. If you care about how complex systems actually behave under intervention (rather than how they are modeled or narrated), the entry points are concrete — the assets are public:

**Reproduce a validation cell.** [`scenario-composition-gate`](https://github.com/godon-dev/godon/tree/main/examples/bench/scenario-composition-gate) (three agents, chained coupling — the linear composition cell), [`scenario-door-chain`](https://github.com/godon-dev/godon/tree/main/examples/bench/scenario-door-chain) (nonlinearity in the edge — the nonlinear composition cell), and `scenario-verification-star` (per-receiver curve separation with an uncoupled witness) each run via a single `bench-characterization.yml` dispatch on your own cluster. A reproduction report — confirmed, diverged, or surprising — is a genuine contribution.

**Extend the boundary map.** The detection boundary is an open grid: coupling strength × noise × shape. The published cells — 21 detection cells ([`papers/detection/experiments/`](https://github.com/godon-dev/godon/tree/main/papers/detection/experiments)), a 32-cell characterization sweep ([`papers/characterization`](https://github.com/godon-dev/godon/tree/main/papers/characterization)), and the nonlinear-composition campaign ([`papers/composition`](https://github.com/godon-dev/godon/tree/main/papers/composition)) — each cost one bench run. Pick an untested cell, run it, append the result.

**Analyze the open data.** The sweep data and characterization curve exports (raw points with uncertainty bars) are in the repository. Independent statistical treatment — alternative estimators, stopping-rule critique, visualizations — strengthens the instrument.

**Port a bench.** The generic bench is one Rust container; a scenario is one topology YAML. A bench for a substrate you operate (with its real coupling physics) is the highest-value contribution there is.

**Attack the stopping rule.** Curve retirement uses a deliberately simple information-price argument (remaining ignorance = gap jump × width, vs the local measurement bar). The tests and data to attack it are public; a rigorous replacement would harden every measurement the engine makes.

The method reference: [The Impulse Protocol](https://doi.org/10.5281/zenodo.21962956) (Zenodo DOI) — the protocol, its validation, and its honest boundaries.

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
| [godon-robots](https://github.com/godon-dev/godon-robots) | Systemtender engine — optimization and characterization agents |
| [godon-controller](https://github.com/godon-dev/godon-controller) | Lifecycle logic — systemtender coordination, cleanup cascades |
| [godon-documentation](https://github.com/godon-dev/godon-documentation) | Documentation site and source |
