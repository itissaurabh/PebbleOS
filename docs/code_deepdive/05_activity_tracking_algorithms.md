# Activity Tracking Algorithms: Step Counting, Sleep Detection, and IMU Sensors

This guide provides a comprehensive explanation of how step counting, activity detection, and sleep tracking algorithms work using accelerometer and IMU sensors. We cover the fundamental concepts, mathematical foundations, and link to PebbleOS's actual implementations.

## Table of Contents

1. [Introduction to Motion Sensors](#1-introduction-to-motion-sensors)
2. [Fundamentals of Step Detection](#2-fundamentals-of-step-detection)
3. [The Kraepelin Algorithm (PebbleOS Implementation)](#3-the-kraepelin-algorithm-pebbleos-implementation)
4. [Sleep Detection Algorithm](#4-sleep-detection-algorithm)
5. [Distance and Calorie Calculations](#5-distance-and-calorie-calculations)
6. [PebbleOS Code References](#6-pebbleos-code-references)

---

## 1. Introduction to Motion Sensors

### 1.1 Accelerometer Basics

An accelerometer measures **acceleration forces** acting on a body along three perpendicular axes (X, Y, Z). These forces include:

- **Static forces**: Gravity (always ~9.8 m/s² pointing toward Earth's center)
- **Dynamic forces**: Motion, vibration, shock

```
           +Z (up)
            │
            │
            │
            └──────── +Y (forward)
           /
          /
         +X (right)
```

**Output**: Each axis outputs acceleration in **milli-g** (mg) where 1000 mg = 1g = 9.8 m/s²

**Example readings when stationary**:
- Watch flat on table: X≈0, Y≈0, Z≈1000 mg (gravity)
- Watch on side: X≈1000, Y≈0, Z≈0 mg

### 1.2 IMU (Inertial Measurement Unit)

A 6-axis IMU combines:
- **3-axis accelerometer**: Measures linear acceleration
- **3-axis gyroscope**: Measures angular velocity (rotation rate)

A 9-axis IMU adds:
- **3-axis magnetometer**: Measures magnetic field (compass heading)

**PebbleOS supports multiple sensors**:
| Sensor | Type | Location |
|--------|------|----------|
| LSM6DSO | 6-axis IMU | `src/fw/drivers/imu/lsm6dso/` |
| LIS2DW12 | 3-axis Accelerometer | `src/fw/drivers/imu/lis2dw12/` |
| BMI160 | 6-axis IMU | `src/fw/drivers/imu/bmi160/` |
| BMA255 | 3-axis Accelerometer | `src/fw/drivers/imu/bma255/` |
| LIS3DH | 3-axis Accelerometer | `src/fw/drivers/imu/lis3dh/` |

### 1.3 Key Sensor Parameters

**Sampling Rate**: How often the sensor takes measurements
- PebbleOS activity tracking: **25 Hz** (25 samples/second)
- Higher rates (100+ Hz) used for tap detection, games

**Full-Scale Range**: Maximum measurable acceleration
- ±2g: High sensitivity, low max acceleration
- ±4g: Balanced (typical for wearables)
- ±16g: Low sensitivity, high max acceleration

**Resolution**: Smallest detectable change
- Typically 12-16 bits per axis
- LSM6DSO: 16-bit resolution

### 1.4 The Walking Signal

When a person walks, the wrist-worn watch experiences characteristic accelerations:

```
Acceleration │    ╭──╮      ╭──╮      ╭──╮
    (Z-axis) │   ╱    ╲    ╱    ╲    ╱    ╲
             │──╱      ╲──╱      ╲──╱      ╲──
             │
             └──────────────────────────────────► Time
               │←─Step─→│←─Step─→│←─Step─→│
```

**Key characteristics**:
- **Periodicity**: Steps are roughly regular (0.5-2 seconds apart)
- **Frequency**: Walking = 1-2 Hz (60-120 steps/min), Running = 2.5-4 Hz
- **Arm swing**: Creates additional signal at ~half the step frequency
- **Multi-axis**: Motion appears on all three axes

---

## 2. Fundamentals of Step Detection

### 2.1 The Challenge

Step detection must distinguish walking/running from:
- Random arm movements
- Driving in a car (vibrations)
- Washing dishes (repetitive motion)
- Watch sitting on a table
- Fidgeting

### 2.2 Common Approaches

#### Approach 1: Peak Detection (Time Domain)

The simplest approach detects peaks in the acceleration signal:

```
Algorithm:
1. Combine 3 axes into magnitude: |a| = √(x² + y² + z²)
2. Apply low-pass filter to remove noise
3. Find peaks above threshold
4. Count peaks that are:
   - Above minimum threshold
   - Separated by minimum time interval
   - Below maximum time interval
```

**Pros**: Simple, low power
**Cons**: Poor at rejecting non-walking activities

#### Approach 2: Frequency Analysis (Frequency Domain)

Analyzes the **frequency content** of the signal using FFT:

```
Algorithm:
1. Collect N samples (e.g., 5 seconds worth)
2. Apply windowing function (reduces edge effects)
3. Compute FFT to get frequency spectrum
4. Find dominant frequency in walking range (1-4 Hz)
5. If energy concentrated at walking frequency → count steps
```

**Pros**: Better at distinguishing walking from noise
**Cons**: Higher computation, requires more samples

**This is the approach used by PebbleOS** (Kraepelin Algorithm)

#### Approach 3: Machine Learning

Modern approaches train neural networks on labeled data:
- Convolutional Neural Networks (CNN) on raw signals
- Recurrent Neural Networks (RNN) for temporal patterns
- Decision trees/Random forests on features

**Pros**: Highest accuracy
**Cons**: Requires training data, high computation

### 2.3 Vector Magnitude Counts (VMC)

VMC is a standardized measure of activity intensity, designed to match research-grade devices (ActiGraph):

```
VMC Calculation:
1. For each axis, apply bandpass filter (0.25-1.75 Hz)
2. Sum absolute values of filtered samples
3. Combine axes: VMC = √(sum_x² + sum_y² + sum_z²)
```

VMC correlates with:
- Energy expenditure
- Activity intensity
- Sleep/wake state

---

## 3. The Kraepelin Algorithm (PebbleOS Implementation)

PebbleOS uses the **Kraepelin Algorithm**, an FFT-based step counting algorithm developed by Nathaniel T. Stockham.

**Source file**: `src/fw/services/normal/activity/kraepelin/kraepelin_algorithm.c`

### 3.1 Algorithm Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    KRAEPELIN ALGORITHM                          │
└─────────────────────────────────────────────────────────────────┘

Raw Accelerometer Data (25 Hz, 3 axes)
            │
            ▼
┌─────────────────────────┐
│ Buffer 125 samples      │  (5 seconds = 1 epoch)
│ (5 sec × 25 Hz)         │
└───────────┬─────────────┘
            │
            ├─────────────────────────────┐
            │                             │
            ▼                             ▼
┌─────────────────────────┐   ┌─────────────────────────┐
│ Butterworth Filter      │   │ Apply Cosine Window     │
│ (0.25-1.75 Hz bandpass) │   │ Remove DC offset        │
│ Per-axis PIM (VMC)      │   │ Zero-pad to 128 samples │
└───────────┬─────────────┘   └───────────┬─────────────┘
            │                             │
            ▼                             ▼
┌─────────────────────────┐   ┌─────────────────────────┐
│ Calculate VMC           │   │ 128-point FFT           │
│ (Activity Intensity)    │   │ (per axis)              │
└───────────┬─────────────┘   └───────────┬─────────────┘
            │                             │
            │                             ▼
            │                 ┌─────────────────────────┐
            │                 │ Compute Magnitude       │
            │                 │ |F| = √(Fx² + Fy² + Fz²)│
            │                 └───────────┬─────────────┘
            │                             │
            └──────────────┬──────────────┘
                           │
                           ▼
            ┌─────────────────────────────┐
            │ Find Peak in 7-20 Hz range  │
            │ (walking/running frequency) │
            └───────────────┬─────────────┘
                            │
                            ▼
            ┌─────────────────────────────┐
            │ Calculate Scores:           │
            │ - Signal energy ratio       │
            │ - High-freq ratio (driving) │
            │ - Low-freq ratio (dishes)   │
            └───────────────┬─────────────┘
                            │
                            ▼
            ┌─────────────────────────────┐
            │ Decision: Is this stepping? │
            │ (thresholds on scores + VMC)│
            └───────────────┬─────────────┘
                            │
               ┌────────────┴────────────┐
               │                         │
               ▼                         ▼
        ┌───────────┐             ┌───────────┐
        │ STEPPING  │             │ NOT STEP  │
        │steps = Hz │             │ steps = 0 │
        └───────────┘             └───────────┘
```

### 3.2 Step-by-Step Explanation

#### Step 1: Data Collection (5-second epochs)

```c
// From kraepelin_algorithm.c:57
#define KALG_N_SAMPLES_EPOCH  (5 * 25)  // 125 samples per epoch
#define KALG_SAMPLE_HZ 25               // 25 samples per second
```

The algorithm collects **125 samples** (5 seconds at 25 Hz) before processing. This window provides:
- Enough data for reliable frequency analysis
- Multiple step cycles for pattern recognition
- Balance between latency and accuracy

#### Step 2: Butterworth Bandpass Filter

A **2nd-order Butterworth filter** with passband 0.25-1.75 Hz isolates walking-related motion:

```c
// From kraepelin_algorithm.c:458-468
// Butterworth filter coefficients for 0.25-1.75 Hz bandpass
static const Fixed_S64_32 cb[5] = {
    0.027859766117136,   // b0
    0.0,                 // b1
    -0.055719532234272,  // b2
    0.0,                 // b3
    0.027859766117136    // b4
};
static const Fixed_S64_32 ca[4] = {
    -3.426993307709624,  // a1
    4.453028117259779,   // a2
    -2.612358264068663,  // a3
    0.586919508061190    // a4
};
```

**Why 0.25-1.75 Hz?**
- 0.25 Hz (15 steps/min): Very slow walking
- 1.75 Hz (105 steps/min): Fast walking
- Higher frequencies (running) analyzed separately

The filter removes:
- DC offset (gravity)
- High-frequency noise
- Very low frequency drift

#### Step 3: Calculate VMC (Vector Magnitude Counts)

```c
// From kraepelin_algorithm.c:791-808
static uint32_t prv_calc_raw_vmc(uint32_t *pims) {
    uint32_t d[3];

    // Cap values to prevent overflow
    const uint32_t max_value = 37500;
    for (int axis = 0; axis < 3; axis++) {
        d[axis] = MIN(pims[axis] / 10, max_value);
    }

    // VMC = √(x² + y² + z²)
    return 10 * sqrt(d[0]*d[0] + d[1]*d[1] + d[2]*d[2]);
}
```

VMC calculation:
1. Apply bandpass filter to each axis
2. Sum absolute values: `PIM[axis] = Σ|filtered_sample|`
3. Combine: `VMC = √(PIM_x² + PIM_y² + PIM_z²)`

#### Step 4: FFT Analysis

The algorithm performs a **128-point real FFT** on each axis:

```c
// From kraepelin_algorithm.c:74-77
#define KALG_FFT_WIDTH  128
static const int16_t KALG_FFT_WIDTH_PWR_TWO = 7;  // 2^7 = 128
```

**Pre-processing before FFT**:

1. **Cosine window**: Tapers signal to zero at edges (reduces spectral leakage)
```c
// From kraepelin_algorithm.c:727-734
static void prv_filt_cosine_win_mean0(int16_t *d, int16_t width, int32_t g_factor) {
    int32_t d_mean = prv_mean(d, width, 1);
    for (uint16_t i = 0; i < width; i++) {
        d[i] = ((d[i] - d_mean) * g_factor *
                sin_lookup((TRIG_MAX_ANGLE * i) / (2 * width))) / TRIG_MAX_RATIO;
    }
}
```

2. **Zero-padding**: Extend 125 samples to 128 (power of 2 for efficient FFT)

3. **FFT computation**: Uses Sorensen's real-valued radix-2 algorithm
```c
// From kraepelin_algorithm.c:553-616
static void prv_fft_2radix_real(int16_t *d, int16_t width, int16_t width_log_2);
```

4. **Magnitude calculation**: Combine real and imaginary parts
```c
// From kraepelin_algorithm.c:622-630
static void prv_fft_mag(int16_t *d, int16_t width) {
    for (int16_t i = 1; i < (width / 2); i++) {
        d[i] = sqrt(d[i] * d[i] + d[width - i] * d[width - i]);
    }
}
```

5. **3-axis combination**:
```c
// From kraepelin_algorithm.c:1238-1242
for (int i = 0; i < FFT_WIDTH / 2; i++) {
    magnitude[i] = sqrt(X[i]² + Y[i]² + Z[i]²);
}
```

#### Step 5: Frequency Analysis

**Find the dominant walking frequency** (7-20 Hz range in FFT bins, corresponding to ~1.4-4 Hz actual frequency):

```c
// From kraepelin_algorithm.c:92-94
static const int KALG_MIN_STEP_FREQ = 7;   // FFT bin 7 ≈ 1.4 Hz ≈ 84 steps/min
static const int KALG_MAX_STEP_FREQ = 20;  // FFT bin 20 ≈ 4 Hz ≈ 240 steps/min
```

The algorithm also accounts for **harmonics**:
- Walking frequency (fundamental)
- Arm swing at ~half the walking frequency
- 2nd, 3rd, 4th, 5th harmonics

```c
// From kraepelin_algorithm.c:819-883
static uint32_t prv_compute_signal_energy(int16_t *d, int16_t d_len,
                                          uint16_t walk_hz, bool log) {
    // Walking frequency energy
    uint32_t walk_energy = d[walk_hz];

    // Arm swing at half frequency
    uint32_t arm_energy = d[walk_hz / 2];

    // Harmonics
    uint32_t walk_2_energy = d[walk_hz * 2];     // 2nd harmonic
    uint32_t walk_3_energy = d[walk_hz * 3];     // 3rd harmonic
    // ... up to 5th harmonic

    return walk_energy + arm_energy + walk_2_energy + ...;
}
```

#### Step 6: Score Calculation

Three scores determine if the signal represents stepping:

**1. Signal Score (score_0)**: Ratio of walking energy to total energy
```c
// From kraepelin_algorithm.c:977-981
score_0 = (signal_energy * 100) / total_energy;
```
- Higher score = more concentrated energy at walking frequency
- Threshold: ≥15 for stepping

**2. High-Frequency Score (score_hf)**: Ratio of high-frequency energy
```c
// From kraepelin_algorithm.c:985-988
score_hf = 100 * integral(d, 50, max) / signal_energy;
```
- High value indicates driving/engine vibration
- Threshold: Must be ≤120

**3. Low-Frequency Score (score_lf)**: Ratio of low-frequency energy
```c
// From kraepelin_algorithm.c:992-995
score_lf = 100 * integral(d, 0, 4) / signal_energy;
```
- High value indicates non-walking motion (washing dishes)
- Threshold: Must be ≤145

#### Step 7: Step Decision

```c
// From kraepelin_algorithm.c:1012-1081
static bool prv_is_stepping(KAlgState *state, uint16_t max_mag_hz,
                            uint16_t score_0, uint16_t score_hf,
                            uint16_t score_lf, uint32_t real_vmc_5s,
                            int32_t total_energy, bool *partial_steps) {

    // Thresholds
    const uint16_t k_min_score = 15;
    const uint16_t k_min_vmc = 135;

    bool is_stepping = false;

    // Check frequency range
    if ((max_mag_hz >= 7) && (max_mag_hz <= 20)) {
        // Check score and VMC thresholds
        if ((score_0 >= k_min_score) && (real_vmc_5s >= k_min_vmc)) {
            is_stepping = true;
        }
    }

    // Reject if too much high-frequency content (driving)
    if (score_hf > 120) is_stepping = false;

    // Reject if too much low-frequency content (non-walking)
    if (score_lf > 145) is_stepping = false;

    // Reject high step rate with low VMC
    if (max_mag_hz >= 12 && real_vmc_5s < 1000) is_stepping = false;

    return is_stepping;
}
```

#### Step 8: Step Count Output

If stepping is detected, **the dominant frequency becomes the step count**:

```c
// From kraepelin_algorithm.c:1113
uint16_t step_count = stepping ? max_mag_hz : 0;
```

Since the FFT analyzes 5 seconds of data:
- FFT bin 7 = 7 cycles in 5 seconds = 1.4 Hz = 84 steps/minute → 7 steps in 5 seconds
- FFT bin 15 = 15 cycles in 5 seconds = 3 Hz = 180 steps/minute → 15 steps in 5 seconds

**Partial epoch handling**: When starting/stopping a walk, add half the steps:
```c
// From kraepelin_algorithm.c:1115-1121
if (state->prev_partial_steps && (step_count > 0)) {
    // Transition from non-walking to walking
    return_steps += step_count / 2;
} else if ((state->prev_5s_steps > 0) && partial_steps) {
    // Transition from walking to non-walking
    return_steps += state->prev_5s_steps / 2;
}
```

### 3.3 Mathematical Summary

The complete step counting formula:

```
Given: N = 125 samples, fs = 25 Hz, T = 5 seconds

1. Filter: y[n] = Butterworth_bandpass(x[n], 0.25Hz, 1.75Hz)

2. VMC: VMC = √(Σ|y_x|² + Σ|y_y|² + Σ|y_z|²)

3. FFT: F[k] = Σ(n=0 to N-1) x[n] × e^(-j2πkn/N)

4. Magnitude: |F[k]| = √(Re[k]² + Im[k]²)

5. 3-axis combine: M[k] = √(|F_x[k]|² + |F_y[k]|² + |F_z[k]|²)

6. Find peak: k_peak = argmax(M[k]) for k ∈ [7, 20]

7. Signal energy: E_signal = M[k_peak] + harmonics

8. Score: score = E_signal / Σ M[k] × 100

9. Decision: stepping = (score ≥ 15) AND (VMC ≥ 135) AND
                        (score_hf ≤ 120) AND (score_lf ≤ 145)

10. Steps: count = k_peak if stepping else 0
```

---

## 4. Sleep Detection Algorithm

Sleep detection uses minute-level VMC data rather than raw accelerometer samples.

**Source**: `src/fw/services/normal/activity/kraepelin/kraepelin_algorithm.c` (lines 1252-1399+)

### 4.1 Algorithm Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                  SLEEP DETECTION ALGORITHM                       │
└─────────────────────────────────────────────────────────────────┘

Per-Minute Data: VMC, Orientation, Plugged-in status
                        │
                        ▼
            ┌─────────────────────────┐
            │ Store in 9-minute       │
            │ sliding window buffer   │
            └───────────┬─────────────┘
                        │
                        ▼
            ┌─────────────────────────┐
            │ Compute Sleep Score     │
            │ (weighted convolution)  │
            └───────────┬─────────────┘
                        │
            Score = Σ(weights[i] × VMC[i])
                        │
                        ▼
            ┌─────────────────────────┐
            │ Classify minute:        │
            │ Sleep: score ≤ 330      │
            │ Wake:  score > 330      │
            └───────────┬─────────────┘
                        │
      ┌─────────────────┼─────────────────┐
      │                 │                 │
      ▼                 ▼                 ▼
 ┌─────────┐      ┌─────────┐      ┌─────────┐
 │ Track   │      │ Track   │      │ Validate│
 │ Sleep   │      │ Wake    │      │ Session │
 │ Minutes │      │ Minutes │      │         │
 └─────────┘      └─────────┘      └─────────┘
      │                 │                 │
      │  5+ sleep       │  11-14 wake     │  ≥60 min
      │  minutes        │  minutes        │  duration
      ▼                 ▼                 ▼
 [Sleep Entry]    [Sleep Exit]     [Valid Sleep]
```

### 4.2 Sleep Score Calculation

The sleep score uses a **weighted convolution** over 9 minutes:

```c
// From kraepelin_algorithm.c:1257-1268
static uint32_t prv_compute_sleep_score(KAlgSleepMinute *samples, int i) {
    // Weights for 9-minute window (centered on current minute)
    const int weights[9] = {10, 15, 28, 31, 85, 15, 10, 0, 0};
    const int weight_divisor = 100;

    uint32_t score = 0;
    for (int j = 0; j < 9; j++) {
        score += weights[j] * samples[i - 4 + j].vmc;
    }
    return score / weight_divisor;
}
```

**Weight distribution**:
```
Time:     -4   -3   -2   -1    0   +1   +2   +3   +4  (minutes)
Weights:  10   15   28   31   85   15   10    0    0
                              ↑
                        Current minute
```

The center minute (weight 85) dominates, but surrounding minutes provide context.

### 4.3 Sleep Parameters

```c
// From kraepelin_algorithm.c:213-230
static const KAlgSleepParams KALG_SLEEP_PARAMS = {
    .max_sleep_minute_score = 330,      // Below this = "sleep minute"
    .force_wake_minute_score = 8000,    // Above this = definitely awake
    .force_wake_minute_vmc = 10000,     // VMC above this = definitely awake
    .min_sleep_minutes = 5,             // Need 5+ sleep minutes to start
    .max_wake_minutes_early = 14,       // 14 wake minutes = exit (early in sleep)
    .max_wake_minutes_late = 11,        // 11 wake minutes = exit (later in sleep)
    .min_sleep_cycle_len_minutes = 60,  // Minimum valid sleep = 60 minutes
    .max_active_minutes_pct = 89,       // Max 89% active minutes in sleep
    .max_avg_vmc = 180,                 // Average VMC must be below 180
};
```

### 4.4 Sleep State Machine

```
                    ┌──────────────┐
                    │    AWAKE     │
                    └──────┬───────┘
                           │
                    5+ consecutive
                    sleep minutes
                           │
                           ▼
                    ┌──────────────┐
                    │   SLEEPING   │
                    └──────┬───────┘
                           │
               ┌───────────┴───────────┐
               │                       │
        11-14 consecutive        Session validation:
        wake minutes             - ≥60 minutes total
               │                 - <89% active minutes
               │                 - avg VMC < 180
               ▼                       │
        ┌──────────────┐               │
        │    AWAKE     │ ◄─────────────┘
        └──────────────┘     Invalid: discard
                             Valid: record session
```

### 4.5 Deep Sleep Detection

Deep sleep (restful sleep) is detected within a sleep session:

```c
// From kraepelin_algorithm.c:273-277
static const KAlgDeepSleepParams KALG_DEEP_SLEEP_PARAMS = {
    .max_deep_score = 160,          // Very low activity
    .min_deep_score_count = 20,     // 20+ consecutive minutes
    .min_minutes_after_sleep_entry = 10,  // Wait 10 min after sleep starts
};
```

Deep sleep requires:
- Sleep score ≤ 160 (vs ≤ 330 for light sleep)
- 20+ consecutive minutes of low scores
- At least 10 minutes after sleep entry

### 4.6 Not-Worn Detection

Distinguishes sleep from watch sitting on a table:

```c
// From kraepelin_algorithm.c:316-320
static const KAlgNotWornParams KALG_NOT_WORN_PARAMS = {
    .max_non_worn_vmc = 2500,     // Below this + flat = maybe not worn
    .min_worn_vmc = 4,            // Below this = definitely not worn
    .max_low_vmc_run_m = 180,     // 3 hours of low VMC = not worn
};
```

Key indicators of not-worn:
- Consistent orientation (watch flat)
- Very low VMC (no micro-movements)
- Extended duration of stillness

---

## 5. Distance and Calorie Calculations

**Source**: `src/fw/services/normal/activity/activity_calculators.c`

### 5.1 Distance Calculation

Distance is calculated from steps using stride length estimation:

```c
// From activity_calculators.c:47-89
uint32_t activity_private_compute_distance_mm(uint32_t steps, uint32_t ms) {
    // Stride length formula: stride = (a × cadence + b) × height
    // where cadence = steps/minute

    const uint64_t k_a_x10000 = 31;      // 0.003129
    const uint64_t k_b_x10000 = 1449;    // 0.14485

    // stride_len_component = a × (steps/ms × 60000) + b
    uint64_t stride_len_component =
        (k_a_x10000 * steps * 60000 / ms) + k_b_x10000;

    // distance = stride_len × height × steps
    uint32_t distance_mm = stride_len_component * height_mm * steps / 10000;

    return distance_mm;
}
```

**Formula**:
```
stride_length = (0.003129 × steps_per_min + 0.14485) × height_mm
distance = stride_length × steps
```

**Example**:
- Height: 1750 mm (5'9")
- Cadence: 100 steps/min
- Stride = (0.003129 × 100 + 0.14485) × 1750 = 724 mm
- For 1000 steps: distance = 724 × 1000 = 724,000 mm = 724 m

### 5.2 Active Calorie Calculation

Uses the validated caloric expenditure formula:

```c
// From activity_calculators.c:124-152
uint32_t activity_private_compute_active_calories(uint32_t distance_mm, uint32_t ms) {
    // Walking threshold: 120 m/min = 2 mm/ms
    const uint32_t k_max_walking_rate = 120000;  // mm/min

    uint64_t rate_mm_per_min = distance_mm * 60000 / ms;

    bool walking = (rate_mm_per_min <= k_max_walking_rate);

    // Walking:  calories = 0.501 × distance_m × weight_kg
    // Running:  calories = 1.002 × distance_m × weight_kg
    uint64_t k_constant_x1000 = walking ? 501 : 1002;

    uint32_t calories = k_constant_x1000 * distance_mm * weight_dag /
                        (1000 * 1000 * 100);  // Convert units

    return calories;
}
```

**Formula**:
```
Walking (< 4.5 mph):  active_cal = 0.501 × distance_m × weight_kg
Running (≥ 4.5 mph):  active_cal = 1.002 × distance_m × weight_kg
```

### 5.3 Resting Calorie Calculation

Uses the Mifflin-St Jeor equation for Basal Metabolic Rate:

```c
// From activity_calculators.c:156-183
uint32_t activity_private_compute_resting_calories(uint32_t elapsed_minutes) {
    // Mifflin-St Jeor formula (kcal/day):
    // Men:   10 × weight_kg + 6.25 × height_cm - 5 × age + 5
    // Women: 10 × weight_kg + 6.25 × height_cm - 5 × age - 161

    uint32_t calories_per_day =
        (100 * weight_dag) +      // 10 × weight_kg
        (625 * height_mm) -       // 6.25 × height_cm (using mm)
        (5000 * age_years);       // 5 × age

    if (gender == Male) {
        calories_per_day += 5000;     // +5
    } else if (gender == Female) {
        calories_per_day -= 161000;   // -161
    }

    // Scale to requested minutes
    return calories_per_day * elapsed_minutes / (24 * 60);
}
```

---

## 6. PebbleOS Code References

### 6.1 Complete File Reference

| Component | File Path | Description |
|-----------|-----------|-------------|
| **Step Algorithm** | `src/fw/services/normal/activity/kraepelin/kraepelin_algorithm.c` | FFT-based step counting |
| **Algorithm Header** | `src/fw/services/normal/activity/kraepelin/kraepelin_algorithm.h` | Algorithm interface |
| **Activity Service** | `src/fw/services/normal/activity/activity.c` | Main activity service |
| **Activity Private** | `src/fw/services/normal/activity/activity_private.h` | Internal structures |
| **Calculators** | `src/fw/services/normal/activity/activity_calculators.c` | Distance/calorie math |
| **Sessions** | `src/fw/services/normal/activity/activity_sessions.c` | Walk/run/sleep tracking |
| **Health API** | `src/fw/applib/health_service.h` | Public health API |
| **Accel Manager** | `src/fw/services/common/accel_manager.h` | Accelerometer service |
| **LSM6DSO Driver** | `src/fw/drivers/imu/lsm6dso/lsm6dso.c` | 6-axis IMU driver |
| **LIS2DW12 Driver** | `src/fw/drivers/imu/lis2dw12/lis2dw12.c` | Accelerometer driver |
| **Accel Interface** | `src/fw/drivers/accel.h` | Driver abstraction |

### 6.2 Key Functions

**Step Counting**:
- `kalg_analyze_samples()` - Main entry point for step analysis
- `prv_analyze_epoch()` - Process one 5-second epoch
- `prv_fft_2radix_real()` - FFT computation
- `prv_compute_scores()` - Calculate stepping scores
- `prv_is_stepping()` - Decision function

**Sleep Detection**:
- `kalg_activities_update()` - Process minute data
- `prv_compute_sleep_score()` - Calculate sleep score
- `prv_not_worn_update()` - Not-worn detection

**VMC Calculation**:
- `prv_pim_filter()` - Butterworth filter for VMC
- `prv_calc_raw_vmc()` - Combine axes into VMC
- `prv_real_counts_from_raw()` - Convert to ActiGraph units

### 6.3 Data Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     COMPLETE DATA FLOW                          │
└─────────────────────────────────────────────────────────────────┘

IMU Hardware (LSM6DSO/LIS2DW12)
        │
        │ Raw samples via SPI/I2C
        ▼
┌───────────────────┐
│ IMU Driver        │ src/fw/drivers/imu/*/
│ (lsm6dso.c, etc.) │
└─────────┬─────────┘
          │ AccelRawData samples
          ▼
┌───────────────────┐
│ Accel Manager     │ src/fw/services/common/accel_manager.c
│                   │
└─────────┬─────────┘
          │ Batched samples (25 Hz)
          ▼
┌───────────────────┐
│ Kraepelin Alg     │ src/fw/services/normal/activity/kraepelin/
│                   │
│ ├─ Step counting  │ kalg_analyze_samples()
│ └─ VMC calc       │ kalg_minute_stats()
└─────────┬─────────┘
          │ Steps per epoch, VMC per minute
          ▼
┌───────────────────┐
│ Activity Service  │ src/fw/services/normal/activity/activity.c
│                   │
│ ├─ Sleep detect   │ kalg_activities_update()
│ ├─ Walk/run       │ activity_sessions.c
│ └─ Metrics        │ activity_metrics.c
└─────────┬─────────┘
          │ Aggregated metrics
          ▼
┌───────────────────┐
│ Health Service    │ src/fw/applib/health_service.c
│ (Public API)      │
└─────────┬─────────┘
          │ health_service_sum()
          │ health_service_peek_current_value()
          ▼
┌───────────────────┐
│ User Application  │
└───────────────────┘
```

---

## Summary

The PebbleOS activity tracking system demonstrates a sophisticated approach to step counting and activity detection:

1. **FFT-based Analysis**: More robust than simple peak detection
2. **Multi-score Validation**: Rejects non-walking activities (driving, dishes)
3. **Harmonic Analysis**: Accounts for arm swing and step harmonics
4. **VMC Integration**: Provides activity intensity independent of step count
5. **Minute-level Sleep**: Uses weighted convolution for sleep/wake classification
6. **Validated Formulas**: Distance and calories use peer-reviewed methods

The algorithms balance accuracy with the computational constraints of embedded systems, achieving reliable activity tracking with minimal power consumption.
