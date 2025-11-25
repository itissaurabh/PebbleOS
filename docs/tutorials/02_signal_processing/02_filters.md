# Tutorial 2.2: Filter Design and Implementation

## Learning Objectives

By the end of this tutorial, you will:
- Understand why filtering is essential for biosignals
- Design Butterworth, Chebyshev, and other filter types
- Implement FIR and IIR filters in C and Python
- Apply filters to real-world biosignal data
- Choose the right filter for each application

## Prerequisites
- Tutorial 1.1 (Mathematical Foundations)
- Tutorial 2.1 (DSP Basics)

---

## Table of Contents

1. [Why Filtering Matters](#1-why-filtering-matters)
2. [Filter Fundamentals](#2-filter-fundamentals)
3. [FIR Filters](#3-fir-filters)
4. [IIR Filters](#4-iir-filters)
5. [Butterworth Filter Design](#5-butterworth-filter-design)
6. [Other Filter Types](#6-other-filter-types)
7. [Implementation Examples](#7-implementation-examples)
8. [Filter Selection Guide](#8-filter-selection-guide)
9. [Learning Resources](#9-learning-resources)

---

## 1. Why Filtering Matters

### 1.1 Biosignals Are Noisy

Raw sensor data contains **signal + noise + artifacts**:

```
    Raw PPG Signal
    │
    │   ╭─╮  ╭─╮          Motion        ╭─╮
    │  ╱   ╲╱   ╲  artifact  ╭─────╮   ╱   ╲
    │ ╱         ╲          ╱       ╲ ╱      ╲
    │╱           ╲________╱         ╲         ╲
    │
    └────────────────────────────────────────────→ Time

    Signal components:
    - Cardiac pulse: 0.5-4 Hz (desired)
    - Respiratory modulation: 0.1-0.5 Hz
    - Motion artifacts: Variable, often low frequency
    - Power line interference: 50/60 Hz
    - High-frequency noise: >10 Hz
```

### 1.2 Filtering Separates Components

```
    ┌─────────────────────────────────────────────────────────┐
    │            FREQUENCY DOMAIN VIEW                        │
    └─────────────────────────────────────────────────────────┘

    Amplitude
        │
        │  Baseline           Cardiac      High-freq
        │   drift              pulse        noise
        │    ▓▓                ▓▓▓
        │   ▓▓▓▓              ▓▓▓▓▓         ▓▓
        │  ▓▓▓▓▓▓            ▓▓▓▓▓▓▓       ▓▓▓▓
        │ ▓▓▓▓▓▓▓▓          ▓▓▓▓▓▓▓▓▓     ▓▓▓▓▓▓
        └───────────────────────────────────────────→
          0.1 Hz    0.5    1     2     5    10    50 Hz

    Bandpass filter (0.5-4 Hz) keeps only cardiac signal:

        │
        │                     ▓▓▓
        │                    ▓▓▓▓▓
        │                   ▓▓▓▓▓▓▓
        │                  ▓▓▓▓▓▓▓▓▓
        └───────────────────────────────────────────→
```

---

## 2. Filter Fundamentals

### 2.1 Filter Types by Frequency Response

```
    ┌─────────────────────────────────────────────────────────┐
    │                    FILTER TYPES                         │
    └─────────────────────────────────────────────────────────┘

    LOW-PASS                     HIGH-PASS
    (keeps low, removes high)    (keeps high, removes low)
    │                            │
    │▓▓▓▓▓▓▓▓▓▓                  │          ▓▓▓▓▓▓▓▓▓▓
    │          ╲                 │         ╱
    │           ╲                │        ╱
    │            ╲               │       ╱
    └──────────────────→ f       └──────────────────→ f
            fc (cutoff)                 fc

    BAND-PASS                    BAND-STOP (Notch)
    (keeps middle band)          (removes middle band)
    │                            │
    │        ▓▓▓▓▓               │▓▓▓▓▓▓▓      ▓▓▓▓▓▓▓
    │       ╱     ╲              │       ╲    ╱
    │      ╱       ╲             │        ╲  ╱
    │     ╱         ╲            │         ╲╱
    └──────────────────→ f       └──────────────────→ f
         f_low  f_high                fc (notch freq)
```

### 2.2 Key Filter Parameters

```
    Magnitude Response

    |H(f)|
    (dB)
      │
    0 │▓▓▓▓▓▓▓▓▓▓▓▓▓▓────────────────── Passband (signal passes)
      │               ╲
   -3 │                ╲ ← -3dB point (cutoff frequency)
      │                 ╲
      │                  ╲
      │                   ╲ Transition band
  -40 │                    ╲______________ Stopband (signal blocked)
      │
      └─────────────────────────────────────→ Frequency
                    fc        fs

    Parameters:
    ┌────────────────────┬────────────────────────────────────┐
    │ Cutoff frequency   │ Where response drops to -3dB       │
    │ Passband ripple    │ Variation in passband (ideally 0)  │
    │ Stopband atten.    │ How much signal is reduced         │
    │ Transition width   │ fc to fs (sharper = higher order)  │
    │ Order              │ Number of poles (steeper = higher) │
    └────────────────────┴────────────────────────────────────┘
```

### 2.3 Phase Response

Filters also affect **phase** (timing) of signals:

```
    Phase Response
    ∠H(f)
    (degrees)
      │
    0 │──────╮
      │       ╲
  -90 │        ╲
      │         ╲
 -180 │──────────────╲
      │               ╲
 -270 │                ╲──────────
      └────────────────────────────→ Frequency

    Phase effects:
    - Linear phase: All frequencies delayed equally (FIR)
    - Nonlinear phase: Different frequencies delayed differently (IIR)

    For biosignals:
    - ECG QRS detection: Linear phase preferred (preserve shape)
    - Heart rate calculation: Nonlinear OK (just finding peaks)
```

---

## 3. FIR Filters

### 3.1 FIR Filter Structure

**Finite Impulse Response**: Output depends only on current and past inputs

```
    FIR Filter: y[n] = Σ h[k] · x[n-k], k = 0 to M-1

    Block diagram (M = 4):

    x[n] ─────┬─────────┬─────────┬─────────┐
              │         │         │         │
              │   z⁻¹   │   z⁻¹   │   z⁻¹   │
              │    │    │    │    │    │    │
              ▼    ▼    ▼    ▼    ▼    ▼    ▼
             h[0] h[1] h[2] h[3] h[4]
              │    │    │    │    │
              ▼    ▼    ▼    ▼    ▼
              └────┴────┴────┴────┘
                       │
                       ▼
                     y[n] = h[0]x[n] + h[1]x[n-1] + h[2]x[n-2] + ...
```

### 3.2 FIR Filter Properties

| Advantage | Disadvantage |
|-----------|--------------|
| Always stable | Requires many taps for sharp cutoff |
| Linear phase possible | Higher computational cost |
| Easy to design | More memory for coefficients |
| No feedback issues | |

### 3.3 FIR Filter Implementation

```c
// FIR Filter Implementation in C
typedef struct {
    float *coefficients;  // Filter coefficients h[k]
    float *delay_line;    // Previous input samples
    int num_taps;         // Number of coefficients
    int delay_index;      // Current position in circular buffer
} FIRFilter;

void fir_filter_init(FIRFilter *f, float *coeffs, float *delay, int taps) {
    f->coefficients = coeffs;
    f->delay_line = delay;
    f->num_taps = taps;
    f->delay_index = 0;

    // Clear delay line
    for (int i = 0; i < taps; i++) {
        f->delay_line[i] = 0.0f;
    }
}

float fir_filter_process(FIRFilter *f, float input) {
    // Store new sample in delay line
    f->delay_line[f->delay_index] = input;

    // Compute output: y = sum(h[k] * x[n-k])
    float output = 0.0f;
    int index = f->delay_index;

    for (int k = 0; k < f->num_taps; k++) {
        output += f->coefficients[k] * f->delay_line[index];

        // Move backward through circular buffer
        index--;
        if (index < 0) {
            index = f->num_taps - 1;
        }
    }

    // Advance delay line index
    f->delay_index++;
    if (f->delay_index >= f->num_taps) {
        f->delay_index = 0;
    }

    return output;
}

// Example: 11-tap moving average filter (simple low-pass)
#define MA_TAPS 11
float ma_coeffs[MA_TAPS] = {
    1.0f/11, 1.0f/11, 1.0f/11, 1.0f/11, 1.0f/11,
    1.0f/11,
    1.0f/11, 1.0f/11, 1.0f/11, 1.0f/11, 1.0f/11
};
float ma_delay[MA_TAPS];
FIRFilter moving_avg;

void init_example(void) {
    fir_filter_init(&moving_avg, ma_coeffs, ma_delay, MA_TAPS);
}

float filter_sample(float x) {
    return fir_filter_process(&moving_avg, x);
}
```

### 3.4 FIR Filter Design in Python

```python
import numpy as np
from scipy import signal
import matplotlib.pyplot as plt

def design_fir_lowpass(num_taps, cutoff_hz, sample_rate):
    """
    Design FIR lowpass filter using window method

    Parameters:
    - num_taps: Number of filter coefficients (odd for symmetric)
    - cutoff_hz: Cutoff frequency in Hz
    - sample_rate: Sampling frequency in Hz

    Returns:
    - coefficients: Filter coefficients
    """
    # Normalized cutoff (0 to 1, where 1 = Nyquist)
    nyquist = sample_rate / 2
    normalized_cutoff = cutoff_hz / nyquist

    # Design filter using Hamming window
    coefficients = signal.firwin(num_taps, normalized_cutoff, window='hamming')

    return coefficients

# Example: Low-pass filter for accelerometer (remove noise above 10 Hz)
fs = 100  # 100 Hz sampling rate
fc = 10   # 10 Hz cutoff
taps = 31 # Number of taps

coeffs = design_fir_lowpass(taps, fc, fs)

# Print coefficients for C implementation
print("// FIR coefficients (generated by Python)")
print(f"#define NUM_TAPS {taps}")
print("float fir_coeffs[NUM_TAPS] = {")
for i, c in enumerate(coeffs):
    end = "," if i < len(coeffs) - 1 else ""
    print(f"    {c:.10f}{end}")
print("};")

# Visualize frequency response
w, h = signal.freqz(coeffs)
frequencies = w * fs / (2 * np.pi)

plt.figure(figsize=(10, 4))
plt.subplot(1, 2, 1)
plt.plot(frequencies, 20 * np.log10(np.abs(h)))
plt.xlabel('Frequency (Hz)')
plt.ylabel('Magnitude (dB)')
plt.title('FIR Filter Frequency Response')
plt.axvline(fc, color='r', linestyle='--', label='Cutoff')
plt.legend()
plt.grid(True)

plt.subplot(1, 2, 2)
plt.stem(coeffs)
plt.xlabel('Tap')
plt.ylabel('Coefficient')
plt.title('Filter Coefficients')
plt.grid(True)
plt.tight_layout()
plt.show()
```

---

## 4. IIR Filters

### 4.1 IIR Filter Structure

**Infinite Impulse Response**: Output depends on inputs AND previous outputs (feedback)

```
    IIR Filter (Direct Form I):

    y[n] = (1/a₀)[b₀x[n] + b₁x[n-1] + ... - a₁y[n-1] - a₂y[n-2] - ...]

    Block diagram (2nd order):

    x[n] ──┬──►[b₀]──┬────────────────────┬──► y[n]
           │        │                    │
         [z⁻¹]      │                  [-a₁]
           │        │                    │
           ├──►[b₁]─┼────────┬───────────┤
           │        │        │           │
         [z⁻¹]      │      [z⁻¹]         │
           │        │        │           │
           └──►[b₂]─┴────────┴──►[-a₂]───┘
                            (feedback from output)
```

### 4.2 IIR Filter Properties

| Advantage | Disadvantage |
|-----------|--------------|
| Fewer coefficients needed | Can be unstable |
| Lower computational cost | Nonlinear phase |
| Sharp cutoff possible | Sensitive to coefficient quantization |
| Matches analog filters | Feedback can cause overflow |

### 4.3 Biquad Implementation (2nd Order IIR)

The **biquad** (biquadratic) filter is the building block for IIR filters:

```c
// Biquad (2nd order IIR) filter
typedef struct {
    // Coefficients (normalized: a0 = 1)
    float b0, b1, b2;  // Feedforward (numerator)
    float a1, a2;      // Feedback (denominator)

    // State variables
    float z1, z2;      // Previous intermediate values (Direct Form II)
} Biquad;

void biquad_init(Biquad *bq, float b0, float b1, float b2,
                             float a0, float a1, float a2) {
    // Normalize coefficients by a0
    bq->b0 = b0 / a0;
    bq->b1 = b1 / a0;
    bq->b2 = b2 / a0;
    bq->a1 = a1 / a0;
    bq->a2 = a2 / a0;

    // Clear state
    bq->z1 = 0.0f;
    bq->z2 = 0.0f;
}

float biquad_process(Biquad *bq, float input) {
    // Direct Form II Transposed (most numerically stable)
    float output = bq->b0 * input + bq->z1;

    bq->z1 = bq->b1 * input - bq->a1 * output + bq->z2;
    bq->z2 = bq->b2 * input - bq->a2 * output;

    return output;
}

// Cascaded biquads for higher-order filters
#define MAX_BIQUADS 4

typedef struct {
    Biquad stages[MAX_BIQUADS];
    int num_stages;
} CascadedBiquad;

float cascaded_biquad_process(CascadedBiquad *cb, float input) {
    float output = input;
    for (int i = 0; i < cb->num_stages; i++) {
        output = biquad_process(&cb->stages[i], output);
    }
    return output;
}
```

### 4.4 Fixed-Point Biquad (Q15)

```c
// Q15 fixed-point biquad for efficient embedded implementation
typedef struct {
    int16_t b0, b1, b2;
    int16_t a1, a2;
    int16_t z1, z2;
} BiquadQ15;

int16_t biquad_q15_process(BiquadQ15 *bq, int16_t input) {
    // Use 32-bit accumulator to prevent overflow
    int32_t acc;

    // Feedforward path
    acc = (int32_t)bq->b0 * input;
    acc += (int32_t)bq->b1 * bq->z1;
    acc += (int32_t)bq->b2 * bq->z2;

    // Feedback path (note: a1, a2 are negated in storage)
    acc -= (int32_t)bq->a1 * bq->z1;
    acc -= (int32_t)bq->a2 * bq->z2;

    // Scale back to Q15 with rounding
    int16_t output = (int16_t)((acc + 0x4000) >> 15);

    // Saturate if overflow
    if (acc > 0x3FFFFFFF) output = 32767;
    if (acc < -0x40000000) output = -32768;

    // Update state
    bq->z2 = bq->z1;
    bq->z1 = output;

    return output;
}
```

---

## 5. Butterworth Filter Design

### 5.1 Butterworth Characteristics

The **Butterworth filter** has maximally flat passband response:

```
    Butterworth Magnitude Response

    |H(jω)|²
      │
    1 │▓▓▓▓▓▓▓▓▓▓▓▓▓▓╲
      │               ╲
      │                ╲  Order 2
      │                 ╲
    0.5│                  ╲
      │                   ╲________
      │                            ╲  Order 4
      │                             ╲________
      │                                      ╲  Order 8
    0 │─────────────────────────────────────────────
      └──────────────────────────────────────────→
                      ωc              ω

    Magnitude formula:
    |H(jω)|² = 1 / (1 + (ω/ωc)^(2n))

    Where:
    - ωc = cutoff frequency
    - n = filter order

    Properties:
    - No ripple in passband or stopband
    - -3dB at cutoff frequency
    - Roll-off: 20n dB/decade (n = order)
```

### 5.2 Butterworth Design Equations

```
    2nd Order Butterworth Low-Pass

    Transfer function:
    H(s) = ωc² / (s² + √2·ωc·s + ωc²)

    Bilinear transform to digital (s → z):
    s = (2/T) · (1 - z⁻¹) / (1 + z⁻¹)

    Where T = 1/fs (sampling period)

    Pre-warping:
    ωc_digital = (2/T) · tan(ω_analog · T / 2)

    This accounts for frequency warping in bilinear transform.
```

### 5.3 Butterworth Design in Python

```python
from scipy import signal
import numpy as np

def design_butterworth_lowpass(order, cutoff_hz, sample_rate):
    """
    Design digital Butterworth low-pass filter

    Returns second-order sections (SOS) for numerical stability
    """
    nyquist = sample_rate / 2
    normalized_cutoff = cutoff_hz / nyquist

    # Design filter and return as second-order sections
    sos = signal.butter(order, normalized_cutoff, btype='low', output='sos')

    return sos

def design_butterworth_bandpass(order, low_hz, high_hz, sample_rate):
    """
    Design digital Butterworth band-pass filter
    """
    nyquist = sample_rate / 2
    low_normalized = low_hz / nyquist
    high_normalized = high_hz / nyquist

    sos = signal.butter(order, [low_normalized, high_normalized],
                        btype='band', output='sos')

    return sos

# Example: Design filters for step detection
fs = 25  # 25 Hz accelerometer

# Low-pass: Remove high-frequency noise
lpf_sos = design_butterworth_lowpass(order=2, cutoff_hz=5, sample_rate=fs)

# Band-pass: Isolate walking frequency (0.5-3 Hz)
bpf_sos = design_butterworth_bandpass(order=2, low_hz=0.5, high_hz=3, sample_rate=fs)

# Convert SOS to biquad coefficients for C implementation
def sos_to_biquads(sos):
    """Convert SOS array to individual biquad coefficients"""
    biquads = []
    for section in sos:
        b = section[:3]  # b0, b1, b2
        a = section[3:]  # a0 (=1), a1, a2
        biquads.append({
            'b0': b[0], 'b1': b[1], 'b2': b[2],
            'a0': a[0], 'a1': a[1], 'a2': a[2]
        })
    return biquads

# Generate C code
print("// Butterworth Low-Pass Filter Coefficients")
print("// Cutoff: 5 Hz, Sample Rate: 25 Hz, Order: 2")
biquads = sos_to_biquads(lpf_sos)
for i, bq in enumerate(biquads):
    print(f"\n// Biquad stage {i+1}")
    print(f"biquad_init(&lpf_stage{i+1},")
    print(f"    {bq['b0']:.10f}f,  // b0")
    print(f"    {bq['b1']:.10f}f,  // b1")
    print(f"    {bq['b2']:.10f}f,  // b2")
    print(f"    {bq['a0']:.10f}f,  // a0")
    print(f"    {bq['a1']:.10f}f,  // a1")
    print(f"    {bq['a2']:.10f}f); // a2")
```

### 5.4 Complete Butterworth Filter in C

```c
// butterworth.h
#ifndef BUTTERWORTH_H
#define BUTTERWORTH_H

#include <stdint.h>
#include <stdbool.h>

// 2nd order Butterworth low-pass filter
typedef struct {
    // Biquad coefficients
    float b0, b1, b2;
    float a1, a2;

    // Filter state
    float x1, x2;  // Previous inputs
    float y1, y2;  // Previous outputs
} ButterworthLP2;

// Initialize filter with cutoff frequency
void butterworth_lp2_init(ButterworthLP2 *f, float cutoff_hz, float sample_rate);

// Process single sample
float butterworth_lp2_process(ButterworthLP2 *f, float input);

// Reset filter state
void butterworth_lp2_reset(ButterworthLP2 *f);

#endif // BUTTERWORTH_H
```

```c
// butterworth.c
#include "butterworth.h"
#include <math.h>

#ifndef M_PI
#define M_PI 3.14159265358979323846
#endif

void butterworth_lp2_init(ButterworthLP2 *f, float cutoff_hz, float sample_rate) {
    // Pre-warp the cutoff frequency
    float wc = 2.0f * sample_rate * tanf(M_PI * cutoff_hz / sample_rate);

    // Bilinear transform coefficients
    float k = 2.0f * sample_rate;
    float k2 = k * k;
    float wc2 = wc * wc;
    float sqrt2_wc_k = sqrtf(2.0f) * wc * k;

    // Denominator
    float a0 = k2 + sqrt2_wc_k + wc2;

    // Normalized coefficients
    f->b0 = wc2 / a0;
    f->b1 = 2.0f * wc2 / a0;
    f->b2 = wc2 / a0;
    f->a1 = (2.0f * wc2 - 2.0f * k2) / a0;
    f->a2 = (k2 - sqrt2_wc_k + wc2) / a0;

    // Initialize state
    butterworth_lp2_reset(f);
}

float butterworth_lp2_process(ButterworthLP2 *f, float input) {
    // Direct Form I implementation
    float output = f->b0 * input
                 + f->b1 * f->x1
                 + f->b2 * f->x2
                 - f->a1 * f->y1
                 - f->a2 * f->y2;

    // Update state
    f->x2 = f->x1;
    f->x1 = input;
    f->y2 = f->y1;
    f->y1 = output;

    return output;
}

void butterworth_lp2_reset(ButterworthLP2 *f) {
    f->x1 = 0.0f;
    f->x2 = 0.0f;
    f->y1 = 0.0f;
    f->y2 = 0.0f;
}

// Example usage for accelerometer filtering
#include <stdio.h>

int main() {
    ButterworthLP2 accel_filter;

    // Initialize: 5 Hz cutoff, 25 Hz sample rate
    butterworth_lp2_init(&accel_filter, 5.0f, 25.0f);

    // Process some test samples
    float test_signal[] = {0.0f, 0.5f, 1.0f, 0.8f, 0.3f, -0.2f, -0.5f, -0.3f};
    int n = sizeof(test_signal) / sizeof(test_signal[0]);

    printf("Input\t\tOutput (filtered)\n");
    for (int i = 0; i < n; i++) {
        float filtered = butterworth_lp2_process(&accel_filter, test_signal[i]);
        printf("%.3f\t\t%.3f\n", test_signal[i], filtered);
    }

    return 0;
}
```

---

## 6. Other Filter Types

### 6.1 Chebyshev Filters

**Type I**: Ripple in passband, flat stopband
**Type II**: Flat passband, ripple in stopband

```
    Chebyshev Type I vs Butterworth

    |H(f)|
      │      ╭─╮╭─╮
    1 │▓▓▓▓▓╱   ╲   ╲  Chebyshev (ripple in passband)
      │           ╲   ╲
      │            ╲   ╲ Sharper transition
      │             ╲
      │▓▓▓▓▓▓▓▓▓▓▓▓▓▓╲  Butterworth (flat)
    0 │─────────────────────────────
      └────────────────────────────→ f

    When to use Chebyshev:
    - Need sharper cutoff with fewer coefficients
    - Can tolerate some passband ripple
    - ECG: Often acceptable (ripple < 1 dB)
```

```python
# Chebyshev Type I design
def design_chebyshev1_bandpass(order, ripple_db, low_hz, high_hz, sample_rate):
    """
    Chebyshev Type I bandpass filter

    Parameters:
    - ripple_db: Maximum ripple in passband (dB)
    """
    nyquist = sample_rate / 2
    low_norm = low_hz / nyquist
    high_norm = high_hz / nyquist

    sos = signal.cheby1(order, ripple_db, [low_norm, high_norm],
                        btype='band', output='sos')
    return sos

# Example: ECG bandpass filter (0.5-40 Hz)
ecg_filter = design_chebyshev1_bandpass(
    order=4,
    ripple_db=0.5,
    low_hz=0.5,
    high_hz=40,
    sample_rate=250
)
```

### 6.2 Notch Filter (Band-Stop)

Removes a specific frequency (e.g., 50/60 Hz power line interference):

```c
// Notch filter to remove power line interference
typedef struct {
    float b0, b1, b2;
    float a1, a2;
    float z1, z2;
} NotchFilter;

void notch_filter_init(NotchFilter *f, float notch_freq_hz, float sample_rate,
                       float bandwidth_hz) {
    float w0 = 2.0f * M_PI * notch_freq_hz / sample_rate;
    float bw = 2.0f * M_PI * bandwidth_hz / sample_rate;

    // Quality factor
    float Q = notch_freq_hz / bandwidth_hz;

    // Design coefficients
    float alpha = sinf(w0) / (2.0f * Q);
    float cos_w0 = cosf(w0);

    float a0 = 1.0f + alpha;

    f->b0 = 1.0f / a0;
    f->b1 = -2.0f * cos_w0 / a0;
    f->b2 = 1.0f / a0;
    f->a1 = -2.0f * cos_w0 / a0;
    f->a2 = (1.0f - alpha) / a0;

    f->z1 = 0.0f;
    f->z2 = 0.0f;
}

float notch_filter_process(NotchFilter *f, float input) {
    // Direct Form II Transposed
    float output = f->b0 * input + f->z1;
    f->z1 = f->b1 * input - f->a1 * output + f->z2;
    f->z2 = f->b2 * input - f->a2 * output;
    return output;
}

// Example: Remove 60 Hz interference from ECG
NotchFilter power_line_filter;
notch_filter_init(&power_line_filter, 60.0f, 250.0f, 2.0f);  // 60 Hz notch, 2 Hz bandwidth
```

### 6.3 Moving Average Filter

Simple FIR filter for smoothing:

```c
// Efficient moving average using running sum
typedef struct {
    float *buffer;
    int size;
    int index;
    float sum;
    bool filled;
} MovingAverage;

void moving_average_init(MovingAverage *ma, float *buf, int size) {
    ma->buffer = buf;
    ma->size = size;
    ma->index = 0;
    ma->sum = 0.0f;
    ma->filled = false;

    for (int i = 0; i < size; i++) {
        ma->buffer[i] = 0.0f;
    }
}

float moving_average_process(MovingAverage *ma, float input) {
    // Subtract oldest value from sum
    ma->sum -= ma->buffer[ma->index];

    // Add new value
    ma->buffer[ma->index] = input;
    ma->sum += input;

    // Update index
    ma->index++;
    if (ma->index >= ma->size) {
        ma->index = 0;
        ma->filled = true;
    }

    // Calculate average
    int count = ma->filled ? ma->size : ma->index;
    return ma->sum / count;
}
```

### 6.4 Derivative Filter

High-pass filter that computes rate of change:

```c
// 5-point derivative filter (better than simple difference)
// Coefficients: [-2, -1, 0, 1, 2] / 10
typedef struct {
    float buffer[5];
    int index;
} DerivativeFilter;

void derivative_filter_init(DerivativeFilter *f) {
    for (int i = 0; i < 5; i++) {
        f->buffer[i] = 0.0f;
    }
    f->index = 0;
}

float derivative_filter_process(DerivativeFilter *f, float input, float sample_rate) {
    f->buffer[f->index] = input;

    // 5-point stencil: (-2x[n-2] - x[n-1] + x[n+1] + 2x[n+2]) / (10*Ts)
    int i0 = f->index;
    int i1 = (f->index + 1) % 5;
    int i2 = (f->index + 2) % 5;
    int i3 = (f->index + 3) % 5;
    int i4 = (f->index + 4) % 5;

    float derivative = (-2.0f * f->buffer[i0]
                       - 1.0f * f->buffer[i1]
                       + 0.0f * f->buffer[i2]
                       + 1.0f * f->buffer[i3]
                       + 2.0f * f->buffer[i4]) * sample_rate / 10.0f;

    f->index = (f->index + 1) % 5;

    return derivative;
}
```

---

## 7. Implementation Examples

### 7.1 PPG Signal Processing Pipeline

```c
// Complete PPG processing pipeline for heart rate
typedef struct {
    // Filters
    ButterworthLP2 anti_alias;     // Anti-aliasing before processing
    NotchFilter power_line;        // Remove 50/60 Hz
    ButterworthLP2 baseline_hp;    // High-pass for baseline removal
    ButterworthLP2 smooth_lp;      // Low-pass for smoothing

    // High-pass implemented as HP = input - LP(input)
} PPGPipeline;

void ppg_pipeline_init(PPGPipeline *p, float sample_rate, float power_freq) {
    // Anti-aliasing: 10 Hz low-pass
    butterworth_lp2_init(&p->anti_alias, 10.0f, sample_rate);

    // Power line notch: 50 or 60 Hz
    notch_filter_init(&p->power_line, power_freq, sample_rate, 2.0f);

    // Baseline removal: 0.5 Hz high-pass (implemented via low-pass)
    butterworth_lp2_init(&p->baseline_hp, 0.5f, sample_rate);

    // Smoothing: 4 Hz low-pass
    butterworth_lp2_init(&p->smooth_lp, 4.0f, sample_rate);
}

float ppg_pipeline_process(PPGPipeline *p, float raw_sample) {
    // Step 1: Anti-alias
    float x1 = butterworth_lp2_process(&p->anti_alias, raw_sample);

    // Step 2: Remove power line interference
    float x2 = notch_filter_process(&p->power_line, x1);

    // Step 3: Remove baseline wander (high-pass via DC subtraction)
    float baseline = butterworth_lp2_process(&p->baseline_hp, x2);
    float x3 = x2 - baseline;

    // Step 4: Smooth
    float x4 = butterworth_lp2_process(&p->smooth_lp, x3);

    return x4;
}
```

### 7.2 Accelerometer Processing for Steps

```c
// Accelerometer processing for step detection (PebbleOS style)
typedef struct {
    // Bandpass filter (0.5-3 Hz for walking)
    Biquad bp_stage1;
    Biquad bp_stage2;
} AccelStepFilter;

void accel_step_filter_init(AccelStepFilter *f, float sample_rate) {
    // These coefficients are for 25 Hz sample rate
    // Butterworth bandpass 0.5-3 Hz

    // First biquad stage
    biquad_init(&f->bp_stage1,
        0.0675f, 0.0f, -0.0675f,  // b0, b1, b2
        1.0f, -1.8424f, 0.8651f); // a0, a1, a2

    // Second biquad stage
    biquad_init(&f->bp_stage2,
        1.0f, 0.0f, -1.0f,
        1.0f, -1.9556f, 0.9565f);
}

float accel_step_filter_process(AccelStepFilter *f, float input) {
    float x1 = biquad_process(&f->bp_stage1, input);
    float x2 = biquad_process(&f->bp_stage2, x1);
    return x2;
}
```

---

## 8. Filter Selection Guide

### 8.1 Quick Reference Table

| Application | Filter Type | Cutoff | Order | Notes |
|------------|-------------|--------|-------|-------|
| Accelerometer smoothing | Butterworth LP | 10-20 Hz | 2 | Remove sensor noise |
| Step detection | Butterworth BP | 0.5-3 Hz | 4 | Isolate walking frequency |
| PPG heart rate | Butterworth BP | 0.5-4 Hz | 2-4 | Cardiac frequency range |
| ECG | Butterworth BP | 0.5-40 Hz | 4 | Medical standard |
| Baseline removal | High-pass | 0.05-0.5 Hz | 2 | Remove DC drift |
| Power line | Notch | 50/60 Hz | 2 | Q ≈ 30 |
| Respiratory | Butterworth LP | 0.1-0.5 Hz | 2 | Extract breathing |

### 8.2 Decision Tree

```
    SELECT A FILTER
          │
          ▼
    ┌─────────────────┐
    │ Need linear     │──Yes──► FIR Filter
    │ phase?          │         (more taps needed)
    └────────┬────────┘
             │No
             ▼
    ┌─────────────────┐
    │ Need very sharp │──Yes──► Chebyshev or Elliptic
    │ cutoff?         │         (allows some ripple)
    └────────┬────────┘
             │No
             ▼
    ┌─────────────────┐
    │ Need maximum    │──Yes──► Butterworth
    │ flatness?       │         (most common choice)
    └────────┬────────┘
             │No
             ▼
    ┌─────────────────┐
    │ Need simple     │──Yes──► Moving Average
    │ smoothing?      │         (FIR, easy to implement)
    └────────┬────────┘
             │No
             ▼
    ┌─────────────────┐
    │ Need to remove  │──Yes──► Notch Filter
    │ specific freq?  │         (narrow band-stop)
    └─────────────────┘
```

### 8.3 Common Pitfalls

```
    ⚠️ FILTER DESIGN PITFALLS

    1. Forgetting to pre-warp cutoff frequency
       - Bilinear transform warps frequencies
       - Always pre-warp for accurate cutoff

    2. Coefficient quantization errors
       - Fixed-point can cause instability
       - Use cascaded biquads, not direct form

    3. Ignoring group delay
       - Filters delay the signal
       - Important for real-time beat detection

    4. Wrong filter order
       - Too low: Insufficient rejection
       - Too high: Ringing, computational cost

    5. Not considering transient response
       - Filters need time to "warm up"
       - Initialize state or discard initial samples
```

---

## 9. Learning Resources

### 9.1 Books

| Book | Author | Level | Focus |
|------|--------|-------|-------|
| **"The Scientist and Engineer's Guide to DSP"** | Steven W. Smith | Beginner | Free online, excellent intro |
| **"Digital Signal Processing"** | Proakis & Manolakis | Intermediate | Comprehensive DSP |
| **"Understanding Digital Signal Processing"** | Richard Lyons | Intermediate | Practical focus |
| **"Digital Filter Designer's Handbook"** | C. Britton Rorabaugh | Intermediate | Filter design recipes |

### 9.2 Online Resources

| Resource | Description | Link |
|----------|-------------|------|
| **DSP Guide (free book)** | Complete DSP textbook | [dspguide.com](http://www.dspguide.com/) |
| **Julius O. Smith III** | Stanford DSP courses | [ccrma.stanford.edu](https://ccrma.stanford.edu/~jos/) |
| **DSP StackExchange** | Q&A forum | [dsp.stackexchange.com](https://dsp.stackexchange.com/) |
| **Analog Devices Wiki** | Practical filter design | [wiki.analog.com](https://wiki.analog.com/) |

### 9.3 Tools

| Tool | Purpose | Link |
|------|---------|------|
| **SciPy signal** | Python filter design | [scipy.org](https://docs.scipy.org/doc/scipy/reference/signal.html) |
| **MATLAB fdatool** | Interactive design | MATLAB |
| **Filter Design Tool** | Web-based | [t-filter.engineerjs.com](http://t-filter.engineerjs.com/) |
| **Iowa Hills Filters** | Free Windows app | [iowahills.com](http://www.iowahills.com/) |

### 9.4 YouTube Tutorials

| Channel | Topics |
|---------|--------|
| **Alan V Oppenheim (MIT)** | Classic signals lectures |
| **Brian Douglas** | Control systems, filters |
| **Steve Brunton** | Modern DSP, data-driven |
| **Phil's Lab** | Embedded filter implementation |

---

## Summary

Filter design is essential for biosignal processing:

| Concept | Key Points |
|---------|------------|
| **FIR Filters** | Linear phase, stable, more taps needed |
| **IIR Filters** | Efficient, nonlinear phase, can be unstable |
| **Butterworth** | Maximally flat, most common choice |
| **Biquad** | Building block for IIR, numerically stable |
| **Application** | Match filter to signal characteristics |

**Key Takeaways**:
1. Always design filters in Python/MATLAB first
2. Use cascaded biquads for IIR filters
3. Consider fixed-point for embedded systems
4. Test filters on real data before deployment
5. Account for group delay in real-time applications

---

**Next Tutorial**: [2.3 Frequency Analysis →](03_frequency_analysis.md)
