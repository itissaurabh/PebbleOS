# Section 1: System Architecture Overview

## Introduction

Designing hardware for a smartwatch or wearable health device requires careful consideration of multiple interdependent subsystems. This tutorial provides a comprehensive guide to the hardware considerations for building a smartwatch-type wearable device, covering sensors, power management, signal processing, and regulatory requirements.

## Wearable Hardware Block Diagram

A typical smartwatch system architecture consists of the following major blocks:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           SMARTWATCH SYSTEM                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                   │
│  │   SENSORS    │    │  PROCESSING  │    │   OUTPUT     │                   │
│  ├──────────────┤    ├──────────────┤    ├──────────────┤                   │
│  │ • IMU/Accel  │    │              │    │ • Display    │                   │
│  │ • PPG/SpO2   │───▶│     MCU      │───▶│ • Haptic     │                   │
│  │ • ECG        │    │   (ARM M4+)  │    │ • Audio      │                   │
│  │ • Temp/Baro  │    │              │    │              │                   │
│  │ • Ambient    │    └──────┬───────┘    └──────────────┘                   │
│  └──────────────┘           │                                                │
│                             │                                                │
│  ┌──────────────┐    ┌──────┴───────┐    ┌──────────────┐                   │
│  │    POWER     │    │   WIRELESS   │    │   STORAGE    │                   │
│  ├──────────────┤    ├──────────────┤    ├──────────────┤                   │
│  │ • PMIC       │    │ • BLE 5.x    │    │ • Flash      │                   │
│  │ • Battery    │    │ • NFC        │    │ • EEPROM     │                   │
│  │ • Charging   │    │ • (WiFi)     │    │              │                   │
│  │ • Harvesting │    │              │    │              │                   │
│  └──────────────┘    └──────────────┘    └──────────────┘                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Component Categories Overview

### 1. Sensor Subsystem

The sensor subsystem is the heart of health monitoring functionality:

| Sensor Type | Primary Function | Power Range | Interface |
|-------------|-----------------|-------------|-----------|
| Accelerometer | Motion, activity | 1-10 μA | I2C/SPI |
| Gyroscope | Rotation, gesture | 1-5 mA | I2C/SPI |
| PPG (Optical) | Heart rate, SpO2 | 100 μA - 1mA | I2C |
| ECG | Cardiac rhythm | 50-500 μA | SPI/Analog |
| Temperature | Skin/body temp | 1-10 μA | I2C |
| Barometer | Altitude, weather | 1-5 μA | I2C/SPI |
| Ambient Light | Auto-brightness | 1-5 μA | I2C |

### 2. Processing Subsystem

The main processor requirements for a health-focused wearable:

- **MCU**: ARM Cortex-M4F or higher recommended
  - Hardware FPU for signal processing
  - DSP instructions for filtering
  - 64KB+ SRAM for sensor buffers
  - 256KB+ Flash for firmware

- **Co-processors** (optional):
  - Ultra-low-power sensor hub
  - Hardware accelerators for ML inference
  - Dedicated display controller

### 3. Communication Subsystem

Wireless connectivity options:

| Protocol | Data Rate | Range | Power | Use Case |
|----------|-----------|-------|-------|----------|
| BLE 5.0+ | 2 Mbps | 100m | Low | Primary sync |
| NFC | 424 kbps | 4cm | Very Low | Payments, pairing |
| ANT+ | 60 kbps | 30m | Low | Fitness accessories |
| WiFi | 150 Mbps | 50m | High | Firmware updates |

### 4. Power Subsystem

Critical power management components:

- **Battery**: Li-Po, 100-400 mAh typical
- **PMIC**: Integrated power management IC
  - Multiple LDO/DC-DC regulators
  - Battery charger (wireless/wired)
  - Fuel gauge
- **Power domains**: Separate domains for always-on vs. active components

### 5. User Interface

Output and interaction components:

- **Display**: AMOLED (power-hungry) or MIP/e-ink (power-efficient)
- **Touch**: Capacitive touch controller
- **Haptic**: LRA or ERM motor
- **Buttons**: Mechanical switches
- **Audio**: Optional speaker/microphone

## Design Constraints

### Form Factor

| Constraint | Typical Value | Impact |
|------------|---------------|--------|
| Case diameter | 38-46 mm | PCB area, battery size |
| Case thickness | 10-14 mm | Component stacking |
| Weight | 30-60 g | Battery capacity |
| Strap width | 18-22 mm | Connector routing |

### Environmental

- **Operating temperature**: -10°C to 45°C
- **Storage temperature**: -20°C to 60°C
- **Water resistance**: 3-5 ATM (30-50m) minimum
- **Humidity**: 0-95% RH non-condensing

### Power Budget Example

A typical daily power budget for a health wearable:

| Component | Active Current | Duty Cycle | Daily Energy |
|-----------|---------------|------------|--------------|
| MCU (active) | 10 mA | 5% | 12 mAh |
| MCU (sleep) | 10 μA | 95% | 0.2 mAh |
| Display | 2 mA | 10% | 4.8 mAh |
| PPG sensor | 500 μA | 2% | 0.2 mAh |
| Accelerometer | 10 μA | 100% | 0.2 mAh |
| BLE | 5 mA | 1% | 1.2 mAh |
| **Total** | - | - | **~20 mAh** |

With a 200 mAh battery, this gives approximately 10 days of battery life.

## System Integration Considerations

### Signal Integrity

- Keep analog sensor signals away from switching power supplies
- Use ground planes and proper shielding
- Route I2C/SPI buses with matched lengths

### Thermal Management

- PPG LEDs generate heat near skin
- MCU temperature affects sensor readings
- Battery charging generates significant heat

### Mechanical Stress

- Flex PCBs must handle repeated bending
- Sensor placement affects measurement quality
- Waterproof sealing creates assembly challenges

## Summary

This section provided an overview of the major hardware blocks in a smartwatch system. The following sections will dive deep into each sensor type, power management strategies, signal processing requirements, and regulatory considerations for bringing a wearable health device to market.

---

**Next Section**: [Motion Sensors](02_motion_sensors.md) - Accelerometers, gyroscopes, and IMUs for activity tracking
