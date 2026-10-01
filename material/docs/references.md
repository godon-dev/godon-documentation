---
description: "godon science references — the published work the method rests on: evolutionary-algorithm foundations, probe-block design, CFAR detection, block-oriented composition, fair sharing, and cybernetics."
---

# Science References 
-------------

Each entry is work the method demonstrably rests on. Neighbors the papers position against are named in the papers' related-work sections, not here.

## Core Concepts & Proof of Concept

!!! quote "[1] [Autonomous Configuration of Network Parameters in Operating Systems using Evolutionary Algorithms](https://www.researchgate.net/publication/327392488_Autonomous_Configuration_of_Network_Parameters_in_Operating_Systems_using_Evolutionary_Algorithms)"

    This paper provided the foundational proof-of-concept work that inspired this project, demonstrating the feasibility of applying evolutionary algorithms to configure network parameters.

## Algorithmic Optimization & Parallelization

!!! quote "[2] [A unified view of parallel multi-objective evolutionary algorithms](https://hal.archives-ouvertes.fr/hal-02304734/document)"

    This work underpins the project's approach to accelerating metaheuristics through parallelization, offering a unified perspective on parallel multi-objective evolutionary algorithms.

## Probe-Block Design

!!! quote "[3] [Stochastic Designs in Event-Related fMRI](https://doi.org/10.1006/nimg.1999.0498)"

    Friston, Zarahn, Josephs, Henson and Dale, 1999. The block-design principle: sustained stimulation blocks alternating with rest, because brief events are smeared by the substrate's response. A push/pause probe pair is this design — a perturbation block long enough to survive the substrate's response latency, followed by a rest block that captures recovery.

!!! quote "[4] Statistics for Experimenters: An Introduction to Design, Data Analysis, and Model Building"

    Box, Hunter and Hunter, 1978 (Wiley). The ABA structure — stimulus, rest, stimulus — is standard experimental practice; probe pairs use it so every reading can be compared against its own baseline.

## Signal Detection

!!! quote "[5] Adaptive detection mode with threshold control as a function of spatially sampled clutter-level estimates"

    Finn and Johnson, 1968 (RCA Review). Constant-false-alarm-rate detection, established in radar signal processing. The step-change detector that turns probe responses into coupling verdicts belongs to this family; what is new in godon is the protocol around it — who probes, who holds still — not the detector.

## Sampling Rules for the Walk

!!! quote "[6] [On the Experimental Attainment of Optimum Conditions](https://doi.org/10.1111/j.2517-6161.1951.tb00067.x)"

    Box and Wilson, 1951. Response-surface methodology — sample where the response surface says information pays. The walk's sampling rule is close kin.

!!! quote "[7] [Efficient Global Optimization of Expensive Black-Box Functions](https://doi.org/10.1023/A:1008306431147)"

    Jones, Schonlau and Welch, 1998. Model-based design of experiments: sample where uncertainty or disagreement is highest. The walk applies the same rule inside a live substrate it does not own and cannot pause.

## Composing Measured Blocks

!!! quote "[8] Nonlinear Problems in Random Theory"

    Wiener, 1958 (The Technology Press and Wiley). The origin of modeling a nonlinear system as a composition of measured blocks.

!!! quote "[9] [Block-oriented Nonlinear System Identification](https://doi.org/10.1007/978-1-84996-513-2)"

    Giri and Bai (eds.), 2010 (Springer). The Wiener–Hammerstein lineage in textbook form: identify each block separately, then compose. godon's composition checks follow this material.

## Fair Sharing of Probing Turns

!!! quote "[10] [Analysis and Simulation of a Fair Queueing Algorithm](https://doi.org/10.1145/75246.75248)"

    Demers, Keshav and Shenker, 1990. Fair queueing and the max-min fairness lineage behind it.

!!! quote "[11] [Efficient Fair Queuing Using Deficit Round-Robin](https://doi.org/10.1109/90.502236)"

    Shreedhar and Varghese, 1996. Deficit round-robin, the efficient instantiation of fair sharing. When several probing runs contend for leases, the schedule that shares turns among them descends from this work — applied to lease acquires instead of packets.

## Network Tomography

!!! quote "[12] [Network tomography: estimating source-destination traffic intensities from link data](https://doi.org/10.1080/01621459.1996.10476697)"

    Vardi, 1996. Inferring internal structure from external measurements. 

!!! quote "[13] [Internet Tomography](https://doi.org/10.1109/79.998081)"

    Coates, Hero, Nowak and Yu, 2002. Active probing that selects its measurements adaptively. Both share godon's active-probing philosophy; the difference is that the probers are the coupled nodes themselves, not a central observer.

## Cybernetics Roots

!!! quote "[14] [Cybernetics: Or Control and Communication in the Animal and the Machine](https://doi.org/10.7551/mitpress/11810.001.0001)"

    Wiener, 1948. Perturbing a coupled system and observing its response to discover hidden structure is, at its root, cybernetics — the study of control and communication in coupled systems.

!!! quote "[15] [An Introduction to Cybernetics](https://doi.org/10.5962/bhl.title.5851)"

    Ashby, 1956. The law of requisite variety: a controller must model the complexity of the environment it acts on. The connectome is that model, measured empirically rather than specified analytically.
