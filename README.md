# Quasi-2D Electro-Thermal Poisson-Schrödinger Solver for AlGaN/GaN HEMTs

AlGaN/GaN High Electron Mobility Transistors (HEMTs) drive the next generation of 5G/6G base stations, military radar, and high-density EV chargers. AlGaN/GaN HEMTs solve silicon MOSFET bottlenecks when handling high power and high frequencies simultaneously.

A regular MOSFET relies on heavily doping the silicon to supply free electrons to conduct current. Meanwhile, for AlGaN/GaN HEMTs, the polarization at the AlGaN/GaN interface pulls free electrons into a quantum well. With no dopant atoms there to scatter them, the electrons move incredibly fast.

However, simulating this device accurately is difficult. Standard silicon MOSFET classical drift-diffusion models break down at the AlGaN/GaN heterojunction because they ignore quantum confinement and the extreme self-heating that happens at high power. 

I built this solver to simulate the physics involved in this device using the NumPy, Matplotlib, and SciPy Python libraries. It couples quantum electrostatics with non-linear heat transport to show the relationship between different variables and device performance.

---

## The Physics

### 1. Polarization & 2DEG Generation
* GaN naturally has spontaneous polarization. When you grow a thin AlGaN barrier on top, the lattice mismatch stretches the AlGaN, adding piezoelectric polarization:

$$P_{\text{PE}} = 2 \eta_a \left( e_{31} - e_{33} \frac{C_{13}}{C_{33}} \right)$$

* The steep drop in total polarization at this junction creates a massive sheet of positive bound charge. This bends the conduction band hard enough to pull free electrons into a deep triangular quantum well, forming a dense Two-Dimensional Electron Gas (2DEG).

### 2. Quantum Confinement & Pauli Blocking
* This potential well is only a few nanometers wide, which is on the same scale as the de Broglie wavelength of an electron. Classical physics fails here. Electrons cannot be treated as classical point charges anymore; they instead act as waves restricted to discrete energy subbands, governed by the 1D Schrödinger equation:

$$\left[ -\frac{\hbar^2}{2} \frac{\partial}{\partial z} \left( \frac{1}{m^*(z)} \frac{\partial}{\partial z} \right) + V_{\text{eff}}(z) \right] \psi_m(z) = E_m \psi_m(z)$$

* According to Pauli's Exclusion Principle, as the ground state fills up, quantum crowding forces new electrons into higher energy subbands, pushing the charge distribution deeper into the bulk GaN.

### 3. High-Field Transport & Self-Heating
* Under a high drain bias, electrons accelerate laterally across the channel until they hit a speed limit near the drain edge. When the electric field spikes and electrons gain massive kinetic energy, they shed that energy by emitting optical phonons instead of accelerating, causing intense lattice vibrations. This creates a localized hotspot governed by Joule heating:

$$\nabla \cdot (\kappa \nabla T) = - (\mathbf{J} \cdot \mathbf{E})$$

* The electro-thermal feedback loop works as follows: extreme heat increases phonon scattering, which degrades electron mobility and results in current saturation.

### 4. Deep Buffer Traps & Leakage
* Real GaN crystals have background defects that release stray electrons, causing off-state leakage. Deep acceptor traps, like Carbon, are added deep in the buffer. These traps catch stray electrons based on Fermi-Dirac statistics:

$$f_A = \frac{1}{1 + g_A \exp\left(\frac{E_A - E_{Fn}}{k_B T}\right)}$$

* Therefore, the Fermi level is anchored mid-gap, turning the bulk buffer into an insulator, so current only flows through the 2DEG.

---

## Inside the Repository

I separated the simulations into distinct scripts to isolate the different physical phenomena that dictate AlGaN/GaN HEMT performance and reliability in the industry:

* single_bias.ipynb (Single Bias Point): This is the core physics engine. Before running massive sweeps, this script displays the device behavior at one operating point to capture the spatial distribution of the 2DEG, the quantum subband populations, and the drift velocity scattered along with the localized hotspot.

* transfer_curve.ipynb (Transfer & Transconductance): Sweeps gate voltages to monitor the transconductance to find the sweet spot of gate control before self-heating effects kick in.

* output_curve.ipynb (Output Characteristics): Sweeps drain-to-source voltages to demonstrate Negative Differential Resistance (NDR), where extreme Joule heating causes current to drop.

* fT_fmax.ipynb (High-Frequency RF Limits): Displays RF switching speeds. It tracks how parasitic capacitances and intrinsic transit delays negatively influence high-frequency operation, finding the exact gate bias where current gain and power gain peak before AC small-signal performance drops off.

* DIBL_SCE.ipynb (Scaling Limits): Demonstrates the effects of scaling down gate lengths in hopes of boosting speed and density. In reality, a short gate loses electrostatic control, letting the drain's electric field prematurely pull the threshold voltage down (Drain-Induced Barrier Lowering).

* offstate_breakdown.ipynb (Off-State Reliability): When a HEMT is off but blocking high voltage, extreme electric field crowding happens at the gate edge. This script looks for the exact voltage where impact ionization triggers an avalanche breakdown of electron-hole pairs, destroying the device's blocking capability.

---

## Setup & Running

```bash
pip install -r requirements.txt
jupyter notebook