# Daily Development Log

## Day 1 — 26 November 2026

**Project:** Intelligent Battery Management System (BMS) with Advanced Fault Diagnosis and Adaptive Protection for Electric Mobility  
**Platform:** MATLAB Simulink (R2026b)  
**Battery Configuration:** 4S1P LiFePO₄ battery pack (15 Ah)  
**Date:** 26 November 2026

---

### Work Completed Today

#### 1. Battery Plant – SOC Modeling
- Developed the battery State of Charge (SOC) calculation model using Coulomb counting.
- Implemented the current-to-SOC conversion using a **Gain block** ($k = -\frac{1}{15 \times 3600}$) and **Discrete-Time Integrator**.
- Added a **Saturation block** to constrain SOC strictly between `0` and `100%` (`[0.0, 1.0]`).
- Configured initial SOC to `80%` (`0.8`), nominal battery capacity to `15 Ah` ($54{,}000\text{ Coulombs}$), and fixed sample time to `1 second`.
- Validated the model using a constant `1 A` discharge current over `3600 seconds`.
- Verified that the SOC decreases from `80%` to approximately `73.33%`, confirming the model's theoretical and empirical behavior:
  $$\Delta \text{SOC} = \frac{1\text{ A} \times 3600\text{ s}}{54{,}000\text{ C}} \approx 0.0667 \quad (6.67\%)$$

#### 2. Variable Current Profile Generation
- Created a dynamic variable current profile using multiple **Step blocks** and an **Add block** ($++++$).
- Designed the load profile to simulate varying discharge conditions over time:
  | Time Interval | Current |
  |:---:|:---:|
  | **0–500 s** | 1 A |
  | **500–1000 s** | 3 A |
  | **1000–1500 s** | 5 A |
  | **1500–3600 s** | 2 A |
- Connected the generated profile directly to the **Battery Plant** subsystem.
- Added a dedicated **`Current_Profile_Scope`** and output **`Scope`** to observe excitation and monitor SOC response under changing dynamic load conditions.

#### 3. BMS Controller – Fault Detection
- Implemented individual multi-fault detection logic using **Relational Operator blocks**:
  - **Cell Undervoltage Fault (UVP):** Threshold set to **$2.5\text{ V}$**
  - **Cell Overvoltage Fault (OVP):** Threshold set to **$3.65\text{ V}$**
  - **Overcurrent Fault (OCP):** Threshold set to **$2\text{ A}$**
  - **Overtemperature Fault (OTP):** Threshold set to **$60^\circ\text{C}$**
- Connected dedicated **Display blocks** to monitor each individual fault detection status in real time.

#### 4. Subsystem Organization and Model Structure
- Organized the fault detection logic into clean, separate visual sections using rectangular annotation areas.
- Created dedicated visual sections for:
  - Overcurrent Fault
  - Cell Undervoltage Fault
  - Cell Overvoltage Fault
  - Overtemperature Fault
- Significantly improved the readability, modularity, and maintainability of the **BMS Controller** subsystem.

#### 5. Protection System
- Developed the **Protection System** subsystem (`Protection Sys.`).
- Combined the four fault signals (`Overcurrent_Fault`, `Cell_Undervoltage_Fault`, `Cell_Overvoltage_Fault`, `Overtemperature_Fault`) using a 4-input **Logical Operator OR**.
- Implemented a unified **`Protection_Trip`** output signal.
- The system generates an active protection trip whenever any of the monitored fault conditions becomes active.
- Added a top-level **Display block** to monitor the unified Protection System trip status in real time.

#### 6. Monitoring Subsystem Integration
- Integrated key operational and diagnostic signals into the **Monitoring** subsystem:
  - Battery State of Charge (`SOC`)
  - Pack Current (`Pack_Current`)
  - Cell Voltage (`Cell_Voltage`)
  - Battery Temperature (`Battery_Temperature`)
  - Protection Trip (`Protection_Trip`)
- Connected dedicated **Display blocks** to monitor all five critical parameters.
- Fully integrated the **BMS Controller** and **Protection System** with the top-level system model.

---

### Current Model Status

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

### Key Learning Outcomes

- Understood the complete Simulink subsystem architecture for an automotive/industrial BMS.
- Learned how Coulomb counting estimates SOC from current measurements.
- Understood the mathematical relationship between battery current, capacity, sample time, and SOC variation.
- Learned how Step blocks and Sum blocks generate piecewise variable load profiles.
- Understood the purpose of sensor offset/error modeling.
- Mastered threshold-based fault detection using logical comparison and relational operator blocks.
- Learned how to design safety shutdown logic using unified logical OR protection structures.

---

### Next Planned Work

1. Complete and test the variable-current simulation under extended dynamic drive profiles.
2. Verify each fault condition individually by systematically injecting test input values (voltage, current, temperature).
3. Validate the `Protection_Trip` output response for each fault scenario.
4. Continue developing remaining BMS functionality, including improved sensor modeling (noise, drift) and adaptive protection logic.
5. Progress toward cell balancing and advanced neural/model-based fault diagnosis.

---

### Short Summary (Daily Progress Log)
> Today, the SOC model of the 4S LiFePO₄ battery pack was developed and validated using Coulomb counting. A constant-current discharge simulation was performed to verify SOC behavior, followed by the setup of a variable-current load profile using Step blocks. The BMS Controller was developed with cell undervoltage, cell overvoltage, overcurrent, and overtemperature detection logic. These fault signals were integrated into the Protection System using OR logic to generate a unified Protection Trip signal. The model was also organized using visual subsystem annotations, and the SOC, pack current, cell voltage, temperature, and protection-trip signals were integrated into the Monitoring subsystem.
