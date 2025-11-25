# Heart Rate, HRV, and Stress Monitoring Algorithms Guide

This guide provides a comprehensive explanation of heart rate measurement, heart rate variability (HRV), heart rate zones, and stress monitoring concepts in wearable devices, with detailed references to PebbleOS's implementation.

## Table of Contents

1. [Introduction to Heart Rate Monitoring](#1-introduction-to-heart-rate-monitoring)
2. [PPG Sensor Technology](#2-ppg-sensor-technology)
3. [Heart Rate Measurement Algorithm](#3-heart-rate-measurement-algorithm)
4. [Heart Rate Variability (HRV)](#4-heart-rate-variability-hrv)
5. [Heart Rate Zones](#5-heart-rate-zones)
6. [Stress Monitoring Concepts](#6-stress-monitoring-concepts)
7. [PebbleOS Code References](#7-pebbleos-code-references)

---

## 1. Introduction to Heart Rate Monitoring

### 1.1 Why Heart Rate Monitoring Matters

Heart rate provides valuable health insights:
- **Resting heart rate**: Overall cardiovascular fitness
- **Exercise heart rate**: Workout intensity and calorie burn
- **Heart rate zones**: Training effectiveness
- **Heart rate variability**: Recovery, stress, and autonomic health

### 1.2 Heart Rate Basics

| Term | Definition | Typical Range |
|------|------------|---------------|
| Resting HR | Heart rate while completely at rest | 60-100 BPM |
| Maximum HR | Highest achievable heart rate | ~220 - age |
| Heart Rate Reserve | Max HR - Resting HR | 100-140 BPM |
| HRV | Beat-to-beat variation in heart rate | 20-100+ ms |

### 1.3 Wearable HR Monitoring Approach

Wearables use **optical sensors** (PPG - Photoplethysmography) rather than electrical (ECG):

```
┌─────────────────────────────────────────────────────────────────┐
│                    PPG SENSOR PRINCIPLE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│    LED (Green light) ────► Skin ────► Blood vessels            │
│                              │                                  │
│                              ▼                                  │
│                        Blood volume                             │
│                        pulsates with                            │
│                        each heartbeat                           │
│                              │                                  │
│                              ▼                                  │
│    Photodetector ◄──── Reflected light varies                   │
│                        with blood volume                        │
│                              │                                  │
│                              ▼                                  │
│                     Waveform analysis                           │
│                     extracts heart rate                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. PPG Sensor Technology

### 2.1 The AS7000 Sensor

PebbleOS uses the **AMS AS7000** optical heart rate sensor.

**Source**: `src/fw/drivers/hrm/as7000/as7000.c`

**Key specifications**:
- **Core**: ARM Cortex-M0 SoC with integrated PPG peripherals
- **Sampling**: 12.5 kHz accelerometer, ~1 Hz PPG output
- **Interface**: I2C with handshake protocol
- **Features**: BPM, HRV, presence detection, GSR (unused)

### 2.2 Sensor Applications

**Source**: `src/fw/drivers/hrm/as7000/as7000.c:156-163`

```c
typedef enum AS7000AppId {
    AS7000AppId_Idle = 0x00,      // Sensor idle
    AS7000AppId_Loader = 0x01,    // Firmware loader
    AS7000AppId_HRM = 0x02,       // Heart Rate Monitor (BPM)
    AS7000AppId_PRV = 0x04,       // Pulse Rate Variability (HRV)
    AS7000AppId_GSR = 0x08,       // Galvanic Skin Response
    AS7000AppId_NTC = 0x10,       // Temperature
} AS7000AppId;
```

### 2.3 PPG Data Structure

**Source**: `src/fw/services/common/hrm/hrm_manager.h:154-159`

```c
typedef struct {
    int num_samples;
    uint8_t indexes[MAX_PPG_SAMPLES];   // Sample indices (up to 20)
    uint16_t ppg[MAX_PPG_SAMPLES];      // Raw PPG values
    uint16_t tia[MAX_PPG_SAMPLES];      // Transimpedance amplifier values
} HRMPPGData;
```

### 2.4 Presence Detection

**Source**: `src/fw/drivers/hrm/as7000/as7000.c:84-90`

The sensor detects whether the watch is on-wrist using light reflection:

```c
// Presence detection thresholds (ADC counts)
#define PRES_DETECT_THRSH_BLACK  (54 * 64)  // 3456 for dark skin/band
#define PRES_DETECT_THRSH_WHITE  (78 * 64)  // 4992 for light skin/band
```

Lower reflected light = higher blood absorption = watch on wrist.

---

## 3. Heart Rate Measurement Algorithm

### 3.1 HRM Data Structure

**Source**: `src/fw/services/common/hrm/hrm_manager.h:161-174`

```c
typedef struct {
    uint16_t led_current_ua;        // LED current in microamps
    uint8_t hrm_bpm;                // Heart rate in BPM
    HRMQuality hrm_quality;         // Signal quality

    uint16_t hrv_ppi_ms;            // Peak-to-peak interval (HRV)
    HRMQuality hrv_quality;         // HRV quality
    uint8_t hrm_status;             // AS7000 status register

    HRMAccelData accel_data;        // Motion data
    HRMPPGData ppg_data;            // Raw PPG samples
} HRMData;
```

### 3.2 Signal Quality Index (SQI)

**Source**: `src/fw/drivers/hrm/as7000/as7000.c:137-147`

The AS7000 provides a Signal Quality Index (0-254):

```c
enum AS7000SQIThreshold {
    AS7000SQIThreshold_Excellent = 2,      // SQI 0-2
    AS7000SQIThreshold_Good = 5,           // SQI 3-5
    AS7000SQIThreshold_Acceptable = 8,     // SQI 6-8
    AS7000SQIThreshold_Poor = 10,          // SQI 9-10
    AS7000SQIThreshold_Worst = 20,         // SQI 11-20
    AS7000SQIThreshold_OffWrist = 254,     // Watch removed
};
```

### 3.3 Quality Enum Mapping

**Source**: `src/fw/drivers/hrm/as7000/as7000.c:338-354`

```c
typedef enum {
    HRMQuality_NoAccel = -2,        // No accelerometer data
    HRMQuality_OffWrist = -1,       // Watch removed
    HRMQuality_NoSignal = 0,        // No PPG signal
    HRMQuality_Worst,               // SQI > 20
    HRMQuality_Poor,                // SQI 10-20
    HRMQuality_Acceptable,          // SQI 8-10
    HRMQuality_Good,                // SQI 5-8
    HRMQuality_Excellent,           // SQI 0-2
} HRMQuality;
```

### 3.4 HR Sampling Strategy

**Source**: `src/fw/services/normal/activity/activity_private.h:67-87`

```c
// HR sampling parameters
#define ACTIVITY_DEFAULT_HR_PERIOD_SEC      (10 * SECONDS_PER_MINUTE)  // 10 min
#define ACTIVITY_DEFAULT_HR_ON_TIME_SEC     (SECONDS_PER_MINUTE)        // 1 min
#define ACTIVITY_MIN_NUM_SAMPLES_SHORT_CIRCUIT  (15)  // Early exit after 15 good samples
#define ACTIVITY_MIN_NUM_SAMPLES_FOR_HR_ZONE    (10)  // Min samples for zone
#define ACTIVITY_MIN_HR_QUALITY_THRESH      (HRMQuality_Good)
#define ACTIVITY_MAX_HR_SAMPLES             (3 * SECONDS_PER_MINUTE)   // 180 max
```

**Sampling Strategy**:
1. Every 10 minutes, activate HR sensor for 1 minute
2. Collect up to 180 samples
3. Short-circuit after 15 "good" quality samples
4. Compute median for stability

### 3.5 Valid HR Range

**Source**: `src/fw/services/normal/activity/activity.h:102-104`

```c
#define ACTIVITY_DEFAULT_MIN_HR  40
#define ACTIVITY_DEFAULT_MAX_HR  200
```

Values outside 40-200 BPM are considered invalid.

---

## 4. Heart Rate Variability (HRV)

### 4.1 What is HRV?

Heart Rate Variability measures the variation in time between consecutive heartbeats:

```
    ┌───┐     ┌───┐     ┌───┐     ┌───┐
    │   │     │   │     │   │     │   │
────┘   └─────┘   └─────┘   └─────┘   └────
        │←─────→│ │←─────→│ │←─────→│
          820ms     780ms     850ms

        PPI (Peak-to-Peak Interval) varies
        Higher variation = Better HRV
```

### 4.2 HRV Data Structure

**Source**: `src/fw/kernel/events.h:633-636`

```c
typedef struct HRMHRVData {
    uint16_t ppi_ms;        // Peak-to-peak interval in milliseconds
    HRMQuality quality:8;   // Signal quality
} HRMHRVData;  // 3 bytes
```

### 4.3 HRV Feature Request

**Source**: `src/fw/services/common/hrm/hrm_manager.h:40-56`

```c
typedef enum {
    HRMFeature_BPM = (1 << 0),          // Heart rate in BPM
    HRMFeature_HRV = (1 << 1),          // HRV (PPI intervals)
    HRMFeature_LEDCurrent = (1 << 2),   // LED current (internal)
    HRMFeature_Diagnostics = (1 << 3),  // Raw PPG & accel data
} HRMFeature;
```

### 4.4 HRV Metrics (Theoretical)

While PebbleOS collects raw PPI data, common HRV metrics can be derived:

**Time-Domain Metrics**:
| Metric | Formula | Meaning |
|--------|---------|---------|
| SDNN | std(all PPI intervals) | Overall HRV |
| RMSSD | √(mean(diff²)) | Short-term variability |
| pNN50 | % of intervals differing by >50ms | Parasympathetic activity |

**Frequency-Domain Metrics**:
| Metric | Frequency Band | Meaning |
|--------|----------------|---------|
| VLF | 0.003-0.04 Hz | Thermoregulation, hormones |
| LF | 0.04-0.15 Hz | Sympathetic + Parasympathetic |
| HF | 0.15-0.4 Hz | Parasympathetic (vagal) tone |
| LF/HF Ratio | LF power / HF power | Sympathovagal balance |

**Note**: These calculations are NOT implemented in PebbleOS firmware but can be computed from the PPI data.

---

## 5. Heart Rate Zones

### 5.1 Zone Definitions

**Source**: `src/fw/services/normal/activity/hr_util.h`

```c
typedef enum {
    HRZone_Zone0,    // Below Zone 1 (warm-up)
    HRZone_Zone1,    // Light intensity
    HRZone_Zone2,    // Moderate intensity
    HRZone_Zone3,    // High intensity
    HRZone_Max,
} HRZone;
```

### 5.2 Zone Threshold Calculation

**Source**: `src/fw/services/normal/activity/activity.h:88-95`

Zones are calculated using the **Karvonen Formula** (Heart Rate Reserve method):

```c
#define ACTIVITY_HEART_RATE_DEFAULT_PREFERENCES { \
    .resting_hr = 70,                               \
    .elevated_hr = 100,                             \
    .max_hr = 220 - ACTIVITY_DEFAULT_AGE_YEARS,     \  // 220 - 30 = 190
    .zone1_threshold = 130,  // 50% of HRR          \
    .zone2_threshold = 154,  // 70% of HRR          \
    .zone3_threshold = 172,  // 85% of HRR          \
}
```

### 5.3 Karvonen Formula

```
Heart Rate Reserve (HRR) = Max HR - Resting HR

Zone Threshold = Resting HR + (HRR × Percentage)
```

**Example** (Age 30, Resting HR 70):
```
Max HR = 220 - 30 = 190 BPM
HRR = 190 - 70 = 120 BPM

Zone 1 (50%): 70 + (120 × 0.50) = 130 BPM
Zone 2 (70%): 70 + (120 × 0.70) = 154 BPM
Zone 3 (85%): 70 + (120 × 0.85) = 172 BPM
```

### 5.4 Zone Classification

**Source**: `src/fw/services/normal/activity/hr_util.c:22-36`

```c
HRZone hr_util_get_hr_zone(int bpm) {
    const int zone_thresholds[HRZone_Max] = {
        activity_prefs_heart_get_zone1_threshold(),  // 130
        activity_prefs_heart_get_zone2_threshold(),  // 154
        activity_prefs_heart_get_zone3_threshold(),  // 172
    };

    HRZone zone;
    for (zone = HRZone_Zone0; zone < HRZone_Max; zone++) {
        if (bpm < zone_thresholds[zone]) {
            break;
        }
    }
    return zone;
}
```

### 5.5 Zone Characteristics

| Zone | % HRR | Intensity | Purpose |
|------|-------|-----------|---------|
| Zone 0 | < 50% | Very Light | Warm-up, recovery |
| Zone 1 | 50-70% | Light | Fat burning, endurance |
| Zone 2 | 70-85% | Moderate | Aerobic fitness |
| Zone 3 | > 85% | Hard | Anaerobic threshold |

### 5.6 Elevated HR Detection

**Source**: `src/fw/services/normal/activity/hr_util.c:38-40`

```c
bool hr_util_is_elevated(int bpm) {
    return bpm >= activity_prefs_heart_get_elevated_hr();  // default: 100 BPM
}
```

### 5.7 Zone Tracking Metrics

**Source**: `src/fw/services/normal/activity/activity.h:131-133`

```c
ActivityMetricHeartRateZone1Minutes,  // Time in Zone 1
ActivityMetricHeartRateZone2Minutes,  // Time in Zone 2
ActivityMetricHeartRateZone3Minutes,  // Time in Zone 3
```

---

## 6. Stress Monitoring Concepts

### 6.1 Important Note

**PebbleOS does NOT implement stress detection algorithms**. While the AS7000 sensor supports GSR (Galvanic Skin Response), this feature is not used in the firmware.

```c
// From as7000.c - GSR app defined but never used
AS7000AppId_GSR = 0x08,  // Galvanic Skin Response (stress indicator)
```

### 6.2 How Stress Monitoring Works (Theory)

Modern wearables use several signals to estimate stress:

#### Method 1: HRV-Based Stress

Low HRV correlates with stress:

```
┌─────────────────────────────────────────────────────────────────┐
│                  HRV-STRESS RELATIONSHIP                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  High HRV ────────────────────────────► Low Stress              │
│  (Variable intervals)                   (Relaxed, recovered)    │
│                                                                 │
│  Low HRV ─────────────────────────────► High Stress             │
│  (Regular intervals)                    (Fight-or-flight)       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Common metrics**:
- **RMSSD**: Root Mean Square of Successive Differences
  - Higher = more relaxed
  - Lower = more stressed

- **LF/HF Ratio**: Sympathetic/Parasympathetic balance
  - Higher ratio = sympathetic dominance (stress)
  - Lower ratio = parasympathetic dominance (relaxed)

#### Method 2: GSR-Based Stress

Galvanic Skin Response measures skin conductance:

```
Stress → Sympathetic activation → Sweat glands activate →
    → Skin conductance increases → GSR sensor detects change
```

**Note**: AS7000 has GSR capability but PebbleOS doesn't use it.

#### Method 3: Combined Approach

Advanced systems combine multiple signals:
- Heart rate elevation
- HRV decrease
- Skin conductance increase
- Respiratory rate increase
- Activity context (stationary = stress more likely)

### 6.3 Implementing Stress Detection (Conceptual)

A basic stress algorithm using available PebbleOS data:

```
Algorithm: Simple HRV-Based Stress Estimate

1. Collect PPI intervals over 5-minute window
2. Calculate RMSSD:
   RMSSD = √(Σ(PPI[n+1] - PPI[n])² / (N-1))

3. Normalize to 0-100 scale:
   stress_score = 100 - (RMSSD - min_rmssd) / (max_rmssd - min_rmssd) * 100

4. Apply context:
   if (activity_level > threshold):
       stress_score = adjusted_for_exercise

5. Output:
   0-30:  Low stress
   31-60: Moderate stress
   61-100: High stress
```

**Note**: This is theoretical - not implemented in PebbleOS.

### 6.4 Data Available for Stress Analysis

PebbleOS provides the raw data needed:

| Data | Source | Use for Stress |
|------|--------|----------------|
| PPI intervals | HRMFeature_HRV | Calculate RMSSD, LF/HF |
| Heart rate | HRMFeature_BPM | Detect elevation |
| Activity level | VMC | Context (resting vs active) |
| Time of day | RTC | Baseline adjustment |

---

## 7. PebbleOS Code References

### 7.1 Key Files

| Component | File Path |
|-----------|-----------|
| AS7000 driver | `src/fw/drivers/hrm/as7000/as7000.c` |
| AS7000 header | `src/fw/drivers/hrm/as7000/as7000.h` |
| HRM manager | `src/fw/services/common/hrm/hrm_manager.c` |
| HRM manager header | `src/fw/services/common/hrm/hrm_manager.h` |
| HR utility functions | `src/fw/services/normal/activity/hr_util.c` |
| Activity definitions | `src/fw/services/normal/activity/activity.h` |
| Activity insights | `src/fw/services/normal/activity/activity_insights.c` |
| HRM events | `src/fw/kernel/events.h` |

### 7.2 Key Structures

**HeartRatePreferences**:
```c
// src/fw/services/normal/activity/activity.h:57-64
typedef struct PACKED HeartRatePreferences {
    uint8_t resting_hr;         // Resting heart rate
    uint8_t elevated_hr;        // Elevated threshold
    uint8_t max_hr;            // Maximum heart rate
    uint8_t zone1_threshold;    // Zone 1 lower bound
    uint8_t zone2_threshold;    // Zone 2 lower bound
    uint8_t zone3_threshold;    // Zone 3 lower bound
} HeartRatePreferences;
```

**HRMData**:
```c
// src/fw/services/common/hrm/hrm_manager.h:161-174
typedef struct {
    uint16_t led_current_ua;
    uint8_t hrm_bpm;
    HRMQuality hrm_quality;
    uint16_t hrv_ppi_ms;
    HRMQuality hrv_quality;
    uint8_t hrm_status;
    HRMAccelData accel_data;
    HRMPPGData ppg_data;
} HRMData;
```

### 7.3 Key Functions

| Function | File | Purpose |
|----------|------|---------|
| `hr_util_get_hr_zone()` | hr_util.c:22 | Determine HR zone |
| `hr_util_is_elevated()` | hr_util.c:38 | Check if HR elevated |
| `as7000_get_data()` | as7000.c | Read sensor data |
| `hrm_manager_subscribe()` | hrm_manager.c | Subscribe to HR events |

### 7.4 Complete HR Data Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    HEART RATE PIPELINE                          │
└─────────────────────────────────────────────────────────────────┘

AS7000 PPG Sensor
        │
        │ LED reflects off blood vessels
        │ Photodetector captures signal
        ▼
┌───────────────────┐
│ AS7000 Firmware   │
│ (Cortex-M0)       │──► On-chip PPG processing
│                   │──► Peak detection
│                   │──► BPM calculation
│                   │──► SQI quality assessment
└───────────────────┘
        │
        │ I2C handshake
        ▼
┌───────────────────┐
│ AS7000 Driver     │
│ (as7000.c)        │──► Read BPM, PPI, quality
│                   │──► Map SQI to HRMQuality
└───────────────────┘
        │
        │ HRMData struct
        ▼
┌───────────────────┐
│ HRM Manager       │
│ (hrm_manager.c)   │──► Event generation
│                   │──► Subscription handling
└───────────────────┘
        │
        ├──────────────────┐
        │                  │
        ▼                  ▼
┌───────────────┐  ┌───────────────┐
│ Activity      │  │ Health        │
│ Service       │  │ Service API   │
│               │  │               │
│ - Zone time   │  │ - Get current │
│ - Median HR   │  │ - Historical  │
│ - Insights    │  │ - Subscribe   │
└───────────────┘  └───────────────┘
        │
        ▼
┌───────────────────┐
│ User Application  │
│ (Watchface/App)   │
└───────────────────┘
```

---

## Summary

### What PebbleOS Implements

| Feature | Status | Details |
|---------|--------|---------|
| Heart Rate (BPM) | ✅ Implemented | AS7000 PPG sensor, quality assessment |
| HRV (PPI) | ✅ Implemented | Peak-to-peak intervals with quality |
| Heart Rate Zones | ✅ Implemented | Karvonen formula, 4 zones |
| Zone Time Tracking | ✅ Implemented | Minutes in each zone |
| Elevated HR Detection | ✅ Implemented | Threshold-based (default 100 BPM) |
| Stress Detection | ❌ Not Implemented | GSR capability exists but unused |

### Key Parameters

| Parameter | Default Value | Purpose |
|-----------|---------------|---------|
| Resting HR | 70 BPM | Baseline for calculations |
| Elevated HR | 100 BPM | Alert threshold |
| Max HR | 220 - age | Zone calculations |
| Zone 1 | 50% HRR | Light intensity threshold |
| Zone 2 | 70% HRR | Moderate intensity threshold |
| Zone 3 | 85% HRR | High intensity threshold |
| Valid HR range | 40-200 BPM | Data validation |

### Extending to Stress Detection

While not implemented, stress detection could be added using:
1. **Available data**: PPI intervals, heart rate, activity level
2. **Algorithm**: RMSSD-based HRV analysis
3. **Context**: Activity level to distinguish exercise from stress
4. **Hardware**: GSR capability in AS7000 (currently unused)

The firmware provides all necessary infrastructure; only the stress algorithm implementation is missing.
