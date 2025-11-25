# Section 6: Chemical Sensors

## Introduction

Chemical sensors enable non-invasive or minimally invasive monitoring of biochemical markers. This section covers glucose sensing for diabetes management, lactate monitoring for athletic performance, and emerging sweat-based biomarker analysis. These sensors represent the cutting edge of wearable health technology.

## Continuous Glucose Monitoring (CGM)

### Clinical Importance

| Condition | Glucose Range | Monitoring Need |
|-----------|--------------|-----------------|
| Normal | 70-100 mg/dL (fasting) | Wellness |
| Pre-diabetes | 100-125 mg/dL | Risk assessment |
| Diabetes | >126 mg/dL | Continuous monitoring |
| Hypoglycemia | <70 mg/dL | Urgent alerts |
| Hyperglycemia | >180 mg/dL | Treatment guidance |

### CGM Technologies

| Technology | Type | Accuracy | Duration | Example Products |
|------------|------|----------|----------|------------------|
| Subcutaneous electrochemical | Invasive | ±10-15% | 7-14 days | Dexcom, Libre |
| Optical (near-IR) | Non-invasive | Research stage | Continuous | In development |
| Microwave/RF | Non-invasive | Research stage | Continuous | In development |
| Transdermal | Minimally invasive | ±15-20% | Hours | Research |
| Tear fluid | Non-invasive | ±20% | Continuous | Contact lens |

### Electrochemical Glucose Sensors

#### Operating Principle

```
Glucose + O₂ ──[Glucose Oxidase]──▶ Gluconolactone + H₂O₂

H₂O₂ ──▶ O₂ + 2H⁺ + 2e⁻ (at working electrode)

Current ∝ [Glucose]
```

#### Sensor Construction

```
┌─────────────────────────────────────────┐
│           Permselective Membrane         │ ← Blocks interferents
├─────────────────────────────────────────┤
│           Enzyme Layer (GOx)             │ ← Glucose oxidase
├─────────────────────────────────────────┤
│           Working Electrode              │ ← Pt or Au
├─────────────────────────────────────────┤
│           Reference Electrode            │ ← Ag/AgCl
├─────────────────────────────────────────┤
│           Counter Electrode              │ ← Pt
└─────────────────────────────────────────┘
```

#### Key Parameters

| Parameter | Typical Value | Notes |
|-----------|---------------|-------|
| Sensitivity | 0.5-10 nA/mM/mm² | Enzyme loading dependent |
| Linear range | 0-30 mM (0-540 mg/dL) | Covers clinical range |
| Response time | 1-5 minutes | Membrane diffusion |
| Working potential | +0.6-0.7V vs Ag/AgCl | H₂O₂ oxidation |
| MARD | 10-15% | Mean Absolute Relative Difference |

### Non-Invasive Glucose Approaches

| Method | Principle | Status | Challenges |
|--------|-----------|--------|------------|
| NIR Spectroscopy | Glucose absorption bands | Research | Small signal, water interference |
| Raman Spectroscopy | Molecular fingerprint | Research | Weak signal, cost |
| Photoacoustic | IR absorption + ultrasound | Research | Complexity |
| Microwave | Dielectric properties | Research | Calibration |
| Impedance | Tissue impedance changes | Research | Specificity |
| Optical Coherence | Refractive index | Research | Motion artifact |

**Reality Check**: As of 2024, no truly non-invasive glucose monitor has achieved medical device approval. Claims of non-invasive glucose monitoring in consumer devices should be viewed skeptically.

## Lactate Monitoring

### Physiological Significance

| Application | Lactate Level | Interpretation |
|-------------|---------------|----------------|
| Rest | 0.5-2 mM | Normal metabolism |
| Aerobic exercise | 2-4 mM | Sustainable effort |
| Lactate threshold | 4 mM | Anaerobic threshold |
| High intensity | 4-10+ mM | Anaerobic metabolism |
| Medical | >4 mM | Potential sepsis indicator |

### Lactate Sensing Methods

| Method | Principle | Sample |
|--------|-----------|--------|
| Electrochemical | Lactate oxidase + amperometry | Blood, sweat |
| Optical | Fluorescent enzyme | Blood |
| Colorimetric | Color change indicator | Sweat |

### Electrochemical Lactate Sensor

```
L-Lactate + O₂ ──[Lactate Oxidase]──▶ Pyruvate + H₂O₂

Similar to glucose sensing with different enzyme
```

#### Sweat Lactate Monitoring

| Consideration | Challenge | Solution |
|---------------|-----------|----------|
| Sweat rate | Variable dilution | Flow rate compensation |
| Sample volume | Microliters | Microfluidic collection |
| Contamination | Old sweat | Continuous flow design |
| Correlation | Sweat vs blood | Algorithm calibration |

## Sweat Analysis Platform

### Sweat Composition

| Analyte | Concentration | Clinical Relevance |
|---------|---------------|-------------------|
| Sodium | 10-90 mM | Hydration, cystic fibrosis |
| Chloride | 10-90 mM | Cystic fibrosis diagnosis |
| Potassium | 4-24 mM | Electrolyte balance |
| Lactate | 5-60 mM | Exercise intensity |
| Glucose | 0.1-0.5 mM | Diabetes (poor correlation) |
| Urea | 10-50 mM | Kidney function |
| pH | 4.5-7.0 | Metabolic status |
| Cortisol | 0.1-0.5 μg/dL | Stress marker |

### Sweat Collection Methods

| Method | Pros | Cons |
|--------|------|------|
| Absorbent patch | Simple | Contamination risk |
| Microfluidic | Continuous flow | Complex fabrication |
| Capillary | Self-filling | Limited volume |
| Iontophoresis | Induces sweating | Skin irritation |

### Microfluidic Sweat Sensor

```
┌──────────────────────────────────────────────────────────────┐
│                    Microfluidic Channel                       │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐      │
│  │Sweat    │──▶│ Na⁺     │──▶│ Lactate │──▶│ Glucose │──▶   │
│  │Inlet    │   │ ISE     │   │ Enzyme  │   │ Enzyme  │ Waste│
│  └─────────┘   └─────────┘   └─────────┘   └─────────┘      │
│                     │             │             │            │
│                     ▼             ▼             ▼            │
│                ┌─────────────────────────────────────┐       │
│                │      Readout Electronics            │       │
│                └─────────────────────────────────────┘       │
└──────────────────────────────────────────────────────────────┘
```

### Ion-Selective Electrodes (ISE)

| Ion | Membrane Type | Sensitivity |
|-----|---------------|-------------|
| Na⁺ | Glass, polymer | 59 mV/decade |
| K⁺ | Valinomycin | 59 mV/decade |
| Cl⁻ | AgCl, polymer | -59 mV/decade |
| H⁺ | ISFET, glass | 59 mV/pH |

## Sensor Fabrication Technologies

### Printing Methods

| Method | Resolution | Materials | Cost |
|--------|------------|-----------|------|
| Screen printing | 50-100 μm | Carbon, Ag | Low |
| Inkjet | 20-50 μm | Conductive inks | Medium |
| Aerosol jet | 10 μm | Various | High |
| Photolithography | 1 μm | Metals | High |

### Flexible Substrates

| Substrate | Properties | Use Case |
|-----------|------------|----------|
| PET | Cheap, stable | Disposable sensors |
| PI (Kapton) | High temp | Processing flexibility |
| TPU | Stretchable | Skin-conformable |
| Paper | Biodegradable | Low-cost diagnostics |
| Textile | Wearable | Integrated clothing |

## Calibration Challenges

### Drift Sources

| Source | Timescale | Mitigation |
|--------|-----------|------------|
| Enzyme degradation | Days-weeks | Sensor replacement |
| Biofouling | Hours-days | Membrane design |
| Temperature | Minutes | Compensation |
| Interferents | Immediate | Selectivity layer |

### Calibration Methods

| Method | Description | Accuracy |
|--------|-------------|----------|
| Factory calibration | Pre-calibrated | Moderate |
| Finger-stick | Blood glucose reference | Good |
| Zero-point | Known zero sample | Moderate |
| Multi-point | Multiple standards | Best |

## Signal Processing Considerations

### Noise Sources

| Source | Frequency | Mitigation |
|--------|-----------|------------|
| Electrochemical | Low frequency | Filtering |
| Motion | 1-10 Hz | Accelerometer reference |
| Electromagnetic | 50/60 Hz | Shielding |
| Thermal | Slow drift | Temperature compensation |

### Algorithms

| Algorithm | Purpose |
|-----------|---------|
| Kalman filter | Sensor fusion, drift compensation |
| Lag correction | Account for ISF-blood delay |
| Outlier rejection | Remove motion artifacts |
| Trend prediction | Hypo/hyper alerts |

## Power Requirements

| Sensor Type | Power | Measurement Frequency |
|-------------|-------|----------------------|
| Electrochemical (amperometric) | 10-100 μW | Continuous |
| Potentiometric (ISE) | 1-10 μW | Continuous |
| Optical (fluorescence) | 1-10 mW | Intermittent |
| Impedimetric | 100 μW - 1 mW | Intermittent |

## Regulatory Pathway

### Medical Device Classification

| Region | Glucose CGM | Lactate Monitor | Sweat Analysis |
|--------|-------------|-----------------|----------------|
| FDA (US) | Class II | Class II | Class I/II |
| CE (EU) | Class IIb | Class IIa | Class I/IIa |
| Pathway | 510(k) | 510(k) | De novo possible |

### Clinical Validation Requirements

| Metric | Target | Standard |
|--------|--------|----------|
| MARD | <15% | ISO 15197 |
| Consensus error grid | >99% A+B | Clarke/Parkes |
| Precision | CV <10% | CLSI |
| Stability | >7 days | Manufacturer claim |

## Future Directions

### Emerging Technologies

| Technology | Promise | Timeline |
|------------|---------|----------|
| Microneedle arrays | Minimally invasive multi-analyte | 2-5 years |
| Aptamer sensors | Reusable, stable | 3-7 years |
| Nanomaterial sensors | Higher sensitivity | 3-5 years |
| Implantable CGM | Months-long duration | 2-4 years |
| Optical glucose | True non-invasive | Unknown |

### Integration with Wearables

| Challenge | Current State | Future Goal |
|-----------|---------------|-------------|
| Form factor | External sensor + watch | Integrated in strap |
| Power | Separate battery | Wearable battery |
| Connectivity | Dedicated transmitter | Direct BLE |
| Display | Phone app | Watch face |

## Summary

Chemical sensors are advancing rapidly for wearable health monitoring:

- **Glucose**: Electrochemical subcutaneous sensors mature; non-invasive remains elusive
- **Lactate**: Sweat-based monitoring viable for sports
- **Sweat analysis**: Multi-analyte platforms emerging
- **Fabrication**: Printed electronics enabling disposable sensors
- **Challenges**: Calibration, drift, and regulatory approval

---

**Previous Section**: [Environmental Sensors](05_environmental_sensors.md)
**Next Section**: [Sensor Selection Criteria](07_sensor_selection.md) - Specifications and selection guidelines
