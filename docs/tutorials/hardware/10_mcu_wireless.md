# Section 10: MCU Selection and Wireless Connectivity

## Introduction

The microcontroller (MCU) and wireless subsystem form the computational and communication backbone of a smartwatch. This section covers processor selection criteria, memory requirements, and wireless connectivity options for wearable devices.

## MCU Selection Criteria

### Key Requirements

| Requirement | Importance | Impact |
|-------------|------------|--------|
| Power efficiency | Critical | Battery life |
| Processing power | High | Feature capability |
| Peripheral set | High | Sensor interface |
| Memory | High | Firmware + data |
| Cost | Medium | BOM budget |
| Supply chain | Medium | Production risk |

### Processor Architectures

| Architecture | Examples | Characteristics |
|--------------|----------|-----------------|
| ARM Cortex-M0/M0+ | nRF52810, STM32L0 | Ultra-low power, limited features |
| ARM Cortex-M4/M4F | nRF52840, STM32L4 | DSP, FPU, balanced |
| ARM Cortex-M33 | nRF5340, STM32U5 | TrustZone, modern |
| ARM Cortex-M7 | STM32H7, i.MX RT | High performance |
| RISC-V | ESP32-C3, BL602 | Open ISA, emerging |

### Performance Comparison

| Processor | CoreMark/MHz | DMIPS/MHz | FPU | DSP |
|-----------|-------------|-----------|-----|-----|
| Cortex-M0+ | 2.46 | 0.95 | No | No |
| Cortex-M4F | 3.40 | 1.25 | Yes | Yes |
| Cortex-M33 | 4.02 | 1.50 | Yes | Yes |
| Cortex-M7 | 5.01 | 2.14 | Yes (DP) | Yes |

### Power Efficiency Metrics

| Mode | Cortex-M0+ | Cortex-M4F | Cortex-M33 |
|------|------------|------------|------------|
| Active (μA/MHz) | 30-50 | 50-100 | 30-80 |
| Sleep (μA) | 1-5 | 2-10 | 1-5 |
| Deep sleep (μA) | 0.3-1 | 0.5-2 | 0.3-1 |
| Shutdown (nA) | 20-100 | 50-200 | 20-100 |

## Memory Requirements

### Flash Memory

| Component | Size | Notes |
|-----------|------|-------|
| Bootloader | 16-32 KB | Secure boot, DFU |
| Application | 128-512 KB | Main firmware |
| File system | 64-256 KB | Settings, logs |
| OTA staging | 128-512 KB | Update buffer |
| **Total** | **512 KB - 2 MB** | Typical range |

### RAM Requirements

| Component | Size | Notes |
|-----------|------|-------|
| Stack | 2-8 KB | Per task if RTOS |
| Heap | 8-32 KB | Dynamic allocation |
| Display buffer | 10-100 KB | Resolution dependent |
| Sensor buffers | 4-16 KB | FIFO caching |
| BLE stack | 8-16 KB | Protocol stack |
| **Total** | **64-256 KB** | Typical range |

### External Storage

| Type | Capacity | Interface | Use Case |
|------|----------|-----------|----------|
| QSPI Flash | 2-16 MB | QSPI | Assets, firmware |
| eMMC | 4-64 GB | SDIO | Data logging (rarely) |
| EEPROM | 8-64 KB | I2C | Calibration, settings |

## Recommended MCU Platforms

### Integrated BLE + MCU Solutions

| Part | Core | Flash | RAM | BLE | Features |
|------|------|-------|-----|-----|----------|
| nRF52832 | M4F 64MHz | 512KB | 64KB | 5.0 | Popular choice |
| nRF52840 | M4F 64MHz | 1MB | 256KB | 5.0 | USB, crypto |
| nRF5340 | M33 128MHz + M33 | 1MB | 512KB | 5.2 | Dual core |
| STM32WB55 | M4F 64MHz | 1MB | 256KB | 5.0 | Thread, Zigbee |
| DA14695 | M33 96MHz | 512KB | 512KB | 5.1 | Display controller |
| CC2640R2 | M3 48MHz | 128KB | 28KB | 5.0 | Low cost |

### Application Processor Options

| Part | Core | RAM | GPU | Use Case |
|------|------|-----|-----|----------|
| STM32MP1 | A7 + M4 | External | Yes | Full smartwatch OS |
| i.MX RT1062 | M7 600MHz | 1MB | 2D | Rich UI |
| ESP32-S3 | Xtensa + AI | 512KB | No | AI at edge |

### Power Comparison (Typical)

| Part | Active @ 64MHz | Sleep (RTC) | Radio TX |
|------|----------------|-------------|----------|
| nRF52832 | 4.8 mA | 2.0 μA | 7.2 mA |
| nRF52840 | 5.3 mA | 1.5 μA | 7.5 mA |
| STM32WB55 | 5.5 mA | 1.2 μA | 6.2 mA |
| nRF5340 | 3.2 mA | 0.9 μA | 4.6 mA |

## Wireless Connectivity

### Bluetooth Low Energy (BLE)

BLE is the primary connectivity for wearables:

| BLE Version | Key Features | Data Rate |
|-------------|--------------|-----------|
| 4.0 | Initial LE, 2.4 GHz | 1 Mbps |
| 4.2 | LE Data Length Extension | 1 Mbps |
| 5.0 | 2M PHY, Long Range, Advertising Extensions | 2 Mbps |
| 5.1 | Direction Finding (AoA/AoD) | 2 Mbps |
| 5.2 | LE Audio, Isochronous Channels | 2 Mbps |
| 5.3 | Enhanced ATT, Subrating | 2 Mbps |

### BLE Power Profiles

| Connection Interval | Current (avg) | Use Case |
|--------------------|---------------|----------|
| 7.5 ms | 1-2 mA | Real-time streaming |
| 30 ms | 200-400 μA | Interactive |
| 100 ms | 50-100 μA | Background sync |
| 1000 ms | 10-20 μA | Minimal activity |
| Advertising (slow) | 10-30 μA | Discoverable |

### BLE Data Throughput

| PHY | Theoretical | Practical | Latency |
|-----|-------------|-----------|---------|
| 1M | 1 Mbps | 100-200 kbps | 7.5 ms min |
| 2M | 2 Mbps | 200-400 kbps | 7.5 ms min |
| Coded (S=2) | 500 kbps | 50-100 kbps | Higher |
| Coded (S=8) | 125 kbps | 10-30 kbps | Higher |

### Other Wireless Options

| Technology | Data Rate | Range | Power | Use Case |
|------------|-----------|-------|-------|----------|
| NFC | 424 kbps | 4 cm | Very Low | Payment, pairing |
| ANT+ | 60 kbps | 30 m | Low | Sports accessories |
| WiFi | 150 Mbps | 50 m | High | Firmware updates |
| LTE-M | 1 Mbps | Cellular | High | Standalone |
| GPS | N/A | Global | High | Location |

### NFC Implementation

| Component | Purpose |
|-----------|---------|
| NFC controller | Protocol handling |
| Antenna | Inductive coupling |
| Secure element | Payment credentials |

### GNSS/GPS

| Aspect | Specification |
|--------|---------------|
| Constellations | GPS, GLONASS, Galileo, BeiDou |
| Acquisition time | 1-30 seconds (cold/hot start) |
| Accuracy | 2-5 meters typical |
| Power | 10-30 mA active |

| GNSS Module | Size | Power | TTFF (cold) |
|-------------|------|-------|-------------|
| u-blox MAX-M10S | 4.5×4.5 mm | 6 mA | 27s |
| Sony CXD5605 | 3.6×3.8 mm | 5 mA | 26s |
| Quectel L76-L | 10×10 mm | 22 mA | 32s |

## Peripheral Requirements

### Essential Peripherals

| Peripheral | Count | Use |
|------------|-------|-----|
| I2C | 1-2 | Sensors, PMIC |
| SPI | 1-2 | Display, Flash |
| UART | 1 | Debug |
| ADC | 4-8 ch | Battery, analog sensors |
| PWM | 2-4 | Backlight, haptic |
| GPIO | 10-20 | Buttons, interrupts |
| RTC | 1 | Timekeeping |

### Advanced Peripherals

| Peripheral | Use Case |
|------------|----------|
| DMA | Efficient data transfer |
| Hardware crypto | Secure boot, encryption |
| USB | Charging detect, data |
| I2S | Audio |
| QSPI | External flash |
| LCD controller | Display interface |

## Software Considerations

### Operating System Options

| Option | RAM | Features | Complexity |
|--------|-----|----------|------------|
| Bare metal | <10 KB | Minimal, predictable | Low |
| FreeRTOS | 10-30 KB | Task scheduling | Medium |
| Zephyr | 50-100 KB | Full RTOS, BLE stack | Medium-High |
| RIOT | 20-50 KB | IoT focused | Medium |
| Linux | 4+ MB | Full OS | High |

### BLE Stack Options

| Stack | RAM | Flash | License |
|-------|-----|-------|---------|
| SoftDevice (Nordic) | 10 KB | 112 KB | Proprietary |
| STM32WB Stack | 12 KB | 150 KB | Proprietary |
| Zephyr BLE | 20 KB | 100 KB | Apache 2.0 |
| NimBLE | 8 KB | 80 KB | Apache 2.0 |

### Firmware Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        FIRMWARE ARCHITECTURE                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                         APPLICATION                                  │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐            │    │
│  │  │   UI     │  │  Health  │  │  Comms   │  │  System  │            │    │
│  │  │ Manager  │  │ Tracking │  │  Manager │  │ Services │            │    │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘            │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                    │                                         │
│  ┌─────────────────────────────────┼───────────────────────────────────┐    │
│  │                         MIDDLEWARE                                   │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐            │    │
│  │  │   BLE    │  │  File    │  │  Power   │  │  Sensor  │            │    │
│  │  │  Stack   │  │  System  │  │  Manager │  │   HAL    │            │    │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘            │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                    │                                         │
│  ┌─────────────────────────────────┼───────────────────────────────────┐    │
│  │                          RTOS / HAL                                  │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐            │    │
│  │  │Scheduler │  │ Drivers  │  │   DMA    │  │  IRQ     │            │    │
│  │  │          │  │          │  │          │  │          │            │    │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘            │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                    │                                         │
│                              HARDWARE                                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Design for Low Power

### MCU Sleep Strategy

```
                    User Activity
                         │
                         ▼
┌─────────┐  timeout  ┌─────────┐  timeout  ┌─────────┐
│  ACTIVE │──────────▶│  IDLE   │──────────▶│ STANDBY │
│         │◀──────────│         │◀──────────│         │
└─────────┘  event    └─────────┘  event    └─────────┘
   5 mA                   1 mA                 20 μA

Events: Button, touch, gesture, sensor interrupt, BLE
```

### Wake Sources

| Wake Source | Latency | Use Case |
|-------------|---------|----------|
| RTC alarm | 1-10 ms | Periodic sampling |
| GPIO interrupt | <100 μs | Button, sensor |
| Accelerometer | 1-5 ms | Motion detection |
| BLE event | 1-5 ms | Connection |
| Touch | 1-5 ms | User input |

### Peripheral Power Management

| Strategy | Description | Savings |
|----------|-------------|---------|
| Clock gating | Disable unused peripherals | 10-30% |
| Power domains | Gate power to subsystems | 20-50% |
| DMA transfers | CPU sleeps during transfer | 10-20% |
| Burst mode | Process fast, sleep long | 20-40% |

## Summary

MCU and wireless selection drives wearable capabilities:

- **MCU**: ARM Cortex-M4F or M33 balances performance and power
- **Memory**: 256KB+ RAM, 512KB+ Flash typical
- **BLE**: Version 5.0+ preferred, connection interval critical
- **GPS**: Power-hungry, use sparingly
- **Software**: RTOS recommended for complex applications
- **Low power**: Sleep modes, wake sources, power domains

---

**Previous Section**: [Signal Conditioning](09_signal_conditioning.md)
**Next Section**: [PCB and Mechanical](11_pcb_mechanical.md) - Physical design considerations
