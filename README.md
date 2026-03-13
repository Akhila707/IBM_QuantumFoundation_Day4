# IBM Quantum Foundations — Day 4

<div align="center">

![IBM Quantum](https://img.shields.io/badge/IBM%20Quantum-052FAD?style=flat-square&logo=ibm&logoColor=white)
![Qiskit](https://img.shields.io/badge/Qiskit-6929C4?style=flat-square&logoColor=white)
![Python 3.10](https://img.shields.io/badge/Python%203.10-1a1a2e?style=flat-square&logo=python&logoColor=4fc3f7)
![scipy](https://img.shields.io/badge/scipy-COBYLA-4fc3f7?style=flat-square)
![Day 4](https://img.shields.io/badge/Day%2004-Complete-4fc3f7?style=flat-square)
![Day 5](https://img.shields.io/badge/Day%2005-Loading...-555555?style=flat-square)

</div>

<br/>

```
  ╔══════════════════════════════════════════════════════════════╗
  ║                                                              ║
  ║   E(θ) = ⟨ψ(θ)|H|ψ(θ)⟩                                    ║
  ║                                                              ║
  ║   Quantum prepares the state.                               ║
  ║   Classical finds the minimum.                              ║
  ║   Together — they solve what neither can alone.             ║
  ║                                                              ║
  ╚══════════════════════════════════════════════════════════════╝
```

<div align="center">
<i>Part of the IBM Quantum 20-Day Learning Sprint · VIT Chennai</i>
</div>

---

## What Did We Build Today?

Today's notebook implements **VQE — Variational Quantum Eigensolver**.

It sounds intimidating. It isn't. Here's the honest explanation.

---

## Explain It Like I'm in High School

Imagine you're trying to find the lowest point in a hilly landscape — blindfolded.

You can't see the whole map. You can only feel whether you're going uphill or downhill at each step.

**That's exactly what VQE does** — except the landscape is the energy of a quantum system, and the steps are taken by a classical optimizer adjusting the parameters of a quantum circuit.

```
The landscape   →  energy surface E(θ)
Your position   →  current parameter θ
Feeling uphill  →  energy going up
Feeling downhill→  energy going down
Lowest point    →  ground state energy
```

The quantum computer doesn't solve the whole problem. It just measures the energy at your current position. The classical computer decides which direction to step next.

**Two computers. One problem. Neither could do it alone.**

---

## Why Does This Matter?

Every molecule in the universe has a ground state — its lowest possible energy configuration.

```
If you know the ground state of a molecule:
  → you know how it behaves chemically
  → you know if a drug will bind to a protein
  → you know if a material will conduct electricity
  → you know reaction rates, stability, properties
```

Classical computers can simulate small molecules. For larger ones — the memory required grows as 2^N, where N is the number of electrons. That number becomes impossible fast.

```
10 electrons  →       1,024 states
20 electrons  →   1,048,576 states
50 electrons  →  too large to store classically
```

VQE sidesteps this. Instead of storing every state, it uses N qubits for N electrons and searches for the minimum energy directly.

---

## The Three Moving Parts

### 1 · Ansatz — The Quantum Circuit

The ansatz is a parameterized quantum circuit. Think of it as a machine with knobs.

Each knob is an angle θ. Turning the knobs changes the quantum state the circuit prepares.

```
RY(θ)|0⟩  →  rotates the qubit by angle θ on the Bloch Sphere

θ = 0    →  |0⟩  north pole    energy = +1.0
θ = π/2  →  |+⟩  equator       energy =  0.0
θ = π    →  |1⟩  south pole    energy = -1.0  ← ground state
```

We don't know which θ gives the minimum. That's what we're finding.

---

### 2 · Energy Measurement — The Expectation Value

Once the circuit prepares a state, we measure its energy.

For our Hamiltonian H = Z (Pauli-Z operator):

```
Energy = probability(measuring |0⟩) - probability(measuring |1⟩)

Qubit mostly |0⟩  →  energy close to +1.0
Qubit mostly |1⟩  →  energy close to -1.0
```

We run 1000 shots and count how often we get 0 vs 1. That ratio gives the energy at this θ.

---

### 3 · Classical Optimizer — COBYLA

COBYLA is a classical optimization algorithm — the same family as gradient descent in machine learning.

It receives the energy value from the quantum computer and decides how to update θ to reduce it.

```
Iteration  1:  θ = 2.990  →  energy = -0.992
Iteration  2:  θ adjusted →  energy shifts
...
Iteration 14:  θ = 3.177  →  energy = -1.0000  ← converged!
```

14 iterations. Exact ground state. Done.

---

## The VQE Loop

```
  Initialize: random θ
        │
        ▼
  Quantum circuit prepares |ψ(θ)⟩
        │
        ▼
  Measure energy E = ⟨ψ|H|ψ⟩
        │
        ▼
  COBYLA receives E
        │
        ▼
  Update θ to reduce E ──────────────── (repeat)
        │
        ▼  (when E stops decreasing)

  Ground state found!
```

---

## Experimental Results

```
Starting point (random):
  θ = 2.990 radians
  E = -0.992

After optimization (14 iterations):
  θ = 3.177 radians  (≈ π = 3.1416)
  E = -1.0000        ← exact ground state

Converged: True
```

The optimizer found θ ≈ π — which rotates the qubit to the south pole |1⟩. That's the state with minimum energy for H = Z. The physics and the math agree completely.

---

## Convergence Curve

```
Energy vs Iteration:

 0.0  │
      │  ╲
-0.5  │   ╲      ╱╲
      │    ╲    ╱   ╲
-1.0  │─────╲──╱─────╲────────────────● ← converged
      │      ╲╱       ╲______________/
      └──────────────────────────────────
      0       4        8       12     14
                    Iteration
```

The curve drops, oscillates briefly as the optimizer explores, then locks onto -1.0. That oscillation isn't failure — it's the optimizer probing the landscape before committing to a direction.

---

## What This Connects To

| Today (simple) | Real quantum chemistry |
|----------------|----------------------|
| 1 qubit · H = Z | Dozens of qubits · molecular Hamiltonian |
| RY ansatz | UCCSD ansatz (chemistry-inspired) |
| COBYLA | SPSA · Adam · gradient-based methods |
| Energy = -1.0 | Ground state of H₂ · LiH · BeH₂ |
| 14 iterations | Hundreds of iterations |

The structure is identical. The scale is different.

---

## Tech Stack

```python
qiskit          >= 1.0.0    # quantum circuit construction
qiskit-aer      >= 0.17.2   # local simulation
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

## A Note on Where This Is Going

VQE is one half of the variational approach to quantum computing.

Day 5 covers QAOA — the same idea applied to optimisation problems rather than chemistry. Different problem. Same loop. Same intuition.

By Day 7, both run on real IBM quantum hardware — not a simulator. Real noise. Real qubits. Real results.

---

<div align="center">

[![GitHub](https://img.shields.io/badge/Akhila707-181717?style=flat-square&logo=github)](https://github.com/Akhila707)
&nbsp;·&nbsp;
[![IBM Quantum](https://img.shields.io/badge/IBM%20Quantum-052FAD?style=flat-square&logo=ibm&logoColor=white)](https://quantum.ibm.com)

</div>
