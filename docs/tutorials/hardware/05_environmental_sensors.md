# Section 5: Environmental Sensors

## Introduction

Environmental sensors measure conditions around and on the body, including temperature, barometric pressure, humidity, and altitude. These sensors provide context for health metrics and enable features like weather display, altitude tracking, and thermal comfort monitoring.

## Temperature Sensors

### Types of Temperature Sensing

| Type | Description | Accuracy | Use Case |
|------|-------------|----------|----------|
| Skin temperature | Surface contact | ±0.1-0.5°C | Fever detection, activity |
| Ambient temperature | Environmental | ±0.5-1°C | Weather, comfort |
| Core body temperature | Estimated from skin | ±0.3°C | Medical monitoring |

### Skin vs. Core Temperature

```
Core Body Temperature: ~37°C (98.6°F) - stable
                │
                ▼ Heat flow
        ┌───────────────┐
        │    Body       │
        │   Interior    │
        └───────┬───────┘
                │
        ┌───────┴───────┐
        │  Peripheral   │ - Variable based on blood flow
        │   Tissues     │
        └───────┬───────┘
                │
        ┌───────┴───────┐
        │     Skin      │ - 28-35°C typical
        └───────┬───────┘
                │
        ┌───────┴───────┐
        │   Ambient     │ - Environment dependent
        └───────────────┘

Skin temperature at wrist: 28-35°C (activity dependent)
```

### Temperature Sensor Technologies

| Technology | Principle | Accuracy | Response | Cost |
|------------|-----------|----------|----------|------|
| NTC Thermistor | Resistance vs. temp | ±0.1°C | Fast | Low |
| RTD (Pt100) | Platinum resistance | ±0.03°C | Medium | High |
| Thermocouple | Seebeck effect | ±1°C | Very fast | Low |
| IC Sensor | Bandgap reference | ±0.5°C | Medium | Medium |
| Infrared | Thermal radiation | ±0.5°C | Fast | Medium |

### Recommended Temperature ICs for Wearables

| Part | Type | Range | Accuracy | Current | Interface |
|------|------|-------|----------|---------|-----------|
| TMP117 | Digital IC | -55 to 150°C | ±0.1°C | 3.5 μA | I2C |
| MAX30208 | Clinical IC | 30 to 45°C | ±0.1°C | 6.5 μA | I2C |
| Si7051 | Digital IC | -40 to 125°C | ±0.13°C | 1 μA | I2C |
| MCP9808 | Digital IC | -40 to 125°C | ±0.25°C | 200 μA | I2C |
| NTC 10K | Thermistor | -40 to 125°C | ±0.2°C | <1 μA | Analog |

### Thermal Design Considerations

| Challenge | Impact | Mitigation |
|-----------|--------|------------|
| Self-heating | MCU raises local temp | Isolate sensor from heat sources |
| Air gap | Reduces skin contact | Ensure proper fit |
| Sweat | Evaporative cooling | Sensor placement |
| Blood flow | Temp varies with activity | Algorithm compensation |
| Ambient | Watch body absorbs heat | Shielding, calibration |

### Skin Temperature Placement

| Location | Advantages | Disadvantages |
|----------|------------|---------------|
| Watch back (center) | Good skin contact | Heat from electronics |
| Watch back (edge) | Less PCB heat | Variable contact |
| Strap sensor | Away from electronics | Contact reliability |
| Dual sensor | Compensation possible | Added cost |

## Barometric Pressure Sensors

### Applications

| Application | Precision Needed | Update Rate |
|-------------|-----------------|-------------|
| Altitude tracking | ±10 Pa (~1m) | 1-10 Hz |
| Stair counting | ±10 Pa | 10 Hz |
| Weather trends | ±100 Pa | 0.1 Hz |
| Activity context | ±50 Pa | 1 Hz |

### Pressure to Altitude Conversion

```
Standard atmosphere formula:

h = (T₀/L) × [1 - (P/P₀)^(R×L/g×M)]

Where:
- h = altitude (m)
- T₀ = 288.15 K (sea level temp)
- L = 0.0065 K/m (lapse rate)
- P = measured pressure
- P₀ = 101325 Pa (sea level pressure)
- R = 8.31447 J/(mol·K)
- g = 9.80665 m/s²
- M = 0.0289644 kg/mol

Simplified: Δh ≈ 8.3 m per hPa near sea level
```

### Barometric Sensor Types

| Type | Principle | Resolution | Accuracy |
|------|-----------|------------|----------|
| Piezoresistive | Membrane deflection | 0.01 Pa | ±100 Pa |
| Capacitive | Capacitance change | 0.1 Pa | ±50 Pa |
| Resonant | Frequency shift | 0.001 Pa | ±10 Pa |

### Recommended Barometric Sensors

| Part | Range | Resolution | Accuracy | Current | Features |
|------|-------|------------|----------|---------|----------|
| BMP390 | 300-1250 hPa | 0.003 Pa | ±0.5 hPa | 3.4 μA | Temperature compensated |
| LPS22HH | 260-1260 hPa | 0.004 Pa | ±0.5 hPa | 4 μA | Water resistant |
| DPS310 | 300-1200 hPa | 0.06 Pa | ±1 hPa | 1.7 μA | Low power |
| MS5611 | 10-1200 mbar | 0.012 mbar | ±1.5 mbar | 1 μA | High resolution |

### Altitude Accuracy Factors

| Factor | Error Contribution | Mitigation |
|--------|-------------------|------------|
| Weather changes | ~10m per hPa | Reference pressure updates |
| Temperature | ~0.3%/°C | Temperature compensation |
| Humidity | ~0.4% | Correction factor |
| Sensor noise | ±1-2m | Averaging/filtering |
| Calibration drift | ±2m/year | Periodic recalibration |

### FIFO and Filtering

Most modern pressure sensors include:
- FIFO buffers (32+ samples)
- Integrated IIR filters
- Temperature compensation
- Interrupt on pressure change

## Humidity Sensors

### Applications in Wearables

| Application | Measurement | Use Case |
|-------------|-------------|----------|
| Comfort index | RH + Temp | Heat stress warning |
| Sweat detection | Sudden RH increase | Activity context |
| Skin hydration | Surface RH | Skin health |
| Weather | Ambient RH | Forecast display |

### Humidity Sensor Types

| Type | Principle | Range | Accuracy | Response |
|------|-----------|-------|----------|----------|
| Capacitive | Polymer dielectric | 0-100% RH | ±2% | 1-30s |
| Resistive | Ionic conductivity | 20-90% RH | ±3% | 10-60s |
| Thermal | Heat dissipation | 0-100% RH | ±3% | 1-5s |

### Recommended Humidity Sensors

| Part | RH Range | Accuracy | Temp Accuracy | Current | Features |
|------|----------|----------|---------------|---------|----------|
| SHT40 | 0-100% | ±1.8% | ±0.2°C | 1.5 μA | Fast response |
| HDC2080 | 0-100% | ±2% | ±0.2°C | 0.55 μA | Low power |
| BME280 | 0-100% | ±3% | ±1°C | 1.8 μA | P+T+RH combo |
| SHTC3 | 0-100% | ±2% | ±0.2°C | 0.2 μA | Ultra-low power |

### Combined Environmental Sensors

Many applications benefit from integrated sensors:

| Part | Sensors | Power | Interface | Notes |
|------|---------|-------|-----------|-------|
| BME280 | P, T, RH | 3.6 μA | I2C/SPI | Popular combo |
| BME680 | P, T, RH, Gas | 2.1 μA | I2C/SPI | Air quality |
| MS8607 | P, T, RH | 2.5 μA | I2C | High accuracy |
| ENS210 | T, RH | 1.6 μA | I2C | Humidity focused |

## UV Sensors

### UV Index Measurement

| UV Index | Category | Protection Needed |
|----------|----------|------------------|
| 0-2 | Low | None |
| 3-5 | Moderate | Shade, sunscreen |
| 6-7 | High | Sun protection |
| 8-10 | Very High | Extra protection |
| 11+ | Extreme | Avoid sun exposure |

### UV Sensor Types

| Wavelength | Range | Detection |
|------------|-------|-----------|
| UVA | 315-400 nm | Aging, indirect damage |
| UVB | 280-315 nm | Burns, vitamin D |
| UVC | 100-280 nm | (Blocked by atmosphere) |

### Recommended UV Sensors

| Part | Channels | Range | Sensitivity | Current |
|------|----------|-------|-------------|---------|
| VEML6075 | UVA, UVB | 0-20 UV Index | High | 100 μA |
| SI1145 | UV Index | 0-20+ | Medium | 150 μA |
| LTR-390UV | UVS, ALS | 0-20+ | High | 60 μA |
| GUVA-S12SD | UVA | Wide | Analog | 50 μA |

## Gas Sensors (Air Quality)

### Applications

| Parameter | Sensor Type | Use Case |
|-----------|-------------|----------|
| CO₂ | NDIR, MOX | Indoor air quality |
| VOCs | MOX | Air freshness |
| CO | Electrochemical | Safety |
| Particulates | Optical | Pollution |

### Compact Gas Sensors

| Part | Detection | Response | Power | Size |
|------|-----------|----------|-------|------|
| BME680 | VOC, CO₂(eq) | 1s | 0.9 mA | 3x3mm |
| SGP40 | VOC Index | 10s | 2.6 mA | 2.4x2.4mm |
| CCS811 | VOC, CO₂(eq) | 1s | 30 mA | 2.7x4mm |
| ENS160 | AQI, VOC, CO₂(eq) | 1s | 10 mA | 3x3mm |

**Note**: Most compact VOC sensors provide equivalent CO₂ (eCO₂) estimates, not direct CO₂ measurement. True CO₂ sensing requires larger NDIR sensors.

## Sensor Placement Strategy

### Thermal Isolation

```
┌─────────────────────────────────────────┐
│              Watch Back                  │
│  ┌─────────┐              ┌─────────┐   │
│  │  PPG    │              │  Temp   │   │
│  │ Sensor  │              │ Sensor  │   │
│  └────┬────┘              └────┬────┘   │
│       │                        │        │
│   ════════ Thermal barrier ════════     │
│       │                        │        │
│  ┌────┴────┐              ┌────┴────┐   │
│  │   LED   │              │  MCU    │   │
│  │ Drivers │              │  PMIC   │   │
│  └─────────┘              └─────────┘   │
│              Main PCB                    │
└─────────────────────────────────────────┘
```

### Environmental Considerations

| Sensor | Optimal Placement | Avoid |
|--------|------------------|-------|
| Temperature | Skin contact, isolated from PCB | Near MCU, battery |
| Pressure | Sealed with vent hole | Water ingress paths |
| Humidity | Exposed to air | Sealed compartments |
| UV | External facing | Under cover glass |
| ALS | Near display | PPG LEDs |

## Data Fusion Examples

### Altitude Tracking

```
                   ┌──────────┐
  Barometer ──────▶│          │
                   │  Fusion  │──────▶ Accurate
  GPS altitude ───▶│Algorithm │       Altitude
                   │          │
  Temperature ────▶│          │
                   └──────────┘
```

### Heat Stress Index

```
  Skin Temp ────▶ ┌──────────┐
                  │          │
  Ambient RH ────▶│  Heat    │──────▶ Warning
                  │  Index   │       Threshold
  Activity ──────▶│          │
                  └──────────┘
```

## Summary

Environmental sensors provide critical context for wearable health devices:

- **Temperature**: Skin vs. core measurement, thermal design critical
- **Pressure**: ±1 hPa accuracy for reliable altitude
- **Humidity**: Combined sensors common, fast response preferred
- **UV**: Simple integration, useful for outdoor activity
- **Placement**: Thermal isolation and exposure considerations

---

**Previous Section**: [Electrical Biosensors](04_electrical_biosensors.md)
**Next Section**: [Chemical Sensors](06_chemical_sensors.md) - Glucose, lactate, and sweat analysis
