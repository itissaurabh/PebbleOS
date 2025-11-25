# Understanding Health Detection Algorithms: A Complete Tutorial

This tutorial explains the fundamental logic and intuition behind activity, sleep, stress, and cardiac detection algorithms used in wearable devices. It's designed to help you understand **why** these algorithms work, not just **how**.

## Table of Contents

1. [Fundamentals: Sensors and Signals](#1-fundamentals-sensors-and-signals)
2. [Activity Detection: From Motion to Steps](#2-activity-detection-from-motion-to-steps)
3. [Sleep Detection: Understanding Rest States](#3-sleep-detection-understanding-rest-states)
4. [Stress Detection: Reading the Autonomic Nervous System](#4-stress-detection-reading-the-autonomic-nervous-system)
5. [Cardiac Monitoring: ECG and Arrhythmia Detection](#5-cardiac-monitoring-ecg-and-arrhythmia-detection)
6. [Learning Resources](#6-learning-resources)

---

## 1. Fundamentals: Sensors and Signals

### 1.1 The Accelerometer: Your Motion Sensor

**What it measures**: Acceleration in three axes (X, Y, Z) measured in g-forces (1g = 9.8 m/s²)

**The key insight**: When you're stationary, the accelerometer measures gravity (1g pointing down). When you move, it measures gravity PLUS your motion acceleration.

```
At rest (lying flat):     Walking:                Running:
    Z                         Z                       Z
    ↑ (1g)                   ↑ (1g + bounce)         ↑ (1g + big bounce)
    │                        │  ↗                    │    ↗↗
    ●───→ X                  ●───→ X                 ●───→ X
   ╱                        ╱                       ╱
  Y                        Y                       Y

Signal magnitude:         Signal magnitude:        Signal magnitude:
~1g (constant)            ~1g ± 0.3g (rhythmic)    ~1g ± 1.0g (strong rhythm)
```

**Why walking creates a pattern**:
1. Each step creates a vertical impact (heel strike)
2. Your body oscillates up and down rhythmically
3. This creates a periodic signal at your **step frequency** (typically 1-2 Hz)

### 1.2 The Photoplethysmograph (PPG): Your Pulse Sensor

**What it measures**: Changes in blood volume in your tissue using light

**How it works**:
```
    LED (green/red light)
         │
         ↓ light shines into skin
    ┌────────────────┐
    │   Skin         │
    │   ┌──────┐     │
    │   │Blood │ ←── Blood absorbs light
    │   │Vessel│     │
    │   └──────┘     │
    │                │
    └────────────────┘
         ↑
    Photodetector measures reflected light

When heart beats → more blood → more absorption → less reflected light
Between beats    → less blood  → less absorption → more reflected light
```

**The PPG signal**:
```
Reflected     ╭╮    ╭╮    ╭╮    ╭╮    ╭╮
Light      ───╯╰────╯╰────╯╰────╯╰────╯╰───
              │     │     │     │     │
              └──R──┘     └──R──┘     └──R──┘
              R-R interval (time between beats)
```

**Key measurements from PPG**:
- **Heart Rate**: Count peaks per minute
- **HRV (Heart Rate Variability)**: Variation in R-R intervals
- **SpO2**: Ratio of red to infrared light absorption (oxygenated vs deoxygenated blood)

### 1.3 Electrodermal Activity (EDA/GSR): Your Stress Sensor

**What it measures**: Skin electrical conductance, which changes with sweat gland activity

**The autonomic connection**:
```
Stress/Arousal → Sympathetic Nervous System → Sweat Glands Activate
                                                     ↓
                                              Skin gets more conductive
                                                     ↓
                                              EDA signal increases
```

**EDA Signal Components**:
```
EDA Level (μS)
    │
    │      ╭──╮ Phasic response
    │     ╱    ╲ (quick stress response)
    │    ╱      ╲
    │───╱        ╲────────── Tonic level (baseline)
    │
    └──────────────────────→ Time
         Stimulus
```

### 1.4 Electrocardiogram (ECG): Your Heart's Electrical Signal

**What it measures**: Electrical activity of the heart through skin electrodes

**The PQRST Complex**:
```
Voltage
    │     R
    │     ╱╲
    │    ╱  ╲
    │   ╱    ╲
    │  P      ╲   T
    │ ╱╲       ╲ ╱╲
    │╱  ╲   Q   ╲╱  ╲
────┴────╲──┴────────╲─────→ Time
          S

P wave:  Atrial depolarization (atria contract)
QRS:     Ventricular depolarization (ventricles contract)
T wave:  Ventricular repolarization (ventricles reset)
```

**Why single-lead smartwatch ECG works**:
- Measures electrical potential between wrist and fingertip
- Captures the overall direction of electrical activity
- Sufficient for rhythm analysis (regular vs irregular)
- NOT sufficient for detailed morphology analysis (heart attack detection)

---

## 2. Activity Detection: From Motion to Steps

### 2.1 The Intuition Behind Step Detection

**Core insight**: Walking creates a **periodic signal** at a **characteristic frequency**.

Human walking frequency range:
- Slow walk: ~1.0 Hz (60 steps/min, one foot)
- Normal walk: ~1.5 Hz (90 steps/min)
- Fast walk: ~2.0 Hz (120 steps/min)
- Running: ~2.5-3.5 Hz (150-210 steps/min)

### 2.2 Method 1: Peak Detection (Simple)

**Algorithm**:
```
1. Calculate magnitude: |a| = √(ax² + ay² + az²)
2. Apply low-pass filter (remove high-frequency noise)
3. Find peaks above threshold
4. Count peaks = step count

    |a|
     │     *     *     *     *
     │    ╱╲    ╱╲    ╱╲    ╱╲
     │   ╱  ╲  ╱  ╲  ╱  ╲  ╱  ╲
     │──╱────╲╱────╲╱────╲╱────╲── threshold
     │
     └────────────────────────────→ Time
           1     2     3     4  ← step count
```

**Problems**:
- Sensitive to threshold setting
- Fails during arm swing without steps
- Double-counts during running (heel + toe)

### 2.3 Method 2: Frequency Analysis (PebbleOS Approach)

**Core insight**: Instead of counting peaks, find the **dominant frequency** in the walking range.

**Algorithm (Kraepelin)**:
```
1. Collect 5 seconds of data (125 samples at 25Hz)
2. Apply bandpass filter (0.25-1.75 Hz) → isolate walking frequencies
3. Compute FFT (128-point) → convert to frequency domain
4. Find dominant frequency in step range (0.875-2.5 Hz)
5. If energy at dominant frequency > threshold → walking detected
6. Steps = frequency × time

Time Domain:                    Frequency Domain:
    ╭╮  ╭╮  ╭╮  ╭╮             │
    │╲╱╲│╲╱╲│╲╱╲│╲╱╲           │    ╭╮
    │               │    FFT    │   ╱  ╲
    │ messy signal  │  ──────→  │  ╱    ╲
    └───────────────┘           │──╱──────╲────
                                └──────────────→
                                   step freq  Hz
```

**Why FFT works better**:
- Ignores random noise (appears as flat spectrum)
- Extracts rhythmic component (appears as peak)
- Works regardless of signal amplitude
- Robust to sensor orientation

### 2.4 Method 3: Deep Learning (State of the Art)

**Core insight**: Let the algorithm **learn** what walking looks like from data.

**CNN Architecture**:
```
Raw acceleration    Conv1D        Conv1D        Dense      Output
(125 × 3)          (filters)     (filters)     layers
    │                 │             │             │          │
    │   ┌─────────┐   │   ┌─────┐   │   ┌─────┐   │   ┌───┐  │
ax ─┼──→│ 64 ×3×3 │───┼──→│128×3│───┼──→│ 256 │───┼──→│ N │──┼──→ Steps
ay ─┤   └─────────┘   │   └─────┘   │   └─────┘   │   └───┘  │
az ─┤                 │             │             │          │
    │    Learn local  │   Learn     │   Combine   │  Regress │
    │    patterns     │   patterns  │   features  │  steps   │
```

**Why deep learning excels**:
1. **Learns invariances**: Handles different sensor positions, body types
2. **Multi-scale patterns**: Captures both fine (step) and coarse (gait) features
3. **Personalization**: Can fine-tune to individual users
4. **Handles edge cases**: Stairs, uneven terrain, arm movements

---

## 3. Sleep Detection: Understanding Rest States

### 3.1 The Physiology of Sleep

**Sleep Architecture** (one night):
```
Awake ━━━━                              ━━━━━━
REM       ╲    ╱╲    ╱╲    ╱╲    ╱╲   ╱
Light      ╲  ╱  ╲  ╱  ╲  ╱  ╲  ╱  ╲ ╱
Deep        ╲╱    ╲╱    ╲╱    ╲╱    ╲
            └──────────────────────────→
            11pm                    7am

Each cycle: ~90 minutes
Deep sleep: More in first half of night
REM sleep: More in second half of night
```

### 3.2 What Changes During Sleep?

| Parameter | Awake | Light Sleep | Deep Sleep | REM |
|-----------|-------|-------------|------------|-----|
| Movement | High | Low | Very Low | Very Low (paralysis) |
| Heart Rate | Variable | Decreasing | Lowest | Variable (dreams) |
| HRV (RMSSD) | Lower | Increasing | Highest | Variable |
| Breathing | Irregular | Regular | Very Regular | Irregular |
| Body Temp | Normal | Dropping | Lowest | Rising |

### 3.3 Method 1: Actigraphy (Movement Only) - PebbleOS Approach

**Core insight**: Sleep = low movement. Deep sleep = very low movement.

**PebbleOS Sleep Score Calculation**:
```python
def calculate_sleep_score(vmc_history, current_minute):
    """
    VMC = Vector Magnitude Counts (activity intensity)
    Uses 9-minute weighted convolution
    """
    weights = [10, 15, 28, 31, 85, 15, 10, 0, 0]  # Center-weighted

    score = 0
    for i in range(9):
        minute = current_minute - 4 + i
        score += weights[i] * vmc_history[minute]

    return score / 100

# Classification
if sleep_score < 330:
    state = "Sleep"
    if sleep_score < 160:
        state = "Deep Sleep"
else:
    state = "Awake"
```

**Why weighted convolution?**
- Sleep/wake transitions are gradual, not instant
- The center weight (85) emphasizes current minute
- Surrounding weights (10, 15, 28, 31...) smooth transitions
- Prevents flickering between states

**Limitations**:
- Cannot detect REM (no movement difference from deep)
- Misclassifies quiet wakefulness as sleep
- Accuracy: ~85% sleep/wake, ~60% staging

### 3.4 Method 2: HRV-Based Sleep Staging

**Core insight**: Each sleep stage has a distinct **autonomic signature**.

**HRV During Sleep Stages**:
```
           Awake       Light       Deep        REM
RMSSD:     Lower      Medium      Highest     Variable
           │           │           │           │
           ├───────────┼───────────┼───────────┤
     20ms  █████       ██████████  █████████████  ██████████
           │           │           │           │
LF/HF:     │           │           │           │
           │ Higher    │ Medium    │ Lower     │ Higher
           │           │           │           │
HR:        │ Variable  │ Decreasing│ Lowest    │ Variable
```

**Algorithm**:
```python
def classify_sleep_stage(hrv_features, movement):
    """
    Input: HRV features from 5-minute window
    """
    rmssd = hrv_features['rmssd']
    lf_hf = hrv_features['lf_hf_ratio']
    hr = hrv_features['heart_rate']
    motion = movement['vmc']

    if motion > WAKE_THRESHOLD:
        return 'WAKE'

    if rmssd > DEEP_RMSSD_THRESHOLD and lf_hf < DEEP_LFHF_THRESHOLD:
        return 'DEEP'

    if lf_hf > REM_LFHF_THRESHOLD and hr_variability > REM_HRV_THRESHOLD:
        return 'REM'

    return 'LIGHT'
```

### 3.5 Method 3: Deep Learning Sleep Staging

**Architecture** (from research):
```
PPG (25Hz) ──┐
             ├──→ CNN ──→ LSTM ──→ Attention ──→ Dense ──→ 4-class
ACC (25Hz) ──┘   (local   (temporal  (focus on    output
                 features) patterns)  important
                                      moments)
```

**Why combine CNN + LSTM**:
- **CNN**: Extracts features from 30-second epochs
  - Pulse shape variations
  - Movement patterns
  - Breathing-related modulations

- **LSTM**: Captures sleep stage transitions
  - Sleep follows predictable patterns
  - Deep sleep unlikely right after waking
  - REM increases through the night

**Performance**: 76-80% accuracy, κ = 0.62-0.65

---

## 4. Stress Detection: Reading the Autonomic Nervous System

### 4.1 The Stress Response

**Autonomic Nervous System**:
```
                    ┌─────────────────────────────────┐
                    │     Autonomic Nervous System    │
                    └─────────────────────────────────┘
                              ╱           ╲
                    ┌─────────┐           ┌─────────┐
                    │Sympathetic│         │Parasympathetic│
                    │"Fight/Flight"│      │"Rest/Digest"│
                    └─────────┘           └─────────┘
                          │                     │
            ┌─────────────┼─────────────┐       │
            ↓             ↓             ↓       ↓
         Heart rate↑   Sweat↑      Pupils↑   Heart rate↓
         HRV↓           EDA↑       Alert↑    HRV↑
         BP↑                                 Digestion↑
```

### 4.2 HRV Features for Stress Detection

**Time Domain Features**:

```
R-R Intervals: 800ms, 850ms, 780ms, 920ms, 810ms

SDNN = Standard Deviation of all intervals
     = σ([800, 850, 780, 920, 810])
     = 52ms (higher = healthier, less stressed)

RMSSD = Root Mean Square of Successive Differences
      = √(mean([50², 70², 140², 110²]))
      = √(mean([2500, 4900, 19600, 12100]))
      = √(9775) = 99ms (higher = more parasympathetic)

pNN50 = % of differences > 50ms
      = 3/4 = 75% (higher = more parasympathetic)
```

**Frequency Domain Features**:
```
       Power
         │
         │    LF            HF
         │   ╱╲            ╱╲
         │  ╱  ╲          ╱  ╲
         │ ╱    ╲        ╱    ╲
         │╱      ╲______╱      ╲
         └─────────────────────────→
           0.04  0.15    0.15  0.4  Hz

LF (0.04-0.15 Hz): Mixed sympathetic/parasympathetic
HF (0.15-0.4 Hz):  Parasympathetic (vagal) activity

LF/HF Ratio:
- Relaxed: ~1.0 (balanced)
- Stressed: >2.0 (sympathetic dominant)
- Deep relaxation: <0.5 (parasympathetic dominant)
```

### 4.3 EDA for Stress Detection

**Why EDA is powerful for stress**:
- Sweat glands ONLY controlled by sympathetic nervous system
- No parasympathetic influence = direct stress indicator
- Fast response time (~1-3 seconds)

**Feature Extraction**:
```python
def extract_eda_features(eda_signal, sampling_rate=4):
    """
    EDA typically sampled at 4 Hz (adequate for slow changes)
    """
    # Decompose into tonic and phasic
    tonic, phasic = decompose_eda(eda_signal)

    # Tonic features (baseline stress level)
    mean_scl = np.mean(tonic)
    scl_slope = linear_regression_slope(tonic)

    # Phasic features (stress responses)
    peaks = find_peaks(phasic, threshold=0.01)
    scr_count = len(peaks)
    scr_amplitude = np.mean([phasic[p] for p in peaks]) if peaks else 0

    return {
        'mean_scl': mean_scl,        # Baseline conductance
        'scl_slope': scl_slope,      # Increasing = building stress
        'scr_count': scr_count,      # Number of stress responses
        'scr_amplitude': scr_amplitude  # Intensity of responses
    }
```

### 4.4 Multi-Modal Stress Detection

**Best practice**: Combine HRV + EDA for highest accuracy

```python
def detect_stress_level(hrv_features, eda_features):
    """
    Combine multiple physiological indicators
    """
    # Normalize features to 0-1 scale
    stress_indicators = [
        1 - normalize(hrv_features['rmssd'], 20, 100),  # Low RMSSD = stress
        normalize(hrv_features['lf_hf'], 0.5, 3.0),    # High LF/HF = stress
        normalize(eda_features['mean_scl'], 1, 20),     # High SCL = stress
        normalize(eda_features['scr_count'], 0, 10),    # Many SCRs = stress
    ]

    # Weighted combination
    weights = [0.25, 0.25, 0.30, 0.20]  # EDA weighted slightly higher
    stress_score = sum(w * s for w, s in zip(weights, stress_indicators))

    return stress_score  # 0 = relaxed, 1 = very stressed
```

---

## 5. Cardiac Monitoring: ECG and Arrhythmia Detection

### 5.1 Normal Heart Rhythm

**Sinus Rhythm Characteristics**:
```
1. Regular R-R intervals (±10% variation)
2. P wave before every QRS
3. Consistent P-R interval (120-200ms)
4. Narrow QRS complex (<120ms)

    Normal Sinus Rhythm:
    ─P─┬─Q─R─S─┬─T──────P─┬─Q─R─S─┬─T──────
       │       │          │       │
       └───────┘          └───────┘
        ~800ms             ~800ms  (consistent)
```

### 5.2 Atrial Fibrillation (AFib)

**What goes wrong**:
- Atria fire chaotically (300-600 times/min)
- AV node filters randomly → irregular ventricular response
- No organized atrial contraction

```
Atrial Fibrillation:
─f─f─f─f─┬─Q─R─S─f─f─f─f─f─┬─Q─R─S─f─f─f─f─f─f─┬─Q─R─S─
         │                 │                   │
         └─────────────────┘                   │
              1200ms                           │
                           └───────────────────┘
                                 650ms  (irregular!)

f = fibrillatory waves (chaotic atrial activity)
No clear P waves
Irregularly irregular R-R intervals
```

### 5.3 AFib Detection Algorithm

**Method 1: Statistical Analysis**
```python
def detect_afib(rr_intervals):
    """
    AFib characterized by "irregularly irregular" rhythm
    """
    # Calculate variability metrics
    rmssd = calculate_rmssd(rr_intervals)

    # Poincaré plot analysis
    # Plot each RR against the next RR
    # AFib shows scattered, non-linear pattern
    sd1, sd2 = poincare_analysis(rr_intervals)

    # Turning point ratio
    # Count direction changes in RR sequence
    # Random (AFib) has more turning points than regular
    tpr = calculate_turning_point_ratio(rr_intervals)

    # Shannon entropy
    # Higher entropy = more randomness = more likely AFib
    entropy = calculate_entropy(rr_intervals)

    # Classification
    afib_score = (
        0.3 * normalize(rmssd, 20, 100) +
        0.3 * normalize(entropy, 0.5, 2.0) +
        0.2 * normalize(sd1/sd2, 0.5, 2.0) +
        0.2 * normalize(tpr, 0.4, 0.7)
    )

    return afib_score > 0.6  # True if AFib likely
```

**Method 2: Deep Learning**
```
30-second ECG segment (200 Hz = 6000 samples)
    ↓
Preprocessing (bandpass 0.5-40 Hz, normalize)
    ↓
┌────────────────────────────────────────────┐
│  33-layer ResNet-style CNN                 │
│  - Convolutional blocks                    │
│  - Residual connections                    │
│  - Batch normalization                     │
│  - Dropout                                 │
└────────────────────────────────────────────┘
    ↓
Output: [Normal, AFib, Other rhythms...]
```

### 5.4 Ventricular Fibrillation (V-Fib)

**What it looks like**:
```
Normal:          V-Fib (lethal arrhythmia):

  R                ╱╲  ╱╲╱╲  ╱╲
 ╱╲   R           ╱  ╲╱    ╲╱  ╲╱╲
╱  ╲ ╱╲          ╱              ╱╲╱
    ╲╱  ╲       ╱                  ╲
         ╲

Regular, organized       Chaotic, no identifiable QRS
                         Frequency: 3-7 Hz
                         No effective heart pumping
```

**Detection (research-grade)**:
```python
def detect_vfib(ecg_segment, sampling_rate=200):
    """
    V-Fib detection typically NOT in consumer devices
    Requires high reliability due to life-threatening nature
    """
    # Spectral analysis
    spectrum = np.fft.fft(ecg_segment)
    freqs = np.fft.fftfreq(len(ecg_segment), 1/sampling_rate)

    # V-Fib has dominant frequency 3-7 Hz
    vfib_band_power = np.sum(np.abs(spectrum[(freqs > 3) & (freqs < 7)])**2)
    total_power = np.sum(np.abs(spectrum)**2)

    # Sample entropy (V-Fib is chaotic but not random)
    samp_entropy = calculate_sample_entropy(ecg_segment)

    # Threshold detector
    # Phase space analysis, complexity measures, etc.

    return is_vfib_likely
```

**Why consumer devices don't detect V-Fib**:
1. Requires continuous monitoring (battery constraint)
2. Single lead insufficient for reliable detection
3. False alarms could cause panic
4. Requires immediate medical intervention
5. Medical device regulatory requirements are extreme

---

## 6. Learning Resources

### 6.1 Books

#### Signal Processing & Biosignals
- **"Biomedical Signal Processing and Signal Modeling"** by Eugene N. Bruce
  - Comprehensive foundation in biosignal analysis
  - ECG, EEG, EMG processing techniques

- **"ECG Signal Processing, Classification and Interpretation"** by Adam Gacek
  - Deep dive into ECG analysis algorithms
  - Machine learning for cardiac signals

- **"Heart Rate Variability"** by Marek Malik (Task Force Standard)
  - The definitive reference for HRV analysis
  - Time domain, frequency domain, nonlinear methods

#### Machine Learning for Healthcare
- **"Deep Learning for Medical Image Analysis"** by Kevin Zhou
  - Neural network architectures for medical signals
  - Transfer learning, data augmentation

- **"Machine Learning for Healthcare"** by Marzyeh Ghassemi et al.
  - MIT course materials available online
  - Clinical applications of ML

### 6.2 Online Courses

#### Free Courses
| Course | Platform | Topics |
|--------|----------|--------|
| [Foundations of HRV](https://elitehrv.com/academy/foundations-of-hrv) | Elite HRV | HRV fundamentals, ANS, stress |
| [HRV Basics](https://www.heartmath.org/resources/courses/hrv/) | HeartMath | Physiological mechanisms |
| [Deep Learning Specialization](https://www.coursera.org/specializations/deep-learning) | Coursera | Neural networks, CNNs, RNNs |
| [MIT 6.S897 Machine Learning for Healthcare](https://ocw.mit.edu/) | MIT OCW | Clinical ML applications |

#### Paid Courses
| Course | Platform | Topics |
|--------|----------|--------|
| AI for Medicine | Coursera (DeepLearning.AI) | Diagnosis, prognosis, treatment |
| Wearable Technologies | edX | Sensor fundamentals, signal processing |

### 6.3 YouTube Channels & Videos

#### Signal Processing
- **Steve Brunton** - [Data-Driven Science & Engineering](https://www.youtube.com/c/Eigensteve)
  - FFT tutorials, signal processing fundamentals
  - Excellent mathematical intuition

- **3Blue1Brown** - [Fourier Transform Visualized](https://www.youtube.com/watch?v=spUNpyF58BY)
  - Beautiful visualization of FFT concepts

- **StatQuest with Josh Starmer** - [Machine Learning Fundamentals](https://www.youtube.com/c/joshstarmer)
  - Clear explanations of ML algorithms
  - Neural networks, decision trees, etc.

#### Biomedical Signals
- **MIT OpenCourseWare** - Biomedical Signal Processing
  - Academic lectures on biosignal analysis

- **Khan Academy** - Circulatory System
  - Heart physiology for understanding ECG

### 6.4 Research Papers (Must-Read)

#### Activity Recognition
1. **"Deep Learning for Sensor-based Activity Recognition: A Survey"** (2017)
   - Comprehensive review of DL methods for HAR
   - [Link](https://arxiv.org/abs/1707.03502)

2. **"Self-Supervised Machine Learning for Step Counting"** (2024)
   - UK Biobank large-scale validation
   - [PMC Link](https://pmc.ncbi.nlm.nih.gov/articles/PMC11402590/)

#### Sleep Staging
3. **"Deep Learning for Sleep Staging from PPG"** (2020)
   - CNN-RNN architecture for wearables
   - [Nature Digital Medicine](https://www.nature.com/articles/s41746-020-00331-1)

4. **"Validation of Wearable Sleep Trackers"** (2024)
   - Commercial device accuracy comparison
   - [SLEEP Advances](https://academic.oup.com/sleepadvances/article/6/2/zpaf021/8090472)

#### Stress Detection
5. **"State-of-the-Art of Stress Prediction from HRV Using AI"** (2023)
   - Comprehensive review of ML methods
   - [Springer](https://link.springer.com/article/10.1007/s12559-023-10200-0)

6. **"Detection and Monitoring of Stress Using Wearables"** (2024)
   - Systematic review of wearable stress detection
   - [Frontiers](https://www.frontiersin.org/journals/computer-science/articles/10.3389/fcomp.2024.1478851/full)

#### ECG/Arrhythmia
7. **"Cardiologist-Level Arrhythmia Detection"** (2019)
   - Stanford's 33-layer CNN for ECG
   - [arXiv](https://arxiv.org/abs/1707.01836)

8. **"Apple Watch ECG Meta-Analysis"** (2024)
   - Diagnostic accuracy for AFib
   - [JACC Advances](https://www.jacc.org/doi/10.1016/j.jacadv.2024.101538)

### 6.5 Open Source Code & Datasets

#### Datasets
| Dataset | Description | Link |
|---------|-------------|------|
| UCI HAR | Human Activity Recognition | [UCI ML Repository](https://archive.ics.uci.edu/ml/datasets/human+activity+recognition+using+smartphones) |
| WESAD | Wearable Stress & Affect | [UCI](https://archive.ics.uci.edu/ml/datasets/WESAD+%28Wearable+Stress+and+Affect+Detection%29) |
| Sleep-EDF | Sleep polysomnography | [PhysioNet](https://physionet.org/content/sleep-edfx/) |
| MIT-BIH | Arrhythmia database | [PhysioNet](https://physionet.org/content/mitdb/) |

#### Code Libraries
| Library | Language | Purpose |
|---------|----------|---------|
| [NeuroKit2](https://github.com/neuropsychology/NeuroKit) | Python | Biosignal processing, HRV |
| [HeartPy](https://github.com/paulvangentcom/heartrate_analysis_python) | Python | Heart rate analysis |
| [pyEDA](https://github.com/HealthSciTech/pyEDA) | Python | EDA signal processing |
| [BioSPPy](https://github.com/PIA-Group/BioSPPy) | Python | Biosignal processing |
| [WFDB](https://github.com/MIT-LCP/wfdb-python) | Python | PhysioNet data tools |

#### GitHub Projects
- [LSTM-Human-Activity-Recognition](https://github.com/guillaume-chevalier/LSTM-Human-Activity-Recognition)
  - Complete TensorFlow implementation for HAR

- [awesome-gsr](https://github.com/mintisan/awesome-gsr)
  - Curated list of GSR/EDA resources

### 6.6 Blogs & Websites

| Resource | Focus | URL |
|----------|-------|-----|
| Elite HRV Knowledge Base | HRV education | [help.elitehrv.com](https://help.elitehrv.com) |
| HeartMath Research | HRV science | [heartmath.org/research](https://www.heartmath.org/research/) |
| Kubios Blog | HRV analysis methods | [kubios.com/blog](https://www.kubios.com/blog/) |
| Machine Learning Mastery | ML tutorials | [machinelearningmastery.com](https://machinelearningmastery.com) |
| PhysioNet | Biosignal databases | [physionet.org](https://physionet.org/) |
| Oura Blog | Sleep science | [ouraring.com/blog](https://ouraring.com/blog/) |

### 6.7 Tools for Experimentation

| Tool | Purpose | Cost |
|------|---------|------|
| Kubios HRV | HRV analysis software | Free (limited) |
| NeuroKit2 | Python biosignal processing | Free |
| MATLAB | Signal processing | Paid |
| iMotions | Multi-modal biosignal platform | Enterprise |
| Empatica E4 | Research wearable (EDA, PPG, ACC) | ~$1,690 |

---

## Summary

Understanding health detection algorithms requires knowledge across multiple domains:

1. **Sensor Physics**: How accelerometers, PPG, EDA, and ECG sensors work
2. **Physiology**: What happens in the body during activity, sleep, stress
3. **Signal Processing**: Filtering, FFT, feature extraction
4. **Machine Learning**: From decision trees to deep neural networks
5. **Validation**: Understanding accuracy metrics and limitations

**Key Takeaways**:
- Simple algorithms (peak detection, thresholds) work but have limitations
- Modern approaches combine multiple signals (PPG + accelerometer)
- Deep learning provides best accuracy but requires more resources
- Real-world performance is always lower than lab performance
- Understanding the physiology helps interpret algorithm failures

**PebbleOS Context**:
- Uses proven, efficient algorithms optimized for low power
- Kraepelin algorithm is sophisticated for its resource constraints
- Missing features (stress, sleep staging) could be added with HRM data
- Open-source nature allows community improvements

---

## Quick Reference Card

```
┌─────────────────────────────────────────────────────────────┐
│                  DETECTION QUICK REFERENCE                  │
├─────────────────────────────────────────────────────────────┤
│ STEPS:     Acc → Filter (0.5-3Hz) → FFT → Peak @ walk freq  │
│ ACTIVITY:  Acc → Features → ML Classifier → Activity class  │
│ SLEEP:     Movement (+ HRV) → Thresholds → Wake/Light/Deep  │
│ STRESS:    HRV (↓RMSSD, ↑LF/HF) + EDA (↑SCL) → Score        │
│ AFIB:      ECG RR intervals → Irregularity metrics → Detect │
├─────────────────────────────────────────────────────────────┤
│ Key frequencies:  Walking: 1-2 Hz | HRV LF: 0.04-0.15 Hz    │
│                   HRV HF: 0.15-0.4 Hz | VFib: 3-7 Hz        │
├─────────────────────────────────────────────────────────────┤
│ Best accuracy:    Steps: CNN-LSTM (99%)                     │
│                   Sleep: PPG+Acc DL (80%)                   │
│                   Stress: HRV+EDA ML (95%)                  │
│                   AFib: ECG+DL (95%)                        │
└─────────────────────────────────────────────────────────────┘
```
