# Tutorial 4.6: Continuous Glucose Monitoring - Technology, Chemistry, and Algorithms

## Learning Objectives

By the end of this tutorial, you will:
- Understand the chemistry behind glucose sensing
- Know how CGM devices work at the hardware and software level
- Implement glucose signal processing and calibration algorithms
- Design glucose prediction and alert systems
- Understand the future of non-invasive glucose monitoring

## Prerequisites
- Tutorial 1.2 (Biology Foundations - Glucose Metabolism)
- Tutorial 2.2 (Filter Design)
- Basic chemistry knowledge

---

## Table of Contents

1. [Introduction to Glucose Monitoring](#1-introduction-to-glucose-monitoring)
2. [Electrochemical Sensor Technology](#2-electrochemical-sensor-technology)
3. [CGM System Architecture](#3-cgm-system-architecture)
4. [Signal Processing Algorithms](#4-signal-processing-algorithms)
5. [Calibration Algorithms](#5-calibration-algorithms)
6. [Glucose Prediction](#6-glucose-prediction)
7. [Alert and Safety Systems](#7-alert-and-safety-systems)
8. [Advanced Topics](#8-advanced-topics)
9. [Learning Resources](#9-learning-resources)

---

## 1. Introduction to Glucose Monitoring

### 1.1 Why Continuous Monitoring?

```
    Traditional SMBG vs CGM

    ┌─────────────────────────────────────────────────────────┐
    │       SELF-MONITORING BLOOD GLUCOSE (SMBG)             │
    │                                                        │
    │   Glucose                                              │
    │   (mg/dL)                                              │
    │       │    Only see these                              │
    │   250 │         points                                  │
    │       │           ●                                     │
    │   200 │                    ●                            │
    │       │                                                 │
    │   150 │     ●                        ●                  │
    │       │                                                 │
    │   100 │ ●                                    ●          │
    │       └──────────────────────────────────────→          │
    │         6am  9am  12pm  3pm  6pm  9pm  12am             │
    │                                                        │
    │   Problem: Miss all the action between measurements!    │
    └─────────────────────────────────────────────────────────┘

    ┌─────────────────────────────────────────────────────────┐
    │       CONTINUOUS GLUCOSE MONITORING (CGM)              │
    │                                                        │
    │   Glucose                                              │
    │   (mg/dL)      See the full picture                    │
    │       │                                                 │
    │   250 │         ╭─╮                                     │
    │       │        ╱   ╲                                    │
    │   200 │       ╱     ╲      ╭──╮                        │
    │       │      ╱       ╲    ╱    ╲                       │
    │   150 │    ╱          ╲──╱      ╲                      │
    │       │   ╱                      ╲    ╭──╮             │
    │   100 │──╱                        ╲──╱    ╲──          │
    │       └──────────────────────────────────────→          │
    │         6am  9am  12pm  3pm  6pm  9pm  12am             │
    │                                                        │
    │   Benefit: See trends, patterns, and variability       │
    └─────────────────────────────────────────────────────────┘
```

### 1.2 CGM System Components

```
    ┌─────────────────────────────────────────────────────────┐
    │              CGM SYSTEM OVERVIEW                        │
    └─────────────────────────────────────────────────────────┘

    1. SENSOR (under skin)
       ┌────────────────────────────────────────────────────┐
       │                                                    │
       │   Insertion site: Upper arm or abdomen            │
       │                                                    │
       │       Skin surface                                 │
       │   ════════════════════════                        │
       │           │                                        │
       │           ▼ Needle (removed after insertion)      │
       │       ┌───────┐                                   │
       │       │Sensor │ ← Electrochemical sensor          │
       │       │ tip   │   (5-7mm into subcutaneous tissue)│
       │       └───────┘                                   │
       │           ↑                                        │
       │       Interstitial fluid (ISF)                    │
       │       Contains glucose that diffuses from blood   │
       │                                                    │
       └────────────────────────────────────────────────────┘

    2. TRANSMITTER (on skin)
       ┌────────────────────────────────────────────────────┐
       │ • Connects to sensor                              │
       │ • Measures current from sensor                    │
       │ • Converts to digital signal                      │
       │ • Transmits via Bluetooth                         │
       │ • Battery: Rechargeable or 90-day disposable      │
       └────────────────────────────────────────────────────┘

    3. RECEIVER/SMARTPHONE APP
       ┌────────────────────────────────────────────────────┐
       │ • Receives sensor data                            │
       │ • Applies calibration algorithm                   │
       │ • Displays glucose value and trend                │
       │ • Generates alerts (high/low/rate of change)      │
       │ • Stores historical data                          │
       └────────────────────────────────────────────────────┘
```

### 1.3 Commercial CGM Systems

| System | Manufacturer | Sensor Life | Calibration | Accuracy (MARD) |
|--------|--------------|-------------|-------------|-----------------|
| Dexcom G7 | Dexcom | 10 days | Factory | 8.2% |
| FreeStyle Libre 3 | Abbott | 14 days | Factory | 7.9% |
| Medtronic Guardian 4 | Medtronic | 7 days | 2x/day | 8.7% |
| Eversense E3 | Senseonics | 180 days | 2x/day | 8.5% |

*MARD = Mean Absolute Relative Difference (lower is better)

---

## 2. Electrochemical Sensor Technology

### 2.1 The Enzyme Electrode

The heart of CGM is an **enzyme electrode** that converts glucose concentration to electrical current:

```
    ┌─────────────────────────────────────────────────────────┐
    │          GLUCOSE OXIDASE ENZYME ELECTRODE               │
    └─────────────────────────────────────────────────────────┘

    Layer structure (from outside to inside):

    Interstitial Fluid (glucose source)
           │
           ▼
    ╔═══════════════════════════════════╗
    ║  OUTER MEMBRANE                   ║ ← Biocompatible
    ║  (Polyurethane/Silicone)          ║   Limits glucose diffusion
    ╠═══════════════════════════════════╣
    ║  ENZYME LAYER                     ║ ← Contains Glucose Oxidase (GOx)
    ║  (Immobilized GOx in matrix)      ║   Catalyzes glucose → gluconic acid
    ╠═══════════════════════════════════╣
    ║  INTERFERENCE REJECTION           ║ ← Blocks acetaminophen, uric acid
    ║  (Nafion or similar)              ║
    ╠═══════════════════════════════════╣
    ║  ELECTRODE                        ║ ← Platinum or Carbon
    ║  (Working electrode)              ║   Detects H2O2 or mediator
    ╚═══════════════════════════════════╝
           │
           ▼
    Electrical current output (nanoamps)
```

### 2.2 The Chemistry

**First Generation (Oxygen-based)**:
```
    Chemical Reaction:

    Step 1: Enzyme reaction
    ┌────────────────────────────────────────────────────────┐
    │                                                        │
    │   Glucose + O₂  ──[GOx]──►  Gluconic Acid + H₂O₂      │
    │                                                        │
    │   GOx = Glucose Oxidase enzyme (catalyst)              │
    │   H₂O₂ = Hydrogen Peroxide (measurable)                │
    │                                                        │
    └────────────────────────────────────────────────────────┘

    Step 2: Electrochemical detection
    ┌────────────────────────────────────────────────────────┐
    │                                                        │
    │   At platinum electrode (+0.6V vs Ag/AgCl):            │
    │                                                        │
    │   H₂O₂  ──►  O₂ + 2H⁺ + 2e⁻                           │
    │                                                        │
    │   Current (i) ∝ [H₂O₂] ∝ [Glucose]                    │
    │                                                        │
    │   Typical range: 0.1 - 50 nA for 40-400 mg/dL         │
    │                                                        │
    └────────────────────────────────────────────────────────┘
```

**Second Generation (Mediator-based)**:
```
    Uses electron mediator instead of oxygen:

    Glucose + Mediator(ox)  ──[GOx]──►  Gluconic Acid + Mediator(red)

    Mediator(red)  ──electrode──►  Mediator(ox) + e⁻

    Advantages:
    - Works at lower potential (less interference)
    - Independent of oxygen concentration
    - Faster response

    Common mediators:
    - Ferrocene derivatives
    - Osmium complexes (used in Abbott FreeStyle)
    - Prussian Blue
```

### 2.3 Sensor Current to Glucose

The relationship between sensor current and glucose is approximately linear:

```
    Current (nA)
        │
    50  │                           ╱
        │                         ╱
    40  │                       ╱
        │                     ╱
    30  │                   ╱
        │                 ╱
    20  │               ╱  Linear region
        │             ╱   (most sensors work here)
    10  │           ╱
        │         ╱
     0  │───────╱─────────────────────────────────
        └───────────────────────────────────────────→
         0    100    200    300    400    500   Glucose (mg/dL)

    Basic relationship:
    i = S × G + i₀

    Where:
    i = sensor current (nA)
    S = sensitivity (nA per mg/dL) - typically 0.05-0.5
    G = glucose concentration (mg/dL)
    i₀ = background current (nA) - offset from interferents

    Rearranged for glucose:
    G = (i - i₀) / S
```

### 2.4 Sensor Response Time

```
    ┌─────────────────────────────────────────────────────────┐
    │           SENSOR RESPONSE DYNAMICS                      │
    └─────────────────────────────────────────────────────────┘

    Step change in glucose:

    Glucose    ┌────────────────────────────────
    (blood)    │
               │
    ───────────┘

    Sensor                     ╭─────────────────
    Output         ╭──────────╱
               ╭───╱
    ──────────╱
              ↑
              │
         Time lag (τ) = 5-15 minutes

    Response is first-order:
    y(t) = y_final × (1 - e^(-t/τ))

    This lag comes from:
    1. Diffusion from blood to interstitial fluid (~5 min)
    2. Diffusion through sensor membranes (~2 min)
    3. Enzyme reaction kinetics (~1 min)
```

---

## 3. CGM System Architecture

### 3.1 Hardware Block Diagram

```
    ┌─────────────────────────────────────────────────────────┐
    │              CGM TRANSMITTER HARDWARE                   │
    └─────────────────────────────────────────────────────────┘

    ┌─────────────┐
    │   SENSOR    │ Working electrode (WE)
    │   ══════════│ Reference electrode (RE)
    │   (3-wire)  │ Counter electrode (CE)
    └──────┬──────┘
           │ nA-level current
           ▼
    ┌─────────────┐
    │ POTENTIOSTAT│ Maintains constant voltage
    │             │ Converts current to voltage
    └──────┬──────┘
           │ Analog voltage (mV)
           ▼
    ┌─────────────┐
    │     ADC     │ 16-24 bit resolution
    │             │ Sample rate: 1-10 Hz
    └──────┬──────┘
           │ Digital counts
           ▼
    ┌─────────────┐
    │     MCU     │ Signal processing
    │             │ Calibration
    │             │ Compression
    └──────┬──────┘
           │
           ▼
    ┌─────────────┐
    │  BLUETOOTH  │ Transmit to phone/receiver
    │     LE      │ Every 1-5 minutes
    └─────────────┘
```

### 3.2 Potentiostat Circuit

The potentiostat maintains a constant voltage at the working electrode:

```c
// Simplified potentiostat control
// In practice, this is analog circuitry

/*
 *     ┌─────────────────────────────────────────┐
 *     │                                         │
 *     │    V_ref (DAC) ──►(+)─┐                 │
 *     │                       │                 │
 *     │    V_re ─────────►(-)─┼──► Op-Amp ──► V_ce
 *     │    (from sensor)      │                 │
 *     │                       │                 │
 *     │    Feedback loop maintains:             │
 *     │    V_we - V_re = V_applied              │
 *     │    (typically +0.4 to +0.7V)            │
 *     │                                         │
 *     └─────────────────────────────────────────┘
 *
 *     Current measurement:
 *     i_we flows through transimpedance amplifier
 *     V_out = i_we × R_feedback
 */

typedef struct {
    float applied_voltage;      // V_applied (constant)
    float transimpedance_gain;  // R_feedback (ohms)
    float adc_reference;        // ADC reference voltage
    uint16_t adc_resolution;    // ADC bits
} PotentiostatConfig;

float adc_to_current_nA(PotentiostatConfig *cfg, uint16_t adc_reading) {
    // Convert ADC reading to voltage
    float voltage = (adc_reading * cfg->adc_reference) / (1 << cfg->adc_resolution);

    // Convert voltage to current (nanoamps)
    float current_nA = (voltage / cfg->transimpedance_gain) * 1e9;

    return current_nA;
}

// Example configuration
PotentiostatConfig cgm_config = {
    .applied_voltage = 0.6f,          // 600 mV
    .transimpedance_gain = 1e9f,      // 1 GΩ (1 nA = 1 V output)
    .adc_reference = 3.3f,            // 3.3V reference
    .adc_resolution = 16              // 16-bit ADC
};
```

### 3.3 Data Flow

```
    ┌─────────────────────────────────────────────────────────┐
    │                 CGM DATA PIPELINE                       │
    └─────────────────────────────────────────────────────────┘

    Sensor Current (1-10 Hz raw samples)
           │
           ▼
    ┌─────────────────┐
    │  Noise Filter   │ Low-pass filter (0.1 Hz cutoff)
    │                 │ Remove high-frequency noise
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │   Decimation    │ Average to 1 sample per minute
    │                 │ Reduces data volume
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │   Calibration   │ Convert current to glucose
    │   Algorithm     │ Apply factory or user calibration
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │  Trend Filter   │ Smooth rapid fluctuations
    │                 │ Calculate rate of change
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │    Validation   │ Check for sensor errors
    │                 │ Flag suspicious readings
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │    Display &    │ Show glucose + trend arrow
    │    Alerts       │ Generate notifications
    └─────────────────┘
```

---

## 4. Signal Processing Algorithms

### 4.1 Noise Reduction

CGM sensor signals are noisy due to:
- Electrochemical noise
- Motion artifacts
- Electronic interference

```c
// Low-pass filter for CGM signal
// Using IIR filter from Tutorial 2.2

#include "butterworth.h"

typedef struct {
    ButterworthLP2 noise_filter;
    float sample_rate;
    float cutoff_freq;
} CGMNoiseFilter;

void cgm_noise_filter_init(CGMNoiseFilter *f) {
    f->sample_rate = 1.0f;    // 1 Hz (1 sample/second)
    f->cutoff_freq = 0.1f;    // 0.1 Hz cutoff (10 second time constant)

    butterworth_lp2_init(&f->noise_filter, f->cutoff_freq, f->sample_rate);
}

float cgm_noise_filter_process(CGMNoiseFilter *f, float raw_current) {
    return butterworth_lp2_process(&f->noise_filter, raw_current);
}

// Median filter for spike removal
float median_filter(float *buffer, int size) {
    // Copy to temp array for sorting
    float temp[size];
    memcpy(temp, buffer, size * sizeof(float));

    // Simple bubble sort (small arrays)
    for (int i = 0; i < size - 1; i++) {
        for (int j = 0; j < size - i - 1; j++) {
            if (temp[j] > temp[j + 1]) {
                float swap = temp[j];
                temp[j] = temp[j + 1];
                temp[j + 1] = swap;
            }
        }
    }

    // Return median
    return temp[size / 2];
}
```

### 4.2 Rate of Change Calculation

Trend arrows require calculating the glucose rate of change:

```c
// Rate of change (mg/dL per minute)
typedef struct {
    float glucose_buffer[15];  // 15 minutes of history
    int buffer_index;
    bool buffer_full;
} GlucoseRateCalculator;

void glucose_rate_init(GlucoseRateCalculator *calc) {
    calc->buffer_index = 0;
    calc->buffer_full = false;
    for (int i = 0; i < 15; i++) {
        calc->glucose_buffer[i] = 0.0f;
    }
}

void glucose_rate_add_sample(GlucoseRateCalculator *calc, float glucose) {
    calc->glucose_buffer[calc->buffer_index] = glucose;
    calc->buffer_index++;

    if (calc->buffer_index >= 15) {
        calc->buffer_index = 0;
        calc->buffer_full = true;
    }
}

float glucose_rate_calculate(GlucoseRateCalculator *calc) {
    if (!calc->buffer_full) {
        return 0.0f;  // Not enough data
    }

    // Use linear regression over 15-minute window
    // y = mx + b, where m is the rate (mg/dL/min)

    float sum_x = 0, sum_y = 0, sum_xy = 0, sum_xx = 0;
    int n = 15;

    for (int i = 0; i < n; i++) {
        // Get chronological index
        int idx = (calc->buffer_index + i) % 15;
        float x = i;  // Time in minutes
        float y = calc->glucose_buffer[idx];

        sum_x += x;
        sum_y += y;
        sum_xy += x * y;
        sum_xx += x * x;
    }

    // Slope = rate of change
    float rate = (n * sum_xy - sum_x * sum_y) / (n * sum_xx - sum_x * sum_x);

    return rate;
}

// Convert rate to trend arrow
typedef enum {
    TREND_FALLING_FAST,    // < -3 mg/dL/min  (↓↓)
    TREND_FALLING,         // -3 to -2        (↓)
    TREND_FALLING_SLOW,    // -2 to -1        (↘)
    TREND_STABLE,          // -1 to +1        (→)
    TREND_RISING_SLOW,     // +1 to +2        (↗)
    TREND_RISING,          // +2 to +3        (↑)
    TREND_RISING_FAST      // > +3            (↑↑)
} GlucoseTrend;

GlucoseTrend rate_to_trend(float rate_mg_per_min) {
    if (rate_mg_per_min < -3.0f) return TREND_FALLING_FAST;
    if (rate_mg_per_min < -2.0f) return TREND_FALLING;
    if (rate_mg_per_min < -1.0f) return TREND_FALLING_SLOW;
    if (rate_mg_per_min < 1.0f)  return TREND_STABLE;
    if (rate_mg_per_min < 2.0f)  return TREND_RISING_SLOW;
    if (rate_mg_per_min < 3.0f)  return TREND_RISING;
    return TREND_RISING_FAST;
}

const char* trend_to_arrow(GlucoseTrend trend) {
    switch (trend) {
        case TREND_FALLING_FAST: return "↓↓";
        case TREND_FALLING:      return "↓";
        case TREND_FALLING_SLOW: return "↘";
        case TREND_STABLE:       return "→";
        case TREND_RISING_SLOW:  return "↗";
        case TREND_RISING:       return "↑";
        case TREND_RISING_FAST:  return "↑↑";
        default:                 return "?";
    }
}
```

### 4.3 Sensor Artifact Detection

```c
// Detect and flag sensor artifacts
typedef enum {
    SENSOR_OK,
    SENSOR_SIGNAL_LOW,      // Current too low
    SENSOR_SIGNAL_HIGH,     // Current saturated
    SENSOR_NOISE_HIGH,      // Excessive noise
    SENSOR_RATE_EXCEEDED,   // Physiologically impossible change
    SENSOR_COMPRESSION_LOW  // Pressure on sensor (false low)
} SensorStatus;

typedef struct {
    float min_current;        // nA
    float max_current;        // nA
    float max_noise;          // nA (std dev)
    float max_rate;           // mg/dL/min (physiological limit)
    float compression_threshold;  // Sudden drop detection
} ArtifactDetectorConfig;

SensorStatus detect_artifact(ArtifactDetectorConfig *cfg,
                            float current,
                            float noise_std,
                            float rate_of_change,
                            float previous_glucose,
                            float current_glucose) {

    // Check signal bounds
    if (current < cfg->min_current) {
        return SENSOR_SIGNAL_LOW;
    }
    if (current > cfg->max_current) {
        return SENSOR_SIGNAL_HIGH;
    }

    // Check noise level
    if (noise_std > cfg->max_noise) {
        return SENSOR_NOISE_HIGH;
    }

    // Check physiological plausibility
    // Glucose cannot change faster than ~4 mg/dL/min even in extreme cases
    if (fabsf(rate_of_change) > cfg->max_rate) {
        return SENSOR_RATE_EXCEEDED;
    }

    // Detect compression artifact
    // Sudden drop >30% suggests pressure on sensor
    if (previous_glucose > 0 &&
        (previous_glucose - current_glucose) / previous_glucose > 0.3f) {
        return SENSOR_COMPRESSION_LOW;
    }

    return SENSOR_OK;
}

// Configuration for typical CGM
ArtifactDetectorConfig default_artifact_config = {
    .min_current = 0.5f,      // 0.5 nA minimum
    .max_current = 100.0f,    // 100 nA maximum
    .max_noise = 2.0f,        // 2 nA noise threshold
    .max_rate = 4.0f,         // 4 mg/dL/min maximum rate
    .compression_threshold = 0.3f  // 30% drop threshold
};
```

---

## 5. Calibration Algorithms

### 5.1 Understanding Calibration

CGM calibration converts raw sensor current to glucose concentration:

```
    ┌─────────────────────────────────────────────────────────┐
    │               CALIBRATION CONCEPT                       │
    └─────────────────────────────────────────────────────────┘

    Raw Problem:
    - Sensor sensitivity varies between units
    - Sensitivity changes over sensor lifetime
    - Background current varies

    Solution: Use fingerstick blood glucose to calibrate

    Calibration Model:
    G_cgm = (i_sensor - i_background) / Sensitivity

    Where:
    - G_cgm = estimated glucose (mg/dL)
    - i_sensor = measured current (nA)
    - i_background = baseline current with no glucose
    - Sensitivity = slope (nA per mg/dL)

    These parameters are determined from fingerstick calibrations:

    Current (nA)
        │           Calibration points
    60  │                ● (180, 50 nA)
        │               ╱
    50  │              ╱
        │             ╱
    40  │            ╱
        │           ╱
    30  │          ╱  ● (120, 30 nA)
        │         ╱
    20  │        ╱
        │       ╱
    10  │      ● (70, 15 nA)
        │     ╱
     5  │────╱───── i_background (offset)
        └───────────────────────────────────────→
            70    120      180        Glucose (mg/dL)
```

### 5.2 Linear Calibration

```c
// Linear calibration algorithm
typedef struct {
    float sensitivity;      // Slope (nA per mg/dL)
    float background;       // Offset (nA)
    float r_squared;        // Fit quality
    int num_cal_points;     // Number of calibrations
    uint32_t last_cal_time; // Timestamp of last calibration
} CalibrationState;

typedef struct {
    float glucose_reference;  // Fingerstick glucose (mg/dL)
    float sensor_current;     // CGM current at same time (nA)
    uint32_t timestamp;       // When calibration was taken
} CalibrationPoint;

#define MAX_CAL_POINTS 10

typedef struct {
    CalibrationPoint points[MAX_CAL_POINTS];
    int num_points;
    CalibrationState state;
} CalibrationData;

void calibration_add_point(CalibrationData *cal,
                           float fingerstick_glucose,
                           float sensor_current,
                           uint32_t timestamp) {

    // Add new calibration point
    if (cal->num_points < MAX_CAL_POINTS) {
        cal->points[cal->num_points].glucose_reference = fingerstick_glucose;
        cal->points[cal->num_points].sensor_current = sensor_current;
        cal->points[cal->num_points].timestamp = timestamp;
        cal->num_points++;
    } else {
        // Shift out oldest point
        for (int i = 0; i < MAX_CAL_POINTS - 1; i++) {
            cal->points[i] = cal->points[i + 1];
        }
        cal->points[MAX_CAL_POINTS - 1].glucose_reference = fingerstick_glucose;
        cal->points[MAX_CAL_POINTS - 1].sensor_current = sensor_current;
        cal->points[MAX_CAL_POINTS - 1].timestamp = timestamp;
    }

    // Recalculate calibration parameters using linear regression
    calibration_update_parameters(cal);
}

void calibration_update_parameters(CalibrationData *cal) {
    if (cal->num_points < 2) {
        // Use factory defaults until we have enough points
        cal->state.sensitivity = 0.25f;  // Typical factory value
        cal->state.background = 5.0f;
        return;
    }

    // Linear regression: current = sensitivity * glucose + background
    float sum_g = 0, sum_i = 0, sum_gi = 0, sum_gg = 0;
    int n = cal->num_points;

    for (int j = 0; j < n; j++) {
        float g = cal->points[j].glucose_reference;
        float i = cal->points[j].sensor_current;

        sum_g += g;
        sum_i += i;
        sum_gi += g * i;
        sum_gg += g * g;
    }

    // Calculate slope and intercept
    float denom = n * sum_gg - sum_g * sum_g;
    if (fabsf(denom) < 1e-6f) {
        return;  // Singular matrix, keep current values
    }

    cal->state.sensitivity = (n * sum_gi - sum_g * sum_i) / denom;
    cal->state.background = (sum_i - cal->state.sensitivity * sum_g) / n;

    // Calculate R-squared
    float mean_i = sum_i / n;
    float ss_tot = 0, ss_res = 0;

    for (int j = 0; j < n; j++) {
        float g = cal->points[j].glucose_reference;
        float i = cal->points[j].sensor_current;
        float i_pred = cal->state.sensitivity * g + cal->state.background;

        ss_tot += (i - mean_i) * (i - mean_i);
        ss_res += (i - i_pred) * (i - i_pred);
    }

    cal->state.r_squared = 1.0f - (ss_res / ss_tot);
    cal->state.num_cal_points = n;
}

float calibration_current_to_glucose(CalibrationData *cal, float current) {
    // G = (i - background) / sensitivity
    float glucose = (current - cal->state.background) / cal->state.sensitivity;

    // Clamp to valid range
    if (glucose < 40.0f) glucose = 40.0f;
    if (glucose > 400.0f) glucose = 400.0f;

    return glucose;
}
```

### 5.3 Factory Calibration

Modern CGMs use **factory calibration** (no fingersticks needed):

```c
// Factory calibration uses sensor-specific parameters
// encoded in sensor RFID/NFC tag

typedef struct {
    uint32_t sensor_code;        // Unique sensor identifier
    float sensitivity_factor;    // Individual sensitivity adjustment
    float offset_correction;     // Background current correction
    float temp_coefficient;      // Temperature compensation
    float time_correction[14];   // Day-by-day sensitivity drift
} FactoryCalibration;

float apply_factory_calibration(FactoryCalibration *fc,
                                float raw_current,
                                float temperature,
                                int sensor_day) {

    // Temperature compensation
    float temp_corrected = raw_current *
                          (1.0f + fc->temp_coefficient * (temperature - 25.0f));

    // Time-based sensitivity drift correction
    int day_index = (sensor_day < 14) ? sensor_day : 13;
    float time_corrected = temp_corrected * fc->time_correction[day_index];

    // Apply individual sensor calibration
    float glucose = (time_corrected - fc->offset_correction) / fc->sensitivity_factor;

    return glucose;
}
```

### 5.4 Retrospective Calibration

Some systems recalculate past readings when new calibration data arrives:

```c
// Retrospective calibration smoothing
typedef struct {
    float glucose;
    float current;
    uint32_t timestamp;
    bool recalibrated;
} GlucoseHistory;

#define HISTORY_SIZE 288  // 24 hours at 5-minute intervals

void retrospective_recalibrate(GlucoseHistory *history,
                               int history_count,
                               CalibrationData *old_cal,
                               CalibrationData *new_cal,
                               uint32_t cal_time) {

    // Find how far back to recalibrate
    // Typically 3-6 hours before calibration
    uint32_t recal_window = 3 * 60 * 60;  // 3 hours in seconds

    for (int i = 0; i < history_count; i++) {
        // Only recalibrate readings within window
        if (cal_time - history[i].timestamp < recal_window) {

            // Weight between old and new calibration based on time
            float time_fraction = (float)(cal_time - history[i].timestamp) /
                                  recal_window;

            float old_glucose = calibration_current_to_glucose(old_cal,
                                                               history[i].current);
            float new_glucose = calibration_current_to_glucose(new_cal,
                                                               history[i].current);

            // Smooth transition from old to new calibration
            history[i].glucose = time_fraction * old_glucose +
                                (1.0f - time_fraction) * new_glucose;
            history[i].recalibrated = true;
        }
    }
}
```

---

## 6. Glucose Prediction

### 6.1 Why Predict?

Prediction gives users time to react to dangerous glucose levels:

```
    Without Prediction:           With 30-min Prediction:

    Glucose                       Glucose
        │                             │
    200 │                         200 │
        │                             │      ╱ Predicted
    150 │                         150 │     ╱
        │     ╲                       │    ╱
    100 │      ╲                  100 │   ╱
        │       ╲                     │  ╱
     70 │ ─ ─ ─ ─╲─ ─ Low alert   70 │─╱─ ─ ─ ─ ─ ─ ─
        │         ╲                   │╱
     50 │          ╲              50  │  Alert!
        └───────────────────          └─────────────────
             Now                          30 min before

    Prediction allows earlier warning
```

### 6.2 Linear Prediction

Simple prediction using rate of change:

```c
// Linear prediction
float predict_glucose_linear(float current_glucose,
                            float rate_mg_per_min,
                            float minutes_ahead) {

    float predicted = current_glucose + rate_mg_per_min * minutes_ahead;

    // Clamp to physiological range
    if (predicted < 20.0f) predicted = 20.0f;
    if (predicted > 500.0f) predicted = 500.0f;

    return predicted;
}

// Example: Predict 30 minutes ahead
float glucose_now = 120.0f;  // mg/dL
float rate = -2.0f;          // falling 2 mg/dL/min
float glucose_30min = predict_glucose_linear(glucose_now, rate, 30.0f);
// glucose_30min = 120 + (-2) * 30 = 60 mg/dL
```

### 6.3 Kalman Filter Prediction

More sophisticated prediction using state estimation:

```c
// Kalman filter for glucose prediction
typedef struct {
    // State vector: [glucose, rate, acceleration]
    float x[3];

    // Error covariance matrix (3x3)
    float P[3][3];

    // Process noise covariance
    float Q[3][3];

    // Measurement noise variance
    float R;

    // Time step (minutes)
    float dt;
} GlucoseKalmanFilter;

void kalman_init(GlucoseKalmanFilter *kf, float dt_minutes) {
    kf->dt = dt_minutes;

    // Initial state
    kf->x[0] = 100.0f;  // Glucose (mg/dL)
    kf->x[1] = 0.0f;    // Rate (mg/dL/min)
    kf->x[2] = 0.0f;    // Acceleration (mg/dL/min²)

    // Initial covariance (high uncertainty)
    memset(kf->P, 0, sizeof(kf->P));
    kf->P[0][0] = 1000.0f;
    kf->P[1][1] = 10.0f;
    kf->P[2][2] = 1.0f;

    // Process noise (tuned for glucose dynamics)
    memset(kf->Q, 0, sizeof(kf->Q));
    kf->Q[0][0] = 1.0f;    // Glucose variance
    kf->Q[1][1] = 0.1f;    // Rate variance
    kf->Q[2][2] = 0.01f;   // Acceleration variance

    // Measurement noise
    kf->R = 25.0f;  // CGM variance ~5 mg/dL std dev
}

void kalman_predict(GlucoseKalmanFilter *kf) {
    float dt = kf->dt;
    float dt2 = dt * dt / 2.0f;

    // State transition matrix F:
    // [1  dt  dt²/2]
    // [0   1     dt]
    // [0   0      1]

    // Predict state: x_pred = F * x
    float x_pred[3];
    x_pred[0] = kf->x[0] + kf->x[1] * dt + kf->x[2] * dt2;
    x_pred[1] = kf->x[1] + kf->x[2] * dt;
    x_pred[2] = kf->x[2];  // Assume constant acceleration

    // Update state
    memcpy(kf->x, x_pred, sizeof(x_pred));

    // Predict covariance: P_pred = F * P * F' + Q
    // (simplified for this example)
    kf->P[0][0] += kf->Q[0][0] + 2 * dt * kf->P[0][1] + dt * dt * kf->P[1][1];
    kf->P[1][1] += kf->Q[1][1];
    kf->P[2][2] += kf->Q[2][2];
}

void kalman_update(GlucoseKalmanFilter *kf, float glucose_measurement) {
    // Measurement matrix H = [1, 0, 0] (we only measure glucose)

    // Innovation (measurement residual)
    float y = glucose_measurement - kf->x[0];

    // Innovation covariance
    float S = kf->P[0][0] + kf->R;

    // Kalman gain
    float K[3];
    K[0] = kf->P[0][0] / S;
    K[1] = kf->P[1][0] / S;
    K[2] = kf->P[2][0] / S;

    // Update state estimate
    kf->x[0] += K[0] * y;
    kf->x[1] += K[1] * y;
    kf->x[2] += K[2] * y;

    // Update covariance
    float I_KH = 1.0f - K[0];
    kf->P[0][0] *= I_KH;
    kf->P[1][1] *= (1.0f - K[1]);
    kf->P[2][2] *= (1.0f - K[2]);
}

float kalman_get_glucose(GlucoseKalmanFilter *kf) {
    return kf->x[0];
}

float kalman_get_rate(GlucoseKalmanFilter *kf) {
    return kf->x[1];
}

float kalman_predict_future(GlucoseKalmanFilter *kf, float minutes_ahead) {
    float dt = minutes_ahead;
    float dt2 = dt * dt / 2.0f;

    return kf->x[0] + kf->x[1] * dt + kf->x[2] * dt2;
}
```

### 6.4 Machine Learning Prediction

Modern CGMs use ML for more accurate prediction:

```python
# Example: LSTM-based glucose prediction
import numpy as np
import tensorflow as tf
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import LSTM, Dense, Dropout

def create_glucose_predictor(sequence_length=24, prediction_horizon=6):
    """
    LSTM model for glucose prediction

    Parameters:
    - sequence_length: Number of past readings (e.g., 24 = 2 hours at 5-min intervals)
    - prediction_horizon: How far ahead to predict (e.g., 6 = 30 minutes)
    """
    model = Sequential([
        LSTM(64, input_shape=(sequence_length, 1), return_sequences=True),
        Dropout(0.2),
        LSTM(32, return_sequences=False),
        Dropout(0.2),
        Dense(prediction_horizon)
    ])

    model.compile(optimizer='adam', loss='mse', metrics=['mae'])
    return model

def prepare_glucose_sequences(glucose_data, sequence_length, prediction_horizon):
    """
    Prepare training data from continuous glucose readings
    """
    X, y = [], []

    for i in range(len(glucose_data) - sequence_length - prediction_horizon):
        X.append(glucose_data[i:i+sequence_length])
        y.append(glucose_data[i+sequence_length:i+sequence_length+prediction_horizon])

    return np.array(X).reshape(-1, sequence_length, 1), np.array(y)

# Example usage
# glucose_data = ... # Array of glucose readings at 5-min intervals

# model = create_glucose_predictor()
# X_train, y_train = prepare_glucose_sequences(glucose_data, 24, 6)
# model.fit(X_train, y_train, epochs=50, batch_size=32)

# Predict
# future_glucose = model.predict(recent_24_readings.reshape(1, 24, 1))
```

---

## 7. Alert and Safety Systems

### 7.1 Alert Thresholds

```c
// Alert configuration
typedef struct {
    float low_glucose;          // Low alert threshold (mg/dL)
    float high_glucose;         // High alert threshold (mg/dL)
    float urgent_low;           // Urgent low (mg/dL)
    float rate_falling;         // Falling rate alert (mg/dL/min)
    float rate_rising;          // Rising rate alert (mg/dL/min)
    bool low_prediction_enabled;
    float prediction_time_min;   // How far ahead to predict for alerts
} AlertConfig;

// Default clinical thresholds
AlertConfig default_alerts = {
    .low_glucose = 70.0f,
    .high_glucose = 180.0f,
    .urgent_low = 55.0f,
    .rate_falling = -2.0f,
    .rate_rising = 3.0f,
    .low_prediction_enabled = true,
    .prediction_time_min = 20.0f
};

typedef enum {
    ALERT_NONE,
    ALERT_LOW,
    ALERT_URGENT_LOW,
    ALERT_HIGH,
    ALERT_FALLING_FAST,
    ALERT_RISING_FAST,
    ALERT_LOW_PREDICTED,
    ALERT_SIGNAL_LOSS
} AlertType;

AlertType check_alerts(AlertConfig *cfg,
                       float current_glucose,
                       float rate_of_change,
                       float predicted_glucose,
                       bool signal_valid) {

    // Priority: Signal loss > Urgent low > Low > High > Rate > Predicted

    if (!signal_valid) {
        return ALERT_SIGNAL_LOSS;
    }

    if (current_glucose < cfg->urgent_low) {
        return ALERT_URGENT_LOW;
    }

    if (current_glucose < cfg->low_glucose) {
        return ALERT_LOW;
    }

    if (current_glucose > cfg->high_glucose) {
        return ALERT_HIGH;
    }

    if (rate_of_change < cfg->rate_falling) {
        return ALERT_FALLING_FAST;
    }

    if (rate_of_change > cfg->rate_rising) {
        return ALERT_RISING_FAST;
    }

    if (cfg->low_prediction_enabled && predicted_glucose < cfg->low_glucose) {
        return ALERT_LOW_PREDICTED;
    }

    return ALERT_NONE;
}
```

### 7.2 Alert Escalation

```c
// Alert escalation for critical situations
typedef struct {
    AlertType current_alert;
    uint32_t alert_start_time;
    int snooze_count;
    bool acknowledged;
} AlertState;

typedef struct {
    int initial_interval_sec;    // First alert interval
    int repeat_interval_sec;     // Repeat if not acknowledged
    int escalation_interval_sec; // Time before escalation
    int max_snoozes;            // Maximum snooze count
} AlertEscalationConfig;

void handle_alert_escalation(AlertState *state,
                            AlertType new_alert,
                            AlertEscalationConfig *cfg,
                            uint32_t current_time) {

    // New alert?
    if (new_alert != state->current_alert) {
        state->current_alert = new_alert;
        state->alert_start_time = current_time;
        state->snooze_count = 0;
        state->acknowledged = false;

        if (new_alert != ALERT_NONE) {
            trigger_alert_notification(new_alert, PRIORITY_HIGH);
        }
        return;
    }

    // Ongoing alert - check if we need to escalate
    if (new_alert != ALERT_NONE && !state->acknowledged) {
        uint32_t elapsed = current_time - state->alert_start_time;

        // Escalate if not acknowledged within escalation time
        if (elapsed > cfg->escalation_interval_sec) {
            trigger_emergency_contact();
        }

        // Repeat alert if not acknowledged
        if (elapsed % cfg->repeat_interval_sec == 0) {
            trigger_alert_notification(new_alert, PRIORITY_URGENT);
        }
    }
}

void snooze_alert(AlertState *state, AlertEscalationConfig *cfg) {
    if (state->snooze_count < cfg->max_snoozes) {
        state->snooze_count++;
        state->alert_start_time += cfg->repeat_interval_sec;
    }
    // Don't allow snoozing urgent low
    if (state->current_alert == ALERT_URGENT_LOW) {
        return;  // Cannot snooze urgent low
    }
}
```

---

## 8. Advanced Topics

### 8.1 Non-Invasive Glucose Monitoring (Future)

```
    ┌─────────────────────────────────────────────────────────┐
    │         NON-INVASIVE GLUCOSE APPROACHES                 │
    └─────────────────────────────────────────────────────────┘

    1. NEAR-INFRARED SPECTROSCOPY (NIR)
       ┌────────────────────────────────────────────────────┐
       │ Principle: Glucose absorbs specific NIR wavelengths│
       │            (around 1550 nm and 2150 nm)            │
       │                                                    │
       │ Challenges:                                        │
       │ - Weak absorption (glucose is dilute in blood)     │
       │ - Water, protein, fat also absorb                  │
       │ - Scattering in tissue                             │
       │ - Temperature sensitivity                          │
       │                                                    │
       │ Status: Research stage, no consumer products yet   │
       └────────────────────────────────────────────────────┘

    2. RAMAN SPECTROSCOPY
       ┌────────────────────────────────────────────────────┐
       │ Principle: Laser excites molecules, measuring      │
       │            scattered light shift (Raman effect)    │
       │                                                    │
       │ Advantages:                                        │
       │ - Glucose has unique Raman signature               │
       │ - Less affected by water                           │
       │                                                    │
       │ Challenges:                                        │
       │ - Very weak signal (need sensitive detection)      │
       │ - Requires laser (power, safety concerns)          │
       │ - Fluorescence interference                        │
       │                                                    │
       │ Status: Some promising research results            │
       └────────────────────────────────────────────────────┘

    3. BIOIMPEDANCE
       ┌────────────────────────────────────────────────────┐
       │ Principle: Glucose affects tissue electrical       │
       │            properties (dielectric permittivity)    │
       │                                                    │
       │ Measurement: Apply small AC current, measure       │
       │              impedance at multiple frequencies     │
       │                                                    │
       │ Challenges:                                        │
       │ - Affected by hydration, temperature, sweat        │
       │ - Small glucose effect vs large baseline           │
       │ - Individual calibration required                  │
       │                                                    │
       │ Status: Some devices in development                │
       └────────────────────────────────────────────────────┘

    4. SWEAT/TEAR GLUCOSE
       ┌────────────────────────────────────────────────────┐
       │ Principle: Glucose in sweat/tears correlates with  │
       │            blood glucose                           │
       │                                                    │
       │ Form factors:                                      │
       │ - Smart contact lens (Google/Verily project)       │
       │ - Sweat patch (several startups)                   │
       │                                                    │
       │ Challenges:                                        │
       │ - Weak correlation with blood glucose              │
       │ - Large time lag                                   │
       │ - Affected by sweat rate, contamination            │
       │                                                    │
       │ Status: Most projects discontinued due to accuracy │
       └────────────────────────────────────────────────────┘
```

### 8.2 Closed-Loop Insulin Delivery

CGM enables automated insulin pump control:

```c
// Simplified automated insulin delivery algorithm
// (Based on PID control - real systems use more sophisticated MPC)

typedef struct {
    float target_glucose;        // Target glucose (mg/dL)
    float basal_rate;            // Base insulin rate (U/hr)
    float insulin_sensitivity;   // How much 1 unit drops glucose (mg/dL/U)
    float carb_ratio;           // Grams of carbs per unit insulin
    float correction_factor;    // Insulin sensitivity for corrections
    float max_bolus;            // Safety limit (U)
    float max_basal_multiplier; // Max basal increase factor
} InsulinDeliveryConfig;

typedef struct {
    float kp;  // Proportional gain
    float ki;  // Integral gain
    float kd;  // Derivative gain
    float integral_error;
    float last_error;
} PIDController;

float calculate_insulin_adjustment(InsulinDeliveryConfig *cfg,
                                  PIDController *pid,
                                  float current_glucose,
                                  float rate_of_change,
                                  float dt_hours) {

    // Error: deviation from target
    float error = current_glucose - cfg->target_glucose;

    // PID control
    float p_term = pid->kp * error;

    pid->integral_error += error * dt_hours;
    // Anti-windup: limit integral
    if (pid->integral_error > 100.0f) pid->integral_error = 100.0f;
    if (pid->integral_error < -50.0f) pid->integral_error = -50.0f;
    float i_term = pid->ki * pid->integral_error;

    float derivative = (error - pid->last_error) / dt_hours;
    float d_term = pid->kd * derivative;

    pid->last_error = error;

    // Total adjustment (units/hour relative to basal)
    float adjustment = (p_term + i_term + d_term) / cfg->insulin_sensitivity;

    // Safety limits
    float new_rate = cfg->basal_rate + adjustment;

    // Never go below zero
    if (new_rate < 0.0f) new_rate = 0.0f;

    // Limit maximum rate
    float max_rate = cfg->basal_rate * cfg->max_basal_multiplier;
    if (new_rate > max_rate) new_rate = max_rate;

    // Suspend delivery if glucose is low or falling fast
    if (current_glucose < 70.0f ||
        (current_glucose < 100.0f && rate_of_change < -2.0f)) {
        new_rate = 0.0f;  // Suspend insulin
    }

    return new_rate;
}
```

### 8.3 Time in Range (TIR) Metrics

```c
// Calculate standardized CGM metrics
typedef struct {
    float time_in_range;         // % time 70-180 mg/dL
    float time_below_range;      // % time < 70 mg/dL
    float time_below_range_severe; // % time < 54 mg/dL
    float time_above_range;      // % time > 180 mg/dL
    float time_above_range_severe; // % time > 250 mg/dL
    float gmi;                   // Glucose Management Indicator (estimated HbA1c)
    float cv;                    // Coefficient of Variation
    float mean_glucose;
    float std_glucose;
} CGMMetrics;

void calculate_cgm_metrics(float *glucose_data,
                          int num_readings,
                          CGMMetrics *metrics) {

    // Calculate mean
    float sum = 0;
    for (int i = 0; i < num_readings; i++) {
        sum += glucose_data[i];
    }
    metrics->mean_glucose = sum / num_readings;

    // Calculate standard deviation
    float var_sum = 0;
    for (int i = 0; i < num_readings; i++) {
        float diff = glucose_data[i] - metrics->mean_glucose;
        var_sum += diff * diff;
    }
    metrics->std_glucose = sqrtf(var_sum / (num_readings - 1));

    // Coefficient of variation
    metrics->cv = (metrics->std_glucose / metrics->mean_glucose) * 100.0f;

    // Glucose Management Indicator (GMI)
    // Formula: GMI = 3.31 + 0.02392 × mean glucose (mg/dL)
    metrics->gmi = 3.31f + 0.02392f * metrics->mean_glucose;

    // Time in ranges
    int tir = 0, tbr = 0, tbr_severe = 0, tar = 0, tar_severe = 0;

    for (int i = 0; i < num_readings; i++) {
        float g = glucose_data[i];

        if (g < 54.0f) tbr_severe++;
        else if (g < 70.0f) tbr++;
        else if (g <= 180.0f) tir++;
        else if (g <= 250.0f) tar++;
        else tar_severe++;
    }

    metrics->time_in_range = (float)tir / num_readings * 100.0f;
    metrics->time_below_range = (float)(tbr + tbr_severe) / num_readings * 100.0f;
    metrics->time_below_range_severe = (float)tbr_severe / num_readings * 100.0f;
    metrics->time_above_range = (float)(tar + tar_severe) / num_readings * 100.0f;
    metrics->time_above_range_severe = (float)tar_severe / num_readings * 100.0f;
}

// Clinical targets (2019 Consensus)
// TIR > 70%
// TBR < 4% (<1% severe)
// TAR < 25%
// CV < 36%
```

---

## 9. Learning Resources

### 9.1 Books

| Book | Author | Focus |
|------|--------|-------|
| **"Diabetes Technology: Science and Practice"** | Garg, Hirsch | Comprehensive CGM/pump reference |
| **"Handbook of Diabetes Technology"** | Bruttomesso, Grassi | Technical device details |
| **"Continuous Glucose Monitoring"** | Frontiers collection | Open access research |
| **"Electrochemical Methods"** | Bard, Faulkner | Sensor chemistry fundamentals |

### 9.2 Research Papers

| Paper | Topic | Link |
|-------|-------|------|
| **"Clinical Targets for CGM Data Interpretation"** (Battelino 2019) | TIR targets | [Diabetes Care](https://doi.org/10.2337/dc19-1028) |
| **"Glucose Oxidase Biosensors"** (Wilson 2005) | Sensor chemistry | [Biosensors & Bioelectronics](https://doi.org/10.1016/j.bios.2005.04.009) |
| **"Machine Learning for Glucose Prediction"** | ML approaches | Various on arXiv |

### 9.3 Datasets

| Dataset | Description | Link |
|---------|-------------|------|
| **OhioT1DM** | CGM + insulin + meal data | [ohiot1dm](http://smarthealth.cs.ohio.edu/OhioT1DM-dataset.html) |
| **D1NAMO** | Type 1 diabetes monitoring | [physionet](https://physionet.org/content/d1namo/) |
| **Gluroo** | Open diabetes data | [github](https://github.com/gluroo) |

### 9.4 Online Resources

| Resource | Description | Link |
|----------|-------------|------|
| **Nightscout** | Open source CGM display | [nightscout.info](https://nightscout.info/) |
| **OpenAPS** | DIY closed-loop system | [openaps.org](https://openaps.org/) |
| **Loop** | iOS closed-loop app | [loopkit.github.io](https://loopkit.github.io/loopdocs/) |
| **Tidepool** | Diabetes data platform | [tidepool.org](https://www.tidepool.org/) |

### 9.5 YouTube/Educational

| Resource | Topics |
|----------|--------|
| **DiaTribe** | CGM reviews, diabetes tech |
| **Type 1 University** | Educational diabetes content |
| **JDRF** | Research updates |
| **Adam Brown** | "Bright Spots & Landmines" author |

---

## Summary

CGM technology combines chemistry, electronics, and algorithms:

| Component | Key Concepts |
|-----------|--------------|
| **Sensor Chemistry** | Glucose oxidase, H₂O₂ detection, mediators |
| **Hardware** | Potentiostat, ADC, low-power wireless |
| **Signal Processing** | Noise filtering, rate calculation, artifact detection |
| **Calibration** | Factory vs user calibration, linear regression |
| **Prediction** | Kalman filters, LSTM neural networks |
| **Safety** | Alert thresholds, escalation, closed-loop control |

**Key Takeaways**:
1. CGM measures interstitial fluid, not blood—5-15 min lag
2. Enzyme electrodes convert glucose to electrical current
3. Calibration is critical for accuracy
4. Modern CGMs achieve MARD < 10%
5. Trend arrows and predictions enable proactive management
6. Non-invasive glucose monitoring remains an unsolved challenge

---

**Next Tutorial**: [4.7 Blood Pressure Estimation →](07_blood_pressure_algorithms.md)
