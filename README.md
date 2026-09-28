# Detailed Working and Explanation

## 1. Overview

This project simulates the speed control of a DC motor using a fully controlled SCR converter and also demonstrates regenerative braking.

The main idea is simple: instead of directly applying a fixed DC voltage to the motor, an AC supply is connected to a four-SCR bridge. By controlling when the SCRs are fired, the average voltage supplied to the motor can be changed. This allows the motor speed to be controlled.

The same converter is also used during braking. When regenerative braking is activated, the operating condition of the converter is changed so that the rotating motor can return electrical energy back through the converter.

The entire system was built and tested in MATLAB R2025a using Simulink and Simscape Electrical.

---

## 2. Main Components

The model consists of the following main sections:

- AC voltage source
- Fully controlled SCR bridge
- Wound-field DC motor
- Separate field supply
- SCR gate pulse generation
- Firing-angle control
- Motor speed measurement
- Electrical power measurement
- Regenerative braking control
- Dashboard controls
- Simscape solver and reference blocks

Each part has a specific purpose and together they form the complete motor control system.

---

## 3. DC Motor

The motor used in the simulation is a wound-field DC motor.

The motor has two main electrical sections:

- Armature
- Field winding

The armature receives the controlled voltage from the SCR converter, while the field winding is supplied separately.

The basic relationship is that the motor develops torque based on the armature current and magnetic field. As the applied armature voltage changes, the motor speed changes accordingly.

The mechanical side of the motor is connected to a Mechanical Rotational Reference, allowing Simscape to correctly model the motor's mechanical behaviour.

---

## 4. Fully Controlled SCR Converter

The armature is supplied through a single-phase fully controlled bridge consisting of four SCRs.

The four SCRs are arranged as a bridge so that the AC input can be converted into a controlled DC output.

The SCRs are not switched randomly. They are triggered in pairs.

The two main firing pairs used in the model are:

- SCR1 + SCR4
- SCR2 + SCR3

The pairs are triggered alternately every half cycle of the AC supply.

The AC source used in the simulation has:

- Peak voltage: 100 V
- Frequency: 50 Hz

Therefore, one complete AC cycle takes:

    T = 1 / 50 = 0.02 s

The firing pulses are generated according to the selected firing angle.

---

## 5. Firing Angle Control

The most important control parameter in the converter is the firing angle, represented by `α`.

The firing angle determines how long the SCR waits before turning ON after the appropriate point in the AC cycle.

For example:

- Smaller α → SCRs fire earlier
- Larger α → SCRs fire later

Changing α changes the average voltage applied to the motor armature.

This directly affects the motor speed.

A dashboard slider is used in the model so that the firing angle can be changed during simulation without manually editing the model.

The slider is currently configured to provide a convenient range of firing-angle values.

---

## 6. SCR Gate Pulse Generation

The SCR firing pulses are generated using a MATLAB Function block.

The function receives:

- Accelerator state
- Simulation time
- Firing angle
- Regenerative braking state
- Motor speed

It then calculates when each SCR should receive its gate pulse.

The basic sequence is:

    SCR1 + SCR4 → first firing pulse
    SCR2 + SCR3 → second firing pulse

The process repeats every 20 ms because the AC supply frequency is 50 Hz.

The gate signals are converted into physical signals using Simulink-PS Converter blocks and then applied to the SCR gate-control circuits.

This approach keeps the firing logic inside one MATLAB Function instead of using a large number of separate pulse-generation blocks.

---

## 7. Speed Control

The motor speed is controlled mainly through the firing angle.

When the accelerator is activated, the SCR gate-pulse generator starts firing the converter.

The motor then receives electrical power and begins to accelerate.

If the firing angle is reduced, the converter provides a higher average armature voltage, which generally results in a higher motor speed.

If the firing angle is increased, the average armature voltage decreases and the motor speed decreases.

This gives the project a simple way of demonstrating DC motor speed control.

The relationship can be summarized as:

    Firing angle ↓
          ↓
    Average armature voltage ↑
          ↓
    Motor speed ↑

and approximately:

    Firing angle ↑
          ↓
    Average armature voltage ↓
          ↓
    Motor speed ↓

---

## 8. Motor Speed Measurement

The motor speed is measured using an Ideal Rotational Motion Sensor.

The sensor is connected to the rotating shaft of the DC motor.

The sensor produces angular velocity in rad/s.

Since RPM is easier to understand and display, the signal is converted using:

    RPM = Angular velocity × 30 / π

The resulting RPM signal is sent to a Scope.

This allows the speed response of the motor to be observed throughout the simulation.

---

## 9. Regenerative Braking

The second major part of the project is regenerative braking.

Normally, when a motor is running and the electrical supply is removed, the motor gradually slows down because of mechanical losses and friction.

In regenerative braking, the motor is still rotating but its operating condition is changed so that it can act as a generator.

The mechanical energy stored in the rotating motor is converted back into electrical energy.

The basic energy flow changes from:

    Electrical energy
          ↓
       DC Motor
          ↓
    Mechanical energy

during normal operation, to:

    Mechanical energy
          ↓
       DC Motor
          ↓
    Electrical energy

during regenerative braking.

The recovered electrical energy is transferred through the converter in the reverse direction.

---

## 10. Regenerative Braking Control

A Dashboard Push Button called `Regen Braking` is used to activate braking.

When the button is pressed, the braking logic checks the motor's operating condition.

The braking control is designed so that braking is only activated when the motor is already rotating above a defined speed threshold.

This prevents the braking logic from being unnecessarily active when the motor is already stopped.

Once regenerative braking is activated, the SCR firing control changes the converter operating condition and the motor begins to decelerate.

The RPM can be observed falling rapidly on the Scope.

---

## 11. Power Measurement

To verify that braking is not simply mechanical braking, electrical power is also monitored.

This is important because a motor slowing down by itself does not necessarily mean that regeneration is taking place.

During normal motoring, electrical power flows from the source toward the motor.

During regenerative operation, the power-flow direction can reverse.

With the sign convention used in this model, this reverse flow appears as negative electrical power.

The simulation shows negative power during the regenerative braking interval, with the measured waveform reaching approximately -20 W during the braking event.

The negative portion of the power waveform is therefore used as an indication of reverse electrical power flow during braking.

---

## 12. Dashboard Controls

The model uses simple Dashboard controls to make the simulation easier to operate.

### Accelerator

The `Accelerator` button controls whether the motor is allowed to operate in the normal motoring mode.

When the accelerator is OFF, the firing pulses are disabled.

When the accelerator is ON, the SCR gate pulses are generated according to the selected firing angle.

### Firing Angle Slider

The slider allows the firing angle to be changed during the simulation.

This makes it easy to demonstrate how changing the firing angle affects motor speed.

### Regen Braking

The `Regen Braking` button activates the regenerative braking mode while the motor is rotating.

These controls make the model easier to demonstrate without repeatedly changing parameters inside the Simulink blocks.

---

## 13. Simulation Solver

Since the model contains power-electronic switching devices and electrical dynamics, the simulation requires a suitable solver.

The model uses a variable-step `ode15s` solver.

The maximum step size is limited to:

    1e-4 s

This helps the simulation capture the switching behaviour of the SCR converter more accurately.

The model also contains the required Simscape infrastructure:

- Solver Configuration
- Electrical Reference
- Mechanical Rotational Reference

These blocks provide the necessary physical references and simulation configuration for the Simscape network.

---

## 14. Overall Working Sequence

The complete operation can be understood in two stages.

### Normal Motor Operation

    AC Source
        ↓
    SCR Bridge
        ↓
    Controlled DC Voltage
        ↓
    DC Motor Armature
        ↓
    Motor Rotation
        ↓
    RPM Measurement

The firing angle controls the average armature voltage and therefore controls the motor speed.

### Regenerative Braking

    Motor Running
        ↓
    Regen Braking Activated
        ↓
    Converter Operating Condition Changes
        ↓
    Motor Decelerates
        ↓
    Motor Acts as Generator
        ↓
    Electrical Power Flow Reverses
        ↓
    Negative Power Observed

This gives the project both a controllable motor-speed operation and a regenerative braking mode.

---

## 15. Results Observed

The simulation demonstrates that changing the SCR firing angle changes the steady-state motor speed.

The motor can be accelerated to a high speed using the accelerator control, and the speed remains relatively stable once the operating condition settles.

When regenerative braking is activated, the motor speed decreases significantly.

At the same time, the electrical power waveform shows negative power during the braking event. In the present simulation, the negative power reaches approximately -20 W during the regenerative braking interval.

The combination of:

- Motor deceleration
- Reverse electrical power flow
- Negative measured power

provides the main evidence used to demonstrate the regenerative braking behaviour of the system.

---

## 16. Conclusion

This project provides a simulation-based study of DC motor speed control and regenerative braking using a fully controlled SCR converter.

The project shows how the firing angle of SCRs can be used to control the average armature voltage and consequently the speed of a DC motor. It also demonstrates how changing the converter's operating condition during braking can cause the motor to decelerate while electrical power flows in the reverse direction.

Using MATLAB, Simulink, and Simscape Electrical makes it possible to observe the electrical, mechanical, and control behaviour of the complete system in a single simulation environment.

The project can be further extended in the future by adding closed-loop speed control, load variations, improved current control, efficiency calculations, and more detailed analysis of the regenerated energy.
