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

## Open Research

What is genuinely open, honestly bounded. The solved problems moved to [Detection Capabilities](detection_capabilities.md); this page is the frontier.

### Resolved (context, briefly)

The method-choice question is closed. Passive statistics (FFT, mutual information, transfer entropy, Granger, CCM) failed on this problem class for a structural reason: the receiver's own exploration noise sits ~500× above any coupling signal in passive data. The shipped answer is the **impulse protocol** — deliberate, guarded pushes with the receiver holding still, detected by CFAR — validated across coupling shapes including deeply nonlinear cascaded channels and slow drift, zero false positives on controls, with a published boundary map. Passive methods remain interesting only as accelerators (ranking candidates for probe prioritization at scale).

### Composition on Nonlinear Channels (the keystone)

Edges compose additively — validated: measured two-hop response matches the composed prediction from measured one-hop curves, within uncertainty bars, at every level. The open question is **nonlinear composition**: when the coupling path itself is nonlinear (the intermediate's response function bends the signal), does curve composition still predict, and what correction machinery does it need? This is the make-or-break for prediction, tending, and everything above the map. Requires a bench capability that does not exist yet: nonlinearity in the edge itself.

### Fast Non-Stationarity

Drift slower than the detection window is handled (local reference windows, drift flags on re-measurement disagreement). Phase transitions faster than the window — a system that changes regime mid-probe — remain open. Candidate directions: shorter adaptive blocks, dwell-based protocols, drift-rate estimation as a first-class measurement.

### Scaling the Scan

2-6 agents validated. The O(N²) pairwise scan, turn-taking serialization, and scan-rate vs drift-rate at N >> 6 are open — partly engineering (parallel probe groups, spatial locality), partly fundamental (observability limits under drift). Group-scoped coordination and per-receiver curves (one sender walk measures all listeners) already removed one factor of N.

### Prediction-Driven Rescan (the maintenance loop)

A frozen measured map enables prediction: "if node A moves to X, node D responds by Y." When prediction fails, the map is stale — and the error localizes WHERE structure changed. Rescan the neighborhood, diff against the frozen map. The prediction error is the compass: it points toward ignorance; probe where it's most wrong. Designed, prototyped in analysis, not yet an engine loop.

### Multi-Cadence Coupling Spectroscopy

The coupling structure revealed at one probe cadence is one view: slow probes see steady-state thermal coupling, fast probes see electrical/capacitive coupling. Different cadences excite different mechanisms; each yields a different graph. When one cadence suffices (dominant mechanism) the single map is honest; multiple concurrent mechanisms need multi-cadence probing, and detecting that need — via prediction error — is itself open.

### Meta-Optimization of Sensing

The protocol's own parameters (probe amplitude, block lengths, walk depth, convergence threshold) are tunable against measurement outcomes. The system optimizing its own sensing strategy — choosing experiments by expected information — is a natural, unsolved layer. The current walk is deliberately deterministic; adaptive experiment selection is a rung above it.

### From Detection to Steering

The measured map's consumer loop — agents adapting to known coupling, coordinated moves toward chosen targets — is the project's direction and not yet built. The open problems in order: composition on nonlinear channels (above), then joint action under coupling, then destination-directed steering through the measured map. Each rung gates the next; none is assumed.

### Further Reading

- [Detection Capabilities](detection_capabilities.md) — the validated boundary map
- [Bench Scenarios](bench_scenarios.md) — where these questions get measured
