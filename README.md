# BMS_EMBEDDED_NEURONS

[![MATLAB](https://img.shields.io/badge/MATLAB-R2026b-orange.svg)](https://www.mathworks.com/products/matlab.html)
[![Simulink](https://img.shields.io/badge/Simulink-MBD-blue.svg)](https://www.mathworks.com/products/simulink.html)
[![Chemistry](https://img.shields.io/badge/Chemistry-4S1P%20LiFePO4%20(15Ah)-green.svg)](#key-specifications)
[![Architecture](https://img.shields.io/badge/Architecture-Model--Based%20Design%20(MBD)-purple.svg)](#system-architecture)
[![Status](https://img.shields.io/badge/Status-Active%20Development-brightgreen.svg)](#current-model-status)

An Intelligent Battery Management System (BMS) with Advanced Fault Diagnosis and Adaptive Protection for Electric Mobility, combining **Model-Based Design (MBD)**, physics-informed battery modeling, and embedded intelligence.

---

## 📌 Overview

**BMS_EMBEDDED_NEURONS** bridges the gap between high-fidelity dynamic simulation and production-grade embedded microcontroller firmware. Starting from a **4S1P LiFePO₄ (15 Ah)** battery plant in MATLAB/Simulink, the project implements closed-loop current sensing, Coulomb-counting State of Charge (SOC) tracking, multi-threshold fault protection (UVP, OVP, OCP, OTP), a unified protection trip mechanism, and real-time monitoring destined for embedded firmware deployment ("Embedded Neurons").

---

## ⚡ Key Specifications & Protection Thresholds

| Parameter / Protection | Value / Limit | Description |
| :--- | :--- | :--- |
| **Cell Chemistry** | $\text{LiFePO}_4$ (Lithium Iron Phosphate) | Inherently stable, high thermal runaway margin |
| **Pack Topology** | **4S1P** (4 cells in series, 1 parallel) | Nominal ~12.8 V, Peak ~14.4 V - 14.6 V |
| **Nominal Capacity** | **15 Ah** (54,000 Coulombs) | Capacity base for discrete-time coulomb integration |
| **Initial SOC ($\text{SOC}_0$)** | **80%** (`0.8`) | Configured default initialization |
| **SOC Limits** | `[0.0, 1.0]` (0% - 100%) | Hard bounded by saturation blocks |
| **Cell Undervoltage (UVP)** | **$2.50\text{ V}$** | Triggers `Cell_Undervoltage_Fault` |
| **Cell Overvoltage (OVP)** | **$3.65\text{ V}$** | Triggers `Cell_Overvoltage_Fault` |
| **Overcurrent (OCP)** | **$2.00\text{ A}$** | Triggers `Overcurrent_Fault` |
| **Overtemperature (OTP)** | **$60.0^\circ\text{C}$** | Triggers `Overtemperature_Fault` |
| **Protection Output** | Unified `Protection_Trip` | 4-input Logical OR combining all active faults |
| **Simulation Solver** | Discrete (Fixed-Step) | Step size: $1\text{ s}$, Duration: $0 - 3600\text{ s}$ |

---

## 🏗️ System Architecture

The project employs a modular Model-Based Design (MBD) architecture consisting of five core subsystems:

```
+---------------------------------------------------------------------------------------+
|                                 DYNAMIC LOAD PROFILE                                  |
|   Step Current Profiles (t = 0s, 500s, 1000s, 1500s -> 1A, 3A, 5A, 2A Load Profile)   |
+-------------------------------------------+-------------------------------------------+
                                            |
                                            v
                           +---------------------------------+
                           |       Battery Plant (4S)        |
                           |  - LiFePO4 dynamic cell model   |
                           |  - Discrete Integrator (Coulomb)|
                           +----------------+----------------+
                                            |
       +------------------------------------+------------------------------------+
       |                                    |                                    |
       v                                    v                                    v
+--------------+                   +------------------+                 +-----------------+
| Sensor Model |                   |  BMS Controller  |                 | Protection Sys. |
| - Current    |----(Measured)---->|  - UVP (< 2.5V)  |--(Fault Flags)->| - 4-Input OR    |
| - Voltage    |                   |  - OVP (> 3.65V) |                 | - Unified Trip  |
| - Temp (NTC) |                   |  - OCP (> 2A)    |                 |   Protection    |
+-------+------+                   |  - OTP (> 60°C)  |                 +--------+--------+
        |                          +--------+---------+                          |
        |                                   |                                    |
        +-----------------------------------+------------------------------------+
                                            |
                                            v
                           +---------------------------------+
                           |           Monitoring            |
                           |  - SOC, Current, Voltage, Temp  |
                           |  - Protection Trip Realtime     |
                           |  - Scopes & Displays            |
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
- Hosts dedicated fault detection logic organized with visual annotations:
  - **Cell Undervoltage Fault (UVP):** $V_{\text{cell}} < 2.5\text{ V}$
  - **Cell Overvoltage Fault (OVP):** $V_{\text{cell}} > 3.65\text{ V}$
  - **Overcurrent Fault (OCP):** $I_{\text{pack}} > 2\text{ A}$
  - **Overtemperature Fault (OTP):** $T_{\text{batt}} > 60^\circ\text{C}$
- Real-time display blocks for each monitored fault.

### 4. Protection System (`Protection Sys.`)
- Combines all four fault signals using a **4-input Logical OR block**.
- Generates a unified **`Protection_Trip`** output signal, driving safety contactors/relays upon any anomaly.

### 5. Monitoring & Visualization (`Monitoring` / Scopes)
- Real-time monitoring displays for **SOC**, **Pack Current**, **Cell Voltage**, **Temperature**, and **Protection Trip** status.
- Real-time scopes (`Current_Profile_Scope`, `Scope`) tracking multi-stage charge/discharge dynamics and state trajectory responses.

---

## 📁 Repository Structure

```
BMS_EMBEDDED_NEURONS/
├── Documentation/                 # Technical specs, architecture diagrams, and daily logs
│   └── Daily_Development_Log.md   # Daily development progress log
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

##### 1. Battery Plant – SOC Modeling
- Developed the battery SOC calculation model using Coulomb counting.
- Implemented the current-to-SOC conversion using a **Gain block** ($k = -\frac{1}{15 \times 3600}$) and **Discrete-Time Integrator**.
- Added a **Saturation block** to constrain SOC strictly between `0` and `100%` (`[0.0, 1.0]`).
- Configured initial SOC to `80%` (`0.8`), nominal battery capacity to `15 Ah` ($54{,}000\text{ C}$), and sample time to `1 second`.
- Validated the model using a constant `1 A` discharge current over `3600 seconds`.
- Verified that the SOC decreases from `80%` to approximately `73.33%`, confirming the model's theoretical behavior.

##### 2. Variable Current Profile
- Created a variable current profile using multiple **Step blocks** and an **Add block** ($++++$).
- Designed the load profile to simulate different discharge conditions over time:
  - `0–500 s`: $1\text{ A}$
  - `500–1000 s`: $3\text{ A}$
  - `1000–1500 s`: $5\text{ A}$
  - `1500–3600 s`: $2\text{ A}$
- Connected the current profile to the **Battery Plant** subsystem.
- Added a dedicated **Scope** (`Current_Profile_Scope`) to observe the current profile and monitor SOC response under changing load conditions.

##### 3. BMS Controller – Fault Detection
- Implemented individual fault detection logic using **Relational Operator blocks**:
  - Cell undervoltage detection with a threshold of **$2.5\text{ V}$**.
  - Cell overvoltage detection with a threshold of **$3.65\text{ V}$**.
  - Overcurrent detection with a threshold of **$2\text{ A}$**.
  - Overtemperature detection with a threshold of **$60^\circ\text{C}$**.
- Connected **Display blocks** to monitor the fault detection outputs in real-time.

##### 4. Subsystem Organization and Model Structure
- Organized the fault detection logic into separate visual sections using rectangular annotation areas:
  - *Overcurrent Fault*
  - *Cell Undervoltage Fault*
  - *Cell Overvoltage Fault*
  - *Overtemperature Fault*
- Significantly improved the readability, modularity, and structure of the **BMS Controller** subsystem.

##### 5. Protection System
- Developed the **Protection System** subsystem.
- Combined the four fault signals using a **4-input Logical OR operation**.
- Implemented a unified **`Protection_Trip`** output signal.
- The system generates a protection trip when any of the monitored fault conditions becomes active.
- Added a top-level **Display block** to monitor the Protection System output in real-time.

##### 6. Monitoring Subsystem Integration
- Integrated `SOC`, `Pack_Current`, `Cell_Voltage`, `Battery_Temperature`, and `Protection_Trip` signals into the **Monitoring** subsystem.
- Connected **Display blocks** to monitor these parameters simultaneously.
- Integrated the **BMS Controller** and **Protection System** with the top-level model.

---

## 📊 Current Model Status

| Subsystem / Feature | Status | Implementation Details |
|:---|:---:|:---|
| **Simulink Project Setup** | ✅ Completed | Fixed-step discrete solver ($T_s = 1\text{ s}$), top-level architecture |
| **Battery Plant SOC Model** | ✅ Completed | 4S1P LiFePO₄ pack model with discrete Coulomb counting |
| **Constant-Current SOC Validation** | ✅ Completed | 1 A discharge over 3600 s verified (80% → 73.33%) |
| **Variable-Current Profile Setup** | ✅ Completed | Piecewise step profile (1 A, 3 A, 5 A, 2 A) |
| **Fault Detection Logic** | ✅ Completed | UVP (2.5 V), OVP (3.65 V), OCP (2 A), OTP (60 °C) |
| **Subsystem Organization** | ✅ Completed | Rectangular visual annotation blocks for fault modules |
| **Protection System OR Logic** | ✅ Completed | 4-input Logical OR generating unified `Protection_Trip` |
| **Protection-Trip Monitoring** | ✅ Completed | Top-level visual monitoring display block |
| **Monitoring Subsystem Integration**| ✅ Completed | Display blocks for SOC, Current, Voltage, Temp, Trip |
| **Sensor Offset Error Model** | 🟡 Initial Model | +0.05 A measurement offset implemented |
| **Adaptive Protection** | ⏳ Pending | Dynamic safety derating & adaptive thresholds |
| **Advanced Fault Diagnosis** | ⏳ Pending | Model-based & neural-network diagnostic algorithms |

---

## 🎯 Next Planned Work

1. Complete and test the variable-current simulation under changing load conditions.
2. Verify each fault condition individually by changing input values.
3. Validate the `Protection_Trip` output for each fault.
4. Continue developing the remaining BMS functionality, including improved sensor modeling and adaptive protection logic.
5. Progress toward cell balancing and advanced neural/model-based fault diagnosis.

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
