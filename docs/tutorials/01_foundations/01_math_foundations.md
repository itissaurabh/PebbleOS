# Tutorial 1.1: Mathematical Foundations for Biosignal Processing

## Learning Objectives

By the end of this tutorial, you will:
- Understand linear algebra concepts essential for signal processing
- Apply calculus to continuous and discrete signals
- Use probability and statistics for signal analysis
- Implement mathematical operations in code

## Prerequisites
- Basic algebra (solving equations, manipulating expressions)
- Familiarity with functions and graphs
- Basic programming ability

---

## Table of Contents

1. [Linear Algebra Essentials](#1-linear-algebra-essentials)
2. [Calculus for Signals](#2-calculus-for-signals)
3. [Complex Numbers and Exponentials](#3-complex-numbers-and-exponentials)
4. [Probability and Statistics](#4-probability-and-statistics)
5. [Numerical Methods](#5-numerical-methods)
6. [Exercises](#6-exercises)
7. [Learning Resources](#7-learning-resources)

---

## 1. Linear Algebra Essentials

### 1.1 Why Linear Algebra for Biosignals?

Biosignals are **sequences of numbers**. Processing them means performing **mathematical operations on these sequences**. Linear algebra provides the framework for:

- Representing signals as vectors
- Transforming signals (filtering, frequency analysis)
- Extracting features for classification
- Understanding machine learning algorithms

### 1.2 Vectors

A **vector** is an ordered list of numbers. A biosignal sampled N times is a vector:

```
Signal x with 5 samples:
x = [1.2, 0.8, 1.5, 1.1, 0.9]

In mathematical notation:
x = [x₀, x₁, x₂, x₃, x₄]ᵀ  (column vector, T means transpose)
```

**Key Vector Operations**:

```python
import numpy as np

# Vector creation
x = np.array([1.2, 0.8, 1.5, 1.1, 0.9])

# Addition (element-wise)
y = np.array([0.1, 0.2, 0.1, 0.3, 0.2])
z = x + y  # [1.3, 1.0, 1.6, 1.4, 1.1]

# Scalar multiplication
scaled = 2 * x  # [2.4, 1.6, 3.0, 2.2, 1.8]

# Dot product (inner product)
dot = np.dot(x, y)  # x₀y₀ + x₁y₁ + ... = 0.82

# Magnitude (Euclidean norm)
magnitude = np.linalg.norm(x)  # √(x₀² + x₁² + ...) = 2.57
```

**C Implementation**:
```c
#include <math.h>

// Vector dot product
float dot_product(float *x, float *y, int n) {
    float sum = 0.0f;
    for (int i = 0; i < n; i++) {
        sum += x[i] * y[i];
    }
    return sum;
}

// Vector magnitude
float magnitude(float *x, int n) {
    return sqrtf(dot_product(x, x, n));
}
```

### 1.3 The Dot Product: Measuring Similarity

The dot product tells us **how similar two signals are**:

```
Dot product: x · y = |x| |y| cos(θ)

Where θ is the angle between vectors:
- θ = 0°  → cos(θ) = 1  → vectors point same direction (similar)
- θ = 90° → cos(θ) = 0  → vectors are perpendicular (uncorrelated)
- θ = 180° → cos(θ) = -1 → vectors point opposite (anti-correlated)
```

**Application**: Correlation analysis in biosignals uses dot products to measure similarity between a template and the signal.

### 1.4 Matrices

A **matrix** is a 2D array of numbers. Matrices represent:
- Collections of signals (each row = one signal)
- Transformations (filters)
- Covariance structures

```
Matrix A (3×4):
    ┌                    ┐
    │ a₀₀  a₀₁  a₀₂  a₀₃ │
A = │ a₁₀  a₁₁  a₁₂  a₁₃ │
    │ a₂₀  a₂₁  a₂₂  a₂₃ │
    └                    ┘
```

**Matrix-Vector Multiplication**:

```
y = Ax

Each element of y is a dot product of a row of A with x:
y₀ = a₀₀x₀ + a₀₁x₁ + a₀₂x₂ + a₀₃x₃
y₁ = a₁₀x₀ + a₁₁x₁ + a₁₂x₂ + a₁₃x₃
y₂ = ...
```

**Application**: Filtering is matrix-vector multiplication!

```python
# Applying a filter is equivalent to matrix multiplication
# For a 3-tap moving average filter [1/3, 1/3, 1/3]:

signal = np.array([1, 2, 3, 4, 5])

# Convolution matrix (Toeplitz structure)
H = np.array([
    [1/3, 0,   0,   0,   0  ],
    [1/3, 1/3, 0,   0,   0  ],
    [1/3, 1/3, 1/3, 0,   0  ],
    [0,   1/3, 1/3, 1/3, 0  ],
    [0,   0,   1/3, 1/3, 1/3]
])

filtered = H @ signal  # Matrix multiplication
```

### 1.5 Eigenvalues and Eigenvectors

**Eigenvectors** are special directions that don't change under a transformation (only scaled):

```
Ax = λx

Where:
- A is a matrix (transformation)
- x is an eigenvector
- λ is the eigenvalue (scaling factor)
```

**Application**: Principal Component Analysis (PCA) uses eigenvectors to find the most important directions in multi-sensor data (e.g., combining accelerometer axes).

```python
# Example: Finding principal components of accelerometer data
acc_data = np.array([...])  # Shape: (N_samples, 3) for X, Y, Z

# Compute covariance matrix
cov_matrix = np.cov(acc_data.T)

# Find eigenvalues and eigenvectors
eigenvalues, eigenvectors = np.linalg.eig(cov_matrix)

# First principal component (direction of maximum variance)
pc1 = eigenvectors[:, np.argmax(eigenvalues)]
```

---

## 2. Calculus for Signals

### 2.1 Continuous vs. Discrete

Real-world biosignals are **continuous**, but computers process **discrete** samples:

```
Continuous signal:     x(t) where t ∈ ℝ (any real number)
Discrete signal:       x[n] where n ∈ ℤ (integers only)

Conversion:
x[n] = x(n · Tₛ)    where Tₛ = sampling period = 1/fₛ
```

**Example**:
```
Heart rate signal at 25 Hz sampling:
- Tₛ = 1/25 = 0.04 seconds
- x[0] = value at t = 0.00 s
- x[1] = value at t = 0.04 s
- x[25] = value at t = 1.00 s
```

### 2.2 Derivatives: Rate of Change

The **derivative** measures how fast a signal changes:

```
Continuous:  dx/dt = lim[Δt→0] (x(t+Δt) - x(t)) / Δt

Discrete (forward difference):
x'[n] ≈ (x[n+1] - x[n]) / Tₛ

Discrete (central difference, more accurate):
x'[n] ≈ (x[n+1] - x[n-1]) / (2·Tₛ)
```

**Application**: Detecting peaks in PPG signal for heart rate:

```python
def derivative(signal, fs):
    """
    Compute derivative of discrete signal
    fs: sampling frequency
    """
    Ts = 1.0 / fs
    deriv = np.zeros(len(signal))

    # Central difference (more accurate)
    for i in range(1, len(signal) - 1):
        deriv[i] = (signal[i+1] - signal[i-1]) / (2 * Ts)

    # Handle endpoints with forward/backward difference
    deriv[0] = (signal[1] - signal[0]) / Ts
    deriv[-1] = (signal[-1] - signal[-2]) / Ts

    return deriv
```

**C Implementation**:
```c
void derivative(float *signal, float *deriv, int n, float fs) {
    float Ts = 1.0f / fs;
    float inv_2Ts = 1.0f / (2.0f * Ts);

    // Central difference for interior points
    for (int i = 1; i < n - 1; i++) {
        deriv[i] = (signal[i+1] - signal[i-1]) * inv_2Ts;
    }

    // Endpoints
    deriv[0] = (signal[1] - signal[0]) / Ts;
    deriv[n-1] = (signal[n-1] - signal[n-2]) / Ts;
}
```

### 2.3 Integrals: Accumulation

The **integral** measures the accumulated area under a signal:

```
Continuous:  ∫x(t)dt

Discrete (rectangular rule):
∫x ≈ Tₛ · Σx[n]

Discrete (trapezoidal rule, more accurate):
∫x ≈ Tₛ · Σ(x[n] + x[n+1])/2
```

**Application**: Computing the area under an ECG wave (QRS area) or total activity counts:

```python
def integrate(signal, fs):
    """
    Numerical integration using trapezoidal rule
    """
    Ts = 1.0 / fs
    integral = 0.0

    for i in range(len(signal) - 1):
        integral += (signal[i] + signal[i+1]) / 2 * Ts

    return integral

# Cumulative integral (running sum)
def cumulative_integral(signal, fs):
    Ts = 1.0 / fs
    result = np.zeros(len(signal))

    for i in range(1, len(signal)):
        result[i] = result[i-1] + (signal[i] + signal[i-1]) / 2 * Ts

    return result
```

### 2.4 Convolution: The Heart of Filtering

**Convolution** combines two signals to produce a third. It's the mathematical operation behind filtering:

```
Continuous:  (x * h)(t) = ∫ x(τ) · h(t - τ) dτ

Discrete:    (x * h)[n] = Σₖ x[k] · h[n - k]
```

**Intuition**: Convolution "slides" one signal over another, computing the similarity (dot product) at each position.

```
Signal x:    [1, 2, 3, 4, 5]
Filter h:    [0.2, 0.6, 0.2] (smoothing filter)

Convolution step by step:
n=0: 0.2×0 + 0.6×0 + 0.2×1 = 0.2
n=1: 0.2×0 + 0.6×1 + 0.2×2 = 1.0
n=2: 0.2×1 + 0.6×2 + 0.2×3 = 2.0
n=3: 0.2×2 + 0.6×3 + 0.2×4 = 3.0
n=4: 0.2×3 + 0.6×4 + 0.2×5 = 4.0
...
```

```python
def convolve(x, h):
    """
    Direct convolution implementation
    """
    N = len(x)
    M = len(h)
    y = np.zeros(N + M - 1)

    for n in range(len(y)):
        for k in range(M):
            if 0 <= n - k < N:
                y[n] += h[k] * x[n - k]

    return y

# Using NumPy (faster)
y = np.convolve(x, h, mode='same')  # 'same' keeps output length = input
```

**C Implementation**:
```c
void convolve(float *x, int nx, float *h, int nh, float *y) {
    int ny = nx + nh - 1;

    // Initialize output to zero
    for (int i = 0; i < ny; i++) {
        y[i] = 0.0f;
    }

    // Convolution
    for (int n = 0; n < ny; n++) {
        for (int k = 0; k < nh; k++) {
            int idx = n - k;
            if (idx >= 0 && idx < nx) {
                y[n] += h[k] * x[idx];
            }
        }
    }
}
```

---

## 3. Complex Numbers and Exponentials

### 3.1 Why Complex Numbers?

Complex numbers provide an elegant way to represent **sinusoids** (waves), which are fundamental to signal analysis.

```
Complex number:  z = a + jb

Where:
- a = real part
- b = imaginary part
- j = √(-1) (i in mathematics, j in engineering)
```

**Polar form** (more useful for signals):
```
z = r · e^(jθ)

Where:
- r = |z| = √(a² + b²) = magnitude
- θ = arctan(b/a) = phase angle
```

### 3.2 Euler's Formula: The Bridge to Sinusoids

**Euler's formula** connects exponentials to sinusoids:

```
e^(jθ) = cos(θ) + j·sin(θ)
```

This means:
```
cos(θ) = (e^(jθ) + e^(-jθ)) / 2
sin(θ) = (e^(jθ) - e^(-jθ)) / (2j)
```

**Why this matters**: The Fourier Transform uses complex exponentials to decompose signals into sinusoids.

### 3.3 Complex Exponentials as Rotating Phasors

A complex exponential e^(jωt) represents a **rotating vector** (phasor):

```
                 Im
                  │
                  │    ╱ e^(jωt) at time t
                  │   ╱
                  │  ╱  ↺ rotates counterclockwise
                  │ ╱    at ω radians/second
       ───────────┼──────────── Re
                  │
                  │
```

**Frequency ω and period T**:
```
ω = 2πf         (angular frequency, radians/second)
f = 1/T         (frequency, Hz)
T = 2π/ω        (period, seconds)
```

### 3.4 Complex Arithmetic in Code

```python
import numpy as np

# Complex number creation
z1 = 3 + 4j           # Rectangular form
z2 = 5 * np.exp(1j * np.pi/4)  # Polar form

# Operations
magnitude = np.abs(z1)      # |z| = 5
phase = np.angle(z1)        # θ = 0.927 rad = 53.1°
conjugate = np.conj(z1)     # 3 - 4j

# Euler's formula
theta = np.pi / 3
euler = np.exp(1j * theta)
print(f"e^(jπ/3) = {euler}")  # 0.5 + 0.866j
print(f"cos(π/3) = {np.cos(theta)}")  # 0.5
print(f"sin(π/3) = {np.sin(theta)}")  # 0.866

# Generate a sinusoid using complex exponentials
fs = 100  # Sampling frequency
t = np.arange(0, 1, 1/fs)  # 1 second
f = 5  # 5 Hz sinusoid
omega = 2 * np.pi * f

# Complex exponential
z = np.exp(1j * omega * t)

# Extract real sinusoid
cosine = np.real(z)
sine = np.imag(z)
```

**C Implementation** (using struct):
```c
typedef struct {
    float real;
    float imag;
} Complex;

Complex complex_multiply(Complex a, Complex b) {
    Complex result;
    result.real = a.real * b.real - a.imag * b.imag;
    result.imag = a.real * b.imag + a.imag * b.real;
    return result;
}

float complex_magnitude(Complex z) {
    return sqrtf(z.real * z.real + z.imag * z.imag);
}

float complex_phase(Complex z) {
    return atan2f(z.imag, z.real);
}

// Euler's formula
Complex complex_exp(float theta) {
    Complex result;
    result.real = cosf(theta);
    result.imag = sinf(theta);
    return result;
}
```

---

## 4. Probability and Statistics

### 4.1 Why Statistics for Biosignals?

Biosignals are **noisy** and **variable**. Statistics helps us:
- Characterize signal properties (mean, variance)
- Distinguish signal from noise
- Make decisions under uncertainty (classification)
- Validate algorithms

### 4.2 Descriptive Statistics

**Mean** (average value):
```
μ = (1/N) Σ x[n]
```

**Variance** (spread of values):
```
σ² = (1/N) Σ (x[n] - μ)²
```

**Standard Deviation**:
```
σ = √(σ²)
```

```python
def signal_statistics(signal):
    """
    Compute basic statistics of a signal
    """
    n = len(signal)

    # Mean
    mean = sum(signal) / n

    # Variance (using n-1 for sample variance)
    variance = sum((x - mean)**2 for x in signal) / (n - 1)

    # Standard deviation
    std = variance ** 0.5

    # Root Mean Square (RMS)
    rms = (sum(x**2 for x in signal) / n) ** 0.5

    return {
        'mean': mean,
        'variance': variance,
        'std': std,
        'rms': rms
    }

# NumPy equivalents
mean = np.mean(signal)
variance = np.var(signal, ddof=1)  # ddof=1 for sample variance
std = np.std(signal, ddof=1)
rms = np.sqrt(np.mean(signal**2))
```

**C Implementation**:
```c
typedef struct {
    float mean;
    float variance;
    float std;
    float rms;
} SignalStats;

SignalStats compute_statistics(float *signal, int n) {
    SignalStats stats;

    // Mean
    float sum = 0.0f;
    for (int i = 0; i < n; i++) {
        sum += signal[i];
    }
    stats.mean = sum / n;

    // Variance
    float var_sum = 0.0f;
    float rms_sum = 0.0f;
    for (int i = 0; i < n; i++) {
        float diff = signal[i] - stats.mean;
        var_sum += diff * diff;
        rms_sum += signal[i] * signal[i];
    }
    stats.variance = var_sum / (n - 1);  // Sample variance
    stats.std = sqrtf(stats.variance);
    stats.rms = sqrtf(rms_sum / n);

    return stats;
}
```

### 4.3 Correlation and Covariance

**Covariance** measures how two signals vary together:
```
Cov(x, y) = (1/N) Σ (x[n] - μₓ)(y[n] - μᵧ)
```

**Correlation coefficient** (normalized covariance):
```
ρ = Cov(x, y) / (σₓ · σᵧ)

Range: -1 ≤ ρ ≤ 1
- ρ = 1:  Perfect positive correlation
- ρ = 0:  No correlation
- ρ = -1: Perfect negative correlation
```

**Application**: Measuring similarity between sensor axes or signal templates.

```python
def correlation_coefficient(x, y):
    """
    Pearson correlation coefficient
    """
    n = len(x)
    mean_x = np.mean(x)
    mean_y = np.mean(y)

    cov = sum((x[i] - mean_x) * (y[i] - mean_y) for i in range(n)) / (n - 1)
    std_x = np.std(x, ddof=1)
    std_y = np.std(y, ddof=1)

    return cov / (std_x * std_y)

# NumPy equivalent
rho = np.corrcoef(x, y)[0, 1]
```

### 4.4 Autocorrelation

**Autocorrelation** measures how a signal correlates with a delayed version of itself:

```
R[k] = Σ x[n] · x[n + k]

k = lag (delay in samples)
```

**Application**: Finding periodicity in signals (e.g., heart rate from PPG):

```python
def autocorrelation(signal):
    """
    Compute autocorrelation for all lags
    """
    n = len(signal)
    # Normalize signal
    signal = signal - np.mean(signal)

    result = np.correlate(signal, signal, mode='full')
    result = result[n-1:]  # Keep positive lags only
    result = result / result[0]  # Normalize

    return result

# Find periodicity (e.g., heart rate)
def find_period(signal, fs, min_bpm=40, max_bpm=200):
    """
    Find heart rate from PPG using autocorrelation
    """
    autocorr = autocorrelation(signal)

    # Convert BPM limits to sample limits
    min_lag = int(fs * 60 / max_bpm)  # Samples for max BPM
    max_lag = int(fs * 60 / min_bpm)  # Samples for min BPM

    # Find peak in valid range
    search_region = autocorr[min_lag:max_lag]
    peak_idx = np.argmax(search_region) + min_lag

    # Convert to BPM
    period = peak_idx / fs  # Period in seconds
    bpm = 60 / period

    return bpm, peak_idx
```

### 4.5 Probability Distributions

**Normal (Gaussian) Distribution**:
```
p(x) = (1 / √(2πσ²)) · exp(-(x - μ)² / (2σ²))

μ = mean
σ = standard deviation
```

**Why Gaussian matters**:
- Many noise sources are approximately Gaussian
- Central Limit Theorem: sum of many random variables → Gaussian
- Optimal filters assume Gaussian noise

```python
import scipy.stats as stats

# Generate Gaussian random numbers
noise = np.random.normal(loc=0, scale=1, size=1000)

# Probability of value
prob = stats.norm.pdf(x=1.5, loc=0, scale=1)

# Cumulative probability (P(X < x))
cum_prob = stats.norm.cdf(x=1.96, loc=0, scale=1)  # ~0.975 (97.5%)
```

### 4.6 Signal-to-Noise Ratio (SNR)

**SNR** measures signal quality:

```
SNR = Signal Power / Noise Power
SNR (dB) = 10 · log₁₀(Signal Power / Noise Power)
         = 20 · log₁₀(Signal Amplitude / Noise Amplitude)
```

```python
def calculate_snr(signal, noise):
    """
    Calculate SNR in dB
    """
    signal_power = np.mean(signal ** 2)
    noise_power = np.mean(noise ** 2)

    snr_linear = signal_power / noise_power
    snr_db = 10 * np.log10(snr_linear)

    return snr_db

# Example: Add noise and measure SNR
clean_signal = np.sin(2 * np.pi * 1 * t)  # 1 Hz sinusoid
noise = np.random.normal(0, 0.1, len(t))
noisy_signal = clean_signal + noise

snr = calculate_snr(clean_signal, noise)
print(f"SNR: {snr:.1f} dB")  # ~20 dB for 0.1 noise amplitude
```

---

## 5. Numerical Methods

### 5.1 Fixed-Point vs. Floating-Point

Embedded systems often use **fixed-point** arithmetic for speed:

```c
// Floating-point (slow on microcontrollers without FPU)
float x = 1.234f;

// Fixed-point (fast integer operations)
// Q15 format: 1 sign bit, 15 fractional bits
// Range: -1.0 to +0.99997 with 0.00003 precision
int16_t x_fixed = (int16_t)(1.234f * 32768);  // 40435

// Converting back
float x_float = x_fixed / 32768.0f;
```

**Common Fixed-Point Formats**:
| Format | Bits | Range | Precision |
|--------|------|-------|-----------|
| Q15 | 16 | -1 to ~1 | 0.00003 |
| Q31 | 32 | -1 to ~1 | 4.7×10⁻¹⁰ |
| Q7.8 | 16 | -128 to ~128 | 0.004 |

```c
// Fixed-point multiplication (Q15)
int16_t fixed_multiply_q15(int16_t a, int16_t b) {
    int32_t result = (int32_t)a * (int32_t)b;
    return (int16_t)(result >> 15);  // Shift back to Q15
}

// Fixed-point filter coefficient
// For coefficient 0.25:
int16_t coeff = (int16_t)(0.25f * 32768);  // 8192

// Apply to signal
int16_t apply_coeff(int16_t x, int16_t coeff) {
    return fixed_multiply_q15(x, coeff);
}
```

### 5.2 Lookup Tables

For expensive functions (sin, cos, sqrt), use **lookup tables**:

```c
// Sine lookup table (256 entries, one quadrant)
// Values in Q15 format
static const int16_t sin_table[256] = {
    0, 201, 402, 603, 804, 1005, 1206, 1406,
    // ... (generated offline)
    32767  // sin(90°) = 1.0 in Q15
};

int16_t fast_sin_q15(uint16_t angle) {
    // angle: 0-65535 maps to 0-360°
    uint8_t quadrant = angle >> 14;  // Top 2 bits
    uint8_t index = (angle >> 6) & 0xFF;  // Next 8 bits

    int16_t value;
    switch (quadrant) {
        case 0: value = sin_table[index]; break;
        case 1: value = sin_table[255 - index]; break;
        case 2: value = -sin_table[index]; break;
        case 3: value = -sin_table[255 - index]; break;
    }
    return value;
}
```

### 5.3 Interpolation

When you need values between samples:

**Linear Interpolation**:
```python
def linear_interpolate(x, y, x_new):
    """
    Interpolate y values at new x positions
    """
    result = []
    for xn in x_new:
        # Find surrounding points
        idx = np.searchsorted(x, xn)
        if idx == 0:
            result.append(y[0])
        elif idx >= len(x):
            result.append(y[-1])
        else:
            # Linear interpolation
            x0, x1 = x[idx-1], x[idx]
            y0, y1 = y[idx-1], y[idx]
            t = (xn - x0) / (x1 - x0)
            result.append(y0 + t * (y1 - y0))
    return np.array(result)
```

**C Implementation**:
```c
float linear_interp(float *x, float *y, int n, float x_new) {
    // Find interval
    int idx = 0;
    while (idx < n - 1 && x[idx + 1] < x_new) {
        idx++;
    }

    if (idx >= n - 1) return y[n - 1];

    // Interpolate
    float t = (x_new - x[idx]) / (x[idx + 1] - x[idx]);
    return y[idx] + t * (y[idx + 1] - y[idx]);
}
```

---

## 6. Exercises

### Exercise 1: Vector Operations
Implement a function to calculate the **magnitude of a 3D acceleration vector**:
```python
def acceleration_magnitude(ax, ay, az):
    """
    Returns |a| = sqrt(ax² + ay² + az²)
    """
    # Your implementation here
    pass

# Test
assert abs(acceleration_magnitude(3, 4, 0) - 5.0) < 0.001
assert abs(acceleration_magnitude(1, 0, 0) - 1.0) < 0.001
```

### Exercise 2: Moving Average Filter
Implement a **N-point moving average** using convolution:
```python
def moving_average(signal, N):
    """
    N-point moving average filter
    Returns filtered signal of same length
    """
    # Your implementation here
    pass

# Test with known result
test_signal = [1, 2, 3, 4, 5]
# 3-point MA: [1, 1.33, 2, 3, 4, 4.33, 5] (before trimming)
```

### Exercise 3: Correlation Analysis
Find the **lag** at which two signals have maximum correlation:
```python
def find_lag(signal1, signal2, max_lag):
    """
    Find lag that maximizes cross-correlation
    """
    # Your implementation here
    pass

# Test: signal2 is signal1 delayed by 10 samples
```

### Exercise 4: SNR Calculation
Given a noisy PPG signal and a clean reference, calculate the **SNR**:
```python
def estimate_snr(noisy_signal, clean_signal):
    """
    Estimate SNR in dB
    """
    # Your implementation here
    pass
```

### Exercise 5: Fixed-Point Conversion
Convert a Butterworth filter coefficient from floating-point to Q15:
```python
# Original coefficient
coeff_float = 0.0675  # Example filter coefficient

# Convert to Q15 (your answer)
coeff_q15 = ???  # Should be 2212

# How much error is introduced?
error = ???
```

---

## 7. Learning Resources

### 7.1 Books

#### Foundational Mathematics
| Book | Author | Level | Focus |
|------|--------|-------|-------|
| **"Linear Algebra Done Right"** | Sheldon Axler | Beginner | Pure linear algebra foundations |
| **"Introduction to Linear Algebra"** | Gilbert Strang | Beginner | Applied linear algebra with examples |
| **"Calculus"** | James Stewart | Beginner | Comprehensive calculus reference |
| **"Probability and Statistics for Engineers"** | Montgomery & Runger | Intermediate | Engineering statistics |

#### Signal Processing Mathematics
| Book | Author | Level | Focus |
|------|--------|-------|-------|
| **"Signals and Systems"** | Oppenheim & Willsky | Intermediate | Classic signals text |
| **"Digital Signal Processing"** | Proakis & Manolakis | Intermediate | DSP fundamentals |
| **"The Scientist and Engineer's Guide to DSP"** | Steven W. Smith | Beginner | Free online, practical |

### 7.2 Online Courses

#### Free Courses
| Course | Platform | Link |
|--------|----------|------|
| **Linear Algebra** | MIT OCW (18.06) | [ocw.mit.edu](https://ocw.mit.edu/courses/mathematics/18-06-linear-algebra-spring-2010/) |
| **Signals and Systems** | MIT OCW (6.003) | [ocw.mit.edu](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-003-signals-and-systems-fall-2011/) |
| **Mathematics for Machine Learning** | Coursera (Imperial) | [coursera.org](https://www.coursera.org/specializations/mathematics-machine-learning) |
| **Essence of Linear Algebra** | 3Blue1Brown | [youtube.com](https://www.youtube.com/playlist?list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab) |

### 7.3 YouTube Channels

| Channel | Focus | Best For |
|---------|-------|----------|
| **3Blue1Brown** | Visual math intuition | Linear algebra, calculus visualization |
| **Steve Brunton** | Data-driven dynamics | FFT, signal processing, ML |
| **Zach Star** | Engineering math | Practical applications |
| **Khan Academy** | All math topics | Foundational review |
| **StatQuest** | Statistics & ML | Probability, distributions |

### 7.4 Interactive Tools

| Tool | Purpose | Link |
|------|---------|------|
| **Desmos** | Graphing calculator | [desmos.com](https://www.desmos.com/calculator) |
| **GeoGebra** | Geometry & algebra | [geogebra.org](https://www.geogebra.org/) |
| **Wolfram Alpha** | Computation engine | [wolframalpha.com](https://www.wolframalpha.com/) |
| **Python + Jupyter** | Coding experiments | [jupyter.org](https://jupyter.org/) |

### 7.5 Practice Problems

| Resource | Type | Link |
|----------|------|------|
| **MIT OpenCourseWare** | Problem sets with solutions | [ocw.mit.edu](https://ocw.mit.edu/) |
| **Khan Academy** | Interactive exercises | [khanacademy.org](https://www.khanacademy.org/) |
| **Brilliant.org** | Guided problem solving | [brilliant.org](https://brilliant.org/) |
| **Project Euler** | Math + programming | [projecteuler.net](https://projecteuler.net/) |

---

## Summary

This tutorial covered the mathematical foundations essential for biosignal processing:

| Topic | Key Concepts | Application |
|-------|--------------|-------------|
| **Linear Algebra** | Vectors, matrices, eigenvalues | Signal representation, PCA |
| **Calculus** | Derivatives, integrals, convolution | Filtering, peak detection |
| **Complex Numbers** | Euler's formula, phasors | Fourier analysis |
| **Statistics** | Mean, variance, correlation | Signal characterization |
| **Numerical Methods** | Fixed-point, lookup tables | Embedded implementation |

**Key Takeaways**:
1. Biosignals are vectors; filtering is matrix multiplication
2. Convolution is the foundation of digital filtering
3. Complex exponentials decompose signals into frequencies
4. Statistics quantify signal quality and uncertainty
5. Embedded systems require numerical approximations

---

**Next Tutorial**: [1.2 Biology of Human Physiology →](02_biology_foundations.md)
