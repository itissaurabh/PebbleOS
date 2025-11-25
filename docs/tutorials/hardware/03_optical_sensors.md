# Section 3: Optical Sensors

## Introduction

Optical sensors enable non-invasive measurement of vital signs including heart rate, blood oxygen saturation (SpO2), and respiration rate. This section covers photoplethysmography (PPG) sensors, pulse oximetry, and ambient light sensors commonly used in wearables.

## Photoplethysmography (PPG) Fundamentals

### Operating Principle

PPG measures blood volume changes in the microvascular tissue using light absorption. During cardiac systole (heartbeat), arterial blood volume increases, absorbing more light. This creates a pulsatile signal that tracks heart rate.

```
LED Light Source                    Photodetector
      │                                   │
      ▼                                   │
  ┌───────┐                              │
  │ SKIN  │ ═══════════════════════════▶ │
  ├───────┤   Reflected/Transmitted      │
  │TISSUE │   Light                      │
  ├───────┤                              │
  │BLOOD  │ ◀── Pulsatile absorption     │
  │VESSEL │                              │
  └───────┘                              │
                                         ▼
                              Pulsatile Signal (AC)
                              + Baseline (DC)
```

### Signal Components

| Component | Description | Information |
|-----------|-------------|-------------|
| AC (pulsatile) | Blood volume change | Heart rate, HRV |
| DC (baseline) | Tissue absorption | SpO2 calculation |
| Respiration modulation | Breathing effect | Respiration rate |
| Motion artifact | Movement noise | Must be filtered |

## LED Wavelengths

Different wavelengths penetrate tissue differently and have varying absorption by oxygenated vs. deoxygenated hemoglobin:

### Common Wavelengths

| Wavelength | Color | Penetration | Primary Use |
|------------|-------|-------------|-------------|
| 525-535 nm | Green | Shallow (1-2mm) | Heart rate |
| 630-660 nm | Red | Medium (2-3mm) | SpO2 (oxyHb) |
| 850-940 nm | IR | Deep (3-5mm) | SpO2 (deoxyHb) |
| 405 nm | Blue/Violet | Very shallow | Research |

### LED Selection Considerations

| Parameter | Green LED | Red LED | IR LED |
|-----------|-----------|---------|--------|
| Motion artifact | Lower | Higher | Higher |
| Dark skin performance | Moderate | Good | Good |
| Power consumption | Higher | Lower | Lowest |
| Signal amplitude | Highest | Moderate | Moderate |
| SpO2 capability | No | Required | Required |

### Recommended LED Types

| LED Type | Wavelength | Forward Voltage | Radiant Power |
|----------|------------|-----------------|---------------|
| Green (InGaN) | 530 nm | 3.0-3.5V | 10-20 mW |
| Red (AlGaInP) | 660 nm | 1.8-2.2V | 5-15 mW |
| IR (GaAs) | 880 nm | 1.3-1.6V | 15-30 mW |

## Photodetectors

### Types

| Type | Sensitivity | Bandwidth | Dark Current |
|------|-------------|-----------|--------------|
| Silicon Photodiode | 0.3-0.6 A/W | Wide (visible-NIR) | 1-100 nA |
| Avalanche PD | Higher gain | Narrower | Higher noise |
| PIN Photodiode | 0.5 A/W | Fast response | Low |

### Key Specifications for Wearables

| Parameter | Typical Value | Notes |
|-----------|---------------|-------|
| Active area | 1-3 mm² | Larger = more signal |
| Responsivity | 0.4-0.6 A/W @ 530nm | Wavelength dependent |
| Dark current | <10 nA | Affects SNR |
| NEP | 10⁻¹⁴ W/√Hz | Noise Equivalent Power |
| Capacitance | 20-100 pF | Affects bandwidth |

## Integrated PPG Sensors

Modern wearables typically use integrated analog front-end (AFE) solutions:

### Popular PPG AFE ICs

| Part | LEDs | PD | ADC | Current | Features |
|------|------|-----|-----|---------|----------|
| MAX86150 | 3 (R,IR,G) | 1 | 19-bit | 1.1mA | ECG + PPG combo |
| MAX30101 | 3 (R,IR,G) | 1 | 18-bit | 600μA | SpO2 + HR |
| ADPD4100 | 8 drivers | 4 inputs | 14-bit | 60μA | Highly configurable |
| SFH7072 | 3 (G,G,R) | 2 | Analog | External | Sensor-only |
| PAH8011 | 2 (G,IR) | 1 | 16-bit | 400μA | Low power |

### AFE Block Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    PPG Analog Front-End                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LED Drivers              Signal Chain           Digital        │
│  ┌─────────┐             ┌─────────┐           ┌─────────┐     │
│  │ Green   │──▶ LED ──▶ │   TIA   │──▶ ADC ──▶│  FIFO   │     │
│  │ Red     │             │   +     │           │    +    │     │
│  │ IR      │   ◀── PD ──│  PGA    │           │  I2C/   │     │
│  └────┬────┘             └─────────┘           │  SPI    │     │
│       │                                         └────┬────┘     │
│       │              LED Timing Control              │          │
│       └──────────────────┬───────────────────────────┘          │
│                          │                                       │
│                    ┌─────┴─────┐                                │
│                    │  Control  │                                │
│                    │  Logic    │                                │
│                    └───────────┘                                │
└─────────────────────────────────────────────────────────────────┘
```

## SpO2 Measurement

### Principle

Blood oxygen saturation is calculated using the ratio of light absorption at two wavelengths:

```
R = (AC_red / DC_red) / (AC_ir / DC_ir)

SpO2 = 110 - 25 × R  (simplified linear approximation)
```

### Calibration Requirements

SpO2 measurement requires empirical calibration:

| SpO2 Range | Accuracy Requirement | Calibration Method |
|------------|---------------------|-------------------|
| 70-100% | ±2% | Controlled desaturation study |
| 50-70% | ±3% | Limited clinical data |
| <50% | Not specified | Extrapolation only |

**Regulatory Note**: Medical-grade SpO2 requires FDA 510(k) clearance with clinical validation.

### Multi-Wavelength Benefits

| Configuration | Wavelengths | Benefits |
|--------------|-------------|----------|
| 2-wavelength | Red + IR | Basic SpO2 |
| 3-wavelength | Green + Red + IR | Motion compensation |
| 4+ wavelength | Multiple | Carboxyhemoglobin detection |

## Ambient Light Sensors

### Purpose in Wearables

- **Display brightness control**: Auto-adjust backlight
- **PPG interference rejection**: Ambient light subtraction
- **UV exposure tracking**: Sun safety features

### Types

| Type | Spectral Response | Use Case |
|------|-------------------|----------|
| Visible light | 380-700 nm | Display control |
| IR-filtered | 450-650 nm (photopic) | Human-eye response |
| UV sensor | 280-400 nm | Sun exposure |
| RGB sensor | Separate R,G,B channels | Color temperature |

### Common ALS ICs

| Part | Channels | Range | Resolution | Current |
|------|----------|-------|------------|---------|
| VEML6030 | ALS | 0-120k lux | 16-bit | 2 μA |
| TSL2591 | Visible + IR | 188M:1 range | 16-bit | 0.4 μA |
| APDS-9960 | RGBC + Proximity | 0-16k lux | 16-bit | 1.8 μA |
| LTR-329ALS | ALS | 0-64k lux | 16-bit | 0.15 μA |

## Optical Design Considerations

### LED-Photodetector Spacing

| Spacing | Effect |
|---------|--------|
| 3-5 mm | Shallow tissue sampling, less motion artifact |
| 5-8 mm | Deeper penetration, more motion artifact |
| Multiple spacing | Adaptive depth selection |

### Optical Isolation

Critical to prevent direct LED-to-PD light leakage:

```
        ┌───────────────────────────────┐
        │         OPTICAL WINDOW        │
        └───────────────────────────────┘
               │              │
               ▼              ▼
        ┌──────────┐   ┌──────────┐
        │   LED    │   │    PD    │
        └──────────┘   └──────────┘
               │              │
        ═══════════════════════════════  ◀── Optical barrier
               │              │
        ┌──────────────────────────────┐
        │            PCB               │
        └──────────────────────────────┘
```

**Methods**:
- Physical barriers (black epoxy walls)
- Recessed cavities in housing
- Optical coatings on window

### Skin Contact

| Factor | Impact | Mitigation |
|--------|--------|------------|
| Air gap | Signal loss, ambient light | Proper fit, flexible housing |
| Pressure | Occludes blood flow | Light contact force |
| Hair | Light blockage | Sensor placement |
| Motion | Artifact | Signal processing |

## Motion Artifact Rejection

Motion is the primary challenge for wrist-worn PPG:

### Hardware Approaches

| Method | Description | Effectiveness |
|--------|-------------|--------------|
| Green LED | Less sensitive to motion | Moderate |
| Multi-wavelength | Reference channel | Good |
| Accelerometer reference | Adaptive filtering | Good |
| Multiple PD | Spatial diversity | Moderate |
| Contact sensor | Detect poor contact | Good |

### Algorithm Approaches

| Algorithm | Description | Compute Cost |
|-----------|-------------|--------------|
| Moving average | Simple smoothing | Very low |
| Bandpass filter | 0.5-4 Hz for HR | Low |
| Adaptive filter (LMS) | Accelerometer-aided | Moderate |
| ICA | Blind source separation | High |
| Deep learning | Neural network | Very high |

## Power Management

PPG sensors are significant power consumers due to LED drive current:

### Power Optimization Strategies

| Strategy | Power Savings | Trade-off |
|----------|---------------|-----------|
| Reduce LED current | Linear reduction | Lower SNR |
| Reduce sample rate | Linear reduction | Temporal resolution |
| Duty cycle reduction | Significant | Minimum integration time |
| Adaptive intensity | Variable | Algorithm complexity |
| On-demand measurement | Maximum | Not continuous |

### Example Power Budget

| Parameter | Typical | Low Power |
|-----------|---------|-----------|
| LED current | 50 mA peak | 10 mA peak |
| LED duty cycle | 1% | 0.25% |
| Sample rate | 100 Hz | 25 Hz |
| Average current | 500 μA | 50 μA |

## Signal Processing Pipeline

```
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
│ Raw ADC │──▶│ Ambient │──▶│ Bandpass│──▶│ Motion  │──▶│ Peak    │
│ Samples │   │ Removal │   │ Filter  │   │ Removal │   │Detection│
└─────────┘   └─────────┘   └─────────┘   └─────────┘   └─────────┘
                                                              │
                                                              ▼
                                                        ┌─────────┐
                                                        │Heart    │
                                                        │Rate (HR)│
                                                        └─────────┘
```

## Summary

Optical sensors enable critical health metrics in wearables:

- **Green PPG**: Best motion rejection, standard for HR
- **Red + IR PPG**: Required for SpO2 measurement
- **Integrated AFEs**: Simplify design, reduce power
- **Motion artifact**: Primary challenge requiring multi-modal solutions
- **Optical design**: Critical for signal quality

---

**Previous Section**: [Motion Sensors](02_motion_sensors.md)
**Next Section**: [Electrical Biosensors](04_electrical_biosensors.md) - ECG, EDA, and bioimpedance
