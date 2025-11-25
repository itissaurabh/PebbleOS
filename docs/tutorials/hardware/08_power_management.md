# Section 8: Power Management and Battery Considerations

## Introduction

Power management is critical for wearable devices where battery life directly impacts user experience. This section covers battery selection, power architecture design, energy budgeting, and optimization techniques for achieving week-long battery life in a smartwatch form factor.

## Battery Technologies

### Lithium-Ion Chemistry Comparison

| Chemistry | Voltage | Energy Density | Cycle Life | Safety |
|-----------|---------|----------------|------------|--------|
| LiCoO₂ (LCO) | 3.7V nom | 150-200 Wh/kg | 500-1000 | Moderate |
| LiFePO₄ (LFP) | 3.2V nom | 90-120 Wh/kg | 2000+ | High |
| Li-polymer | 3.7V nom | 150-200 Wh/kg | 500-1000 | Good |
| Li-ion (NMC) | 3.6V nom | 150-220 Wh/kg | 1000-2000 | Moderate |

### Wearable Battery Specifications

| Form Factor | Capacity | Dimensions | Typical Use |
|-------------|----------|------------|-------------|
| Coin cell (rechargeable) | 30-100 mAh | 12-20mm dia, 2-4mm thick | Small wearables |
| Pouch cell (small) | 100-200 mAh | Custom shapes | Smartwatches |
| Pouch cell (medium) | 200-400 mAh | Custom shapes | Feature watches |
| Curved cell | 100-300 mAh | Watch-shaped | Premium wearables |

### Battery Selection Criteria

| Parameter | Typical Requirement | Notes |
|-----------|-------------------|-------|
| Nominal voltage | 3.7V | Standard Li-ion |
| Capacity | 200-400 mAh | 5-10 day target |
| Max discharge | 1-2C | Peak 400-800 mA |
| Operating temp | -10°C to 45°C | Skin contact constraint |
| Cycle life | >500 cycles | 2+ year lifespan |
| Self-discharge | <3%/month | Shelf life |

### Voltage Curves

```
Voltage (V)
4.2 ┤ ─────╮
    │      ╰──────╮
4.0 ┤             ╰────╮
    │                  ╰────╮
3.7 ┤                       ╰────────────────╮
    │                                        ╰────╮
3.4 ┤                                             ╰───╮
    │                                                 ╰──╮
3.0 ┤                                                    ╰─
    │
    └─────────────────────────────────────────────────────
    0%   10%   20%   30%   40%   50%   60%   70%   80%   90%  100%
                         State of Charge

Note: Most capacity between 3.5V and 4.1V
      Below 3.0V: Risk of damage
      Above 4.2V: Safety hazard
```

## Power Architecture

### System Power Tree

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          POWER ARCHITECTURE                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌────────┐    ┌────────────┐                                               │
│  │Battery │───▶│  Charger   │◀─── USB/Wireless                              │
│  │3.0-4.2V│    │ + Fuel     │                                               │
│  └───┬────┘    │   Gauge    │                                               │
│      │         └─────┬──────┘                                               │
│      │               │                                                       │
│      └───────┬───────┘                                                      │
│              │ VBAT                                                         │
│              ▼                                                               │
│      ┌───────────────┐                                                      │
│      │     PMIC      │                                                      │
│      ├───────────────┤                                                      │
│      │ DC-DC  │ LDO  │                                                      │
│      └───┬────┴──┬───┘                                                      │
│          │       │                                                           │
│          ▼       ▼                                                           │
│     ┌────────┐ ┌────────┐                                                   │
│     │ 1.8V   │ │ 3.3V   │                                                   │
│     │ Core   │ │ I/O    │                                                   │
│     └───┬────┘ └───┬────┘                                                   │
│         │          │                                                         │
│         ▼          ▼                                                         │
│    MCU Core    Sensors, Display, Wireless                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Regulator Types

| Type | Efficiency | Noise | Dropout | Use Case |
|------|------------|-------|---------|----------|
| Buck (DC-DC) | 85-95% | Higher | N/A | High current rails |
| Boost (DC-DC) | 80-90% | Higher | N/A | LED drivers |
| LDO | 60-90%* | Lower | 100-300mV | Low noise rails |
| Charge pump | 70-85% | Medium | N/A | Small loads |

*LDO efficiency = Vout/Vin

### Power Domain Strategy

| Domain | Voltage | Current | When Active |
|--------|---------|---------|-------------|
| Always-on | 1.8V | 10-50 μA | Continuously |
| Core | 1.2V | 1-10 mA | Processing |
| Sensors | 1.8/3.3V | 100 μA-2 mA | Measurement |
| Wireless | 1.8/3.3V | 5-15 mA | Communication |
| Display | 3.3V | 0.5-5 mA | Screen on |
| Haptic | VBAT | 50-200 mA | Notifications |

## Energy Budgeting

### Daily Energy Budget Example

Target: 7-day battery life with 300 mAh battery
Daily budget: 300 mAh / 7 days = 43 mAh/day

| Component | Active | Duty Cycle | Daily mAh |
|-----------|--------|------------|-----------|
| MCU sleep | 10 μA | 95% | 0.2 |
| MCU active | 5 mA | 5% | 6.0 |
| Accelerometer | 10 μA | 100% | 0.2 |
| PPG (HR) | 500 μA | 2% | 0.2 |
| Display (MIP) | 50 μA | 100% | 1.2 |
| Display backlight | 5 mA | 5% | 6.0 |
| BLE advertising | 100 μA | 100% | 2.4 |
| BLE connected | 5 mA | 2% | 2.4 |
| PMIC quiescent | 20 μA | 100% | 0.5 |
| Notifications | 50 mA | 0.1% | 1.2 |
| **Total** | | | **~20 mAh** |

Margin: 43 - 20 = 23 mAh for additional features

### Activity-Based Budgeting

| User Activity | Duration | Extra Consumption |
|---------------|----------|-------------------|
| Workout (GPS) | 1 hr/day | 15-25 mAh |
| Music control | 0.5 hr/day | 2-3 mAh |
| Notifications | 50/day | 1-2 mAh |
| Always-on display | 8 hr/day | 10-15 mAh |
| SpO2 measurement | 5 min/day | 0.5-1 mAh |

## Power Management IC (PMIC)

### PMIC Functions

| Function | Description |
|----------|-------------|
| Battery charger | CC/CV charging |
| Fuel gauge | State of charge estimation |
| DC-DC converter | High efficiency voltage conversion |
| LDO | Low noise voltage regulation |
| Load switch | Power domain control |
| Power path | Seamless USB/battery transition |
| Protection | OVP, OCP, thermal shutdown |

### Popular PMICs for Wearables

| Part | Channels | Charger | Fuel Gauge | Current |
|------|----------|---------|------------|---------|
| MAX77650 | 1 buck + 2 LDO | Yes | Optional | 8 μA quiescent |
| BQ25125 | 1 buck + 1 LDO | Yes | No | 700 nA |
| AXP2101 | 2 buck + 4 LDO | Yes | Yes | 20 μA |
| DA9062 | 2 buck + 4 LDO | Yes | No | 25 μA |

### Fuel Gauge Accuracy

| Method | Accuracy | Complexity |
|--------|----------|------------|
| Voltage-based | ±15% | Low |
| Coulomb counting | ±5-10% | Medium |
| ModelGauge | ±3-5% | Low (IC) |
| Impedance track | ±1-3% | High (IC) |

## Charging Systems

### Charging Methods

| Method | Power | Convenience | Complexity |
|--------|-------|-------------|------------|
| Pogo pins | 5W+ | Dock required | Low |
| Magnetic | 2-5W | Easy alignment | Medium |
| Wireless (Qi) | 1-5W | Universal | High |
| USB-C | 5-15W | Standard cable | Medium |

### Charging Profile

```
Current/Voltage
    │
    │   ┌────────────────────────┐
    │   │     CC Phase           │
    │   │   (Constant Current)   │
Imax├───┤                        │
    │   │                        │    ┌────────────────
    │   │                        │    │   CV Phase
    │   │                        │    │ (Constant Voltage)
    │   │                        │    │
    │   │                        │    │
    │   │                        ╰────┤
    │   │                             │
    │   │                             ╰──────────────
    │   │                                   Termination
    └───┴────────────────────────────────────────────▶ Time

    CC: Charge at maximum current (e.g., 0.5C)
    CV: Hold at 4.2V, current tapers
    Terminate: When current falls below threshold (e.g., C/20)
```

### Thermal Considerations

| Charging State | Heat Generation | Limit |
|----------------|-----------------|-------|
| Fast charge | Highest | Battery temp <45°C |
| Normal charge | Moderate | Skin temp <43°C |
| Trickle charge | Low | Safe indefinitely |

## Power Modes

### MCU Power Modes

| Mode | Current | Wake Time | Retention |
|------|---------|-----------|-----------|
| Run | 1-10 mA | N/A | Full |
| Sleep | 100 μA - 1 mA | <1 μs | Full |
| Stop | 1-10 μA | 10-100 μs | RAM only |
| Standby | 0.5-2 μA | 100 μs - 1 ms | RTC only |
| Shutdown | 10-100 nA | ms (reboot) | None |

### Sensor Power Modes

| Sensor | Active | Low-power | Standby |
|--------|--------|-----------|---------|
| Accelerometer | 50 μA | 2 μA | 0.5 μA |
| PPG AFE | 500 μA | 10 μA | 0.5 μA |
| Pressure | 5 μA | 0.5 μA | 0.1 μA |
| Temperature | 5 μA | 1 μA | 0.1 μA |

### System Power States

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        SYSTEM POWER STATES                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐                  │
│  │    ACTIVE   │◀────▶│    IDLE     │◀────▶│    SLEEP    │                  │
│  │   10+ mA    │      │   1-5 mA    │      │  50-200 μA  │                  │
│  │             │      │             │      │             │                  │
│  │ User        │      │ Display on  │      │ Display off │                  │
│  │ interaction │      │ Background  │      │ Monitoring  │                  │
│  │ Workout     │      │ monitoring  │      │ only        │                  │
│  └─────────────┘      └─────────────┘      └──────┬──────┘                  │
│                                                    │                         │
│                                             ┌──────┴──────┐                  │
│                                             │  DEEP SLEEP │                  │
│                                             │   <20 μA    │                  │
│                                             │             │                  │
│                                             │ RTC only    │                  │
│                                             │ Motion wake │                  │
│                                             └─────────────┘                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Energy Harvesting

### Harvesting Sources

| Source | Power Available | Form Factor | Challenges |
|--------|-----------------|-------------|------------|
| Solar (indoor) | 10-50 μW/cm² | Watch face | Inconsistent |
| Solar (outdoor) | 1-10 mW/cm² | Watch face | Size limited |
| Thermoelectric | 10-100 μW | Body heat | Low ΔT |
| Kinetic | 1-10 mW | Motion | Intermittent |

### Practical Considerations

| Aspect | Reality |
|--------|---------|
| Primary power | Rarely sufficient alone |
| Supplemental | Can extend battery 10-30% |
| Complexity | Added circuits, cost |
| User expectation | Still need regular charging |

## Optimization Techniques

### Hardware Optimizations

| Technique | Savings | Complexity |
|-----------|---------|------------|
| Efficient DC-DC | 5-15% | Low |
| Power gating | 10-30% | Medium |
| Voltage scaling | 10-20% | Medium |
| Low-power components | 20-50% | Low |

### Software Optimizations

| Technique | Savings | Implementation |
|-----------|---------|----------------|
| Sensor batching | 20-40% | FIFO, interrupt batching |
| Adaptive sampling | 10-30% | Context-aware rates |
| BLE optimization | 20-50% | Long intervals, batching |
| Display optimization | 30-50% | Timeout, low-power mode |

### Algorithm Optimizations

| Technique | Description |
|-----------|-------------|
| Edge processing | Process on sensor hub |
| Tiered algorithms | Simple always-on, complex on-demand |
| Event-driven | Only process on motion/touch |
| Predictive | Pre-compute expected patterns |

## Battery Safety

### Protection Requirements

| Protection | Purpose | Implementation |
|------------|---------|----------------|
| Over-voltage | Prevent >4.2V | Charger IC, protection IC |
| Under-voltage | Prevent <2.5V | PMIC cutoff |
| Over-current | Prevent short | Fuse, protection IC |
| Over-temperature | Prevent thermal runaway | NTC monitoring |

### Safety Standards

| Standard | Region | Requirement |
|----------|--------|-------------|
| UL 2054 | US | Household batteries |
| UN 38.3 | International | Transport safety |
| IEC 62133 | International | Secondary cells |
| UL 2056 | US | Power banks |

## Summary

Power management determines wearable success:

- **Battery selection**: 200-400 mAh Li-Po typical
- **PMIC integration**: Charger, regulators, fuel gauge
- **Power architecture**: Multiple domains, efficient conversion
- **Energy budgeting**: Plan for 7+ day battery life
- **Optimization**: Hardware, software, and algorithm levels
- **Safety**: Protection circuits and certification required

---

**Previous Section**: [Sensor Selection](07_sensor_selection.md)
**Next Section**: [Signal Conditioning](09_signal_conditioning.md) - Analog front-end design
