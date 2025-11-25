# Section 11: PCB Design and Mechanical Considerations

## Introduction

Smartwatch design presents unique challenges in PCB layout and mechanical engineering. This section covers form factor constraints, PCB design guidelines, waterproofing, thermal management, and assembly considerations for wearable devices.

## Form Factor Constraints

### Typical Dimensions

| Component | Dimension | Notes |
|-----------|-----------|-------|
| Case diameter | 38-46 mm | Wrist size dependent |
| Case thickness | 10-14 mm | Includes crystal |
| PCB diameter | 28-36 mm | Inside case |
| PCB thickness | 0.8-1.2 mm | Rigid or flex-rigid |
| Battery space | 30-50% of volume | Primary constraint |

### Space Allocation

```
┌────────────────────────────────────────────────────────────────┐
│                      WATCH CROSS-SECTION                        │
├────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ═══════════════════════════════════════════════════ Crystal   │
│  ┌──────────────────────────────────────────────────┐ Display  │
│  │░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│          │
│  └──────────────────────────────────────────────────┘          │
│  ┌──────────────────────────────────────────────────┐ PCB 1    │
│  │▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│ (top)    │
│  └──────────────────────────────────────────────────┘          │
│  ┌──────────────────────────────────────────────────┐ Battery  │
│  │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒│          │
│  │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒│          │
│  └──────────────────────────────────────────────────┘          │
│  ┌──────────────────────────────────────────────────┐ PCB 2    │
│  │▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│ (bottom) │
│  └──────────────────────────────────────────────────┘          │
│  ┌──┐ ┌──┐                              ┌──┐ ┌──┐    Sensors   │
│  │PP│ │HR│                              │T │ │EC│              │
│  └──┘ └──┘                              └──┘ └──┘              │
│  ═══════════════════════════════════════════════════ Back      │
│                                                                  │
└────────────────────────────────────────────────────────────────┘
```

## PCB Stack-up

### Rigid PCB Options

| Layers | Thickness | Use Case |
|--------|-----------|----------|
| 4-layer | 0.8 mm | Simple designs |
| 6-layer | 1.0 mm | Standard smartwatch |
| 8-layer | 1.2 mm | Complex designs |
| 10+ layer | 1.4+ mm | Premium, high-density |

### Typical 6-Layer Stack-up

```
Layer 1: Signal (top components)         35 μm Cu
         Prepreg                         100 μm
Layer 2: Ground plane                    35 μm Cu
         Core                            200 μm
Layer 3: Signal/Power                    35 μm Cu
         Prepreg                         100 μm
Layer 4: Signal/Power                    35 μm Cu
         Core                            200 μm
Layer 5: Power plane                     35 μm Cu
         Prepreg                         100 μm
Layer 6: Signal (bottom components)      35 μm Cu

Total: ~1.0 mm
```

### Flex and Flex-Rigid PCB

| Configuration | Application |
|---------------|-------------|
| Flex | Sensor modules, interconnects |
| Flex-rigid | Main board + sensor flex |
| Multi-flex | Complex 3D routing |

### Flex PCB Considerations

| Parameter | Recommendation |
|-----------|----------------|
| Bend radius | >10× thickness |
| Copper | Rolled annealed (RA) for dynamic flex |
| Layers | 1-4 typical |
| Stiffeners | Required at connector areas |

## Component Placement

### Placement Guidelines

| Component | Location | Rationale |
|-----------|----------|-----------|
| MCU/SoC | Center top | Heat dissipation, trace length |
| Crystal | Near MCU | Short traces |
| Power (PMIC) | Edge | Easy routing, thermal |
| Antenna | Edge/corner | Ground clearance |
| Sensors | Bottom PCB | Skin contact |
| Display connector | Edge | Flat flex routing |

### Thermal Considerations

| Heat Source | Mitigation |
|-------------|------------|
| MCU | Thermal vias to inner planes |
| PMIC | Exposed pad to ground plane |
| LED drivers | Thermal relief |
| Battery charging | Distance from battery |

### 3D Component Stacking

```
        Top View                    Side View
    ┌─────────────┐            ┌─────────────┐
    │  ┌───────┐  │            │ Component   │
    │  │ MCU   │  │            │ ▓▓▓▓▓▓▓▓▓▓▓ │
    │  └───────┘  │            ├─────────────┤ PCB 1
    │ ┌───┐ ┌───┐ │            │ ▒▒▒▒▒▒▒▒▒▒▒ │
    │ │PMC│ │MEM│ │            │   Battery   │
    │ └───┘ └───┘ │            │ ▒▒▒▒▒▒▒▒▒▒▒ │
    └─────────────┘            ├─────────────┤ PCB 2
                               │ ▓▓▓▓▓▓▓▓▓▓▓ │
    Package-on-Package (PoP)   │   Sensors   │
    can save board area        └─────────────┘
```

## Antenna Design

### BLE Antenna Options

| Type | Size | Efficiency | Cost |
|------|------|------------|------|
| Chip antenna | 2×1 mm | 50-70% | Low |
| PCB trace | 15×5 mm | 60-80% | Free |
| Flex antenna | Variable | 70-85% | Medium |
| Metal case antenna | Integrated | 50-70% | High |

### Antenna Placement

```
┌──────────────────────────────────────────────────┐
│                    PCB TOP VIEW                   │
│                                                   │
│         Ground Plane                             │
│     ┌─────────────────────────┐                  │
│     │                         │                  │
│     │    Components           │                  │
│     │                         │     ┌────────┐  │
│     │                         │     │Antenna │  │
│     │                         │     │ Keep-  │  │
│     └─────────────────────────┘     │  out   │  │
│                                      │        │  │
│                                      └────────┘  │
│                                                   │
│  ← Keep ground plane away from antenna           │
│                                                   │
└──────────────────────────────────────────────────┘
```

### Antenna Keep-out Rules

| Rule | Specification |
|------|---------------|
| Ground clearance | >5 mm from antenna element |
| Component clearance | No metal within 3 mm |
| Via clearance | Minimize near antenna |
| Ground plane | Solid under feed, clear under element |

## Power Integrity

### Decoupling Strategy

| Frequency | Capacitor | Placement |
|-----------|-----------|-----------|
| Low (<100 kHz) | 10-100 μF | Near PMIC |
| Medium (100k-10M) | 0.1-1 μF | Per IC power pin |
| High (>10 MHz) | 10-100 nF | Adjacent to pins |
| Very high | 1-10 nF | Multiple per BGA |

### Power Plane Design

| Guideline | Purpose |
|-----------|---------|
| Solid planes | Low impedance |
| Split planes | Isolate analog/digital |
| Decoupling vias | Short return path |
| Wide traces | High current paths |

## Signal Integrity

### High-Speed Design Rules

| Signal | Trace Width | Impedance | Max Length |
|--------|-------------|-----------|------------|
| QSPI | 4-5 mil | 50 Ω single | <25 mm |
| I2C | 4-5 mil | N/A | <50 mm |
| SPI (1 MHz) | 4-5 mil | N/A | <50 mm |
| USB 2.0 | Differential | 90 Ω diff | <50 mm |

### Length Matching

| Interface | Matching Requirement |
|-----------|---------------------|
| QSPI | ±5 mm |
| SDIO | ±5 mm |
| USB | ±2 mm (differential) |
| I2C/SPI | Not critical |

## Waterproofing

### IP Ratings

| Rating | Protection | Test |
|--------|------------|------|
| IP54 | Splash resistant | Water spray |
| IP67 | 1m for 30 min | Immersion |
| IP68 | >1m continuous | Pressure test |
| 3 ATM | 30m equivalent | Static pressure |
| 5 ATM | 50m equivalent | Static pressure |

### Sealing Methods

| Method | Application | IP Level |
|--------|-------------|----------|
| Gaskets | Case joints | IP67+ |
| O-rings | Buttons, crown | IP67+ |
| Adhesive | Display, back | IP67+ |
| Potting | PCB protection | IP68 |
| Welding | Case seams | IP68 |

### Pressure Sensor Venting

```
┌─────────────────────────────────────────────────┐
│                PRESSURE VENT DESIGN              │
├─────────────────────────────────────────────────┤
│                                                  │
│       External                 Internal          │
│          │                        │              │
│          ▼                        ▼              │
│       ╔═════╗    ┌─────────┐   ╔═════╗          │
│       ║Vent ║────│ Gore-Tex│───║Sensor║         │
│       ║Hole ║    │Membrane │   ║      ║         │
│       ╚═════╝    └─────────┘   ╚═════╝          │
│                                                  │
│  Membrane: Waterproof, air-permeable            │
│  Typical: Gore-Tex, ePTFE                       │
│                                                  │
└─────────────────────────────────────────────────┘
```

## Sensor Window Design

### Optical Sensor Window

| Parameter | Specification |
|-----------|---------------|
| Material | Glass, sapphire, PC |
| Transmission | >90% at 530/660/880 nm |
| Thickness | 0.3-0.5 mm |
| Coating | AR coating optional |
| Adhesive | Optical-grade, biocompatible |

### Back Case Materials

| Material | Pros | Cons |
|----------|------|------|
| Glass | Optical clarity, premium | Fragile |
| Ceramic | Scratch resistant | Cost |
| Plastic (PC) | Low cost, light | Scratches |
| Sapphire | Most durable | Expensive |

## Electrode Design

### ECG Electrode Considerations

| Factor | Recommendation |
|--------|----------------|
| Material | Stainless steel 316L, Ti |
| Surface | Brushed or polished |
| Size | 5-10 mm diameter |
| Spacing | >10 mm between electrodes |
| Connection | Spring contacts or flex |

### Skin Contact Optimization

```
┌─────────────────────────────────────────────────────┐
│               WATCH BACK SENSOR LAYOUT               │
├─────────────────────────────────────────────────────┤
│                                                      │
│         ┌───────────────────────────┐               │
│         │      PPG Window           │               │
│         │  ┌───┐  ┌───┐  ┌───┐     │               │
│         │  │LED│  │ PD│  │LED│     │               │
│         │  └───┘  └───┘  └───┘     │               │
│         └───────────────────────────┘               │
│                                                      │
│    ┌────┐                           ┌────┐         │
│    │ECG │                           │ECG │         │
│    │ E1 │                           │ E2 │         │
│    └────┘                           └────┘         │
│                                                      │
│    ○ Temp sensor (thermal contact)                  │
│                                                      │
└─────────────────────────────────────────────────────┘
```

## Vibration Motor Mounting

### Motor Types

| Type | Size | Vibration | Response |
|------|------|-----------|----------|
| ERM | 8-12 mm | 1-2 G | 50-100 ms |
| LRA | 6-10 mm | 1.5-2.5 G | 10-20 ms |
| Piezo | 10-15 mm | 0.5-1 G | <5 ms |

### Mounting Considerations

| Consideration | Guideline |
|---------------|-----------|
| Isolation | Rubber mounts |
| Position | Away from sensors |
| Wiring | Flex cable preferred |
| Driver | H-bridge for ERM, sine drive for LRA |

## Thermal Management

### Heat Sources and Sinks

| Source | Power | Mitigation |
|--------|-------|------------|
| MCU | 100-500 mW | Thermal vias |
| Wireless TX | 50-200 mW | Duty cycle |
| Display | 50-200 mW | Efficient tech |
| Charging | 1-5 W | Limit charge rate |

### Thermal Simulation

| Limit | Value | Reason |
|-------|-------|--------|
| Skin temp | <43°C | Comfort, safety |
| Battery temp | <45°C | Cycle life |
| MCU junction | <85°C | Reliability |

## Assembly Considerations

### Manufacturing Process

| Step | Consideration |
|------|---------------|
| SMT | Standard reflow compatible |
| Module attachment | Pre-assembled flex modules |
| Battery | Manual or automated |
| Final assembly | Clean room for optical |
| Sealing | Precise adhesive dispensing |
| Testing | AOI, functional, waterproof |

### Design for Manufacturing (DFM)

| Guideline | Purpose |
|-----------|---------|
| 0.4mm pitch minimum | Standard SMT capability |
| Test points | Functional testing |
| Fiducials | Pick-and-place alignment |
| Panel design | Efficient production |

## Summary

PCB and mechanical design determines product quality:

- **Form factor**: 38-46mm diameter, 10-14mm thick typical
- **PCB**: 6-8 layer, flex-rigid common
- **Antenna**: Careful placement and keep-out zones
- **Waterproofing**: IP67/68 achievable with proper sealing
- **Sensors**: Optical window and electrode design critical
- **Thermal**: Manage skin temperature below 43°C
- **DFM**: Design for manufacturing from the start

---

**Previous Section**: [MCU and Wireless](10_mcu_wireless.md)
**Next Section**: [Regulatory and Safety](12_regulatory_safety.md) - Compliance requirements
