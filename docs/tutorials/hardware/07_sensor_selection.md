# Section 7: Sensor Selection Criteria and Specifications

## Introduction

Selecting the right sensors for a wearable device requires balancing multiple factors: accuracy, power consumption, size, cost, and integration complexity. This section provides a framework for evaluating sensors and making informed selection decisions.

## Key Specification Parameters

### Resolution and Precision

| Term | Definition | Example |
|------|------------|---------|
| Resolution | Smallest detectable change | 14-bit ADC = 0.006% of range |
| Precision | Repeatability of measurements | ±0.1°C variation |
| Accuracy | Closeness to true value | ±0.5°C absolute error |
| LSB | Least Significant Bit | 1 LSB = Range / 2^bits |

#### Resolution Requirements by Application

| Application | Minimum Resolution | Typical Sensor |
|-------------|-------------------|----------------|
| Step counting | ±0.1 g | 10-bit accelerometer |
| Heart rate | 0.1% optical signal | 16-bit ADC |
| Blood pressure | 1 mmHg | 18-bit ADC |
| ECG morphology | 10 μV | 24-bit ADC |
| Temperature | 0.01°C | 16-bit sensor |

### Noise Specifications

| Specification | Unit | Description |
|---------------|------|-------------|
| Noise density | μg/√Hz, μV/√Hz | Noise per bandwidth |
| RMS noise | μg, μV | Total noise in bandwidth |
| Peak-to-peak | μg, μV | Observed noise envelope |
| SNR | dB | Signal-to-noise ratio |

#### Noise Calculation Example

```
Given: Accelerometer noise density = 100 μg/√Hz
       Bandwidth = 100 Hz

RMS Noise = 100 μg/√Hz × √100 Hz = 1000 μg = 1 mg

For 95% confidence (2σ): Peak noise ≈ 2 mg
```

### Power Consumption

| Mode | Description | Typical Range |
|------|-------------|---------------|
| Active | Full operation | 100 μA - 10 mA |
| Low-power | Reduced ODR/resolution | 10-100 μA |
| Sleep/standby | Registers retained | 1-10 μA |
| Power-down | Minimum leakage | 0.1-1 μA |

#### Power Budget Allocation

| Subsystem | Budget % | Typical Current |
|-----------|----------|-----------------|
| MCU | 40-50% | 1-5 mA active |
| Display | 20-30% | 0.5-5 mA |
| Wireless | 15-25% | 5-15 mA TX |
| Sensors | 5-15% | 100 μA - 2 mA |
| PMIC overhead | 5-10% | 10-50 μA |

### Sampling Rate (ODR)

| Application | Minimum ODR | Typical ODR |
|-------------|-------------|-------------|
| Step counting | 25 Hz | 50 Hz |
| Gesture recognition | 50 Hz | 100-200 Hz |
| Heart rate | 25 Hz | 50-100 Hz |
| ECG | 125 Hz | 250-500 Hz |
| SpO2 | 25 Hz | 50-100 Hz |
| Inactivity detection | 1 Hz | 12.5 Hz |

### Dynamic Range

| Sensor | Range Options | Use Case |
|--------|---------------|----------|
| Accelerometer | ±2g, ±4g, ±8g, ±16g | Step/activity: ±8g |
| Gyroscope | ±250, ±500, ±1000, ±2000 °/s | Gesture: ±500 °/s |
| Pressure | 260-1260 hPa | Altitude tracking |
| Temperature | -40 to +85°C | Industrial |
| Temperature | +25 to +45°C | Clinical |

## Interface Selection

### Digital Interfaces

| Interface | Speed | Pins | Power | Multi-Device |
|-----------|-------|------|-------|--------------|
| I2C | 100-400 kHz (std) | 2 | Medium | Yes (address) |
| I2C | 1-3.4 MHz (fast) | 2 | Medium | Yes |
| SPI | 1-20 MHz | 4 | Lower | Yes (CS per device) |
| I3C | 12.5 MHz | 2 | Low | Yes |

#### Interface Selection Guide

| Priority | Choose I2C When | Choose SPI When |
|----------|-----------------|-----------------|
| Pin count | Limited GPIO | Pins available |
| Data rate | <400 Hz ODR | >1 kHz ODR |
| Distance | <10 cm trace | <5 cm trace |
| Devices | Multiple sensors | Single sensor |
| Power | Less critical | Very critical |

### Analog Interface Considerations

| Parameter | Consideration |
|-----------|---------------|
| Output range | Match to ADC input range |
| Output impedance | Lower is better for ADC driving |
| Bandwidth | Must exceed signal bandwidth |
| Noise | Contributes to system noise floor |

## Physical Specifications

### Package Size Comparison

| Package | Typical Size | Height | Notes |
|---------|-------------|--------|-------|
| LGA | 2×2×0.75 mm | 0.75 mm | Standard wearable |
| LGA | 3×3×1 mm | 1 mm | Larger sensors |
| QFN | 3×3×0.9 mm | 0.9 mm | Common option |
| Wafer-level | 1.5×1.5×0.5 mm | 0.5 mm | Smallest |
| Module | 10×10×2 mm | 2 mm | Integrated |

### Mounting Considerations

| Factor | Impact |
|--------|--------|
| Reflow profile | Temperature sensitivity |
| Moisture sensitivity | MSL rating |
| Mechanical stress | Affects calibration |
| Thermal expansion | CTE mismatch |

## Sensor Comparison Tables

### Accelerometer Comparison

| Parameter | Budget | Standard | Premium |
|-----------|--------|----------|---------|
| Part | LIS2DH12 | BMA456 | BMI270 (accel) |
| Resolution | 12-bit | 16-bit | 16-bit |
| Noise | 4 mg | 120 μg | 160 μg |
| Current (active) | 2 μA | 14 μA | 45 μA |
| Current (low-power) | 2 μA | 3.5 μA | 3.5 μA |
| FIFO | 32 samples | 1024 samples | 6144 bytes |
| Features | Basic | Step counter | Wearable features |
| Price tier | $ | $$ | $$$ |

### PPG Sensor Comparison

| Parameter | Basic | Mid-range | Premium |
|-----------|-------|-----------|---------|
| Part | MAX30102 | MAX30101 | ADPD4100 |
| LEDs | Red, IR | Red, IR, Green | 8 drivers |
| Photodiodes | 1 | 1 | 4 inputs |
| ADC | 18-bit | 18-bit | 14-bit |
| Current | 600 μA | 600 μA | 60 μA (standby) |
| SpO2 | Yes | Yes | Configurable |
| HR | Yes | Yes | Yes |
| Price tier | $ | $$ | $$$ |

### Temperature Sensor Comparison

| Parameter | Budget | Standard | Clinical |
|-----------|--------|----------|----------|
| Part | TMP102 | TMP117 | MAX30208 |
| Range | -40 to +125°C | -55 to +150°C | +30 to +45°C |
| Accuracy | ±2°C | ±0.1°C | ±0.1°C |
| Resolution | 12-bit | 16-bit | 16-bit |
| Current | 10 μA | 3.5 μA | 6.5 μA |
| Price tier | $ | $$ | $$$ |

## Decision Framework

### Step 1: Define Requirements

| Category | Questions to Answer |
|----------|-------------------|
| Function | What will you measure? |
| Accuracy | What precision is needed? |
| Power | What is the battery budget? |
| Form factor | Size and weight constraints? |
| Cost | Volume and price targets? |
| Regulatory | Medical device or consumer? |

### Step 2: Create Weighted Matrix

| Criterion | Weight | Sensor A | Sensor B | Sensor C |
|-----------|--------|----------|----------|----------|
| Accuracy | 25% | 8 | 9 | 7 |
| Power | 20% | 9 | 7 | 8 |
| Size | 15% | 7 | 8 | 9 |
| Cost | 15% | 9 | 5 | 7 |
| Features | 15% | 6 | 9 | 8 |
| Supply chain | 10% | 8 | 7 | 6 |
| **Weighted Score** | 100% | **7.8** | **7.5** | **7.5** |

### Step 3: Risk Assessment

| Risk | Mitigation |
|------|------------|
| Single source | Qualify alternate part |
| New technology | Request samples, evaluate |
| Supply shortage | Design for multiple options |
| End of life | Check lifecycle status |

## Vendor Considerations

### Major Sensor Vendors

| Category | Key Vendors |
|----------|-------------|
| Motion (IMU) | Bosch, STMicroelectronics, TDK/InvenSense |
| Optical (PPG) | Maxim/ADI, ams OSRAM, Silan |
| Temperature | TI, Maxim/ADI, Sensirion |
| Pressure | Bosch, STMicroelectronics, Infineon |
| Environmental | Sensirion, Bosch, ams |

### Evaluation Checklist

| Item | Check |
|------|-------|
| Datasheet review | Complete specs available? |
| Application notes | Design guidance provided? |
| Reference design | Evaluation board available? |
| Software support | Drivers, libraries available? |
| Technical support | FAE responsiveness? |
| Supply chain | Distribution availability? |
| Roadmap | Future products planned? |

## Cost Considerations

### Bill of Materials Impact

| Component | % of BOM | Cost Sensitivity |
|-----------|----------|------------------|
| Display | 25-35% | High |
| Battery | 10-15% | Medium |
| MCU | 10-20% | Medium |
| Sensors | 10-20% | Medium-High |
| PMIC | 5-10% | Medium |
| Passives | 5-10% | Low |

### Volume Pricing Trends

| Volume | Typical Discount |
|--------|------------------|
| 1-100 | List price |
| 100-1K | 10-20% off |
| 1K-10K | 20-35% off |
| 10K-100K | 35-50% off |
| 100K+ | Negotiated |

## Summary

Effective sensor selection requires:

- **Define specifications**: Resolution, noise, power, ODR requirements
- **Evaluate trade-offs**: Performance vs. power vs. cost
- **Consider integration**: Interface, package, software support
- **Assess risks**: Supply chain, lifecycle, technology maturity
- **Prototype early**: Validate performance in actual application

---

**Previous Section**: [Chemical Sensors](06_chemical_sensors.md)
**Next Section**: [Power Management](08_power_management.md) - Battery and power system design
