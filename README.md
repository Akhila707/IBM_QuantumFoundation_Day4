# IBM Quantum Foundations — Day 4

<div align="center">

![IBM Quantum](https://img.shields.io/badge/IBM%20Quantum-052FAD?style=flat-square&logo=ibm&logoColor=white)
![Qiskit](https://img.shields.io/badge/Qiskit-6929C4?style=flat-square&logoColor=white)
![Python 3.10](https://img.shields.io/badge/Python%203.10-1a1a2e?style=flat-square&logo=python&logoColor=4fc3f7)
![SciPy](https://img.shields.io/badge/SciPy-COBYLA-4fc3f7?style=flat-square)
![Day 4](https://img.shields.io/badge/Day%2004-Complete-4fc3f7?style=flat-square)
![Day 5](https://img.shields.io/badge/Day%2005-Loading...-555555?style=flat-square)

</div>

<br/>

```
  ╔══════════════════════════════════════════════════════════════╗
  ║                                                              ║
  ║   E(θ) = ⟨ψ(θ)|H|ψ(θ)⟩                                    ║
  ║                                                              ║
  ║   Quantum computer  →  prepares the trial state             ║
  ║   Classical computer →  minimizes the energy                ║
  ║                                                              ║
  ╚══════════════════════════════════════════════════════════════╝
```

<div align="center">
<i>Part of the IBM Quantum 20-Day Learning Sprint · VIT Chennai</i>
</div>

---

## Overview

This notebook implements the **Variational Quantum Eigensolver (VQE)** — a hybrid quantum-classical algorithm for finding the ground state energy of a quantum system.

The quantum computer prepares a parameterized trial state. A classical optimizer iteratively adjusts the circuit parameters to minimize the measured energy. When the energy converges, the ground state has been found.

| Component | Role |
|-----------|------|
| Ansatz (RY gate) | Parameterized quantum circuit — prepares trial state |
| Hamiltonian (H = Z) | Defines the system whose energy we minimize |
| COBYLA optimizer | Classical algorithm that updates circuit parameters |
| AerSimulator | Local quantum simulator (1000 shots per evaluation) |

---

## Background

Every physical system has a ground state — the configuration of lowest energy. In quantum chemistry, knowing the ground state of a molecule determines its chemical behaviour, reaction rates, and binding properties.

Classical simulation of quantum systems requires memory proportional to 2^N, where N is the number of particles. This grows beyond practical limits quickly.

```
N = 10   →        1,024 amplitudes
N = 20   →    1,048,576 amplitudes
N = 50   →  2^50 amplitudes  (intractable classically)
```

VQE addresses this by representing the quantum state directly on a quantum processor and offloading only the parameter optimization to a classical computer. The memory cost scales with N qubits, not 2^N.

---

## Algorithm

### Step 1 · Ansatz Construction

A parameterized RY gate rotates the qubit by angle θ along the Y-axis of the Bloch Sphere:

```
RY(θ)|0⟩  =  cos(θ/2)|0⟩ + sin(θ/2)|1⟩

θ = 0      →  |0⟩   north pole   E = +1.0
θ = π/2    →  |+⟩   equator      E =  0.0
θ = π      →  |1⟩   south pole   E = -1.0  ← ground state
```

### Step 2 · Energy Evaluation

For Hamiltonian H = Z (Pauli-Z), the expectation value is:

```
E(θ) = ⟨ψ(θ)|Z|ψ(θ)⟩
     = P(|0⟩) - P(|1⟩)
```

where P(|0⟩) and P(|1⟩) are measurement probabilities estimated over 1000 shots.

### Step 3 · Classical Optimization

The COBYLA optimizer receives E(θ) and updates θ to reduce the energy. This loop continues until convergence.

```
Initialize θ  (random)
      │
      ▼
Prepare |ψ(θ)⟩  on quantum circuit
      │
      ▼
Measure E(θ) = ⟨ψ|H|ψ⟩
      │
      ▼
COBYLA updates θ
      │
      └──────────────────── repeat until ΔE < tolerance
      │
      ▼
θ_optimal,  E_minimum  (ground state)
```

---

## Results

```
Initial state (random θ):
  θ₀  =  2.990 rad
  E₀  =  −0.992

Final state (after optimization):
  θ*  =  3.177 rad  (≈ π)
  E*  =  −1.0000
  Iterations   :  14
  Converged    :  True
  |E* − E_exact| < 0.001
```

The optimizer recovered θ ≈ π, placing the qubit at the south pole of the Bloch Sphere — the exact ground state of H = Z.

---

## Convergence

```
  0.0 ┤
      │╲
 −0.5 ┤ ╲     ╱╲
      │  ╲   ╱  ╲
 −1.0 ┤───╲─╱────╲────────────────── ● E* = −1.0000
      │    ╲╱     ╲________________/
      └──────────────────────────────────────────────
      0          5          10         14
                        Iteration
```

Initial oscillation reflects the optimizer probing the energy landscape. Convergence to the exact minimum occurs at iteration 14.

---

## Connection to Real Applications

The structure of today's experiment maps directly to quantum chemistry:

| This notebook | Quantum chemistry |
|---------------|-------------------|
| 1 qubit · H = Z | N qubits · molecular Hamiltonian |
| RY ansatz | UCCSD / hardware-efficient ansatz |
| COBYLA | SPSA · L-BFGS-B · gradient-based methods |
| E* = −1.0 | Ground state of H₂ · LiH · BeH₂ |
| 14 iterations | Hundreds to thousands of iterations |

The algorithm is identical. The scale and Hamiltonian change.

---

## Tech Stack

```python
qiskit          >= 1.0.0    # quantum circuit construction
qiskit-aer      >= 0.17.2   # local statevector simulation
scipy           >= 1.10.0   # COBYLA classical optimizer
numpy           >= 1.24.0   # numerical operations
matplotlib      >= 3.7.0    # convergence visualization
python-dotenv   >= 1.0.0    # credential isolation
```

---

## Setup

```bash
git clone https://github.com/Akhila707/IBM_QuantumFoundation_Day4.git
cd IBM_QuantumFoundation_Day4
pip install -r requirements.txt
jupyter notebook
```

---

## Project Structure

```
IBM_QuantumFoundation_Day4/
│
├── notebooks/
│   └── 01_vqe_simulation.ipynb
│
├── results/
│   └── vqe_convergence.png
│
├── .env
├── .gitignore
├── requirements.txt
└── README.md
```

---

## Sprint Progress

```
Day 01  ──  ✅  Qiskit setup · Hello Quantum · first IBM cloud circuit
Day 02  ──  ✅  Superposition · Entanglement · Multi-gate circuits
Day 03  ──  ✅  Gates deep-dive · Grover's algorithm
Day 04  ──  ✅  VQE · parametric circuits · COBYLA optimizer
Day 05  ──  ⬡   QAOA · optimisation landscape · Qiskit Patterns
Day 06  ──  ·   Quantum Error Mitigation · noise models · ZNE
Day 07  ──  ·   Run all experiments on real IBM hardware
·
·
Day 20  ──  ·   Final push · 50+ applications · LinkedIn article
```

---

## Security

- Credentials stored in `.env`, excluded from version control via `.gitignore`
- All results reproducible locally using AerSimulator — no IBM token required

---

<div align="center">

[![GitHub](https://img.shields.io/badge/Akhila707-181717?style=flat-square&logo=github)](https://github.com/Akhila707)
&nbsp;·&nbsp;
[![IBM Quantum](https://img.shields.io/badge/IBM%20Quantum-052FAD?style=flat-square&logo=ibm&logoColor=white)](https://quantum.ibm.com)

</div>
