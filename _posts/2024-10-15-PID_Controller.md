---
layout: post
title: PID Controller
date: 15-10-2024
categories: [Robotics]
tags: [robotics, algorithms, control_systems]
---
**PID** is a widely-used feedback control algorithm that adjusts the control inputs based on three error factors:

1. **Proportional (P)**: Reacts to the current error.
2. **Integral (I)**: Accounts for accumulated past errors.
3. **Derivative (D)**: Predicts future error trends.

A key challenge with the integral term is **integral windup**—when the integral term accumulates too much error, leading to performance issues like overshoot, instability, or slow response. This is typically addressed with **integral clamping**, which resets or limits the integral term when the output saturates.

Additionally, the PID controller in our system runs **asynchronously** to ensure that it can update the control output at a high frequency, independent of the main robot control loop. This allows the PID controller to operate efficiently while the robot continues to perform other tasks.

## Basic C++ Implementation

```cpp
#include "../../include/common/pid.h"

// Initializes PID controller with user-defined constants
PID::PID(PID_consts consts, double outMax, double exitRange) {
    // setting constants the lazy way
    this.consts = consts;
    consts.outMax = outMax; //used to determine saturation
    consts.exitRange = exitRange; //used to determine if the target has been reached
};

// Updates the PID controller based on the setpoint and process variable
float PID::update(double target, double current) {
    state.error = target - current;  //Error is the target value minus the current value
    state.derivative = state.error - state.lastError;  //Get derivative by subtacting current error and last error
    
    state.integral += state.error;  //Add current error to integral
    
    // Integral clamp (anti-windup) to prevent integral windup
    if((state.error > 0 && state.lastError < 0) || (state.error < 0 && state.lastError > 0)) {
        state.integral = 0; // Reset integral if error changes sign
    }
    
    state.lastError = state.error;  //Set last error to what the error currently is
    state.rawOut = consts.kP * state.error + consts.kI * state.integral + consts.kD * state.derivative;

    state.reachedTarget = fabs(state.error) <= consts.exitRange ? true : false;
    return state.rawOut;
}
```

### Key Concepts

1. **PID Constants**:
   - **kP (Proportional Gain)**: Controls the response to the current error.
   - **kI (Integral Gain)**: Controls the accumulated error over time.
   - **kD (Derivative Gain)**: Controls the response based on the rate of change of the error.

2. **Integral Windup**:
   - **Problem**: The integral term in a PID controller can accumulate error indefinitely, especially if the system is not able to reach the setpoint (e.g., when the output is saturated).
   - **Consequence**: When the integral term grows too large, it may lead to excessive control output, causing overshoot or instability when the system finally recovers.
   
3. **Integral Clamping (Anti-Windup)**:
   - **Solution**: To mitigate integral windup, we use **integral clamping**, which resets or limits the integral term when the error changes sign. This ensures that the integral term does not accumulate inappropriately, improving system stability.
   - **How it's Implemented**: In the provided code, the integral term is reset to zero whenever the error changes sign (i.e., the system moves from positive error to negative or vice versa). This is achieved with the following logic:
   
     ```cpp
     if((state.error > 0 && state.lastError < 0) || (state.error < 0 && state.lastError > 0)) {
         state.integral = 0; // Reset integral
     }
     ```

4. **Asynchronous Operation**:
   - The PID controller runs **asynchronously**, meaning that it can continuously update the control output in the background while the rest of the robot performs other tasks.
   - This allows the PID controller to compute the control actions at a higher frequency, improving responsiveness and ensuring smooth adjustments to the system, even while the robot is handling other real-time tasks like sensor reading, movement, or data logging.
   - The asynchronous execution is typically implemented using a separate thread or task, enabling the PID calculations to run independently of the main control loop, which is crucial for maintaining high performance and precise control.

### Explanation of the Code

1. **PID Constants Initialization**:
   - The constructor `PID::PID()` initializes the PID controller with user-defined constants (proportional, integral, derivative gains) and sets the output maximum (`outMax`) and target reach range (`exitRange`).

2. **Update Method (`PID::update`)**:
   - The `PID::update()` method calculates the **error**, **derivative**, and **integral** terms at each step.
   - It applies the PID formula to compute the **raw output**:
     ```cpp
     state.rawOut = consts.kP * state.error + consts.kI * state.integral + consts.kD * state.derivative;
     ```

3. **Integral Clamping**:
   - To prevent integral windup, the code checks if the error has crossed zero (i.e., if the error changes sign). If so, the integral term is reset to zero:
     ```cpp
     if((state.error > 0 && state.lastError < 0) || (state.error < 0 && state.lastError > 0)) {
         state.integral = 0; // Reset integral
     }
     ```

4. **Exit Condition**:
   - The method also checks whether the **target** has been reached by comparing the **error** with a threshold (`exitRange`):
     ```cpp
     state.reachedTarget = fabs(state.error) <= consts.exitRange ? true : false;
     ```

### Applications in Illini VEX Robotics

- **Drivetrain Control**: In the drivetrain, the PID controller ensures the robot's motors adjust smoothly and follow a desired trajectory or speed.
- **Arm Control**: For positioning arms or actuators, PID control is used to reach a target position smoothly, with integral clamping ensuring the arm does not overshoot or exhibit oscillations.

### Preventing Integral Windup:
To prevent integral windup in practical applications:
1. **Clamp the Integral**: Use the approach shown here to reset or limit the integral term when necessary.
2. **Tune PID Constants**: Proper tuning of the PID constants can minimize the need for clamping. Start with a low integral gain (`kI`) and gradually increase it as needed.
3. **Saturation Limits**: Ensure that your system output is bounded by hardware constraints, such as motor speed limits.

## Conclusion

The provided PID controller implementation includes **integral clamping** to address the issue of **integral windup**, ensuring smoother and more stable control for robotics applications. By resetting the integral term when the error changes sign, the controller prevents excessive error accumulation and improves the system's ability to follow the desired trajectory or reach the target efficiently. Additionally, running the PID controller **asynchronously** allows for efficient real-time control without interrupting other tasks, ensuring high performance in dynamic environments.

For more information on PID control, check out the [Purdue SIGBots Wiki on PID Controllers](https://wiki.purduesigbots.com/software/control-algorithms/pid-controller).