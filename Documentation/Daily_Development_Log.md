# Daily Development Log

## Day 1 — 26 November 2026

**Project:** Intelligent Battery Management System (BMS) with Advanced Fault Diagnosis and Adaptive Protection for Electric Mobility  
**Platform:** MATLAB Simulink  
**Battery Configuration:** 4S1P LiFePO₄ battery pack  
**Date:** 26 November 2026

---

## Work Completed

### 1. Simulink Project Setup
- Created the main Simulink model: `Intelligent_BMS_4S_LiFePO4.slx`.
- Established the top-level system architecture with the following subsystems:
  - Battery Plant
  - Sensor Model
  - BMS Controller
  - Protection System
  - Monitoring
- Configured the simulation for discrete-time operation:
  - **Start Time:** 0 seconds
  - **Stop Time:** 3600 seconds
  - **Solver:** Fixed-step, discrete
  - **Sample Time:** 1 second

### 2. Battery Plant — SOC Estimation
Implemented the initial battery State of Charge (SOC) model using Coulomb counting.

**Implemented signal flow:**
```
Pack_Current → Gain → Discrete-Time Integrator → Saturation → SOC
```

**Model parameters:**
- **Battery Capacity:** 15 Ah
- **Capacity in Coulombs:** 54,000 C
- **Initial SOC:** 80% (0.8)
- **SOC limits:** 0 to 1
- **Sample Time:** 1 second
- **Current convention:** Positive current represents discharge; negative current represents charging.

The SOC calculation is based on:
$$\text{SOC}[k] = \text{SOC}[k-1] - \frac{I[k]\Delta t}{Q}$$

### 3. Constant Current Simulation
- Tested the battery model with a constant 1 A discharge current.
- Verified the expected linear decrease in SOC.
- SOC decreased from 80% to approximately 73.33% over 3600 seconds.

This validated the initial Coulomb-counting implementation.

### 4. Variable Current Profile Generation
Created a variable load-current profile using multiple Step blocks and a Sum block.

| Time Interval | Current |
|---|---:|
| 0–500 s | 1 A |
| 500–1000 s | 3 A |
| 1000–1500 s | 5 A |
| 1500–3600 s | 2 A |

The current profile was successfully simulated and observed using a dedicated `Current_Profile_Scope`. This test allows the battery model to be evaluated under changing load conditions instead of only constant-current operation.

### 5. Sensor Model — Initial Implementation
Started implementing the Sensor Model subsystem:
- Added `Pack_Current_Actual` as the input.
- Introduced a constant measurement offset of 0.05 A.
- Used a Sum/Add block to generate the measured current:
  $$I_{\text{measured}} = I_{\text{actual}} + 0.05$$
This represents a simple sensor offset/error model for future testing of BMS robustness.

### 6. Overcurrent Fault Detection — Initial Implementation
Started implementing the first fault-detection logic inside the BMS Controller subsystem:
- Used the measured pack current as the input.
- Added a configurable current threshold.
- Implemented a Relational Operator to compare measured current against the threshold.
- Generated an `Overcurrent_Fault` signal.
- Added a Display block to observe the fault output.

The initial threshold was experimented with during implementation and will be finalized as the model develops.

---

## Current Project Status

| Subsystem | Status | Description |
|---|:---:|---|
| Simulink Project Setup | ✅ Completed | Fixed-step discrete solver, top-level architecture |
| Battery Plant | ✅ Completed | 4S1P LiFePO₄ pack representation |
| SOC Coulomb Counting | ✅ Completed | Discrete integrator with gain & saturation |
| Constant Current Test | ✅ Completed | 1 A discharge over 3600 s verified (80% → 73.33%) |
| Variable Current Profile | ✅ Completed | Piecewise step profile (1 A, 3 A, 5 A, 2 A) |
| Sensor Model | 🟡 In Progress | Initial +0.05 A current offset model implemented |
| BMS Controller | 🟡 In Progress | Initial overcurrent logic and threshold comparison |
| Protection System | ⏳ Pending | Interlocks for OVP, UVP, OCP, OTP |
| Monitoring System | ⏳ Pending | Real-time diagnostic telemetry & fault logs |
| Cell Voltage Monitoring | ⏳ Pending | Individual 4S cell tap acquisition & analysis |
| Temperature Monitoring | ⏳ Pending | NTC thermistor thermal modeling & thresholding |
| Adaptive Protection | ⏳ Pending | Dynamic safety derating and adaptive limits |
| Advanced Fault Diagnosis | ⏳ Pending | Machine learning / model-based diagnostic algorithms |

---

## Key Learning Outcomes

- Understood the basic Simulink subsystem architecture for a BMS.
- Learned how Coulomb counting estimates SOC from current measurements.
- Understood the relationship between battery current, capacity, sample time, and SOC variation.
- Learned how Step blocks and Sum blocks can generate piecewise variable load profiles.
- Understood the purpose of sensor offset/error modeling.
- Began implementing threshold-based fault detection using logical comparison blocks.

---

## Next Steps

1. Continue developing the BMS Controller.
2. Implement and validate overcurrent fault detection.
3. Develop the Protection System.
4. Introduce cell voltage monitoring and voltage fault detection.
5. Add temperature monitoring and thermal fault detection.
6. Develop monitoring and fault-status outputs.
7. Progress toward adaptive protection and advanced fault diagnosis.

---

**Day 1 Summary:**  
Established the initial Simulink architecture, implemented and validated SOC estimation using Coulomb counting, tested both constant and variable current profiles, introduced a simple current sensor offset model, and began implementing overcurrent fault detection.
