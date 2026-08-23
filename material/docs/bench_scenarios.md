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

## Bench Scenarios

Bench scenarios are planted-reality experiments: a simulator with a known coupling topology (the ground truth), target and breeder definitions, and a GitHub Actions workflow that runs the full protocol against it. You plant the truth, the engine measures it, the comparison validates the instrument.

### Generic Bench (workhorse)

The configurable synthetic coupling bench. Any topology: node count, per-node parameter and objective counts, coupling shapes (`linear`, `threshold`, `saturation`, `polynomial`), edge strengths, stacked noise (white, colored, drifting). Deterministic per seed.

| Property | Value |
|---|---|
| Directory | [`examples/bench/`](https://github.com/godon-dev/godon/tree/main/examples/bench) (workflows: `bench-generic.yml`, `bench-characterization.yml`) |
| Image | `ghcr.io/godon-dev/godon-bench-generic` |
| Channel type | Any (per topology) |
| Validation | 21-cell detection sweep + full characterization suite |

**Characterization scenarios** (2-3 breeders, one carrier parameter, dead parameters as controls):

- `scenario-characterization` — threshold carrier, the original loop-validation scenario
- `scenario-characterization-saturation` — saturation carrier; validated against planted truth to ≤0.7σ per point
- `scenario-characterization-ch1` — edge feeding objective_1: the per-channel mapping cell
- `scenario-verification-star` — three agents, one edge, one uncoupled witness: per-receiver curve separation
- `scenario-composition-gate` — chain topology (A → C → B): composed two-hop response vs measured — the composition validation cell

**Detection sweep scenarios**: `scenario-generic-chain4` (4-node chain, topology recovery), `scenario-generic-pair`, `scenario-generic-noisy`, `scenario-generic-nonlinear`.

Sweep results (linear coupling, noise σ=0.02): detection floor between 0.1 and 0.2 coupling strength; measurement error under 5%; **zero false positives** across all control cells; shape-agnostic at adequate coupling — saturation, threshold, and polynomial shapes all detected at strength 0.7. Honest boundary: at noise σ=0.10 with coupling 0.5, the signal sits permanently below the detection threshold — an SNR limit, not a budget limit.

### Microgrid

Linear additive coupling through a shared power bus. The first bench the engine was validated on; detection works across the full strength range (0.0-0.9).

| Property | Value |
|---|---|
| Directory | [`examples/bench/scenario-microgrid`](https://github.com/godon-dev/godon/tree/main/examples/bench/scenario-microgrid) (+ `scenario-microgrid-6breeder` for the 6-agent scale cell) |
| Channel type | Linear additive |
| Validation | Pairwise detection 0.0-0.9; 6-breeder scale run |

### Greenhouse

Deeply nonlinear cascaded coupling: waste heat and CO2 through thermal inertia, multiplicative growth with dead zones, crop-phase drift, irreversible damage thresholds. The historically hardest channel — and the one the impulse protocol was designed for: bidirectional detection validated at strong coupling, zero false positives on the uncoupled control.

| Property | Value |
|---|---|
| Directory | [`examples/bench/scenario-4`](https://github.com/godon-dev/godon/tree/main/examples/bench/scenario-4) |
| Channel type | Deeply nonlinear cascaded + non-stationary (crop drift) |
| Validation | Bidirectional detection at coupling 0.9; clean control at 0.0 |

### Detection Coverage

| Scenario | Channel type | Status |
|---|---|---|
| Generic (all shapes) | Configurable | Validated: sweep + characterization suite |
| Microgrid | Linear additive | Validated (0.0-0.9) |
| Microgrid 6-breeder | Linear additive | Validated at scale |
| Greenhouse | Nonlinear cascaded, non-stationary | Validated (strong coupling) |

Open cells (honest boundaries): non-stationarity with phase transitions faster than the detection window; coupling-path nonlinearity for composition (the bench composes additively — a nonlinearity-in-the-edge bench capability is future work).

### Adding a New Bench

Scenario structure:

```
examples/bench/scenario-<name>/
├── topology.yaml          # planted ground truth (generic bench)
│                          # — or docker-compose.yml for other simulators
├── targets/
│   └── node-N.yaml        # one target per node
└── breeders/
    └── breeder-N.yml      # one breeder config per node
```

For the generic bench, only the topology file changes between scenarios. The characterization workflow discovers breeders and targets from the scenario directory (any node count). See an existing scenario as the reference; the generic bench's HTTP contract (`/{node}/apply`, `/{node}/metrics/json`) is the target interface.

### Further Reading

- [Detection Capabilities](detection_capabilities.md) — what the validated method covers
- [Getting Started](getting_started.md) — running your first scenario
