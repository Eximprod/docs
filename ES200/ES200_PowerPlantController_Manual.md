# 1. ES200 Power Plant Controller Manual

[**Table of Contents:**](#toc)
- [1. ES200 Power Plant Controller Manual](#1-es200-power-plant-controller-manual)
  - [1.1. Introduction](#11-introduction)
  - [1.2. Legal Information](#12-legal-information)
  - [1.3. General dispositions](#13-general-dispositions)
  - [1.4. Understanding the Configuration Variables](#14-understanding-the-configuration-variables)
  - [1.5. Understanding the Operating Modes](#15-understanding-the-operating-modes)
    - [1.5.1. Mode P](#151-mode-p)
    - [1.5.2. Mode P(f)](#152-mode-pf)
    - [1.5.3. Mode Q](#153-mode-q)
    - [1.5.4. Mode Q(U)](#154-mode-qu)
  - [1.6. Understanding the pre-configured points](#16-understanding-the-pre-configured-points)

## 1.1. Introduction
The **Power Plant Controller** is designed to manage and control power output in a dynamic and efficient manner. By leveraging PID control strategies, the controller is capable of handling both active and reactive power adjustments, as well as rapidly responding to system frequency and voltage deviations. 
This document provides detailed information about the configuration variables and operating modes that enable the controller to optimize performance under various conditions.

## 1.2. Legal Information

The information in this document is subject to change without prior notice and is not a commitment on the part of the supplier. Eximprod does not take responsibility for the use of the information in this document. 

Eximprod is not responsible for any direct or indirect matter of any nature that may result from the use of this document or any product mentioned in the document.

The software described in this document is licensed and may only be used in accordance with the terms of this license.

The information in this document cannot be reproduced or copied without the written permission of Eximprod and the content cannot be given to a third party for unauthorized use.

## 1.3. General dispositions

This document provides information about the ES200 Power Plant Controller software and its main features. The information available in this manual is intended for personnel who will use this product to configure the PPC equipment or for current use.

## 1.4. Understanding the Configuration Variables

<div style="text-align: center;">
  <img src="ppc-images/config-vars.png" style="max-width:100%; height:auto">
  <p>Figure 1: PPC configuration variables in ES200 Dashboard</p>
</div>


- **Testing:** This flag determines **whether** the controller is operating in testing mode. When enabled, you can change the PID controller parameters while the process is running, allowing for faster **iteration** during testing and quicker fine tuning of the PID parameters.
- **Proportional Gain:** Represents the proportional gain of the PID controller. **Default:** 1
- **Integral Gain:** Represents the integral gain of the PID controller. **Default:** 0.1
- **Derivative Gain:** Represents the derivative gain of the PID controller. **Default:** 0
- **Time Step:** A time value in milliseconds (ms) that determines the calculation time step for the controller. This value dictates how fast the controller calculates commands for the system and should be set according to how quickly the system can accept commands to avoid integral wind-up. **Default:** 400 ms
- **Ramp Value:** A percentage value that determines the speed of the output ramp trend of the controller. **Default:** 0
- **Max System Power:** A power value in megawatts (MW) representing the maximum output power of the controlled system. **Default:** 0
- **System Response Time:** A time value in seconds (s) that represents how fast the system responds to a command in a closed loop. This defines the execution time for a command, only affected by the PID parameters and with no ramped output. **Default:** 0 s
- **Inverter Delay Time:** A time value in seconds (s) used for inverters or setups that have a known delay between receiving commands and starting execution. **Default:** 0 s
- **Minimum Command and Maximum Command:** Lower and upper bounds for the control commands. The unit depends on the command type (absolute values or percentages).
  - **Absolute values:** Defaults are a lower bound of `0` for active power control or `-Max System Power` for reactive power control and an upper bound of `Max System Power`.
  - **Percentage commands:** Defaults are a lower bound of `0` for active power control or `-100` for reactive power control and an upper bound of `100`.
- **Scaling Factor:** A scalar value used to scale the output commands of the PID controller. Scaling is applied before the command is distributed to all connected points.
- **Operating Mode:** The controller operates in either `Active Power` or `Reactive Power` mode.
  - In **Active Power** mode, there are two submodes:
    - Setpoint-based active power control with automatic frequency response.
    - Active power control that strictly follows frequency changes within a narrow window while still accepting setpoints.
  - In **Reactive Power** mode, there are also two submodes:
    - Setpoint-based reactive power control.
    - A hybrid reactive power control scheme that follows the voltage curve within a defined window while still accepting setpoints.
- **Command Type:** The controller can output commands as an **absolute value** or as a **percentage** (the percentage being based on the specified `Max System Power`).
- **Statism:** A percentage value that describes the ratio between the relative deviation of frequency and the relative deviation of active power. This parameter is required only when operating in `Active Power` mode.
- **Nominal Voltage:** The nominal voltage value of the system (measured in kV). This parameter is required only when operating in `Reactive Power` mode.
- **Voltage Interval:** A percentage value describing the range within which the system voltage can vary. For example, for a 20 kV system with a voltage interval of 5%, the maximum allowed voltage setpoint is 21 kV. This parameter is required only when operating in `Reactive Power` mode.
- **Process Point:** The primary data point for the control process measurement. For `Active Power`, this should be the measured power output of the system; for `Reactive Power`, the measured reactive power output.
- **Process Complement Point:** The auxiliary measurement data point. For `Active Power`, this is the system frequency; for `Reactive Power`, it is the system voltage.
- **Available Power Point:** A data point providing an estimated `Available Power` for the system (e.g., an estimate from a weather station).
- **Inverter Points:** Data points, formatted as a comma-separated list, representing the registers of the inverters to which the controller's commands should be sent.
- **Needs Activation Command:** A flag indicating whether the inverter requires an activation command to execute the current command.
- **Inverter Activation Points:** Similar to **Inverter Points** in formatting, these represent the inverter registers that receive the activation commands. These only need to be specified if the **Needs Activation Command** flag is checked.

---

## 1.5. Understanding the Operating Modes

The Power Plant Controller has two main operating modes, each with two submodes that offer distinct characteristics and functionality:

- **Active Power**
  - [Mode P (RFA-SC/CR)](#151-mode-p)
  - [Mode P(f) (RFA)](#152-mode-pf)
- **Reactive Power**
  - [Mode Q](#153-mode-q)
  - [Mode Q(U)](#154-mode-qu)

### 1.5.1. Mode P
This is the primary **Active Power** operating mode, responsible for processing incoming active power setpoints while dynamically responding to frequency deviations. Mode P validates new setpoints and continuously monitors system frequency, adjusting the active power output as needed to maintain stability.

**Default Setpoint:** The mode starts with a setpoint equal to `Max System Power`.

- **Setpoint Handling:** When a new active power setpoint is received, Mode P verifies that it is within acceptable bounds (i.e., between 0 and `Max System Power`). If the setpoint is out of bounds, it reverts to the last valid value and flags this with an appropriate status update.
- **Ramped Output Strategy:** For valid setpoints, Mode P initiates a ramped output strategy by breaking the transition from the current power level to the new setpoint into incremental steps. This ensures a smooth and controlled change.
- **Frequency Correction:** Mode P detects if the system frequency deviates beyond predefined thresholds. In such cases, it calculates a corrective command based on the deviation from the nominal frequency and adjusts the active power output accordingly. This frequency-based control is synchronized with inverter setup delays to ensure proper command execution.

### 1.5.2. Mode P(f)
This secondary **Active Power** operating mode is designed to rapidly counteract deviations in system frequency while accommodating active power setpoint adjustments. Mode P(f) operates within a very narrow insensibility band (±0.01) to promptly address any frequency variations.

- **Frequency Correction:** When the measured frequency deviates outside the defined reference limits, Mode P(f) computes a corrective command by comparing the nominal frequency to the current process measurement, thereby adjusting the active power output.
- **Inverter Setup and Timing:** Mode P(f) incorporates inverter setup delays and carefully manages timing intervals to ensure that frequency corrections are applied accurately and only when the system is ready.
- **Ramped Output Transition:** If a new active power setpoint is issued, Mode P(f) transitions to a ramped output strategy by generating intermediate ramp points to smoothly transition from the current power level to the new setpoint. This approach prevents sudden changes while prioritizing frequency corrections.
- **Internal Updates:** Mode P(f) continuously updates its internal parameters, such as the previous power value and insensibility limits, and uses status callbacks to signal key events (e.g., executing a frequency command, engaging ramp output, or detecting an out-of-bound setpoint).

### 1.5.3. Mode Q
This is the primary **Reactive Power** operating mode, dedicated to processing and executing reactive power setpoints with rapid response. Mode Q bypasses ramped output strategies in favor of immediate command execution.

**Default Setpoint:** The mode starts with a setpoint equal to `0`.

- **Setpoint Validation:** When a new reactive power setpoint is received, Mode Q first validates it against preset boundaries defined by `Max System Power`. If the setpoint exceeds these limits, it is automatically rejected and reverts to the previous valid value, with a corresponding status update.
- **Inverter Setup Delay:** For valid setpoint changes, a brief inverter setup delay may be initiated (as defined by the `Inverter Delay Time` parameter). During this delay, the new command is held until the system is ready to execute the change.
- **Immediate Execution:** Once the delay elapses, the reactive power command is committed, and the new setpoint is stored and executed immediately. Throughout this process, Mode Q provides continuous status feedback regarding the command's execution.

### 1.5.4. Mode Q(U)
This is the secondary **Reactive Power (Voltage)** operating mode, responsible for processing incoming voltage setpoints and converting them into reactive power commands for the system.

- **Setpoint and Safety Validation:** Mode Q(U) begins by validating the incoming voltage setpoint against fixed safety boundaries. Once validated, it computes dynamic, sliding voltage limits that define a permissible range around the setpoint.
- **Reactive Power Adjustment:** If the measured voltage deviates outside these sliding limits, Mode Q(U) calculates the necessary reactive power adjustment (ΔQ) by comparing the reference voltage to the current process value, and applies this ΔQ to drive the system back toward the desired voltage level.
- **Ramped Output Strategy:** To ensure smooth transitions and avoid abrupt changes, Mode Q(U) implements a ramped output strategy. It leverages a PID controller to generate a sequence of ramp points that gradually guide the system from its current state to the new setpoint.
- **Status Feedback:** Throughout its operation, Mode Q(U) employs status callbacks to indicate critical states (e.g., when a voltage command is attempted, when a setpoint is out of bounds, during ramp execution, or upon successful command execution), ensuring that the broader control system remains informed.

## 1.6. Understanding the pre-configured points
Whenever a **Power Plant Controller** equipment is added in the database configuration using ES200 Dashboard, a number of pre-configured points will be added to the equipment based on the specified configuration variables. These points will be different based on wether you select **Active Power** or **Reactive Power** as the operating mode. 
Additionally, some other points will be added based on the status of the **Testing** flag which will be showcased later in this section.
First, let's showcase the difference between the two operating modes. 
For **Active Power**, the generated points are the following:

<div style="text-align: center;">
  <img src="ppc-images/active-power-gen-points-db-no-testing.png" style="max-width:100%; height:auto">
  <p>Figure 2: Pre-Configured points for Active Power operating mode in ES200 Dashboard</p>
</div>

1. **[AO] Setpoint_P**  
   - **Point Type:** *Analog Output*  
   - **Purpose:** Represents the *desired active power setpoint* that the controller will attempt to maintain. This point is used for receiving setpoints which will act as an input into the controller logic (the SCADA or operator writes this value, and the PPC reads it). Measured in MW.

2. **[AI] SetpointValue_P**  
   - **Point Type:** *Analog Input*  
   - **Purpose:** Holds the *validated or current active power setpoint value* used internally by the controller. It may mirror or slightly differ from the raw “Setpoint” above if any validation (e.g., bounds checking) or processing is applied.

3. **[AI] Translation_f**  
   - **Point Type:** *Analog Input*  
   - **Purpose:** Stores the *frequency‐related command* value calculated by the PPC. For instance, when frequency deviations occur, the PPC needs to calculate a delta for that frequency deviation and *translate* it into an active power value which will be used as the new setpoint for the system.

4. **[AI] Status**  
   - **Point Type:** *Analog Input*  
   - **Purpose:** Reflects the *current status* of the Power Plant Controller. This is a numeric code indicating modes such as "Executing Ramp", "Setpoint Out of Bounds" or "Executing Command". All status point codes will be showcased in a later section of the documentation.

5. **[BO] ModeSwitch**  
   - **Point Type:** *Binary Output*  
   - **Purpose:** A *command from SCADA or an operator* to switch between secondary operating modes (e.g., from mode P to mode P(f)). Writing 0 or 1 to this point triggers the PPC to change to the specific secondary mode. For example, for **Active Power** the default secondary mode is **mode P**, so writing 1 to this point will switch the secondary mode to **mode P(f)**.

6. **[BI] PrimaryModeStatus**  
   - **Point Type:** *Binary Input*  
   - **Purpose:** Indicates what the *primary mode* (e.g., Active Power or Reactive Power) of the PPC is. A value of 0 means **Active Power**, and a value of 1 means **Reactive Power**. The primary operating mode cannot be changed once the equipment was initialised, but this point exists to be able to tell what type of PPC is currently running without relying on naming conventions or other arbitrary methods.


7. **[BI] SecondaryModeStatus**  
   - **Point Type:** *Binary Input*  
   - **Purpose:** Indicates which of the *secondary modes* (e.g., Mode P(f) or mode P for active power frequency and Mode Q or mode Q(U) for reactive power) is currently active.

8. **[AI] RampPoint**  
   - **Point Type:** *Analog Input*  
   - **Purpose:** Displays the *current ramp step or intermediate command value* during a transition. When the controller ramps from one power level to another, it calculates intermediate points; the active step is shown here.

For **Reactive Power** the generated points are the following:

<div style="text-align: center;">
  <img src="ppc-images/reactive-power-gen-points-db-no-testing.png" style="max-width:100%; height:auto">
  <p>Figure 3: Pre-Configured points for Reactive Power operating mode in ES200 Dashboard</p>
</div>

As can be seen from the image, most of the generated points are very similar to their **Active Power** counterparts, except for the naming adjustments to reflect the parameters they actually operate on. A point that is unique to the **Reactive Power** operating mode is the **Setpoint_U** point with it's respective **SetpointValue_U**, so let's look at that in more detail.

**[AO] Setpoint_U**  
   - **Point Type:** *Analog Output*  
   - **Purpose:** Represents the *desired voltage setpoint* that the controller will attempt to maintain. This point is used for receiving setpoints which will act as an input into the controller logic (the SCADA or operator writes this value, and the PPC reads it). Measured in kV. Unlike frequency, which has a desired fixed value according to national energy regulation, voltage can vary in a certain range so the operator needs to be able to specify the desired voltage, which will subsequently be translated into a reactive power command.

**[AI] SetpointValue_U**  
   - **Point Type:** *Analog Input*  
   - **Purpose:** Holds the *validated or current voltage setpoint value* used internally by the controller. It may mirror or slightly differ from the raw “Setpoint” above if any validation (e.g., bounds checking) or processing is applied.

The **Testing** flag allows for much faster iteration when trying to parametize the controll by way of allowing the person configuring and testing to switch the PID parameters while the process is running. Otherwise this would have to be done by going to the ES200 Dashboard, reconfiguring the equipment with the desired parameters and reloading the database onto ES200, which depending on the size of your database can take some time. 
While the **Testing** flag is powerful and allows for a lot of flexibility, it is **strongly dicouraged** that you leave this checked whenever deploying the Power Plant Controller in a production scenario as this can turn into a serious liability, since anyone with access to your ES200 instance can modify the parameters of your controller.

<div style="text-align: center;">
  <img src="ppc-images/testing-gen-points-db.png" style="max-width:100%; height:auto">
  <p>Figure 3: Pre-Configured points for the Testing flag in ES200 Dashboard</p>
</div>

When the **Testing** flag is enabled, these analog output points become writable in real time. This allows operators or engineers to adjust key PID and timing parameters on the fly while the controller is actively regulating power. Below is a breakdown of each point shown in the table:

1. **[AO] currentProportionalGain**  
   - **Point Type:** *Analog Output*  
   - **Purpose:** Allows the user to **manually override** the proportional (Kp) term of the PID controller while the system is running. Increasing or decreasing this value changes how aggressively the controller responds to the difference between the measured and desired setpoints.

2. **[AO] currentIntegralGain**  
   - **Point Type:** *Analog Output*  
   - **Purpose:** Enables real-time adjustments to the **integral (Ki) gain** of the PID loop. By modifying this value, users can influence how the controller accumulates and corrects for steady-state errors over time.

3. **[AO] currentSystemResponseTime**  
   - **Point Type:** *Analog Output*  
   - **Purpose:** Lets the operator **dynamically set** the expected system response time (in seconds) used by the controller’s logic. This value helps the controller estimate how quickly the plant can react to new setpoints or ramp commands, influencing ramp calculations.

4. **[AO] currentInverterDelayTime**  
   - **Point Type:** *Analog Output*  
   - **Purpose:** Used to **adjust** the inverter delay time (in seconds) on the fly. This delay accounts for the time needed by inverters to begin executing commands. Changing it in real time helps refine the coordination between command issuance and actual inverter response.