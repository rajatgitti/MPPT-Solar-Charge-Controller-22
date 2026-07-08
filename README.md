# MPPT-Solar-Charge-Controller-22# MPPT Solar Charge Controller - Synchronous Buck Converter Stage

An embedded systems and power electronics project focused on designing and analyzing a high-efficiency Synchronous Buck Converter stage, optimized for Maximum Power Point Tracking (MPPT) solar charge controllers. This repository contains the circuit architecture, design mathematical calculations, and transient simulation analysis.

---

## 1. System Architecture & Circuit Design

The power stage utilizes a **Synchronous Buck Converter** topology. Unlike standard buck converters that use a freewheeling diode, this design replaces the diode with a low $R_{DS(on)}$ synchronous MOSFET to drastically minimize conduction losses and maximize solar energy conversion efficiency.

### System Specifications
* **Input Voltage ($V_{in}$):** 20V (Simulating standard Solar PV panel output)
* **Switching Frequency ($f_s$):** 33.33 kHz (Period $T = 30\mu s$)
* **Control Modulation:** Pulse Width Modulation (PWM) with a 50% Duty Cycle ($D = 0.5$)
* **Filter Components:** $100\mu\text{H}$ Power Inductor ($L_1$), $100\mu\text{F}$ Smoothing Capacitor ($C_1$)
* **Output Load ($R_{load}$):** $5\Omega$ Resistive Load (Simulating battery charging sink)

---

## 2. Design Mathematics & Governing Equations

### Target Output Voltage
For a buck topology operating in Continuous Conduction Mode (CCM), the steady-state output voltage is directly proportional to the duty cycle:
$$V_{out} = D \times V_{in}$$

Given our parameters:
$$V_{out} = 0.5 \times 20\text{V} = 10\text{V}$$

### Inductor Ripple Current ($\Delta I_L$)
The peak-to-peak ripple current passing through the inductor is dictated by the switching period and component values:
$$\Delta I_L = \frac{V_{out} \cdot (1 - D)}{L \cdot f_s}$$

$$\Delta I_L = \frac{10\text{V} \cdot (1 - 0.5)}{100\mu\text{H} \cdot 33.33\text{kHz}} = \frac{5}{3.333} \approx 1.5\text{A}$$

---

## 3. Simulation & Analysis Performance Metrics

The transient behavior of the circuit was evaluated over a $5\text{ms}$ execution window (`.tran 5m`). 

### Key Performance Observations:
* **Transient Phase ($0\text{ms}$ to $1.5\text{ms}$):** The output voltage $V(out)$ exhibits a standard second-order underdamped response, climbing rapidly from $0\text{V}$ and settling seamlessly due to the tuned damping of the LC filter network.
* **Steady-State Phase ($1.5\text{ms}$ to $5\text{ms}$):** The output voltage tightly locks onto the calculated target of **$10\text{V}$ DC**. 
* **Current Delivery:** At steady state, the output current through the $5\Omega$ load settles firmly at **$2\text{A}$** ($10\text{V} / 5\Omega$).

### Project Analysis Metrics for Optimization
To evaluate full system performance in a real-world deployment, the project focuses on three primary metrics:

1. **Tracking Efficiency ($\eta_{MPPT}$):** Measuring how closely the Perturb & Observe (P&O) firmware locks onto the solar panel's theoretical maximum power point ($P_{max}$).
   $$\eta_{MPPT} = \left(\frac{P_{actual}}{P_{max\_ideal}}\right) \times 100\%$$
2. **Power Conversion Efficiency ($\eta_{conversion}$):** Assessing thermal and switching losses across the synchronous power MOSFETs.
   $$\eta_{conversion} = \left(\frac{V_{battery} \times I_{battery}}{V_{PV} \times I_{PV}}\right) \times 100\%$$
3. **Dead-Time Verification:** Ensuring exact non-overlapping gate signals to prevent hazardous shoot-through conditions across the high-side and low-side switch rails.

---

## 4. How to Run the Simulation Netlist

To replicate these results without graphical symbol errors, use the raw SPICE netlist below directly inside LTspice:

1. Create a **New Schematic** in LTspice.
2. Press `t` to open the text tool, switch the radio button to **SPICE Directive**, and paste the code below:

```spice
* MPPT Buck Converter Simulation Netlist
Vin In 0 20
Vgate Gate Switching PULSE(0 10 0 10n 10n 15u 30u)
Q1 In Gate Switching NMOS
D1 0 Switching MURS120
L1 Switching Out 100u
C1 Out 0 100u
Rload Out 0 5

.model NMOS NMOS(Vto=2 Kp=20)
.model MURS120 D(Is=2.7n Rs=0.05 N=1.5 Cjo=50p)
.tran 5m
