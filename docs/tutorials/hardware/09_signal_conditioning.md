# Section 9: Signal Conditioning and Analog Front-End

## Introduction

Signal conditioning is the critical interface between analog sensors and digital processing. This section covers amplifier design, filtering, analog-to-digital conversion, and noise management for wearable biosignal acquisition.

## Signal Chain Overview

### Generic Signal Chain

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ANALOG SIGNAL CHAIN                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Sensor      Protection    Amplifier    Filter       ADC        Digital     │
│  ┌─────┐     ┌─────┐      ┌─────┐     ┌─────┐     ┌─────┐     ┌─────┐      │
│  │     │────▶│ ESD │─────▶│ Gain│────▶│ LPF │────▶│     │────▶│     │      │
│  │     │     │ TVS │      │     │     │ HPF │     │ SAR │     │ DSP │      │
│  │     │     │     │      │     │     │     │     │Sigma│     │     │      │
│  └─────┘     └─────┘      └─────┘     └─────┘     │Delta│     └─────┘      │
│                                                    └─────┘                   │
│                                                                              │
│  μV - mV     Limit        mV - V      Remove       Bits        Process      │
│  signal      voltage      range       noise                    filter       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Signal Levels by Application

| Signal | Amplitude | Bandwidth | Dynamic Range |
|--------|-----------|-----------|---------------|
| ECG | 0.5-3 mV | 0.05-150 Hz | 60 dB |
| EDA | 0.1-100 μS | DC-5 Hz | 60 dB |
| PPG | 1-10 mV (AC) | 0.5-10 Hz | 40-60 dB |
| EMG | 10 μV - 10 mV | 10-500 Hz | 80 dB |
| EEG | 1-100 μV | 0.5-100 Hz | 40 dB |

## Input Protection

### ESD Protection

| Protection Method | Clamping Voltage | Capacitance | Leakage |
|-------------------|------------------|-------------|---------|
| TVS diode | 5-12V | 0.5-2 pF | <1 nA |
| ESD diode array | 6-15V | 0.3-1 pF | <1 nA |
| Resistor + diode | Variable | Adds 1-10 pF | <1 nA |
| Varistor | 5-20V | 10-50 pF | <10 nA |

### Protection Circuit Example

```
                     TVS
Electrode ────┬──────┤├───────┬──── GND
              │               │
              R (10k-100k)    │
              │               │
              └───────────────┴──── To Amplifier Input

TVS: Bidirectional TVS rated for ESD (±8kV)
R: Current limiting resistor
```

### Input Filtering

| Purpose | Component | Value | Notes |
|---------|-----------|-------|-------|
| RF rejection | Series resistor | 10-100 kΩ | Forms RC filter |
| RF rejection | Shunt capacitor | 10-100 pF | fc = 1/(2πRC) |
| EMI filtering | Ferrite bead | 10-100 Ω @ 100MHz | Common mode |

## Amplifier Design

### Instrumentation Amplifier (In-Amp)

For differential biopotential signals (ECG, EMG):

```
                        ┌─────────────────────┐
       V+  ─────────────┤                     │
          ┌─────────────┤─    A1    +         │
          │             │           │         │
          │    ┌────────┤───────────┼─────────┤─────── Vout
          │    │        │           │         │
          │    Rgain    │    ┌──────┼─────────┤
          │    │        │    │      │         │
          │    ├────────┤───────────┘         │
          │             │           │         │
       V- ──────────────┤─    A2    +         │
                        │                     │
                        └─────────────────────┘

Gain = 1 + (2 × 49.4kΩ / Rgain)

For G=100: Rgain ≈ 1 kΩ
```

### Key Specifications

| Parameter | ECG Requirement | PPG Requirement |
|-----------|-----------------|-----------------|
| Input impedance | >10 MΩ | >1 MΩ |
| CMRR | >80 dB | >60 dB |
| Input bias current | <1 nA | <10 nA |
| Input offset voltage | <100 μV | <1 mV |
| Bandwidth | 0.05-150 Hz | DC-10 Hz |
| Noise | <10 μVpp | <100 μVpp |
| Gain | 100-1000 | 1-100 |

### Recommended Amplifier ICs

| Part | Type | CMRR | Input Current | Noise | Current |
|------|------|------|---------------|-------|---------|
| AD8220 | In-amp | 100 dB | 25 pA | 15 nV/√Hz | 750 μA |
| INA333 | In-amp | 100 dB | 25 pA | 50 nV/√Hz | 50 μA |
| OPA2333 | Op-amp | - | 30 pA | 55 nV/√Hz | 17 μA |
| MAX4461 | Op-amp | - | 1 pA | 22 nV/√Hz | 24 μA |

### Transimpedance Amplifier (TIA)

For photodiode signals (PPG):

```
              Rf
         ┌────/\/\/────┐
         │             │
         │   ┌─────┐   │
    PD ──┴───┤-    │   │
             │ TIA ├───┴──── Vout
         ────┤+    │
         │   └─────┘
        Vbias

Vout = Iphotodiode × Rf

Typical: Rf = 100kΩ - 10MΩ
         Cf = stability compensation
```

### TIA Design Considerations

| Parameter | Consideration |
|-----------|---------------|
| Gain (Rf) | Higher = more signal, more noise |
| Bandwidth | Set by Cf and Rf |
| Stability | Cf required for photodiode capacitance |
| Dark current | Contributes DC offset |
| Noise | Shot noise, thermal noise, amp noise |

## Filtering

### Filter Types

| Filter | Purpose | Implementation |
|--------|---------|----------------|
| High-pass | Remove DC offset, baseline wander | Passive RC, active |
| Low-pass | Anti-aliasing, noise reduction | Passive RC, active |
| Band-pass | Signal of interest | Active, HPF + LPF |
| Notch | 50/60 Hz rejection | Twin-T, active |

### Common Filter Frequencies

| Application | High-pass | Low-pass | Notch |
|-------------|-----------|----------|-------|
| ECG (diagnostic) | 0.05 Hz | 150 Hz | 50/60 Hz |
| ECG (monitoring) | 0.5 Hz | 40 Hz | 50/60 Hz |
| PPG heart rate | 0.5 Hz | 5 Hz | Optional |
| PPG SpO2 | DC | 10 Hz | Optional |
| EDA | DC | 5 Hz | Optional |

### Anti-Aliasing Filter Design

For sampling at fs:
- Nyquist frequency: fs/2
- Anti-alias cutoff: <0.4 × fs (for sigma-delta)
- Anti-alias cutoff: <0.45 × fs (for SAR)

```
Example: ECG sampled at 250 Hz
         Nyquist = 125 Hz
         Cutoff = 100 Hz (4th order Butterworth)
         Attenuation at 125 Hz: ~12 dB
```

### Active Filter Topologies

| Topology | Order | Components | Q Range |
|----------|-------|------------|---------|
| Sallen-Key | 2 | 1 op-amp, 4 passive | <10 |
| Multiple feedback | 2 | 1 op-amp, 5 passive | <20 |
| State variable | 2 | 3 op-amps | High |
| Biquad | 2 | 2 op-amps | High |

### Digital Post-Filtering

| Filter Type | Use Case | Latency |
|-------------|----------|---------|
| Moving average | Smoothing | Low |
| IIR low-pass | Anti-noise | Low |
| FIR band-pass | Signal extraction | Moderate |
| Adaptive | Motion artifact | High |

## Analog-to-Digital Conversion

### ADC Types for Wearables

| Type | Resolution | Speed | Power | Use Case |
|------|------------|-------|-------|----------|
| SAR | 12-18 bit | 1-10 MSPS | Low-Medium | General purpose |
| Sigma-Delta | 16-24 bit | 10-10k SPS | Low | High resolution |
| Pipeline | 10-14 bit | 10-100 MSPS | High | Not typical |

### SAR vs Sigma-Delta

| Aspect | SAR | Sigma-Delta |
|--------|-----|-------------|
| Resolution | 12-18 bit effective | 16-24 bit effective |
| Speed | Fast conversion | Slower (oversampling) |
| Anti-alias filter | Sharp required | Relaxed (digital) |
| Latency | Low | Higher |
| Multiplexing | Easy | Difficult |
| Power | Lower | Higher at high speed |

### ADC Selection Criteria

| Parameter | ECG | PPG | General |
|-----------|-----|-----|---------|
| Resolution | 16-24 bit | 16-18 bit | 12-16 bit |
| Sample rate | 250-1000 SPS | 100-500 SPS | Variable |
| Channels | 1-3 | 2-4 | Multiple |
| SNR | >90 dB | >70 dB | Application |
| INL/DNL | <1 LSB | <2 LSB | <2 LSB |

### Popular ADC ICs

| Part | Type | Resolution | Channels | Current |
|------|------|------------|----------|---------|
| ADS1292 | Sigma-Delta | 24-bit | 2 | 335 μA |
| ADS1115 | Sigma-Delta | 16-bit | 4 (mux) | 150 μA |
| MAX11131 | SAR | 12-bit | 16 (mux) | 2.4 mA |
| AD7124 | Sigma-Delta | 24-bit | 8 | 250 μA |

## Integrated Analog Front-Ends

### AFE Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    INTEGRATED AFE (e.g., MAX86150)                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐     │
│  │   Input     │   │    PGA      │   │   Filter    │   │    ADC      │     │
│  │   Mux       │──▶│  (1-128x)   │──▶│  (HPF/LPF)  │──▶│ (16-24 bit) │──┐  │
│  │             │   │             │   │             │   │             │  │  │
│  └─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘  │  │
│                                                                          │  │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐                    │  │
│  │   LED       │   │    TIA      │   │   Sample    │                    │  │
│  │   Drivers   │──▶│             │──▶│   & Hold    │────────────────────┘  │
│  │             │   │             │   │             │                       │
│  └─────────────┘   └─────────────┘   └─────────────┘                       │
│                                                                          │  │
│                                           ┌─────────────────────────────┼──┤
│                                           │        Digital Interface    │  │
│                                           │        (I2C/SPI + FIFO)     │  │
│                                           └─────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Recommended AFE ICs

| Part | Application | Resolution | Channels | Features |
|------|-------------|------------|----------|----------|
| MAX86150 | ECG + PPG | 18-bit | 4 | Integrated combo |
| ADS1292R | ECG | 24-bit | 2 | Respiration, pace |
| ADPD4100 | PPG | 14-bit | 8 LED + 4 PD | Configurable |
| AFE4900 | PPG + ECG | 22-bit | Multi | Wearable focused |
| MAX30003 | ECG | 18-bit | 1 | Ultra-low power |

## Noise Management

### Noise Sources

| Source | Type | Mitigation |
|--------|------|------------|
| Thermal (Johnson) | White | Lower R, lower T |
| Shot | White | Lower current |
| Flicker (1/f) | Pink | Chopper, correlated double sampling |
| Quantization | White | Higher resolution ADC |
| Power supply | Various | Filtering, LDO |
| EMI | Various | Shielding, filtering |

### Noise Calculations

```
Thermal noise: Vn = √(4kTRB)
Where: k = 1.38×10⁻²³ J/K
       T = Temperature (K)
       R = Resistance (Ω)
       B = Bandwidth (Hz)

Example: 10 kΩ resistor, 100 Hz bandwidth, 300K
         Vn = √(4 × 1.38×10⁻²³ × 300 × 10000 × 100)
         Vn = 4.1 μV RMS
```

### Noise Budget Example (ECG)

| Stage | Noise Contribution | Notes |
|-------|-------------------|-------|
| Electrode | 2-5 μVrms | Electrode-skin interface |
| Input resistors | 1-2 μVrms | Protection circuitry |
| In-amp | 1-3 μVrms | Depends on gain, BW |
| Filter | 0.5-1 μVrms | Op-amp noise |
| ADC | 0.5-1 μVrms | Quantization + reference |
| **Total** | **~5-10 μVrms** | RSS addition |

### Layout for Low Noise

| Guideline | Purpose |
|-----------|---------|
| Star grounding | Prevent ground loops |
| Guard rings | Reduce leakage current |
| Separate analog/digital | Prevent coupling |
| Short traces | Reduce antenna effect |
| Ground plane | Shield from EMI |
| Proper decoupling | Clean power supply |

## Practical Design Tips

### Component Selection

| Component | Recommendation |
|-----------|----------------|
| Resistors | Thin film, 0.1% for critical paths |
| Capacitors | C0G/NP0 for filters, X7R for bypass |
| Op-amps | Rail-to-rail for single supply |
| References | Low noise, low drift |

### PCB Considerations

| Aspect | Recommendation |
|--------|----------------|
| Layer stack | Dedicated analog layer |
| Traces | Wide for power, thin for signal |
| Vias | Minimize in signal path |
| Components | Keep analog components close |
| Ground | Solid plane under analog |

## Summary

Signal conditioning fundamentals for wearables:

- **Protection**: ESD protection at all inputs
- **Amplification**: In-amp for biopotentials, TIA for optical
- **Filtering**: HPF for baseline, LPF for anti-aliasing
- **ADC selection**: Sigma-delta for high resolution, SAR for speed
- **Integrated AFE**: Reduces design complexity significantly
- **Noise management**: Systematic approach from source to digital

---

**Previous Section**: [Power Management](08_power_management.md)
**Next Section**: [MCU and Wireless](10_mcu_wireless.md) - Processor and connectivity selection
