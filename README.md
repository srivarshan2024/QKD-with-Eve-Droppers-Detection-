# Professional BB84 Quantum Key Distribution Simulator with Eavesdropper Detection and Simplified Grain Cipher Integration

[![Python 3.12](https://img.shields.io/badge/python-3.12-blue.svg)](https://www.python.org/downloads/)
[![Qiskit](https://img.shields.io/badge/quantum-Qiskit%202.x-613394.svg)](https://qiskit.org/)
[![Tests](https://img.shields.io/badge/tests-34%20passed%20(96%25%20coverage)-brightgreen.svg)](tests/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

An academic, scientifically rigorous, and interactive **BB84 Quantum Key Distribution (QKD) Simulator** built for university research projects, quantum cryptography coursework, and IEEE-style research papers. The application demonstrates single-photon state preparation, transmission over a quantum channel, random-basis intercept-and-resend eavesdropping attacks, classical public basis sifting, quantum bit error rate (QBER) analysis against the Shor-Preskill asymptotic abort bound (~11%), key gating, and educational encryption using the eSTREAM Grain stream cipher architecture.

---

## 1. System Architecture & Project Structure

The project maintains a modular, decoupled architecture where quantum physics, protocol steps, attack models, security thresholds, and cryptographic algorithms are isolated in testable Python modules:

```
Innovation/
├── app.py                      # Main Streamlit Academic Dashboard UI
├── requirements.txt            # System dependencies
├── README.md                   # Academic documentation and execution guide
├── src/
│   ├── __init__.py             # Package descriptor
│   ├── quantum_channel.py      # State vectors, Born rule measurement, and Qiskit circuits
│   ├── bb84.py                 # Alice/Bob preparation, measurement, and basis reconciliation
│   ├── eavesdropper.py         # Intercept-and-resend attack simulation and theoretical formulas
│   ├── qber.py                 # Error calculation, sample disclosure, and standard error bounds
│   ├── security_decision.py    # Threshold evaluation, Shor-Preskill bound, and key gating
│   ├── grain_cipher.py         # Educational 32-bit LFSR/NFSR Grain stream cipher integration
│   └── experiment_runner.py    # Canonical scenario benchmarks, Monte Carlo sweeps, and CSV/JSON export
└── tests/
    ├── test_bb84.py            # Tests for state normalization, reconciliation, reproducibility
    ├── test_eavesdropper.py    # Tests for Eve disabled/enabled, statistical convergence to 25%
    ├── test_qber.py            # Tests for QBER computation, division-by-zero protection, disclosure
    ├── test_security_decision.py # Tests for acceptance, rejection, and insufficient key material
    ├── test_grain_cipher.py    # Tests for keystream determinism, KDF, and round-trip recovery
    ├── test_experiment_runner.py # Tests for 3 scenarios, parametric sweeps, and export validity
    └── test_app.py             # Streamlit AppTest integration tests verifying UI rendering
```

---

## 2. Theoretical Background & Mathematical Formulation

### A. BB84 Quantum Key Distribution Protocol
Proposed by Charles Bennett and Gilles Brassard in 1984, BB84 leverages quantum mechanical principles—specifically the **No-Cloning Theorem** and the **Heisenberg Uncertainty Principle**—to establish an unconditionally secure key between two authenticated parties:

1. **Conjugate Bases:**
   - **Computational Basis ($Z$ / Rectilinear $+$):**
     $$\{|0\rangle, |1\rangle\}, \quad |0\rangle = \begin{pmatrix} 1 \\ 0 \end{pmatrix}, \quad |1\rangle = \begin{pmatrix} 0 \\ 1 \end{pmatrix}$$
   - **Hadamard Basis ($X$ / Diagonal $\times$):**
     $$\{|+\rangle, |-\rangle\}, \quad |+\rangle = \frac{|0\rangle + |1\rangle}{\sqrt{2}}, \quad |-\rangle = \frac{|0\rangle - |1\rangle}{\sqrt{2}}$$

2. **Alice's Preparation:**
   Alice generates a random bit $a_i \in \{0, 1\}$ and independently selects an encoding basis $B_{A,i} \in \{Z, X\}$.
   - If $a_i = 0, B_{A,i} = Z \implies |0\rangle$
   - If $a_i = 1, B_{A,i} = Z \implies |1\rangle$
   - If $a_i = 0, B_{A,i} = X \implies |+\rangle$
   - If $a_i = 1, B_{A,i} = X \implies |-\rangle$

3. **Bob's Independent Measurement:**
   Bob chooses a measurement basis $B_{B,i} \in \{Z, X\}$ uniformly at random ($P(Z)=0.5, P(X)=0.5$) and measures the incoming quantum state according to the **Born Rule**:
   $$P(b_i) = |\langle b_i | \psi \rangle|^2$$
   - When $B_{B,i} = B_{A,i}$ (and no eavesdropper intervened), Bob measures Alice's bit with $100\%$ fidelity ($P(b_i = a_i) = 1$).
   - When $B_{B,i} \neq B_{A,i}$, Bob measures $0$ or $1$ with equal probability ($P=0.5$).

4. **Public Basis Sifting:**
   Alice and Bob publish their basis choices $B_A, B_B$ over an authenticated classical channel.
   - Positions where $B_{A,i} = B_{B,i}$ are retained to form the **Sifted Key**.
   - Positions where $B_{A,i} \neq B_{B,i}$ are discarded.
   - Theoretical expected sifted key fraction: $\lim_{N \to \infty} \frac{|K_{\text{sifted}}|}{N} = 50\%$.

---

### B. Intercept-and-Resend Attack Model
Eve intercepts a fraction $\rho \in [0.0, 1.0]$ of transmitted qubits. Because Eve cannot clone unknown quantum states, she must choose a measurement basis $B_{E,i} \in \{Z, X\}$ without knowing Alice's choice.

Consider a qubit where Alice and Bob coincidentally chose the same basis ($B_A = B_B$):
- **Scenario 1: Eve guesses correctly ($B_E = B_A$):**
  Probability $P(B_E = B_A) = 0.5$. Eve measures the exact bit Alice prepared, collapses the state into that eigenstate, and resends it. Bob measures in $B_B = B_A$ and receives Alice's bit. Error probability = $0\%$.
- **Scenario 2: Eve guesses incorrectly ($B_E \neq B_A$):**
  Probability $P(B_E \neq B_A) = 0.5$. Eve measures in the conjugate basis and collapses the state into her basis eigenstate. When Bob measures in $B_B = B_A$, the probability of Bob obtaining the wrong bit is $|\langle - | 0 \rangle|^2 = 0.5$. Error probability = $50\%$.

Total conditional error probability given an intercepted qubit where bases match:
$$P(\text{error} \mid \text{intercept}, B_A = B_B) = (0.5 \times 0.0) + (0.5 \times 0.5) = 0.25 = 25\%$$

Since interception occurs independently with probability $\rho$:
$$\mathbb{E}[\text{QBER}] = 0.25 \times \rho \times 100\%$$

- When $\rho = 0.0$ (No Eve): $\mathbb{E}[\text{QBER}] = 0\%$
- When $\rho = 0.50$ (50% Interception): $\mathbb{E}[\text{QBER}] = 12.5\%$
- When $\rho = 1.0$ (100% Full Interception): $\mathbb{E}[\text{QBER}] = 25.0\%$

---

### C. Quantum Bit Error Rate (QBER) and Security Decision

$$\text{QBER} = \left( \frac{\sum_{i \in \text{compared}} (a_i \oplus b_i)}{N_{\text{compared}}} \right) \times 100\%$$

- **Finite Sample Standard Error (SE):**
  $$\text{SE} = \sqrt{\frac{p(1-p)}{N_{\text{compared}}}} \times 100\%$$
- **Shor-Preskill Asymptotic Abort Bound:**
  In one-way classical post-processing (error correction and privacy amplification), the secret key extraction rate is bounded by:
  $$R \ge 1 - 2 \cdot H_2(\text{QBER})$$
  where $H_2(p) = -p \log_2(p) - (1-p) \log_2(1-p)$ is the binary Shannon entropy.
  When $\text{QBER} > 11.0\%$, $1 - 2 H_2(0.11) \le 0$, meaning it is information-theoretically impossible to eliminate Eve's mutual information without eliminating all secret key bits.
- **Protocol Decision:**
  - $\text{QBER} \le \tau$: **ACCEPTED** &rarr; Key is retained for cryptographic seeding.
  - $\text{QBER} > \tau$: **REJECTED** &rarr; Session aborted, key discarded.

---

### D. Educational Simplified Grain Stream Cipher Integration
The **Grain stream cipher** family (Hell, Johansson, & Meier, eSTREAM project) is designed for hardware efficiency, combining two 32-bit registers:
1. **Linear Feedback Shift Register (LFSR):**
   $$f(s) = s_0 \oplus s_5 \oplus s_{13} \oplus s_{23} \oplus s_{31}$$
2. **Non-Linear Feedback Shift Register (NFSR):**
   $$g(b) = b_0 \oplus b_9 \oplus b_{14} \oplus b_{21} \oplus b_{28} \oplus (b_3 \cdot b_{15}) \oplus (b_7 \cdot b_{25}) \oplus (b_{12} \cdot b_{18} \cdot b_{27})$$
3. **Non-linear Output Filter $h(s, b)$:**
   $$h(s, b) = (s_3 \cdot b_{12}) \oplus (s_{13} \cdot s_{19}) \oplus (b_4 \cdot b_{23}) \oplus (s_7 \cdot b_9 \cdot s_{25})$$
4. **Keystream Output Bit $z_t$:**
   $$z_t = b_1 \oplus b_{17} \oplus s_0 \oplus s_{24} \oplus h(s, b)$$
5. **Warm-up Phase:**
   Clocked for 64 rounds feeding $z_t$ back into both registers to diffuse Key and IV before keystream generation.
6. **Key Gating:**
   The cipher is only accessible when the QKD session achieves **ACCEPTED** status.

---

## 3. Installation & Quick Start

### Prerequisites
- Python 3.10, 3.11, or 3.12
- `pip` package manager

### Installation Steps
```bash
# 1. Clone or navigate to the repository
cd Innovation

# 2. Install dependencies
pip install -r requirements.txt

# 3. Run the automated test suite
pytest
```

### Launching the Streamlit Application
```bash
streamlit run app.py
```
Open your browser at `http://localhost:8501`.

---

## 4. Navigation & Feature Walkthrough

The web dashboard is organized into 6 focused sections:

1. **1. Overview:** Academic abstract, core quantum principles, interactive transmission diagram, and live session status.
2. **2. BB84 Simulation:** Real-time generation of Alice's bits and bases, Bob's measurements, classical basis sifting table, and an interactive **Qiskit Quantum Circuit diagram** generator for any selected qubit.
3. **3. Eavesdropper Analysis:** Full analysis of Eve's intercept-and-resend attack, theoretical error calculations, and an interactive filtered trace of intercepted qubits showing the exact collapsed states.
4. **4. QBER & Security:** Side-by-side metric tiles (Measured QBER, Threshold $\tau$, Error Count, Standard Error), Shor-Preskill bound discussion, and formal protocol acceptance/rejection verdict banner.
5. **5. Grain Cipher Demo:** Architectural breakdown of LFSR, NFSR, and non-linear filter $h(s,b)$, key derivation function (KDF) from QKD bits, and live encryption/decryption with round-trip verification.
6. **6. Experiment Results:** Three-scenario comparative bar charts (No Eve, Partial Eve, Full Eve), Monte Carlo parametric sweeps across $\rho \in [0.0, 1.0]$ with 95% confidence intervals, and one-click **CSV and JSON data export**.

---

## 5. Automated Testing Suite

The repository contains 34 automated unit and integration tests covering all critical scientific invariants:

```bash
# Run all tests with coverage report
python -m pytest --cov=src tests/
```

### Key Test Validations:
- `test_bb84.py`: State vector normalization, orthogonal projection, 50% sifted key convergence, zero errors on clean channels, reproducibility with fixed seed.
- `test_eavesdropper.py`: Eve disabled behavior, eigenstate resending, theoretical formula verification, convergence to 25% error on full intercept.
- `test_qber.py`: Exact QBER computation, zero-division safety on empty sifted keys, sample disclosure partitioning.
- `test_security_decision.py`: Acceptance/rejection boundaries, insufficient key handling.
- `test_grain_cipher.py`: Deterministic keystream generation, KDF expansion from short/long keys, round-trip ASCII and Unicode encryption/decryption recovery.
- `test_experiment_runner.py`: Execution of canonical 3 scenarios, parametric sweep aggregation, CSV/JSON serialization.
- `test_app.py`: Streamlit AppTest headless execution verifying all 6 tabs render cleanly without unhandled exceptions.

## 6. Publication Benchmark Suite & Generated Datasets

An automated experimental benchmark suite (`run_benchmarks.py`) executes 525+ quantum transmission trials across varying eavesdropping regimes and produces 300 DPI publication figures:

```bash
# Run the complete experimental benchmark suite
python run_benchmarks.py
```

### Generated Artifacts:
- **`experiments/canonical_scenarios.csv` & `.json`:** Three canonical scenarios ($N = 1000$ qubits): Clean Channel ($\rho=0.0$), Partial Intercept ($\rho=0.50$), Full Intercept ($\rho=1.00$).
- **`experiments/parametric_sweep_rho.csv` & `.json`:** 21-point parametric sweep across $\rho \in [0.0, 1.0]$ with 25 Monte Carlo trials per step, measuring Mean QBER, Standard Deviation, and 95% Confidence Intervals.
- **`experiments/finite_key_scaling.csv` & `.json`:** Scaling of sifted key ratio and standard error across $N \in [50, 5000]$ qubits.
- **`experiments/grain_diffusion_metrics.json`:** Statistical balance (49.29% ones) and Strict Avalanche Criterion (SAC = 49.90%) across 10,000 keystream bits.
- **`figures/fig1_scenarios_comparison.png`:** Bar chart comparing measured QBER across scenarios against the 11.0% abort threshold.
- **`figures/fig2_qber_vs_interception_probability.png`:** Empirical QBER convergence vs theoretical expectation ($0.25 \times \rho$).
- **`figures/fig3_finite_sample_scaling.png`:** Sifted key ratio convergence and standard error reduction proportional to $1/\sqrt{N}$.
- **`figures/fig4_grain_diffusion_analysis.png`:** Strict Avalanche Criterion distribution across all 32 key bits.

---

## 7. Scientific Limitations & Assumptions

For academic integrity and IEEE publication standards, users should note the following assumptions:
1. **Idealized Detectors:** The baseline simulator assumes photon detectors have 100% efficiency and zero dark counts. In physical fiber QKD systems, dark counts and channel attenuation introduce a baseline background QBER ($\sim 1\text{--}3\%$).
2. **Threshold Security:** Passing the QBER threshold ($\text{QBER} \le \tau$) is an educational prerequisite, not unconditional mathematical proof of security for finite key lengths. Real systems apply finite-key security parameter bounds $\epsilon_{\text{sec}}$.
3. **Simplified Grain Cipher:** The integrated stream cipher implements an educational 32-bit register model adhering to Grain's structural principles. Production systems use 80-bit Grain v1 or 128-bit Grain-128a.

---

## 7. Citation & References

- C. H. Bennett and G. Brassard, *"Quantum cryptography: Public key distribution and coin tossing,"* in Proceedings of IEEE International Conference on Computers, Systems and Signal Processing, 1984, pp. 175–179.
- P. W. Shor and J. Preskill, *"Simple proof of security of the BB84 quantum key distribution protocol,"* Physical Review Letters, vol. 85, no. 2, p. 441, 2000.
- M. Hell, T. Johansson, and W. Meier, *"Grain - A stream cipher for constrained environments,"* International Journal of Wireless and Mobile Computing, 2007.
- W. K. Wootters and W. H. Zurek, *"A single quantum cannot be cloned,"* Nature, vol. 299, pp. 802–803, 1982.

---

## License
MIT License. Free for academic research, education, and university course demonstrations.
