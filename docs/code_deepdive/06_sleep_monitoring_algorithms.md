# Sleep Monitoring Algorithms Guide

This guide provides a comprehensive explanation of sleep detection, sleep quality assessment, and sleep stage classification algorithms used in wearable devices, with detailed references to PebbleOS's implementation.

## Table of Contents

1. [Introduction to Sleep Monitoring](#1-introduction-to-sleep-monitoring)
2. [Sleep Detection Fundamentals](#2-sleep-detection-fundamentals)
3. [The Sleep Detection Algorithm](#3-the-sleep-detection-algorithm)
4. [Deep Sleep (Restful Sleep) Detection](#4-deep-sleep-restful-sleep-detection)
5. [Sleep Session Validation](#5-sleep-session-validation)
6. [Nap Detection](#6-nap-detection)
7. [Not-Worn Detection](#7-not-worn-detection)
8. [Sleep Quality Metrics](#8-sleep-quality-metrics)
9. [PebbleOS Code References](#9-pebbleos-code-references)

---

## 1. Introduction to Sleep Monitoring

### 1.1 Why Sleep Monitoring Matters

Sleep monitoring in wearables aims to:
- Track total sleep duration
- Identify sleep quality (light vs. deep sleep)
- Detect sleep patterns and consistency
- Provide insights for better sleep hygiene

### 1.2 Clinical Sleep Stages

Medical sleep studies (polysomnography) identify these stages:

| Stage | Name | Characteristics | % of Night |
|-------|------|-----------------|------------|
| W | Wake | Active movement, eyes open | Variable |
| N1 | Light Sleep | Drowsy, easily awakened | 5% |
| N2 | Light Sleep | Body temperature drops, heart rate slows | 45% |
| N3 | Deep Sleep | Slow brain waves, difficult to wake, restorative | 25% |
| REM | REM Sleep | Rapid eye movement, dreaming, muscle paralysis | 25% |

### 1.3 Wearable Sleep Detection Approach

Wearables cannot detect all clinical stages but can distinguish:

```
┌─────────────────────────────────────────────────────────────────┐
│            WEARABLE SLEEP CLASSIFICATION                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  AWAKE ──────────► Significant movement, high VMC               │
│    │                                                            │
│    ▼                                                            │
│  LIGHT SLEEP ────► Low movement, VMC below threshold            │
│    │                                                            │
│    ▼                                                            │
│  DEEP SLEEP ─────► Very low movement, sustained low VMC         │
│    (Restful)                                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

PebbleOS uses **motion-based detection** via accelerometer data:
- **Awake**: Active movement patterns
- **Light Sleep**: Reduced movement, occasional shifts
- **Deep/Restful Sleep**: Minimal movement for extended periods

---

## 2. Sleep Detection Fundamentals

### 2.1 The Key Metric: VMC (Vector Magnitude Counts)

VMC measures overall activity intensity per minute:

```
VMC = √(Σ|filtered_x|² + Σ|filtered_y|² + Σ|filtered_z|²)
```

Where filtered samples use a 0.25-1.75 Hz bandpass Butterworth filter.

**VMC Interpretation**:
| VMC Range | Activity Level | Sleep State |
|-----------|----------------|-------------|
| 0-20 | No movement | Likely not worn |
| 20-180 | Minimal movement | Deep sleep candidate |
| 180-330 | Low movement | Light sleep candidate |
| 330-1000 | Moderate movement | Likely awake |
| >1000 | Active movement | Definitely awake |

### 2.2 Orientation Tracking

Watch orientation helps distinguish sleep from stationary placement:

```c
// From kraepelin_algorithm.c:1151-1172
// Orientation encoded as: 16 * phi_index + theta_index
// phi: angle to Z-axis (0-15)
// theta: angle in X-Y plane (0-15)

uint8_t orientation = KALG_NUM_ANGLES * phi_i + theta;
```

**Orientation uses**:
- Consistent orientation = watch stationary (maybe not worn)
- Varying orientation = watch being worn
- Flat orientation (z-axis = 0x0 or 0x8) = possibly on table

### 2.3 Sleep Score Calculation

**Source**: `src/fw/services/normal/activity/kraepelin/kraepelin_algorithm.c:1256-1268`

The algorithm computes a **sleep score** using weighted convolution:

```c
static uint32_t prv_compute_sleep_score(KAlgSleepMinute *samples, int i) {
    // 9-minute weighted convolution filter
    const int weights[9] = {10, 15, 28, 31, 85, 15, 10, 0, 0};
    const int weight_divisor = 100;

    uint32_t score = 0;
    for (int j = 0; j < 9; j++) {
        uint32_t vmc = samples[i - 4 + j].vmc;
        score += weights[j] * vmc;
    }
    return score / weight_divisor;
}
```

**Weight Distribution**:
```
Minute:  -4   -3   -2   -1    0   +1   +2   +3   +4
Weight:  10   15   28   31   85   15   10    0    0
                              ↑
                       Current minute (85% weight)
```

The current minute dominates (85%), but context from surrounding minutes smooths the decision.

---

## 3. The Sleep Detection Algorithm

### 3.1 Algorithm Parameters

**Source**: `src/fw/services/normal/activity/kraepelin/kraepelin_algorithm.c:193-230`

```c
static const KAlgSleepParams KALG_SLEEP_PARAMS = {
    // Score thresholds
    .max_sleep_minute_score = 330,      // Below = "sleep minute"
    .force_wake_minute_score = 8000,    // Above = definitely awake
    .force_wake_minute_vmc = 10000,     // VMC above = definitely awake

    // Sleep entry/exit
    .min_sleep_minutes = 5,             // 5 consecutive sleep minutes to start
    .max_wake_minutes_early = 14,       // 14 awake minutes = exit (first hour)
    .max_wake_minutes_late = 11,        // 11 awake minutes = exit (after first hour)
    .max_wake_minute_early_offset = 60, // "Early" phase duration

    // Session validation
    .min_sleep_cycle_len_minutes = 60,  // Minimum 1 hour sleep
    .max_active_minutes_pct = 89,       // Max 89% active minutes
    .max_avg_vmc = 180,                 // Average VMC limit
    .vmc_clip = 1000,                   // Clip high VMC values
    .min_sleep_len_for_active_pct_check = 39,

    // Noise floor
    .min_valid_vmc = 20,                // Below = "zero" movement
};
```

**Platform-specific variations** (Asterix has stricter thresholds):
- `max_sleep_minute_score`: 500 (vs 330)
- `min_valid_vmc`: 30 (vs 20)
- `max_avg_vmc`: 250 (vs 180)

### 3.2 Sleep State Machine

**Source**: `src/fw/services/normal/activity/kraepelin/kraepelin_algorithm.c:1830-1954`

```
                ┌─────────────────┐
                │     AWAKE       │
                │ (Default State) │
                └────────┬────────┘
                         │
            5+ consecutive minutes with
            sleep_score ≤ max_sleep_minute_score
                         │
                         ▼
                ┌─────────────────┐
                │    SLEEPING     │
                │  (Entry State)  │
                └────────┬────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
   11-14 wake       VMC > 10000     score > 8000
   minutes in       (any minute)    (any minute)
   a row                 │                │
        │                │                │
        └────────────────┼────────────────┘
                         │
                         ▼
                ┌─────────────────┐
                │   VALIDATING    │
                │ (Check session) │
                └────────┬────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   Duration < 60    Active% > 89%    Avg VMC > 180
   minutes               │                │
        │                │                │
        ▼                ▼                ▼
   ┌─────────┐      ┌─────────┐      ┌─────────┐
   │ REJECTED│      │ REJECTED│      │ REJECTED│
   └─────────┘      └─────────┘      └─────────┘

        All checks pass
             │
             ▼
        ┌─────────┐
        │ VALID   │ ──► Record sleep session
        │ SLEEP   │
        └─────────┘
```

### 3.3 Sleep Entry Detection

```c
// From kraepelin_algorithm.c:1751-1766
// Increment consecutive sleep minute counter
if (sleep_score <= KALG_SLEEP_PARAMS.max_sleep_minute_score) {
    state->consecutive_sleep_minutes++;
    state->consecutive_awake_minutes = 0;
}

// Start sleep when threshold reached
if (state->consecutive_sleep_minutes >= KALG_SLEEP_PARAMS.min_sleep_minutes) {
    // Sleep session begins!
    state->start_time = utc_now - (state->consecutive_sleep_minutes * 60);
    prv_deep_sleep_update(alg_state, KAlgDeepSleepAction_Start, ...);
}
```

**Sleep starts when**:
- 5 or more consecutive minutes have sleep_score ≤ 330
- The start time is backdated to when the streak began

### 3.4 Sleep Exit Detection

**Source**: `src/fw/services/normal/activity/kraepelin/kraepelin_algorithm.c:1769-1817`

```c
// Force wake conditions
if (vmc > KALG_SLEEP_PARAMS.force_wake_minute_vmc ||
    sleep_score > KALG_SLEEP_PARAMS.force_wake_minute_score) {
    // Immediate wake!
    state->consecutive_awake_minutes = max_wake_minutes;
}

// Check consecutive awake minutes
uint16_t max_wake_minutes;
if (session_duration < KALG_SLEEP_PARAMS.max_wake_minute_early_offset) {
    max_wake_minutes = KALG_SLEEP_PARAMS.max_wake_minutes_early;  // 14 min
} else {
    max_wake_minutes = KALG_SLEEP_PARAMS.max_wake_minutes_late;   // 11 min
}

if (state->consecutive_awake_minutes >= max_wake_minutes) {
    // End sleep session
    end_time = utc_now - (state->consecutive_awake_minutes * 60);
    validate_and_record_session();
}
```

**Sleep ends when**:
1. **Consecutive awake minutes**: 14 (early) or 11 (late) in a row
2. **Force wake VMC**: Single minute with VMC > 10,000
3. **Force wake score**: Single minute with score > 8,000

The end time is backdated to when wakefulness began.

---

## 4. Deep Sleep (Restful Sleep) Detection

### 4.1 Deep Sleep Parameters

**Source**: `src/fw/services/normal/activity/kraepelin/kraepelin_algorithm.c:273-277`

```c
static const KAlgDeepSleepParams KALG_DEEP_SLEEP_PARAMS = {
    .max_deep_score = 160,              // Very low movement threshold
    .min_deep_score_count = 20,         // 20 consecutive minutes required
    .min_minutes_after_sleep_entry = 10 // Wait 10 min after sleep starts
};
```

### 4.2 Deep Sleep State Machine

**Source**: `src/fw/services/normal/activity/kraepelin/kraepelin_algorithm.c:1547-1653`

```
            Sleep Session Starts
                    │
                    ▼
            Wait 10 minutes
            (settling period)
                    │
                    ▼
        ┌───────────────────────┐
        │ Check each minute:    │
        │ score ≤ 160?          │
        └───────────┬───────────┘
                    │
        ┌───────────┼───────────┐
        │           │           │
        ▼           ▼           ▼
     YES          YES         NO
   (continue)   (20+ min)   (reset)
        │           │           │
        │           ▼           │
        │    ┌───────────┐      │
        │    │ DEEP SLEEP│      │
        │    │ DETECTED  │      │
        │    └─────┬─────┘      │
        │          │            │
        │    Add to pending     │
        │    sessions list      │
        │          │            │
        └──────────┴────────────┘
                   │
                   ▼
            Sleep Session Ends
                   │
                   ▼
        Validate sleep session
                   │
         ┌─────────┴─────────┐
         │                   │
      VALID               INVALID
         │                   │
         ▼                   ▼
   Register all          Discard all
   deep sleep           deep sleep
   sessions             sessions
```

### 4.3 Deep Sleep Tracking Structure

```c
// From kraepelin_algorithm.c:237-250
#define KALG_MAX_DEEP_SLEEP_SESSIONS 8

typedef struct {
    time_t sleep_start_time;              // Start of parent sleep session
    time_t deep_start_time;               // Start of current deep period
    uint16_t deep_score_count;            // Consecutive deep minutes
    uint16_t non_deep_score_count;        // Consecutive non-deep minutes
    bool ok_to_register;                  // Validation flag

    // Pending deep sleep sessions (up to 8 per night)
    uint8_t num_sessions;
    uint16_t start_delta_sec[8];          // Offset from sleep_start_time
    uint16_t len_m[8];                    // Duration in minutes
} KAlgDeepSleepActivityState;
```

### 4.4 Multiple Deep Sleep Periods

A single sleep session can contain multiple deep sleep periods:

```
Sleep Session: 11:00 PM ─────────────────────────────── 7:00 AM
                    │                                      │
                    ├──┬─────────┬─────────┬─────────┬─────┤
                       │         │         │         │
                    Deep 1    Deep 2    Deep 3    Deep 4
                   (30 min)  (45 min)  (25 min)  (40 min)

                    Each requires 20+ consecutive minutes
                    with score ≤ 160
```

---

## 5. Sleep Session Validation

### 5.1 Validation Criteria

**Source**: `src/fw/services/normal/activity/kraepelin/kraepelin_algorithm.c:1879-1890`

A sleep session is **rejected** if:

| Check | Threshold | Reason |
|-------|-----------|--------|
| Duration | < 60 minutes | Too short to be real sleep |
| Active % | > 89% | Too much movement during session |
| Avg VMC | > 180 (or 250) | Average movement too high |
| Not-worn | Detected | Watch was removed |

### 5.2 Active Minutes Calculation

```c
// Count minutes that appear to be "active" (non-sleep)
uint16_t active_minutes = 0;
for (each minute in session) {
    if (vmc > min_valid_vmc) {  // vmc > 20
        active_minutes++;
    }
}

uint16_t active_pct = (active_minutes * 100) / session_length;

if (active_pct > KALG_SLEEP_PARAMS.max_active_minutes_pct) {  // > 89%
    reject_session();
}
```

### 5.3 Average VMC Calculation

```c
// VMC is clipped to prevent post-wake activity from skewing average
uint32_t vmc_sum = 0;
for (each minute in session) {
    uint16_t clipped_vmc = MIN(vmc, KALG_SLEEP_PARAMS.vmc_clip);  // clip to 1000
    vmc_sum += clipped_vmc;
}

uint16_t avg_vmc = vmc_sum / num_non_zero_minutes;

if (avg_vmc > KALG_SLEEP_PARAMS.max_avg_vmc) {  // > 180
    reject_session();
}
```

---

## 6. Nap Detection

### 6.1 Nap Classification Rules

**Source**: `src/fw/services/normal/activity/kraepelin/activity_algorithm_kraepelin.h:26-34`

```c
#define ALG_PRIMARY_MORNING_MINUTE  (12 * MINUTES_PER_HOUR)   // 12:00 PM
#define ALG_PRIMARY_EVENING_MINUTE  (21 * MINUTES_PER_HOUR)   // 9:00 PM
#define ALG_MAX_NAP_MINUTES         (3 * MINUTES_PER_HOUR)    // 180 minutes
```

**A sleep session is classified as NAP if**:
1. Duration ≤ 180 minutes (3 hours)
2. Start time between 12:00 PM and 9:00 PM
3. End time between 12:00 PM and 9:00 PM

### 6.2 Nap Classification Logic

**Source**: `src/fw/services/normal/activity/kraepelin/activity_algorithm_kraepelin.c:732-811`

```c
bool prv_session_is_nap(ActivitySession *session) {
    // Get local time components
    int start_minute = minutes_since_midnight(session->start_utc);
    int end_minute = minutes_since_midnight(session->end_utc);

    // Check duration
    if (session->length_min > ALG_MAX_NAP_MINUTES) {
        return false;  // Too long for a nap
    }

    // Check time window
    bool start_in_range = (start_minute >= ALG_PRIMARY_MORNING_MINUTE &&
                          start_minute <= ALG_PRIMARY_EVENING_MINUTE);
    bool end_in_range = (end_minute >= ALG_PRIMARY_MORNING_MINUTE &&
                        end_minute <= ALG_PRIMARY_EVENING_MINUTE);

    return start_in_range && end_in_range;
}
```

### 6.3 Session Type Hierarchy

```
Primary Sleep (Night)          Nap (Daytime)
─────────────────────          ─────────────
ActivitySessionType_Sleep      ActivitySessionType_Nap
        │                              │
        ▼                              ▼
ActivitySessionType_RestfulSleep    ActivitySessionType_RestfulNap
```

Restful periods within naps are relabeled as RestfulNap.

---

## 7. Not-Worn Detection

### 7.1 Not-Worn Parameters

**Source**: `src/fw/services/normal/activity/kraepelin/kraepelin_algorithm.c:309-321`

```c
static const KAlgNotWornParams KALG_NOT_WORN_PARAMS = {
    .max_non_worn_vmc = 2500,     // Below this + flat = maybe not worn
    .min_worn_vmc = 4,            // Below this = definitely not worn
    .max_low_vmc_run_m = 180,     // 3 hours of low VMC = not worn
};
```

### 7.2 Not-Worn Detection Logic

**Source**: `src/fw/services/normal/activity/kraepelin/kraepelin_algorithm.c:1379-1438`

```c
static bool prv_not_worn_update(KAlgState *alg_state, time_t utc_now,
                                uint16_t vmc, uint8_t orientation,
                                bool plugged_in) {

    // Check if "maybe not worn"
    bool maybe_not_worn = false;

    // Same orientation as last minute?
    if (orientation == state->prev_orientation) {
        maybe_not_worn = true;
    }

    // Very low VMC for consecutive minutes?
    if (vmc < params->min_worn_vmc &&
        state->prev_vmc < params->min_worn_vmc) {
        maybe_not_worn = true;
    }

    // Watch is flat on table?
    const uint8_t z_axis = orientation >> 4;
    if (z_axis == 0x0 || z_axis == 0x8) {
        maybe_not_worn = true;
    }

    // High VMC means definitely worn
    if (vmc > params->max_non_worn_vmc) {
        maybe_not_worn = false;
    }

    // Track consecutive "maybe not worn" minutes
    if (maybe_not_worn) {
        state->maybe_not_worn_count++;
    } else {
        state->maybe_not_worn_count = 0;
    }

    // Definitely not worn if low VMC for too long
    if (state->maybe_not_worn_count > params->max_low_vmc_run_m) {
        return true;  // Not worn
    }

    return false;
}
```

### 7.3 Not-Worn Indicators

| Indicator | Condition | Meaning |
|-----------|-----------|---------|
| Consistent orientation | Same for multiple minutes | Watch stationary |
| Very low VMC | < 4 (or 15 for Asterix) | No micro-movements |
| Flat position | Z-axis = 0x0 or 0x8 | Watch on table |
| Extended stillness | > 30-180 minutes | Definitely not worn |
| Charging | Plugged in | Likely not on wrist |

---

## 8. Sleep Quality Metrics

### 8.1 Sleep Data Structure

**Source**: `src/fw/services/normal/activity/activity.h:178-183`

```c
typedef enum {
    ActivitySleepStateAwake = 0,
    ActivitySleepStateRestfulSleep,     // Deep/restful sleep
    ActivitySleepStateLightSleep,       // Light sleep
    ActivitySleepStateUnknown,
} ActivitySleepState;
```

### 8.2 Available Sleep Metrics

**Source**: `src/fw/services/normal/activity/activity.h:114-122`

| Metric | Description |
|--------|-------------|
| `SleepTotalSeconds` | Total sleep duration |
| `SleepRestfulSeconds` | Deep/restful sleep duration |
| `SleepEnterAtSeconds` | Time fell asleep (seconds after midnight) |
| `SleepExitAtSeconds` | Time woke up (seconds after midnight) |
| `SleepState` | Current sleep state enum |
| `SleepStateSeconds` | Duration in current state |

### 8.3 Sleep Efficiency Calculation

While not explicitly calculated in PebbleOS, sleep efficiency can be derived:

```
Sleep Efficiency = (Total Sleep Time / Time in Bed) × 100%

Where:
- Total Sleep Time = SleepTotalSeconds
- Time in Bed = SleepExitAtSeconds - SleepEnterAtSeconds
```

### 8.4 Deep Sleep Percentage

```
Deep Sleep % = (SleepRestfulSeconds / SleepTotalSeconds) × 100%

Typical ranges:
- < 10%: Poor deep sleep
- 10-20%: Normal
- > 20%: Good deep sleep quality
```

---

## 9. PebbleOS Code References

### 9.1 Key Files

| Component | File Path |
|-----------|-----------|
| Sleep algorithm | `src/fw/services/normal/activity/kraepelin/kraepelin_algorithm.c` |
| Algorithm header | `src/fw/services/normal/activity/kraepelin/kraepelin_algorithm.h` |
| Algorithm wrapper | `src/fw/services/normal/activity/kraepelin/activity_algorithm_kraepelin.c` |
| Nap thresholds | `src/fw/services/normal/activity/kraepelin/activity_algorithm_kraepelin.h` |
| Activity service | `src/fw/services/normal/activity/activity.c` |
| Activity types | `src/fw/services/normal/activity/activity.h` |
| Private structures | `src/fw/services/normal/activity/activity_private.h` |
| Session management | `src/fw/services/normal/activity/activity_sessions.c` |
| Sleep insights | `src/fw/services/normal/activity/activity_insights.c` |

### 9.2 Key Functions

| Function | File:Line | Purpose |
|----------|-----------|---------|
| `prv_compute_sleep_score()` | kraepelin_algorithm.c:1256 | Calculate sleep score |
| `prv_sleep_activity_update()` | kraepelin_algorithm.c:1830 | Main sleep state machine |
| `prv_deep_sleep_update()` | kraepelin_algorithm.c:1547 | Deep sleep detection |
| `prv_not_worn_update()` | kraepelin_algorithm.c:1379 | Not-worn detection |
| `prv_session_is_nap()` | activity_algorithm_kraepelin.c:732 | Nap classification |

### 9.3 Complete Algorithm Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    SLEEP MONITORING PIPELINE                     │
└─────────────────────────────────────────────────────────────────┘

Accelerometer (25 Hz)
        │
        ▼
┌───────────────────┐
│ Kraepelin Step    │
│ Algorithm         │──► Steps, VMC per minute
│ (5-sec epochs)    │
└───────────────────┘
        │
        ▼
┌───────────────────┐
│ Minute Data       │
│ Collection        │──► VMC, Orientation, Flags
│                   │
└───────────────────┘
        │
        ▼
┌───────────────────┐
│ Sleep Score       │
│ Calculation       │──► 9-minute weighted convolution
│                   │
└───────────────────┘
        │
        ▼
┌───────────────────┐
│ Sleep State       │
│ Machine           │──► Entry/Exit detection
│                   │
└───────────────────┘
        │
   ┌────┴────┐
   │         │
   ▼         ▼
┌──────┐  ┌──────────┐
│Deep  │  │Not-Worn  │
│Sleep │  │Detection │
└──────┘  └──────────┘
   │         │
   └────┬────┘
        │
        ▼
┌───────────────────┐
│ Session           │
│ Validation        │──► Duration, Active%, Avg VMC
│                   │
└───────────────────┘
        │
        ▼
┌───────────────────┐
│ Nap               │
│ Classification    │──► Time window, Duration
│                   │
└───────────────────┘
        │
        ▼
┌───────────────────┐
│ Record Session    │
│ to History        │──► 30-day rolling storage
│                   │
└───────────────────┘
```

---

## Summary

The PebbleOS sleep monitoring system provides:

1. **Motion-based Detection**: Uses VMC from accelerometer data
2. **Weighted Scoring**: 9-minute convolution smooths decisions
3. **Three Sleep States**: Awake, Light Sleep, Deep Sleep
4. **Robust Validation**: Multiple checks prevent false positives
5. **Nap Recognition**: Time-based classification for daytime sleep
6. **Not-Worn Detection**: Distinguishes sleep from watch removal

**Key Thresholds Summary**:

| Parameter | Value | Purpose |
|-----------|-------|---------|
| Sleep score threshold | 330 | Below = sleep minute |
| Deep sleep threshold | 160 | Below = deep sleep |
| Min sleep minutes | 5 | Start detection |
| Max wake minutes | 11-14 | End detection |
| Min session duration | 60 min | Validation |
| Max active % | 89% | Validation |
| Max nap duration | 180 min | Nap classification |
