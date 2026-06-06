# Core Thesis — godon (INTERNAL)

Not for publication. Speculative and aspirational notes.

## The Mechanism (Proven)

Policy-free optimization explores parameter space via watermark-perturbed trials. Each trial is a probe. The observer detects whether breeder A's watermark signal appears in breeder B's objectives. If detected, one edge in the coupling graph is revealed: A → B.

## Transitive Signal Tracing (Proven for Linear Channels)

Each detection reveals one edge:
- A's signal reaches B → edge A→B exists
- B's signal reaches C → edge B→C exists  
- C's signal reaches D → edge C→D exists

Accumulated detections form a partial topology of how the system is coupled internally. Not assumed. Not modeled. Discovered from the system's own dynamics.

## What's Proven

- Interference detection on linear additive channels — bench proven, microgrid scenario
- Watermark propagation through shared infrastructure — measurable, repeatable
- Detection at various coupling strengths — p < 0.001 at 0.9 coupling, works down to 0.1

## What's Hypothesis

- That transitive tracing scales to reveal the full assembly (proven for 2 breeders, 1 edge; unproven for N breeders, N(N-1)/2 edges)
- That accumulated topology represents genuine understanding, not just a partial sketch
- That the approach transfers to nonlinear and non-stationary channels
- That the same epistemological move applies beyond infrastructure

## What's Speculation (Internal Only)

- Application to exogenesis environments
- Application to organoid intelligence breeding
- Understanding complex living systems through transitive signal propagation
- These ideas are genuine but unsubstantiated. Not for publication.

## Phrasing Discipline

Always distinguish:
1. **Proven** — state as fact with bench reference
2. **Hypothesis** — state as hypothesis, explain reasoning
3. **Speculation** — state as open question

Example:
- Proven: "We detected breeder A's watermark in breeder B's objectives through shared memory coupling at p < 0.001"
- Hypothesis: "Accumulated detections may form a partial topology of system coupling"
- Speculation: "This approach could extend to understanding complex biological systems"

## The Assembly Concept

A complex system is an assembly of interconnected components. You can't model it from first principles — too many variables, too many nonlinear interactions, too much hidden coupling. But if you can trace how perturbations propagate transitively through it, the system reveals its own structure. Each signal that survives the coupling path is proof that a connection exists. Each signal that doesn't survive tells you something about the channel's destructive properties.

The topology emerges from the signals. Not from assumptions. Not from models. From the system's own response to being probed.

## Variables That Determine Detection Difficulty

Not just coupling factor. Multiple interacting variables:
- Coupling strength (how much signal transfers)
- Signal-to-noise ratio (receiver's own exploration noise)
- Number of nonlinear transforms in the coupling path
- Transform type (additive = benign, multiplicative = scaling, dead zone = destructive)
- Dimensionality (parameters × objectives)
- Stationarity (does coupling change over time)
- Watermark design (frequency choice, amplitude, number of frequencies)
- Receiver exploration variance (wild exploration drowns weak coupling)

The space is combinatorial but not infinite: roughly 5-6 key variables with 2-3 levels each.

## Channel Classification (Open Design Question)

An assessment that characterizes coupling channels before choosing detection method:
- Linear additive → FFT + permutation test
- Weakly nonlinear → adapted spectral methods
- Strongly nonlinear → no reliable method yet
- Non-stationary → phase-aware methods (research phase)

Where does the classifier live? Part of breeder (startup calibration)? Separate probe unit? Part of observer? Depends on whether channels are stable or shift over time. Non-stationary channels suggest ongoing assessment needed.

## Connection to Meta-Optimization

Channel classification itself is an optimization problem — choosing the right detection method, watermark parameters, trial budget based on channel characteristics. This is the system optimizing its own sensing strategy. Meta-optimization direction.

## godon as a Means, Not an End

godon isn't the point. It's the tool that unlocks a complexity level of systems management that wasn't accessible before. If we can detect and steer interference in shared infrastructure, the same principles apply to any complex adaptive system — biological, computational, hybrid.

## Beyond Infrastructure

We currently have no clue how complex living systems work internally. The exploration-detection loop that maps infrastructure coupling could map biological coupling too — perturb, observe, learn the topology of interactions we can't model from first principles. Same epistemological move, different substrate.
