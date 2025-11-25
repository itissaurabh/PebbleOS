# Section 2: Motion Sensors

## Introduction

Motion sensors are fundamental to smartwatch functionality, enabling activity tracking, gesture recognition, step counting, and sleep monitoring. This section covers accelerometers, gyroscopes, magnetometers, and integrated Inertial Measurement Units (IMUs).

## Accelerometers

### Operating Principles

MEMS accelerometers measure linear acceleration along one or more axes using a proof mass suspended by spring structures. When acceleration occurs, the proof mass deflects, and this displacement is measured capacitively.

```
           Fixed Plate
              ───────
                 │
    ┌────────────┼────────────┐
    │     ┌──────┴──────┐     │
    │     │ Proof Mass  │◀────┼── Acceleration
    │     └──────┬──────┘     │
    │            │            │
    └────────────┼────────────┘
              ───────
           Fixed Plate

    Capacitance changes with displacement
```

### Key Specifications

| Parameter | Low Power | Standard | High Performance |
|-----------|-----------|----------|------------------|
| Range | ±2g to ±8g | ±2g to ±16g | ±2g to ±32g |
| Resolution | 10-12 bit | 12-14 bit | 14-16 bit |
| Noise Density | 200-400 μg/√Hz | 100-200 μg/√Hz | 50-100 μg/√Hz |
| ODR (Output Data Rate) | 1-400 Hz | 1-1600 Hz | 1-6400 Hz |
| Current (Active) | 1-5 μA | 10-50 μA | 100-500 μA |
| Current (Low Power) | 0.5-2 μA | 1-5 μA | 5-20 μA |

### Recommended Parts for Wearables

| Part Number | Manufacturer | Range | Resolution | Current | Features |
|-------------|--------------|-------|------------|---------|----------|
| LIS2DW12 | STMicroelectronics | ±2/4/8/16g | 14-bit | 50 nA - 45 μA | Ultra-low power |
| BMA456 | Bosch | ±2/4/8/16g | 16-bit | 14 μA | Step counter built-in |
| MC3672 | mCube | ±2/4/8/16g | 14-bit | 0.4 μA | Very low power |
| ADXL362 | Analog Devices | ±2/4/8g | 12-bit | 1.8 μA | Motion-activated wake |

### Application Considerations

#### Step Counting
- **Sample rate**: 25-50 Hz typical
- **Filter**: 0.5-3 Hz bandpass for walking cadence
- **Algorithm**: Peak detection with adaptive thresholds
- **Power tip**: Use accelerometer's built-in pedometer if available

#### Activity Classification
- **Sample rate**: 50-100 Hz
- **Features**: Mean, variance, FFT coefficients
- **Activities**: Walking, running, cycling, stationary, sleep

#### Gesture Recognition
- **Sample rate**: 100-200 Hz
- **Gestures**: Wrist raise, double-tap, shake
- **Latency**: <100ms for responsive feel

## Gyroscopes

### Operating Principles

MEMS gyroscopes measure angular velocity using the Coriolis effect. A vibrating proof mass experiences a perpendicular force when rotated, which is measured capacitively.

### Key Specifications

| Parameter | Low Power | Standard | High Performance |
|-----------|-----------|----------|------------------|
| Range | ±250 to ±2000 °/s | ±250 to ±2000 °/s | ±125 to ±4000 °/s |
| Resolution | 16-bit | 16-bit | 16-bit |
| Noise Density | 0.01-0.03 °/s/√Hz | 0.005-0.01 °/s/√Hz | 0.003-0.007 °/s/√Hz |
| Bias Stability | 5-20 °/hr | 1-5 °/hr | 0.5-2 °/hr |
| Current | 1-3 mA | 3-5 mA | 5-10 mA |

### Wearable Applications

- **Gesture recognition**: Detecting wrist rotations
- **Swimming stroke detection**: Arm rotation patterns
- **Fall detection**: Combined with accelerometer
- **Dead reckoning**: Short-term position tracking

**Power Note**: Gyroscopes consume 100-1000x more power than accelerometers. Use motion-triggered activation rather than continuous operation.

## Magnetometers

### Operating Principles

Magnetometers measure magnetic field strength, typically using:
- **Hall effect sensors**: Lower precision, lower power
- **AMR (Anisotropic Magnetoresistive)**: Better sensitivity
- **Fluxgate**: Highest precision, highest power

### Key Specifications

| Parameter | Typical Value |
|-----------|---------------|
| Range | ±4800 μT |
| Resolution | 0.1-0.3 μT |
| Noise Density | 0.1-0.5 μT/√Hz |
| Current | 50-200 μA |
| Offset drift | ±0.5 μT/°C |

### Applications

- **Compass heading**: Combined with accelerometer for tilt-compensation
- **Indoor positioning**: Magnetic field mapping
- **Interference detection**: Identifying metal objects

### Challenges in Wearables

- **Hard iron distortion**: Fixed magnetic sources (speaker magnets, battery)
- **Soft iron distortion**: Ferromagnetic materials in watch case
- **Calibration**: User-initiated figure-8 motion
- **Dynamic interference**: Phone, keys, transit cards

## Inertial Measurement Units (IMUs)

IMUs integrate multiple sensors in a single package, typically:
- 3-axis accelerometer
- 3-axis gyroscope
- (Sometimes) 3-axis magnetometer (9-DOF)

### Benefits of Integration

| Advantage | Description |
|-----------|-------------|
| Smaller footprint | Single package vs. multiple |
| Time synchronization | Hardware-aligned samples |
| Sensor fusion | Built-in algorithms |
| Reduced BOM | Fewer components |

### Popular IMU Options

| Part | Sensors | Accel Noise | Gyro Noise | Current | Features |
|------|---------|-------------|------------|---------|----------|
| BMI270 | 6-axis | 160 μg/√Hz | 0.007 °/s/√Hz | 685 μA | Wearable-focused |
| ICM-42688-P | 6-axis | 70 μg/√Hz | 0.0028 °/s/√Hz | 0.9 mA | Low noise |
| LSM6DSO | 6-axis | 70 μg/√Hz | 4 mdps/√Hz | 0.55 mA | Machine learning core |
| BMI160 | 6-axis | 180 μg/√Hz | 0.008 °/s/√Hz | 925 μA | Cost-effective |

### Sensor Fusion

Modern IMUs often include hardware sensor fusion providing:

- **Orientation**: Quaternion or Euler angles
- **Activity classification**: Built-in step counter, activity detection
- **Gesture engine**: Programmable gesture detection
- **Wake-on-motion**: Ultra-low power motion trigger

#### Sensor Fusion Algorithms

| Algorithm | Description | Use Case |
|-----------|-------------|----------|
| Complementary Filter | Simple, low compute | Basic orientation |
| Kalman Filter | Optimal estimation | Drift compensation |
| Madgwick/Mahony | Efficient quaternion | Real-time 3D tracking |

## Interface Considerations

### I2C vs SPI

| Aspect | I2C | SPI |
|--------|-----|-----|
| Pins | 2 (SDA, SCL) | 4 (MOSI, MISO, CLK, CS) |
| Speed | 100-400 kHz (standard) | 1-10 MHz |
| Multi-device | Yes (addresses) | Yes (multiple CS) |
| Power | Slightly higher (pull-ups) | Lower |
| Best for | Low data rate, multiple sensors | High ODR, single sensor |

### FIFO Buffers

Most motion sensors include FIFO buffers (32-2048 samples):

**Benefits**:
- MCU can sleep while sensor collects data
- Batch processing reduces wake cycles
- Watermark interrupts for efficient timing

**Example Power Savings**:
```
Without FIFO: MCU wakes 50x/second = ~500 μA average
With FIFO:    MCU wakes 1x/second  = ~20 μA average
```

## PCB Layout Guidelines

### Placement
- Mount sensor near watch center (reduces arm rotation error)
- Align sensor axes with watch axes
- Keep away from motors/haptics (vibration)

### Routing
- Keep I2C/SPI traces short (<25mm)
- Add decoupling capacitors (100nF) at sensor VDD
- Consider ground plane under sensor

### Stress Isolation
- Use mechanical isolation from case flex
- Avoid placing near case attachment points
- Consider using flex PCB for sensor module

## Noise and Calibration

### Sources of Error

| Error Type | Source | Mitigation |
|------------|--------|------------|
| Bias offset | Manufacturing variation | Factory calibration |
| Scale factor | Temperature, aging | Runtime calibration |
| Cross-axis | Mechanical alignment | Calibration matrix |
| Vibration | Motor, external | Software filtering |
| Thermal | Temperature gradients | Temperature compensation |

### Calibration Procedure

1. **Static calibration**: Place sensor in 6 known orientations
2. **Calculate offsets**: Average readings vs. expected values
3. **Calculate scale**: Compare measured vs. expected g-force
4. **Store parameters**: Save to non-volatile memory

## Summary

Motion sensors are the foundation of smartwatch activity tracking. Key takeaways:

- **Accelerometers**: Ultra-low power (μA), always-on capability
- **Gyroscopes**: Higher power (mA), use sparingly with motion triggers
- **IMUs**: Integrated solutions with built-in fusion algorithms
- **FIFO usage**: Critical for power optimization
- **Calibration**: Essential for accuracy

---

**Previous Section**: [System Architecture](01_system_architecture.md)
**Next Section**: [Optical Sensors](03_optical_sensors.md) - PPG, SpO2, and ambient light sensing
