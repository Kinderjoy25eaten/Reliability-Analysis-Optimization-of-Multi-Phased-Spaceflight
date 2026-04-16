# Phased-Mission System Reliability Optimization

This repository implements a high-fidelity simulation and optimization framework for analyzing **Phased-Mission Systems (PMS)** using a **Mixed Redundancy Strategy (MRS)**. Our primary objective is to maximize **Crew Survival Probability (CSP)** and **Mission Success Probability (MSP)** under strict weight and cost constraints.

Unlike idealized textbook models, this project bridges the gap between pure mathematics and realistic physical chaos by incorporating transition shocks, thermal cascades, and opportunistic maintenance.

---

## Core Architectural Improvement: Dynamic DFS Path Generation

A significant departure from the reference paper and traditional PMD-BDD methods is our **Dynamic DFS Path Generator**. 

* **The Methodology:** Instead of deriving static Boolean algebraic equations by hand (using `ite` manipulation rules), we engineered a recursive **Depth-First Search (DFS)** algorithm.
* **The Innovation:** The algorithm traverses a mission graph dictionary and dynamically constructs disjoint chronological path requirements for every possible mission branch (Success, Fail, Rescue).
* **The Benefit:** This architecture ensures the engine is infinitely scalable. You can modify the mission flow or add new phases without ever touching the core reliability logic.

---

## Model Overviews

### 1. Baseline Model (`baseline_model.ipynb`)
The baseline serves as the controlled reference point. It evaluates hardware using a standard Continuous-Time Markov Chain (CTMC) while incorporating realistic hardware-level constraints:
* **Continuous Switch Failure ($\lambda_s$):** Modeling the degradation of active switching sensors over time.
* **On-Demand Switch Jamming ($\rho$):** Modeling the discrete probability of a switch failing to activate a Cold Standby unit.

### 2. Advanced Model (`advanced_model.ipynb`)
The advanced model introduces environmental and operational stressors to test system resilience:
* **Cascading Load-Share Failures ($\gamma$):** An intra-component multiplier that increases the failure rate of remaining units as backups are activated.
* **Phase Transition Shocks ($S$):** Discrete damage matrices applied at phase boundaries to simulate the violent vibrations of staging or re-entry.
* **Gap-Time Opportunistic Maintenance ($R$):** A repair matrix applied during inter-phase "quiet periods," allowing the crew to restore probability mass from failed states to healthy ones.

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
4. **Advanced Specifications:** Introduction of the $\gamma$ multiplier, $\pi_{shock}$, and $\pi_{repair}$ values.
5. **Environmental Matrix Engine:** Implements the modified failure rate $\lambda'_i = \lambda \cdot \gamma^i$ to account for thermal stress.
6. **Maintenance & Shock Logic:** Applies discrete matrices such that $\mathbf{P}_{i+1}(0) = \mathbf{P}_i(t) \cdot \mathbf{R} \cdot \mathbf{S}$, simulating the "saw-tooth" reliability curve.
7. **Execution & Profiling:** Final heuristic search and convergence graphing.

---

## Discussion of Results

A key observation during testing was that the **Advanced Model** occasionally yielded a higher CSP than the Baseline. This is not a mathematical error, but a discovery of **Structural Resilience**. 

The PSO algorithm "learned" to design architectures that specifically exploit the **Gap-Time Maintenance** window. By selecting components with high repairability and pairing them with reliable switches, the optimizer created a system where opportunistic repairs effectively outweighed the penalties of shocks and thermal cascades. This proves that a realistic model allows for more intelligent, hardware-specific engineering trade-offs than an idealized one.

---

## Performance Metrics
* **Convergence Time:** < under(10.0 seconds) .
* **Memory Management:** Matrix operations are localized to the component level (typically $4 \times 4$ or $5 \times 5$), preventing global state-space explosion and ensuring computational feasibility on consumer-grade hardware.
