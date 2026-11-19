# PSO-Tuned MACA and MACAW Protocols

**Network and Design Organisation (NDO) – 7th Semester Project**  
**Group 22**  
- **Jay Hirapara**  
- **Darshit Khandelwal**  
- **Kundan Meena**  

---

## 📌 Project Overview

### Intuitive Analogy: The Crowded Room Problem
Imagine a crowded room where 50 people want to speak. If everyone talks at once, they talk over each other (**collision**). To avoid this:
1. People raise their hands and request permission before speaking (**RTS / CTS** handshake in MACA).
2. If two people request at the exact same moment and collide, both pause and wait a random moment before trying again (**Backoff**).
3. If the wait is too brief, they collide immediately again. If the wait is too long, the channel sits idle with wasted silence.

**The Problem:** Traditional MAC protocols used hardcoded waiting rules chosen decades ago. While these rules work adequately for small groups of 3 nodes, they collapse in dense networks: up to **80% of transmission attempts collide** at 50 nodes, severely choking channel capacity.

### Protocol Mechanics
In wireless MAC protocols:
- **MACA** (*Multiple Access with Collision Avoidance*) reserves the channel via Request-to-Send / Clear-to-Send (RTS/CTS) control frames to resolve hidden-terminal contention.
- **MACAW** (*MACA for Wireless LANs*) enhances MACA by adding a receiver acknowledgement (ACK) and a MILD (*Multiplicative Increase, Linear Decrease*) backoff algorithm for smoother adaptation.
- Channel contention is governed by 4 continuous parameters:
  - $CW_{min}$: Minimum contention window (shortest wait after success).
  - $CW_{max}$: Maximum contention window (upper limit cap after repeated collisions).
  - $f$ (Growth Factor): Multiplier expanding the contention window upon a collision.
  - $d$ (Decrease Step): Reduction applied to the contention window after a successful frame.

We employ **Particle Swarm Optimization (PSO)** to discover the optimal parameter tuple $(CW_{min}, CW_{max}, f, d)$ that maximizes throughput and fairness while minimizing collisions.

---

## 🎯 The Fitness Function (How PSO Grades a Backoff Setting)

The **Fitness Function** serves as the objective scoring rubric that PSO evaluates during network simulations:

$$\text{Fitness Score} = (\text{Throughput} \times \text{Fairness}) - (0.1 \times \text{CollisionRate})$$

### 1. Throughput (Network Speed)
- Measures how much useful data actually gets delivered across channel time slots.
- Higher throughput means faster and more efficient data delivery.

### 2. Fairness (Equal Sharing)
- Ensures all competing nodes get an equal opportunity to transmit.
- Calculated using Jain's Fairness Index ($\mathcal{J} \in [0, 1]$), preventing aggressive nodes from starving quieter nodes:
  $$\mathcal{J} = \frac{\left(\sum_{i=1}^n x_i\right)^2}{n \sum_{i=1}^n x_i^2}$$

### 3. Collisions (Wasted Retries)
- Ratio of collision slots to total transmission attempts. Lower is better.
- Fewer collisions saves node energy and eliminates wasted airtime.

### 💡 Why is the Fitness Function structured this way?
1. **Why multiply $(\text{Throughput} \times \text{Fairness})$?**  
   If we only maximized throughput, an optimizer could cheat by letting a single node transmit 100% of the time while 49 nodes starve. Multiplying by fairness guarantees the network must achieve **both high speed AND equal sharing**.
2. **Why subtract $(0.1 \times \text{CollisionRate})$?**  
   Acts as a gentle regularization penalty to discourage reckless retries while keeping throughput delivery the primary goal.

---

## ⚙️ PSO Optimization Framework

### Swarm Intelligence
Inspired by bird flocks or bee swarms searching for food. No individual particle knows the exact optimal configuration alone, but by continuously sharing personal and group discoveries, the entire swarm rapidly converges on the optimal backoff parameters.

### Mathematical Formulation
Each particle updates its velocity $v$ and position $x$ across iterations:
$$v_{t+1} = w \cdot v_t + c_1 r_1 (p_{best} - x_t) + c_2 r_2 (g_{best} - x_t)$$
$$x_{t+1} = x_t + v_{t+1}$$

- **Inertia ($w = 0.7$):** Momentum keeping the particle moving in its current direction.
- **Cognitive Weight ($c_1 = 1.5$):** Memory pulling the particle toward its own historical best.
- **Social Weight ($c_2 = 1.5$):** Swarm wisdom pulling the particle toward the group's global best.
- **Stability Condition:** $c_1 + c_2 = 3.0 < 3.5$, strictly satisfying Clerc & Kennedy's stability theorem to prevent swarm explosion/divergence.
- **Search Budget:** 16 particles $\times$ 18 iterations = 288 full simulation evaluations per protocol.
- **Training Set:** Networks of 2, 8, 20, and 40 nodes.
- **Validation Set:** Unseen sizes (3, 5, 10, 20, 30, 50 nodes) across independent random seeds.

---

## 🔬 Optimized Parameters

| Protocol | $CW_{min}$ | $CW_{max}$ | Growth Factor ($f$) | Shrink Step ($d$) |
| :--- | :---: | :---: | :---: | :---: |
| **Default (Standard BEB)** | 4 | 32 | 2.00 | 1.00 |
| **MACA (PSO-Tuned)** | **3** | **36** | **2.47** | **4.00** |
| **MACAW (PSO-Tuned)** | **7** | **52** | **1.35** | **1.27** |

> **Key Discovery:** In crowded environments, the default $CW_{max}=32$ is too cramped, forcing nodes to retry too close to each other. PSO widened $CW_{max}$ to 36 and 52, spreading out retry intervals. For MACAW, PSO selected a gentler growth factor ($1.35$ vs $2.00$), matching the MILD philosophy to avoid extreme channel latency.

---

## 📊 Experimental Results

### 1. Performance at 50 Nodes (Heavy Congestion)

| Metric | MACA Default | MACA PSO | Δ (Change) | MACAW Default | MACAW PSO | Δ (Change) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Throughput** | 14.8% | **18.5%** | **+25.0%** | 13.0% | **21.9%** | **+68.5%** |
| **Collision Rate** | 80.4% | **75.1%** | **-5.3%** | 70.1% | **44.9%** | **-25.2%** |
| **Fairness (Jain's)** | 0.888 | **0.914** | **+2.9%** | 0.887 | **0.941** | **+6.1%** |

### 2. Comprehensive Scaling Across Network Sizes

#### MACA Results
| Nodes | Default Thr | PSO Thr | Δ Thr (%) | Default Coll | PSO Coll | Default Fair | PSO Fair |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **3** | 0.3200 | 0.3050 | -4.7% | 0.0350 | 0.0450 | 0.9650 | 0.9450 |
| **5** | 0.3000 | 0.2980 | -0.7% | 0.0750 | 0.0820 | 0.9850 | 0.9650 |
| **10** | 0.2650 | 0.2880 | +8.7% | 0.1850 | 0.1700 | 0.9880 | 0.9800 |
| **20** | 0.2250 | 0.2600 | +15.6% | 0.3800 | 0.3250 | 0.9750 | 0.9820 |
| **30** | 0.1850 | 0.2250 | +21.6% | 0.5850 | 0.4900 | 0.9600 | 0.9700 |
| **50** | 0.1480 | 0.1850 | **+25.0%** | 0.8040 | 0.7510 | 0.8880 | 0.9140 |

#### MACAW Results
| Nodes | Default Thr | PSO Thr | Δ Thr (%) | Default Coll | PSO Coll | Default Fair | PSO Fair |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **3** | 0.3100 | 0.2950 | -4.8% | 0.0380 | 0.0420 | 0.9600 | 0.9400 |
| **5** | 0.2850 | 0.2800 | -1.8% | 0.0650 | 0.0600 | 0.9800 | 0.9600 |
| **10** | 0.2400 | 0.2680 | +11.7% | 0.1450 | 0.1150 | 0.9850 | 0.9750 |
| **20** | 0.1950 | 0.2520 | +29.2% | 0.2950 | 0.2100 | 0.9700 | 0.9800 |
| **30** | 0.1600 | 0.2350 | +46.9% | 0.4650 | 0.3150 | 0.9500 | 0.9750 |
| **50** | 0.1300 | 0.2190 | **+68.5%** | 0.7010 | 0.4490 | 0.8870 | 0.9410 |

---

## 📈 Visual Results

All simulation graphs are generated and exported automatically to `results.png`:

![Results Plot](results.png)

---

## 🚀 How to Run the Code

### Prerequisites
- Python 3.8+
- NumPy
- Matplotlib

Install required libraries:
```bash
pip install numpy matplotlib
```

### Execution
Run the full PSO optimization and evaluation script:
```bash
python pso_maca_macaw.py
```

---

## 📁 Repository Structure
```
.
├── pso_maca_macaw.py          # Core simulation & PSO optimizer
├── NDO_final.pptx             # Primary presentation slides (high contrast, clear explanations)
├── NDO_final_backup.pptx      # Original presentation backup
├── results.png                # Generated performance plots
├── README.md                  # Comprehensive project documentation
└── .gitignore                 # Git ignore rules
```

---

## 📚 References
1. Kennedy, J., & Eberhart, R. (1995). *Particle swarm optimization*. Proceedings of IEEE International Conference on Neural Networks (ICNN).
2. Shi, Y., & Eberhart, R. (1998). *A modified particle swarm optimizer*. IEEE International Conference on Evolutionary Computation.
3. Clerc, M., & Kennedy, J. (2002). *The particle swarm: explosion, stability, and convergence in a multidimensional complex space*. IEEE Transactions on Evolutionary Computation, 6(1), 58-73.
4. Poli, R. (2009). *Mean and variance of the sampling distribution of particle swarm optimizers during stagnation*. IEEE Transactions on Evolutionary Computation, 13(4), 712-721.
5. Bharghavan, V., Demers, A., Shenker, S., & Zhang, L. (1994). *MACAW: A media access protocol for wireless LANs*. ACM SIGCOMM Computer Communication Review, 24(4), 212-225.
6. Karn, P. (1990). *MACA - a new channel access method for packet radio*. ARRL/CRRL Amateur Radio 9th Computer Networking Conference.
7. Jain, R., Chiu, D. M., & Hawe, W. R. (1984). *A quantitative measure of fairness and discrimination for resource allocation in shared computer systems*. DEC Research Report TR-301.
8. Hirapara, J., Khandelwal, D., & Meena, K. (2026). *PSO-Tuned MACA and MACAW Implementation*. GitHub Repository: https://github.com/JayH25/NDO---MACA-MACAw-with-PSO-optimization
