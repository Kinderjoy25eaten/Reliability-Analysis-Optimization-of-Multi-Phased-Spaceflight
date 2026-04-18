# Phased-Mission System Reliability Optimization

This repository implements a high-fidelity simulation and optimization framework for analyzing **Phased-Mission Systems (PMS)** using a **Mixed Redundancy Strategy (MRS)**. Our primary objective is to maximize **Crew Survival Probability (CSP)** and **Mission Success Probability (MSP)** under strict weight and cost constraints.

Unlike idealized textbook models that assume hardware is non-repairable, this project bridges the gap between pure mathematics and realistic physical aerospace operations by incorporating transition shocks, thermal cascades, and **continuous online maintenance**.

---

## Core Architectural Improvement: Dynamic DFS Path Generation

A significant departure from the reference paper and traditional PMD-BDD methods is our **Dynamic DFS Path Generator**.

* **The Methodology:** Instead of deriving static Boolean algebraic equations by hand (using `ite` manipulation rules), we engineered a recursive **Depth-First Search (DFS)** algorithm.
* **The Innovation:** The algorithm traverses a mission graph dictionary and dynamically constructs disjoint chronological path requirements for every possible mission branch (Success, Fail, Rescue).
* **The Benefit:** This architecture ensures the engine is infinitely scalable. You can modify the mission flow or add new phases without ever touching the core reliability logic.

---

## Model Overviews

### 1. Baseline Model (`baseline_model.ipynb`)
The baseline serves as the controlled reference point. It evaluates hardware using a standard Continuous-Time Markov Chain (CTMC) as a *Pure Death Process* (non-repairable), while incorporating realistic hardware-level constraints:
* **Continuous Switch Failure ($\lambda_s$):** Modeling the degradation of active switching sensors over time.
* **On-Demand Switch Jamming ($\rho$):** Modeling the discrete probability of a switch failing to activate a Cold Standby unit.

### 2. Advanced Model (`advanced_model.ipynb`)
The advanced model introduces environmental stressors and human intervention to test system resilience:
* **Cascading Load-Share Failures ($\gamma$):** An intra-component multiplier that increases the failure rate of remaining units as backups are activated.
* **Phase Transition Shocks ($S$):** Discrete damage matrices applied at phase boundaries to simulate the violent vibrations of staging or re-entry.
* **Bi-Directional Stochastic Process ($\mu$):** Replaces the textbook "non-repairable" assumption with an active Birth-Death process. It models continuous system recovery via online maintenance rates ($\mu$), allowing probability mass to flow back toward perfect health.

---

## Code Architecture & Mathematical Foundations

### **Baseline Model (6 Blocks)**
1. **Environment Setup:** Standard libraries (NumPy, SciPy, Matplotlib) for matrix exponentials and optimization.
2. **Mission Graphing:** Modeling the mission as a Directed Acyclic Graph (DAG) of phase nodes.
3. **DFS Path Generator:** Recursively identifies all successful and rescue paths through the DAG.
4. **Hardware Parameters:** Defining $\lambda_c$, $\lambda_s$, and $\rho$ for unique components.
5. **Standard Markov Engine:** Solves the forward equation $\mathbf{P}(t) = \mathbf{P}(0)e^{\mathbf{Q}t}$ to compute state probabilities.
6. **APSO Optimizer & Execution:** Adaptive Particle Swarm Optimization to find the optimal vector $\vec{X} = \{n_H, n_C, n_o\}$.

### **Advanced Model (7 Blocks)**
1. **Setup & Imports:** Standard environment initialization.
2. **Mission Graphing:** Logic mapping for success/fail branches.
3. **DFS Path Generator:** Automated temporal mapping of component requirements.
4. **Advanced Specifications:** Introduction of the $\gamma$ multiplier and continuous repair rates ($\mu_c$, $\mu_s$).
5. **State-Dependent Matrix Engine:** * **Active Phases:** Simulates simultaneous wear and repair via a net-rate matrix: $\mathbf{Q}_{net} = \mathbf{Q}_{damage} + \mathbf{Q}_{repair}$.
   * **Idle Phases:** Isolates the restoration force to simulate dedicated system recovery: $\mathbf{Q}_{net} = \mathbf{Q}_{repair}$.
6. **Shock Logic:** Applies discrete boundary matrices such that $\mathbf{P}_{i+1}(0) = \mathbf{P}_i(t) \cdot \mathbf{S}$, simulating staging damage.
7. **Execution & Profiling:** Final heuristic search and convergence graphing.

---

## Discussion of Results

A key observation during testing was that the **Advanced Model** occasionally yielded a higher CSP than the Baseline. This is not a mathematical error, but a discovery of **Structural Resilience**.

The PSO algorithm "learned" to design architectures that specifically exploit the **Online Maintenance** capabilities of the crew. By selecting components with high repair rates ($\mu$) and pairing them with reliable switches, the optimizer created a system where continuous human intervention effectively neutralized the penalties of staging shocks and thermal cascades. It proved mathematically that highly repairable components require fewer heavy cold standbys. This demonstrates that a realistic, birth-death stochastic model allows for much smarter, asymmetric engineering trade-offs than an idealized pure-death model.

---

## Computational Complexity

The framework is designed for high-speed execution by localizing Markovian state-spaces. The total time complexity is defined as:

$$O(I \cdot P \cdot (M \cdot N \cdot K \cdot S^3))$$

Where:
* **I, P**: Iterations and Particles (Search space depth).
* **M**: Number of mission paths (DFS traversal).
* **N, K**: Components and Phases (System scale).
* **S**: Localized state-space size ($n_H + n_C + 2$).

### Scalability Analysis
While matrix operations are traditionally $O(n^3)$, our approach maintains **Linear Scalability** relative to the number of components ($N$). By avoiding a single "Global System Matrix," we prevent the state-space explosion typically associated with complex Phased-Mission Systems. Furthermore, computing $\mathbf{Q}_{net}$ requires only simple matrix addition ($O(S^2)$), ensuring the advanced Birth-Death mechanics add near-zero computational overhead.

---

## Performance Metrics
* **Convergence Time:** < 10.0 seconds (30 particles, 40 iterations).
* **Memory Management:** Matrix operations are localized to the component level (typically $4 \times 4$ or $5 \times 5$), preventing global state-space explosion and ensuring computational feasibility on consumer-grade hardware.
