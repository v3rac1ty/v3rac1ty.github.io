---
layout: post
title: Kalman Filter
date: 14-2-2025
categories: [Robotics]
tags: [robotics, algorithms, filtering, data_analysis]
---
A Kalman filter is an optimal estimation algorithm used to estimate the state of a system when the state cannot be measured directly but can be inferred from indirect and noisy measurements. This document explains the theory behind Kalman filtering and details our implementation for motion tracking.

## Theory of Kalman Filtering

### Basic Concept

Kalman filtering works in two steps:
1. **Prediction**: Using a motion model to predict the next state
2. **Update**: Correcting the prediction using measurements

The filter maintains:
- A state estimate vector (in our case: position, velocity, acceleration)
- A covariance matrix that represents uncertainty in our estimates

### Mathematical Foundation

The Kalman filter is governed by these equations:

**Prediction**:
- State prediction: x̂ₙ⁻ = F·x̂ₙ₋₁ + B·uₙ
- Covariance prediction: P₍ₙ⁻₎ = F·P₍ₙ₋₁₎·F^T + Q

**Update**:
- Measurement residual: y̆ₙ = zₙ - H·x̂ₙ⁻
- Residual covariance: S = H·P₍ₙ⁻₎·H^T + R
- Kalman gain: K = P₍ₙ⁻₎·H^T·S⁻¹
- State update: x̂ₙ = x̂ₙ⁻ + K·y̆ₙ
- Covariance update: Pₙ = (I - K·H)·P₍ₙ⁻₎

## Our Implementation

Our implementation is a three-state Kalman filter that tracks:
- Position
- Velocity
- Acceleration

### Key Features

1. **Full Error Covariance Tracking**:
   - Includes all cross-covariance terms between states
   - Uses proper propagation for higher-order kinematic models

2. **Kinematic Motion Model**:
   - Position update: θ = θ₀ + ω₀·t + (1/2)·α·t²
   - Velocity update: ω = ω₀ + α·t

3. **Adaptive Noise Handling**:
   - Variable `adaptiveFactor` can adjust process noise dynamically
   - Helps handle non-linear behaviors and modeling errors

4. **Numerically Stable Updates**:
   - Joseph form for covariance updates to maintain positive definite matrices

## Usage

### Initialization

```cpp
// Initialize with default parameters
KalmanFilter kf;

// Or specify initial state and noise parameters
KalmanFilter kf(
    initial_position, 
    initial_velocity, 
    initial_acceleration,
    position_variance,
    velocity_variance,
    acceleration_variance,
    position_process_noise,
    velocity_process_noise,
    acceleration_process_noise,
    measurement_noise
);
```

### Filtering Loop

```cpp
// Time step (in seconds)
double dt = 0.01;

// In your loop:
while (true) {
    // Predict next state
    kf.predict(dt);
    
    // Get a measurement
    double measurement = getSensorMeasurement();
    
    // Update filter with measurement
    kf.update(measurement);
    
    // Get estimated state
    double position = kf.get_position();
    double velocity = kf.get_velocity();
    double acceleration = kf.get_acceleration();
}
```

## Parameter Tuning

### Process Noise (Q)

- **Q_pos**: Position process noise - increase to trust measurements more than the model
- **Q_vel**: Velocity process noise - increase for less smooth velocity estimates
- **Q_acc**: Acceleration process noise - increase if acceleration can change rapidly

### Measurement Noise (R)

- **R**: Represents sensor noise variance - increase when measurements are unreliable

### Initial Covariance (P)

- **P_pos**: Initial position uncertainty
- **P_vel**: Initial velocity uncertainty
- **P_acc**: Initial acceleration uncertainty

## Advanced Features

### Adaptive Parameter Adjustment

The filter supports adaptive process noise scaling to handle dynamic situations:

```cpp
// Set base process noise parameters
kf.setAdaptiveNoiseParams(baseQPos, baseQVel, baseQAcc);

// The adaptiveFactor is applied internally to these base values
```

### Reset Function

You can reset the filter state and covariance:

```cpp
kf.reset(
    new_position, 
    new_velocity, 
    new_acceleration,
    new_position_covariance,
    new_velocity_covariance, 
    new_acceleration_covariance
);
```

## Performance Considerations

- The filter uses static variables in the `predict()` and `update()` methods to avoid repeated memory allocation
- All methods are marked `inline` to encourage compiler optimization
- The entire implementation is header-only for ease of inclusion

For more information on Kalman Filters, check out the [Purdue SIGBots Wiki on Kalman Filters](https://wiki.purduesigbots.com/software/control-algorithms/kalman-filter).