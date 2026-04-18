# Reliability Analysis & Optimization of Multi-Phased Spaceflight

A Python-based simulation and optimization framework for evaluating Phased-Mission Systems (PMS) under a Mixed Redundancy Strategy (MRS). The objective of this codebase is to maximize Crew Survival Probability (CSP) and Mission Success Probability (MSP) while strictly adhering to weight and cost constraints.

This implementation extends standard textbook reliability models by incorporating physical operational constraints, specifically thermal load-sharing and continuous online maintenance (a Birth-Death Markov process).

## Key Features

* **Dynamic DFS Path Generation:** Instead of relying on static, hardcoded Boolean Decision Diagrams (BDDs) derived by hand, this engine uses a Depth-First Search (DFS) algorithm to dynamically map chronological path requirements directly from a mission graph dictionary. This makes the system easily scalable to any mission architecture.
* **Continuous Online Maintenance:** Departs from the standard "Pure Death" non-repairable assumption. The engine dynamically sums degradation and restoration matrices ($Q_{net} = Q_{damage} + Q_{repair}$) to simulate a crew actively maintaining hardware during flight.
* **Thermal Cascading:** Implements a cascading multiplier ($\gamma$) to penalize Hot Standby architectures that suffer from thermal and mechanical load-sharing stress.
* **Transition Shocks:** Uses discrete damage matrices ($S$) applied at phase boundaries to simulate staging or re-entry vibrations.

## Repository Structure

### 1. `baseline_model.ipynb`
The control model. Evaluates hardware using a standard Continuous-Time Markov Chain (CTMC) formulated as a *Pure Death Process* (components cannot be repaired). 
* Includes continuous switch degradation ($\lambda_s$).
* Includes on-demand switch jamming probability for Cold Standbys ($\rho$).

### 2. `advanced_model.ipynb`
The primary operational model. Introduces environmental stressors and a Birth-Death stochastic process to evaluate how active maintenance changes optimal system design.
* **Active Phases:** Evaluates simultaneous wear and repair using a net-rate matrix: $Q_{net} = Q_{damage} + Q_{repair}$.
* **Idle Phases:** Isolates the restoration force to simulate dedicated, offline system recovery: $Q_{net} = Q_{repair}$.

## Execution Flow & Architecture

Both models utilize an Adaptive Particle Swarm Optimization (APSO) algorithm to find the optimal redundancy vector $\vec{X} = \{n_H, n_C, n_o\}$.

1. **Graph Traversal:** The DFS maps phase dependencies for Success and Rescue branches.
2. **State Matrix Construction:** The engine builds isolated component-level transition matrices based on specific $\lambda$ (failure) and $\mu$ (repair) rates.
3. **Timeline Simulation:** For each particle in the swarm, the engine evaluates the mission timeline using matrix exponentials ($P(t) = P(0)e^{Qt}$), intercepting the continuum with discrete shock matrices ($S$) at phase boundaries.
4. **Convergence:** The PSO iterates to find the configuration that maximizes CSP within the weight/cost limits.

## Computational Complexity

To prevent the "state-space explosion" typically associated with global Markov models, this framework isolates transition matrices to the component level (usually $4 \times 4$ or $5 \times 5$ arrays). 

Total time complexity:
$$O(I \cdot P \cdot M \cdot N \cdot K \cdot S^3)$$

* $I, P$: PSO Iterations and Particles.
* $M$: Number of valid mission paths.
* $N, K$: Number of Components and Phases.
* $S$: Localized state-space size ($n_H + n_C + 2$).

By keeping $S$ localized, the matrix operations remain computationally trivial, maintaining linear scalability relative to the number of components ($N$). A standard 30-particle, 40-iteration optimization converges in under 10 seconds.

## Discussion of Results

Testing revealed that allowing for a Birth-Death repair process fundamentally changes the optimizer's behavior. Instead of hoarding heavy Cold Standby units, the PSO learns to exploit the active repair rates ($\mu$). 

It mathematically favors components with high repairability, creating an asymmetric architecture where continuous human intervention effectively neutralizes the penalties of thermal cascades and staging shocks. The code demonstrates that assuming hardware is "non-repairable" leads to significantly sub-optimal and overweight spacecraft designs.
