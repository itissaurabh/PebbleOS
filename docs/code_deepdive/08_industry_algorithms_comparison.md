# Industry Algorithms for Activity, Sleep, and Stress Detection

This guide covers the state-of-the-art algorithms used in the wearable industry for activity tracking, sleep monitoring, and stress detection, comparing them with PebbleOS implementations.

## Table of Contents

1. [Activity Recognition Algorithms](#1-activity-recognition-algorithms)
2. [Sleep Stage Detection](#2-sleep-stage-detection)
3. [Stress Detection](#3-stress-detection)
4. [ECG and Arrhythmia Detection](#4-ecg-and-arrhythmia-detection)
5. [Commercial Device Comparison](#5-commercial-device-comparison)
6. [Accuracy Benchmarks](#6-accuracy-benchmarks)

---

## 1. Activity Recognition Algorithms

### 1.1 Traditional Signal Processing Methods

#### Peak Detection (Used by PebbleOS)
PebbleOS uses the **Kraepelin Algorithm** which employs FFT-based peak detection:
- 128-point FFT on 5-second epochs (125 samples at 25Hz)
- Butterworth bandpass filter (0.25-1.75 Hz)
- Detection of dominant frequency in the walking range (0.875-2.5 Hz)

**Accuracy**: ~85-90% in controlled conditions

#### Threshold-Based Methods
Simple algorithms that detect steps when acceleration magnitude crosses a threshold:
```
if |a| > threshold AND time_since_last_step > min_step_interval:
    step_count++
```

**Pros**: Low computational cost, real-time capable
**Cons**: Poor accuracy during varied activities, sensitive to sensor placement

#### Zero-Crossing Detection
Counts the number of times the acceleration signal crosses zero:
- Pre-filter with bandpass (0.5-3 Hz)
- Count zero crossings in vertical axis
- Apply debouncing to prevent double-counting

### 1.2 Machine Learning Approaches

#### Random Forest / Decision Trees
- Extract time-domain features (mean, std, max, min, energy)
- Extract frequency-domain features (dominant frequency, spectral entropy)
- Train classifier on labeled activity data

**Typical Features**:
| Feature | Description |
|---------|-------------|
| Mean | Average acceleration per axis |
| Standard Deviation | Variability measure |
| Signal Magnitude Area (SMA) | Sum of absolute values |
| Energy | Sum of squared FFT components |
| Correlation | Inter-axis correlation |

#### Support Vector Machines (SVM)
- Maps features to high-dimensional space
- Finds optimal hyperplane separating activities
- Kernel functions: RBF, polynomial

**Accuracy**: 90-95% on benchmark datasets

### 1.3 Deep Learning Methods (State of the Art)

#### Convolutional Neural Networks (CNN)
Modern approaches use 1D or 2D CNNs directly on raw sensor data:

```
Input: Raw accelerometer/gyroscope time series
    ↓
Conv1D layers (extract local patterns)
    ↓
Pooling layers (reduce dimensionality)
    ↓
Fully connected layers
    ↓
Softmax output (activity classes)
```

**Architecture Example** (from [Stanford HAR research](https://github.com/guillaume-chevalier/LSTM-Human-Activity-Recognition)):
- 3-5 convolutional layers
- Filter sizes: 64, 128, 256
- Kernel size: 3-5 samples
- Batch normalization + dropout

#### LSTM / Recurrent Neural Networks
Capture temporal dependencies in activity sequences:

```
Input: Windowed sensor data (2.56s windows, 50% overlap)
    ↓
LSTM layers (capture sequential patterns)
    ↓
Dense layers
    ↓
Activity classification
```

**Accuracy**: 91-93% on UCI HAR dataset

#### CNN-LSTM Hybrid (Best Performance)
Combines spatial feature extraction with temporal modeling:

1. **CNN layers**: Extract local features from sensor data
2. **LSTM layers**: Model temporal dependencies
3. **Attention mechanism**: Focus on important time steps

**Recent Results** ([2024 research](https://pmc.ncbi.nlm.nih.gov/articles/PMC9252338/)):
- 99.93% accuracy on H-Activity dataset
- 98.76% on MHEALTH dataset
- 93.11% on UCI-HAR dataset

#### Transformer-Based Models
Using self-attention for activity recognition:
- Convert sensor data to 2D recurrence plots
- Use pre-trained vision transformers (ViT, Swin)
- Fine-tune on activity recognition

### 1.4 Step Counting Accuracy Comparison

| Method | Accuracy | Computational Cost |
|--------|----------|-------------------|
| Simple threshold | 70-80% | Very Low |
| Peak detection (PebbleOS) | 85-90% | Low |
| Machine learning | 90-95% | Medium |
| Deep learning (CNN) | 96-99% | High |
| CNN-LSTM hybrid | 97-99% | Very High |

**Reference**: [UK Biobank step counting study](https://pmc.ncbi.nlm.nih.gov/articles/PMC11402590/) achieved 12.5% mean absolute error vs 65-231% for traditional methods.

---

## 2. Sleep Stage Detection

### 2.1 Sleep Stage Overview

| Stage | Characteristics | Duration |
|-------|----------------|----------|
| **Wake** | High movement, irregular HR | Variable |
| **Light (N1/N2)** | Decreased HR, low movement | 50-60% of night |
| **Deep (N3/SWS)** | Very low HR, minimal movement | 15-20% of night |
| **REM** | Irregular HR, eye movement, paralysis | 20-25% of night |

### 2.2 Actigraphy-Based Methods (Used by PebbleOS)

PebbleOS uses **VMC-based sleep detection**:

```c
// 9-minute weighted convolution filter
const int weights[9] = {10, 15, 28, 31, 85, 15, 10, 0, 0};
sleep_score = Σ(weights[i] × VMC[t-4+i]) / 100;

// Classification
if (sleep_score < 330) → Sleep
if (sleep_score < 160) → Deep Sleep
```

**Limitations**:
- Cannot distinguish REM from light sleep
- Based only on movement (no cardiac data)
- Accuracy: ~85% for sleep/wake, ~60% for stages

### 2.3 PPG + Accelerometer Methods (Industry Standard)

Modern wearables combine **heart rate variability (HRV)** with movement:

#### HRV Features for Sleep Staging

| Feature | Wake | Light | Deep | REM |
|---------|------|-------|------|-----|
| Heart Rate | High/Variable | Moderate | Low | Variable |
| RMSSD | Low | Medium | High | Variable |
| LF/HF Ratio | High | Medium | Low | High |
| Movement | High | Low | Very Low | Very Low |

#### Algorithm Pipeline

```
PPG Signal → Beat Detection → RR Intervals → HRV Features
                                    ↓
Accelerometer → Movement Features →  Feature Fusion
                                    ↓
                              Sleep Stage Classifier
                                    ↓
                              (Wake/Light/Deep/REM)
```

### 2.4 Deep Learning Sleep Staging

#### CNN-RNN Architecture ([Nature Digital Medicine](https://www.nature.com/articles/s41746-021-00510-8))

```
Raw PPG (25Hz) + Accelerometer (25Hz)
    ↓
1D CNN (feature extraction)
    ↓
Bidirectional LSTM (temporal modeling)
    ↓
Dense + Softmax
    ↓
4-class sleep stage prediction
```

**Performance**:
- Accuracy: 76-80%
- Cohen's κ: 0.62-0.65
- Best for: Wake, Deep sleep
- Challenging: N1 vs N2, Light vs REM

#### Transfer Learning Approach
1. Pre-train on large ECG polysomnography dataset
2. Transfer weights to PPG-based model
3. Fine-tune on wearable data

### 2.5 Polysomnography (Gold Standard)

PSG measures multiple signals:
- **EEG**: Brain electrical activity (defines sleep stages)
- **EOG**: Eye movements (REM detection)
- **EMG**: Muscle tone
- **ECG**: Heart activity
- **Respiratory**: Breathing patterns

**Inter-rater reliability**: Human experts agree ~80-85% of the time

### 2.6 Sleep Detection Accuracy by Device

| Device | Sleep/Wake | 4-Stage κ | Deep Sleep Sensitivity |
|--------|------------|-----------|------------------------|
| Oura Ring Gen 3 | 95%+ | 0.65 | 79.5% |
| Apple Watch S8 | 93% | 0.53-0.60 | 50.5% |
| Fitbit Sense | 95%+ | 0.42-0.55 | 61.7% |
| Garmin | 98% | 0.25 | ~60% |
| PebbleOS (VMC only) | ~85% | N/A | ~65% |

**Source**: [2024 Brigham and Women's Hospital study](https://www.mdpi.com/1424-8220/24/20/6532)

---

## 3. Stress Detection

### 3.1 Physiological Markers of Stress

| Signal | Stress Response | Sensor |
|--------|-----------------|--------|
| Heart Rate | Increases | PPG/ECG |
| HRV (RMSSD) | Decreases | PPG/ECG |
| Skin Conductance | Increases | EDA/GSR |
| Skin Temperature | Decreases (periphery) | Thermistor |
| Respiratory Rate | Increases | Accelerometer/PPG |

### 3.2 HRV-Based Stress Detection

#### Time Domain Features

| Feature | Formula | Stress Indication |
|---------|---------|-------------------|
| **SDNN** | σ(RR intervals) | Lower = more stress |
| **RMSSD** | √(mean(ΔRR²)) | Lower = more stress |
| **pNN50** | % of ΔRR > 50ms | Lower = more stress |

#### Frequency Domain Features

| Band | Frequency | Interpretation |
|------|-----------|----------------|
| **VLF** | 0-0.04 Hz | Thermoregulation, hormones |
| **LF** | 0.04-0.15 Hz | Mixed sympathetic/parasympathetic |
| **HF** | 0.15-0.4 Hz | Parasympathetic (vagal) activity |
| **LF/HF** | Ratio | Sympathovagal balance |

**Stress Detection Rule**:
```
High stress indicators:
- Elevated LF/HF ratio (>2.0)
- Reduced RMSSD (<20ms)
- Reduced HF power
- Elevated heart rate
```

### 3.3 EDA/GSR-Based Stress Detection

Electrodermal Activity measures skin conductance changes due to sweat gland activity:

#### Signal Components

```
EDA Signal
    ├── Tonic (SCL): Slow baseline changes
    │   └── Features: Mean level, drift rate
    │
    └── Phasic (SCR): Rapid responses to stimuli
        └── Features: Peak count, amplitude, rise time
```

#### Feature Extraction

| Feature | Description |
|---------|-------------|
| Mean SCL | Average skin conductance level |
| SCR Count | Number of phasic responses |
| SCR Amplitude | Height of phasic peaks |
| Rise Time | Time to peak |
| Recovery Time | Time to return to baseline |

### 3.4 Machine Learning for Stress Detection

#### Best Performing Approaches

| Algorithm | Accuracy | Signals Used |
|-----------|----------|--------------|
| Random Forest | 88-98% | HRV + EDA |
| SVM | 85-95% | HRV + EDA |
| KNN | 92-96% | HRV + EDA |
| CNN | 90-95% | Raw PPG + EDA |
| LSTM | 88-93% | Time series HRV |

**Key Finding**: Studies achieving >95% accuracy consistently use **both EDA and HRV** features. Using only HRV typically caps accuracy at ~86%.

#### Feature Importance (from [2024 meta-analysis](https://www.frontiersin.org/journals/computer-science/articles/10.3389/fcomp.2024.1478851/full))

1. EDA mean level (most important)
2. Heart rate
3. RMSSD
4. LF/HF ratio
5. Skin temperature

### 3.5 Commercial Stress Detection

| Device | Method | Availability |
|--------|--------|--------------|
| Fitbit Sense/2 | EDA sensor + HRV | Yes |
| Apple Watch | HRV only | Limited |
| Garmin | HRV "Body Battery" | Yes |
| Samsung Galaxy Watch | HRV + EDA | Yes |
| Oura Ring | HRV only | Yes |
| **PebbleOS** | **Not implemented** | No |

**Note**: PebbleOS has GSR hardware capability but stress detection is not implemented in software.

### 3.6 Challenges in Real-World Stress Detection

1. **Motion artifacts**: Movement corrupts PPG and EDA signals
2. **Individual variability**: Baseline HRV varies 10x between people
3. **Context dependency**: Same physiological state can be excitement or stress
4. **Sensor contact**: EDA requires good skin contact

---

## 4. ECG and Arrhythmia Detection

### 4.1 How Smartwatch ECG Works

Modern smartwatches (Apple Watch, Samsung Galaxy Watch, Fitbit Sense) can record single-lead ECG:

```
       ┌─────────────────────────────────────┐
       │         Smartwatch Back             │
       │    ┌─────────────────────────┐      │
       │    │   Electrode (negative)  │      │
       │    └─────────────────────────┘      │
       │              ↓                       │
       │         Heart signal                │
       │              ↓                       │
       │    ┌─────────────────────────┐      │
       │    │   Digital Crown/Button  │ ←── Finger touch (positive)
       │    │      (electrode)        │      │
       │    └─────────────────────────┘      │
       └─────────────────────────────────────┘

Lead I equivalent: measures electrical potential between
wrist (negative) and finger (positive)
```

### 4.2 Atrial Fibrillation Detection

#### Algorithm Approaches

**1. Rhythm-Based Detection (Most Common)**
```
Normal sinus rhythm:
  R-R intervals are regular (±10%)

Atrial fibrillation:
  R-R intervals are irregularly irregular
  No discernible P waves
```

**Detection Algorithm**:
```python
def detect_afib(rr_intervals):
    # Calculate RR variability metrics
    rmssd = calculate_rmssd(rr_intervals)
    pnn50 = calculate_pnn50(rr_intervals)

    # Shannon entropy of RR intervals
    entropy = calculate_entropy(rr_intervals)

    # Poincaré plot analysis
    sd1, sd2 = poincare_analysis(rr_intervals)

    # Turning point ratio
    tpr = turning_point_ratio(rr_intervals)

    # Classification threshold
    if entropy > threshold and tpr < threshold:
        return "AFib likely"
```

**2. Deep Learning Detection** ([Stanford model](https://stanfordmlgroup.github.io/projects/ecg/))
- 33-layer CNN architecture
- Input: 30-second ECG at 200Hz
- Output: 14 rhythm classes
- Performance: Cardiologist-level accuracy

### 4.3 Apple Watch AFib Detection Performance

From [2024 meta-analysis](https://www.jacc.org/doi/10.1016/j.jacadv.2024.101538) (11 studies, 4,241 participants):

| Metric | ECG Feature | PPG (IRN) Feature |
|--------|-------------|-------------------|
| Sensitivity | 94.8% | 21.4% |
| Specificity | 95.0% | 100% |
| AUC | 0.96 | - |

**Key Insight**: The ECG feature is highly accurate, but the passive irregular rhythm notification (PPG-based) has low sensitivity - it shouldn't be relied upon to rule out AFib.

### 4.4 Ventricular Fibrillation Detection

V-fib detection is typically NOT implemented in consumer smartwatches due to:
1. Single-lead limitation (V-fib detection typically requires multiple leads)
2. Requires continuous monitoring (battery constraints)
3. Medical device regulatory requirements
4. Life-threatening nature requires higher certainty

**Clinical V-fib Detection**:
```
Characteristics:
- No identifiable QRS complexes
- Chaotic, irregular waveform
- Rate: 150-500 bpm (fibrillatory waves)
- Amplitude: Variable, often decreasing over time
```

**Research Approaches**:
- Spectral analysis (dominant frequency 3-7 Hz)
- Complexity measures (sample entropy)
- Deep learning on ECG morphology

### 4.5 Deep Learning for Arrhythmia Classification

#### State-of-the-Art Architecture

```
Raw ECG (single lead, 200-500 Hz)
    ↓
Preprocessing (bandpass filter, baseline removal)
    ↓
1D CNN layers (32→64→128→256 filters)
    ↓
Residual connections
    ↓
BiLSTM layer (temporal patterns)
    ↓
Attention mechanism
    ↓
Dense layers
    ↓
Multi-class arrhythmia output
```

**Performance** ([2024 review](https://pmc.ncbi.nlm.nih.gov/articles/PMC10542398/)):
- 5-class arrhythmia: 99%+ accuracy
- 14-class arrhythmia: 95%+ accuracy
- F1 scores: >99% for major arrhythmias

---

## 5. Commercial Device Comparison

### 5.1 Sensor Capabilities

| Device | Accelerometer | Gyroscope | PPG | ECG | EDA | SpO2 | Temp |
|--------|--------------|-----------|-----|-----|-----|------|------|
| Apple Watch S9 | ✓ | ✓ | ✓ | ✓ | ✗ | ✓ | ✓ |
| Fitbit Sense 2 | ✓ | ✗ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Samsung Galaxy Watch 6 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Garmin Venu 3 | ✓ | ✓ | ✓ | ✗ | ✗ | ✓ | ✗ |
| Oura Ring Gen 3 | ✓ | ✗ | ✓ | ✗ | ✗ | ✓ | ✓ |
| Whoop 4.0 | ✓ | ✗ | ✓ | ✗ | ✓ | ✓ | ✓ |
| **Pebble** | ✓ | ✓* | ✓* | ✗ | ✓* | ✗ | ✗ |

*Varies by model

### 5.2 Algorithm Features

| Feature | Apple | Fitbit | Samsung | Garmin | Oura |
|---------|-------|--------|---------|--------|------|
| Step counting | ✓ | ✓ | ✓ | ✓ | ✓ |
| Activity recognition | ✓ | ✓ | ✓ | ✓ | ✗ |
| Sleep staging (4-stage) | ✓ | ✓ | ✓ | ✓ | ✓ |
| Sleep score | ✓ | ✓ | ✓ | ✓ | ✓ |
| Stress detection | ✗ | ✓ | ✓ | ✓ | ✓ |
| AFib detection | ✓ | ✓ | ✓ | ✗ | ✗ |
| SpO2 monitoring | ✓ | ✓ | ✓ | ✓ | ✓ |
| Respiratory rate | ✓ | ✓ | ✓ | ✓ | ✓ |

---

## 6. Accuracy Benchmarks

### 6.1 Step Counting (vs. Manual Count)

| Device/Algorithm | Mean Absolute % Error |
|------------------|----------------------|
| Self-supervised ML (UK Biobank) | 12.5% |
| Deep learning CNN | 1-4% |
| Apple Watch | 1-3% |
| Fitbit | 2-5% |
| Garmin | 2-4% |
| Traditional peak detection | 20%+ |

### 6.2 Sleep Staging (vs. PSG, Cohen's κ)

| Device | 2-Stage κ | 4-Stage κ |
|--------|-----------|-----------|
| Oura Ring | 0.75+ | 0.65 |
| Apple Watch | 0.70+ | 0.53-0.60 |
| Fitbit | 0.70+ | 0.42-0.55 |
| Garmin | 0.65+ | 0.25 |
| Research algorithms | 0.80+ | 0.62-0.70 |

### 6.3 Stress Detection (Binary Classification)

| Method | Accuracy | Signals |
|--------|----------|---------|
| HRV + EDA + ML | 93-98% | Multi-modal |
| HRV only + ML | 80-86% | PPG |
| EDA only + ML | 85-92% | GSR |
| Deep learning (raw signals) | 88-95% | PPG + EDA |

### 6.4 AFib Detection (vs. Cardiologist)

| Device | Sensitivity | Specificity |
|--------|-------------|-------------|
| Apple Watch ECG | 94.8% | 95.0% |
| Samsung Galaxy Watch | 87-94% | 90-95% |
| Fitbit ECG | 98.7% | 100% |
| PPG-based (any device) | 85-92% | 85-95% |

---

## Summary: State of the Art vs. PebbleOS

| Capability | State of the Art | PebbleOS |
|------------|-----------------|----------|
| **Step Counting** | CNN-LSTM (97-99%) | Kraepelin FFT (~85-90%) |
| **Activity Recognition** | Transformer/CNN-LSTM | Not implemented |
| **Sleep Detection** | PPG+ACC deep learning (κ=0.65) | VMC convolution (κ≈0.4) |
| **Sleep Staging** | 4-stage (Wake/Light/Deep/REM) | 2-stage (Sleep/Deep) |
| **Stress Detection** | HRV+EDA ML (93%+) | Not implemented |
| **AFib Detection** | ECG + DL (95%+) | Not available |

**Key Gaps in PebbleOS**:
1. No deep learning models (resource constrained)
2. No PPG-based sleep staging
3. No stress detection despite GSR hardware
4. No ECG capability

**PebbleOS Strengths**:
1. Very low power consumption
2. Real-time capable algorithms
3. Proven reliability in production
4. Open-source and modifiable

---

## References

### Activity Recognition
- [Multi-Activity Step Counting with Deep Learning](https://www.mdpi.com/1424-8220/24/21/6927)
- [UK Biobank Step Counting Algorithm](https://pmc.ncbi.nlm.nih.gov/articles/PMC11402590/)
- [CNN-LSTM for HAR](https://pmc.ncbi.nlm.nih.gov/articles/PMC9252338/)

### Sleep Detection
- [Six Wearable Devices Validation](https://academic.oup.com/sleepadvances/article/6/2/zpaf021/8090472)
- [Oura Ring Validation Study](https://www.mdpi.com/1424-8220/24/20/6532)
- [Deep Learning for Sleep Staging](https://www.nature.com/articles/s41746-021-00510-8)

### Stress Detection
- [Stress Detection Systematic Review](https://www.frontiersin.org/journals/computer-science/articles/10.3389/fcomp.2024.1478851/full)
- [HRV-Based Stress Prediction](https://link.springer.com/article/10.1007/s12559-023-10200-0)
- [EDA/GSR Overview](https://imotions.com/products/imotions-lab/modules/eda-gsr-electrodermal-activity/)

### ECG/Arrhythmia
- [Apple Watch ECG Meta-Analysis](https://www.jacc.org/doi/10.1016/j.jacadv.2024.101538)
- [Deep Learning ECG Review](https://pmc.ncbi.nlm.nih.gov/articles/PMC10542398/)
- [Stanford Cardiologist-Level Detection](https://stanfordmlgroup.github.io/projects/ecg/)
