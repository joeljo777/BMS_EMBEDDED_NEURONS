# BMS_EMBEDDED_NEURONS

[![MATLAB](https://img.shields.io/badge/MATLAB-R2026b-orange.svg)](https://www.mathworks.com/products/matlab.html)
[![Simulink](https://img.shields.io/badge/Simulink-MBD-blue.svg)](https://www.mathworks.com/products/simulink.html)
[![Chemistry](https://img.shields.io/badge/Chemistry-4S1P%20LiFePO4%20(15Ah)-green.svg)](#key-specifications)
[![Architecture](https://img.shields.io/badge/Architecture-Model--Based%20Design%20(MBD)-purple.svg)](#system-architecture)
[![Status](https://img.shields.io/badge/Status-Active%20Development-brightgreen.svg)](#current-project-status)

An Intelligent Battery Management System (BMS) with Advanced Fault Diagnosis and Adaptive Protection for Electric Mobility, combining **Model-Based Design (MBD)**, physics-informed battery modeling, and embedded intelligence.

---

## 📌 Overview

**BMS_EMBEDDED_NEURONS** bridges the gap between high-fidelity dynamic simulation and production-grade embedded microcontroller firmware. Starting from a **4S1P LiFePO₄ (15 Ah)** battery plant in MATLAB/Simulink, the project implements closed-loop current sensing, Coulomb-counting State of Charge (SOC) tracking, sensor error modeling, threshold-based overcurrent protection, and modular controller architectures destined for embedded firmware deployment ("Embedded Neurons").

---

## ⚡ Key Specifications

| Parameter | Specification | Description |
| :--- | :--- | :--- |
| **Cell Chemistry** | $\text{LiFePO}_4$ (Lithium Iron Phosphate) | Inherently stable, high thermal runaway margin |
| **Pack Topology** | **4S1P** (4 cells in series, 1 parallel) | Nominal ~12.8 V, Peak ~14.4 V - 14.6 V |
| **Nominal Capacity** | **15 Ah** (54,000 Coulombs) | Capacity base for discrete-time coulomb integration |
| **Initial SOC ($\text{SOC}_0$)** | **80%** (`0.8`) | Configured default initialization |
| **SOC Limits** | `[0.0, 1.0]` (0% - 100%) | Hard bounded by saturation blocks |
| **Simulation Solver** | Discrete (Fixed-Step) | Step size: $1\text{ s}$, Duration: $0 - 3600\text{ s}$ |
| **Simulation Environment** | MATLAB / Simulink R2026b | Dynamic piecewise step profiling |

---

## 🏗️ System Architecture

The project employs a modular Model-Based Design (MBD) architecture consisting of five core subsystems:

```
+-------------------------------------------------------------------------------+
|                             DYNAMIC LOAD PROFILE                              |
|   Step Current Profiles (t = 0s, 500s, 1000s, 1500s -> Charge/Discharge)      |
+---------------------------------------+---------------------------------------+
                                        |
                                        v
                       +---------------------------------+
                       |       Battery Plant (4S)        |
                       |  - LiFePO4 dynamic cell model   |
                       |  - Discrete Integrator (Coulomb)|
                       +----------------+----------------+
                                        |
       +--------------------------------+--------------------------------+
       |                                |                                |
       v                                v                                v
+--------------+               +------------------+             +-----------------+
| Sensor Model |               |  BMS Controller  |             | Protection Sys. |
| - Current    |               |  - SOC / SOH     |             | - OVP / UVP     |
| - Voltage    |               |  - Logic Control |             | - OCP / OTP     |
| - Temp (NTC) |               |  - Balancing     |             | - Fault Flag    |
+-------+------+               +--------+---------+             +--------+--------+
        |                               |                                |
        +-------------------------------+--------------------------------+
                                        |
                                        v
                       +---------------------------------+
                       |           Monitoring            |
                       |  - Telemetry & Diagnostic Logs  |
                       |  - Scopes & Telemetry Output    |
                       +---------------------------------+
```

### 1. Battery Plant (`Battery Plant`)
- Modeled for a **4S1P LiFePO₄ 15 Ah** pack.
- Implements Coulomb Counting via a **Discrete-Time Integrator**:
  $$\text{SOC}[k] = \text{SOC}[k-1] - \frac{I[k]\Delta t}{Q}$$
  with capacity $Q = 54{,}000\text{ C}$ ($15\text{ Ah} \times 3600\text{ s}$), initial condition $\text{SOC}_0 = 0.8$, and saturation limits $[0, 1]$.
  *(Current convention: Positive = Discharge; Negative = Charge)*.

### 2. Sensor Model (`Sensor Model`)
- Simulates physical sensor acquisition and measurement non-idealities:
  $$I_{\text{measured}} = I_{\text{actual}} + 0.05\text{ A}$$
  Provides a configurable offset/drift model for testing BMS robustness.

### 3. BMS Controller (`BMS Controller`)
- Hosts state machine routines, charge/discharge power limiting, cell balancing logic, and fault detection algorithms.
- Includes overcurrent threshold evaluation with relational operators generating `Overcurrent_Fault`.

### 4. Protection System (`Protection Sys.`)
- Safety interlocks guarding against over-voltage (OVP), under-voltage (UVP), over-current (OCP), and thermal faults (OTP/UTP).

### 5. Monitoring & Visualization (`Monitoring` / Scopes)
- Real-time monitoring scopes (`Current_Profile_Scope`, `Scope`) tracking multi-stage charge/discharge dynamics and state trajectory responses.

---

## 📁 Repository Structure

```
BMS_EMBEDDED_NEURONS/
├── Documentation/                 # Technical specs, architecture diagrams, and daily logs
│   └── Daily_Development_Log.md   # Detailed daily progress updates
├── Firmware/                      # C/C++ target implementations (STM32, TI, ESP32)
├── MATLAB_Model/                  # Simulink system simulations & validation harnesses
│   └── Intelligent_BMS_4S_LiFePO4.slx   # 4S LiFePO4 MBD simulation model
├── Test_Data/                     # Battery test cycling data, HIL profiles, benchmark logs
└── README.md                      # Project documentation & status
```

---

## 📝 Daily Development Log

### Day 1 — 26 November 2026

**Project:** Intelligent Battery Management System (BMS) with Advanced Fault Diagnosis and Adaptive Protection for Electric Mobility  
**Platform:** MATLAB Simulink  
**Battery Configuration:** 4S1P LiFePO₄ battery pack  
**Date:** 26 November 2026

#### Work Completed

##### 1. Simulink Project Setup
- Created the main Simulink model: `Intelligent_BMS_4S_LiFePO4.slx`.
- Established the top-level system architecture with the following subsystems:
  - Battery Plant
  - Sensor Model
  - BMS Controller
  - Protection System
  - Monitoring
- Configured the simulation for discrete-time operation:
  - Start Time: `0` seconds
  - Stop Time: `3600` seconds
  - Solver: Fixed-step, discrete
  - Sample Time: `1` second

##### 2. Battery Plant — SOC Estimation
Implemented the initial battery State of Charge (SOC) model using Coulomb counting.

**Implemented signal flow:**
```
Pack_Current → Gain → Discrete-Time Integrator → Saturation → SOC
```

**Model parameters:**
- Battery Capacity: `15 Ah`
- Capacity in Coulombs: `54,000 C`
- Initial SOC: `80%` (`0.8`)
- SOC limits: `0` to `1`
- Sample Time: `1` second
- Current convention: Positive current represents discharge; negative current represents charging.

The SOC calculation is based on:
$$\text{SOC}[k] = \text{SOC}[k-1] - \frac{I[k]\Delta t}{Q}$$

##### 3. Constant Current Simulation
- Tested the battery model with a constant `1 A` discharge current.
- Verified the expected linear decrease in SOC.
- SOC decreased from `80%` to approximately `73.33%` over `3600` seconds:
  $$\Delta \text{SOC} = \frac{1\text{ A} \times 3600\text{ s}}{54{,}000\text{ C}} \approx 0.0667 \quad (6.67\%)$$
- Validated the initial Coulomb-counting implementation.

##### 4. Variable Current Profile Generation
Created a variable load-current profile using multiple Step blocks and a Sum block.

| Time Interval | Current |
|---|---:|
| 0–500 s | 1 A |
| 500–1000 s | 3 A |
| 1000–1500 s | 5 A |
| 1500–3600 s | 2 A |

The current profile was successfully simulated and observed using a dedicated `Current_Profile_Scope`. This enables testing under dynamically changing load conditions.

##### 5. Sensor Model — Initial Implementation
Started implementing the Sensor Model subsystem:
- Added `Pack_Current_Actual` as the input.
- Introduced a constant measurement offset of `0.05 A`.
- Used a Sum/Add block to generate the measured current:
  $$I_{\text{measured}} = I_{\text{actual}} + 0.05$$
- Serves as a baseline sensor offset/error model for testing BMS robustness.

##### 6. Overcurrent Fault Detection — Initial Implementation
Started implementing fault-detection logic inside the BMS Controller subsystem:
- Used measured pack current as input.
- Added a configurable current threshold.
- Implemented a Relational Operator comparing measured current against threshold.
- Generated an `Overcurrent_Fault` signal.
- Added a Display block to observe the fault output in real-time.

---

## 📊 Current Project Status

| Subsystem | Status | Description |
|:---|:---:|:---|
| **Simulink Project Setup** | ✅ Completed | Fixed-step discrete solver, top-level architecture |
| **Battery Plant** | ✅ Completed | 4S1P LiFePO₄ pack representation |
| **SOC Coulomb Counting** | ✅ Completed | Discrete integrator with gain & saturation |
| **Constant Current Test** | ✅ Completed | 1 A discharge over 3600 s verified (80% → 73.33%) |
| **Variable Current Profile** | ✅ Completed | Piecewise step profile (1 A, 3 A, 5 A, 2 A) |
| **Sensor Model** | 🟡 In Progress | Initial +0.05 A current offset model implemented |
| **BMS Controller** | 🟡 In Progress | Initial overcurrent logic and threshold comparison |
| **Protection System** | ⏳ Pending | Interlocks for OVP, UVP, OCP, OTP |
| **Monitoring System** | ⏳ Pending | Real-time diagnostic telemetry & fault logs |
| **Cell Voltage Monitoring** | ⏳ Pending | Individual 4S cell tap acquisition & analysis |
| **Temperature Monitoring** | ⏳ Pending | NTC thermistor thermal modeling & thresholding |
| **Adaptive Protection** | ⏳ Pending | Dynamic safety derating and adaptive limits |
| **Advanced Fault Diagnosis** | ⏳ Pending | Machine learning / model-based diagnostic algorithms |

---

## 💡 Key Learning Outcomes

- Understood the fundamental Simulink subsystem architecture for an automotive/industrial BMS.
- Learned how Coulomb counting estimates SOC from current measurements.
- Mastered the relationship between battery current, capacity, sample time, and SOC variation.
- Learned how Step blocks and Sum blocks generate piecewise variable load profiles.
- Understood the purpose of sensor offset/error modeling in control systems.
- Implemented threshold-based fault detection using logical comparison blocks.

---

## 🎯 Next Steps

1. Continue developing the **BMS Controller**.
2. Finalize and validate overcurrent fault detection.
3. Develop the **Protection System** switchgear and logic.
4. Introduce cell voltage monitoring and voltage fault detection (OVP/UVP).
5. Add temperature monitoring and thermal fault detection (OTP/UTP).
6. Develop comprehensive monitoring and fault-status outputs.
7. Progress toward adaptive protection and advanced fault diagnosis algorithms.

---

## 🚀 Getting Started

### Prerequisites
- **MATLAB & Simulink**: Version **R2026b** (or compatible release with Simscape / DSP System Toolbox).
- Optional: Embedded Coder (for automatic C/C++ code generation).

### Running the Simulink Model
1. Clone the repository:
   ```bash
   git clone https://github.com/joeljo777/BMS_EMBEDDED_NEURONS.git
   cd BMS_EMBEDDED_NEURONS
   ```
2. Open MATLAB and navigate to the project directory.
3. Open the model:
   ```matlab
   open('MATLAB_Model/Intelligent_BMS_4S_LiFePO4.slx');
   ```
4. Run the simulation to view the battery plant response and SOC tracking under dynamic step current profiles.
5. Double-click **`Current_Profile_Scope`** and **`Scope`** to analyze dynamic behavior.

---

# **The CHASE Continues**
