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

## Detection Capabilities

What the impulse protocol — validated, with measured boundaries — covers. One method: non-destructive guarded pushes, ABA block design (push → pause → compare), CFAR detection with adaptive thresholds. The statistical family is radar and sonar's: constant false-alarm rate detection of a known-shape response in unknown noise.

### Linear and Moderately Nonlinear Channels

```
    Sender parameter → [ coupling ] → Receiver objective
```

Fully covered. 21-cell sweep on the generic bench: detection floor between 0.1 and 0.2 coupling at noise σ=0.02; measurement error under 5%; zero false positives across all uncoupled controls. Coupling shapes (linear, saturation, threshold, polynomial) do not matter at adequate strength — the response is detected by its ABA signature, not its frequency content.

### Deeply Nonlinear Cascaded Channels

```
    Sender parameter → [ thermal inertia ] → [ multiplicative growth,
    dead zones ] → [ phase-dependent sensitivity ] → [ sensor noise ]
    → Receiver objective
```

Covered — this is the greenhouse bench, the channel class that defeated every passive method. The impulse survives cascades because it does not rely on the channel preserving any signal structure, only propagating a perturbation: push hard within guardrails, measure whether the receiver's median moves and recovers. Validated bidirectionally at strong coupling, zero false positives on controls.

### Non-Stationary Channels (Slow Drift)

Covered at the bench's drift rates. The CFAR reference window is local (a handful of trials), limiting exposure to slow drift; re-measurements that disagree beyond bars are flagged as drift rather than blended — the instrument reports the change instead of smearing it.

### Honest Boundaries (Measured, Not Guessed)

| Boundary | Where it sits |
|---|---|
| Noise floor | σ=0.05-0.10 at coupling 0.5: signal permanently below threshold — an SNR limit; more budget does not cross it |
| Fast phase transitions | Non-stationarity faster than the detection window remains open |
| Coupling-path nonlinearity | Detection is shape-agnostic; composition of edges validated on additive coupling — nonlinear edge composition is future work |
| Scale | 2-6 agents validated; the coordination regime for 50+ is unbuilt |

### Beyond Detection: Characterization

Detection reports an edge exists. The same protocol, run to depth, measures its **response curve** — the full level→shift shape per (sender, parameter, channel), with uncertainty bars. Full story, calibration numbers, and composition results: [Characterization](characterization.md).

See [Interference Detection](concept_interference_detection.md) for the full method.

### Real-World Channel Examples

- Shared power bus — load affects voltage (linear additive)
- Shared heating/cooling — waste heat through thermal inertia (nonlinear cascaded)
- Shared compute — cache/memory contention (nonlinear, state-dependent)
- Shared data paths — schema or queue coupling (logical coupling, any shape)

### Further Reading

- [Interference Detection](concept_interference_detection.md) — methodology and validation data
- [Bench Scenarios](bench_scenarios.md) — the scenario library behind these claims
- [Open Research](open_research.md) — the open cells and active directions
