# Tutorial 1.2: Biology of Human Physiology for Health Monitoring

## Learning Objectives

By the end of this tutorial, you will:
- Understand the cardiovascular system and why we measure heart rate/ECG
- Know the autonomic nervous system's role in stress response
- Comprehend sleep physiology and what defines sleep stages
- Understand glucose metabolism and diabetes management
- Know the respiratory system basics for breathing monitoring

## Prerequisites
- Basic biology (high school level)
- Completed Tutorial 1.1 (Mathematical Foundations)

---

## Table of Contents

1. [The Cardiovascular System](#1-the-cardiovascular-system)
2. [The Autonomic Nervous System](#2-the-autonomic-nervous-system)
3. [Sleep Physiology](#3-sleep-physiology)
4. [Glucose Metabolism and Diabetes](#4-glucose-metabolism-and-diabetes)
5. [The Respiratory System](#5-the-respiratory-system)
6. [The Musculoskeletal System and Movement](#6-the-musculoskeletal-system-and-movement)
7. [Thermoregulation](#7-thermoregulation)
8. [Learning Resources](#8-learning-resources)

---

## 1. The Cardiovascular System

### 1.1 The Heart: An Electrical Pump

The heart is both a **mechanical pump** and an **electrical system**. Understanding both is essential for heart rate and ECG monitoring.

```
                    ┌─────────────────────────────┐
                    │        HEART ANATOMY        │
                    └─────────────────────────────┘

                         To Lungs ←─ Pulmonary Artery
                              ↑
            ┌─────────────────┼───────────────────┐
            │     ┌───────────┴──────────┐        │
   From     │     │    Right Atrium      │        │    From
   Body  ───┼────→│    (receives blood)   │        │←── Lungs
(Vena Cava) │     └──────────┬───────────┘        │
            │                ↓ Tricuspid Valve    │
            │     ┌──────────┴───────────┐        │
            │     │   Right Ventricle    │        │
            │     │   (pumps to lungs)   │        │
            │     └──────────────────────┘        │
            │                                      │
            │     ┌──────────────────────┐        │
            │     │    Left Atrium       │        │
            │     │  (receives from lungs)│←──────┤
            │     └──────────┬───────────┘        │
            │                ↓ Mitral Valve       │
            │     ┌──────────┴───────────┐        │
            │     │   Left Ventricle     │───────→│ To Body
            │     │   (pumps to body)    │        │  (Aorta)
            └─────┴──────────────────────┴────────┘

    Blood Flow: Body → Right Atrium → Right Ventricle → Lungs
                → Left Atrium → Left Ventricle → Body
```

### 1.2 The Cardiac Conduction System

The heart has its own **electrical system** that coordinates contractions:

```
                    ┌─────────────────────────────┐
                    │  CARDIAC CONDUCTION SYSTEM  │
                    └─────────────────────────────┘

        1. SA Node (Sinoatrial Node) - "Natural Pacemaker"
           Location: Right atrium
           Function: Initiates heartbeat (60-100 bpm)
                              │
                              ↓ (electrical signal spreads)
        2. Atrial Muscle
           Result: Atria contract → P wave on ECG
                              │
                              ↓
        3. AV Node (Atrioventricular Node)
           Location: Between atria and ventricles
           Function: Delays signal (~0.1s) for atria to finish
                              │
                              ↓
        4. Bundle of His → Bundle Branches
           Function: Rapid conduction to ventricles
                              │
                              ↓
        5. Purkinje Fibers
           Function: Distribute signal throughout ventricles
           Result: Ventricles contract → QRS complex on ECG
                              │
                              ↓
        6. Ventricular Repolarization
           Result: Ventricles reset → T wave on ECG
```

### 1.3 The ECG: Reading the Heart's Electrical Activity

The **electrocardiogram (ECG/EKG)** records electrical activity through skin electrodes:

```
    Voltage
    (mV)
      │
    1.0│                R
      │               ╱ ╲
      │              ╱   ╲
    0.5│            ╱     ╲
      │     P     ╱       ╲      T
      │    ╱╲    ╱         ╲    ╱╲
    0 ─┼───╱──╲─╱───────────╲──╱──╲────────
      │  ╱    Q              S
   -0.5│
      │
      └──────────────────────────────────→ Time (ms)
         0   200   400   600   800

    P wave:      Atrial depolarization (0.08-0.10 s)
    PR interval: Conduction through AV node (0.12-0.20 s)
    QRS complex: Ventricular depolarization (0.06-0.10 s)
    T wave:      Ventricular repolarization
    QT interval: Total ventricular activity (0.35-0.44 s)
    RR interval: Time between heartbeats (heart rate)
```

**Key Clinical Intervals**:

| Interval | Normal Range | Abnormality Indicates |
|----------|--------------|----------------------|
| PR | 120-200 ms | Heart block if prolonged |
| QRS | 60-100 ms | Bundle branch block if wide |
| QT | 350-440 ms | Arrhythmia risk if prolonged |
| RR | 600-1000 ms | Bradycardia/tachycardia |

### 1.4 Heart Rate Variability (HRV)

The heart doesn't beat like a metronome—**healthy hearts show variation**:

```
    RR Intervals (ms):
    ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┐
    │ 850 │ 920 │ 880 │ 910 │ 870 │ 940 │ 860 │
    └─────┴─────┴─────┴─────┴─────┴─────┴─────┘

    This variability (HRV) reflects:
    - Respiratory Sinus Arrhythmia (breathing affects HR)
    - Autonomic Nervous System balance
    - Overall cardiovascular health

    Higher HRV = healthier, more adaptable
    Lower HRV = stress, disease, aging
```

**Physiological Sources of HRV**:

| Frequency Band | Range | Source |
|---------------|-------|--------|
| VLF (Very Low) | <0.04 Hz | Thermoregulation, hormones |
| LF (Low) | 0.04-0.15 Hz | Mix of sympathetic + parasympathetic |
| HF (High) | 0.15-0.4 Hz | Parasympathetic (vagal), breathing |

### 1.5 Blood Pressure and Pulse Wave

Blood pressure creates a **pulse wave** that travels through arteries:

```
    Pressure
    (mmHg)
      │
    140│    ╭─╮ Systolic Peak
      │   ╱   ╲
    120│  ╱     ╲
      │ ╱       ╲ Dicrotic Notch
    100│╱         ╲╱╲
      │             ╲
     80│              ╲_______ Diastolic
      │
      └────────────────────────→ Time

    Systolic: Peak pressure (heart contracts)
    Diastolic: Minimum pressure (heart relaxes)
    Pulse Pressure: Systolic - Diastolic

    Normal: 120/80 mmHg
    Hypertension: >140/90 mmHg
```

**Pulse Transit Time (PTT)**: Time for pulse to travel from heart to periphery
- Faster PTT = Higher blood pressure
- Can estimate BP without cuff using ECG + PPG timing

---

## 2. The Autonomic Nervous System

### 2.1 Two Branches: Fight-or-Flight vs. Rest-and-Digest

The **Autonomic Nervous System (ANS)** controls involuntary functions:

```
                    ┌─────────────────────────────┐
                    │   AUTONOMIC NERVOUS SYSTEM  │
                    └─────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              │                               │
    ┌─────────┴─────────┐         ┌──────────┴──────────┐
    │    SYMPATHETIC    │         │   PARASYMPATHETIC   │
    │  "Fight or Flight"│         │  "Rest and Digest"  │
    └─────────┬─────────┘         └──────────┬──────────┘
              │                               │
    ┌─────────┴─────────┐         ┌──────────┴──────────┐
    │ Activated by:     │         │ Activated by:       │
    │ - Stress          │         │ - Relaxation        │
    │ - Exercise        │         │ - Sleep             │
    │ - Fear            │         │ - Digestion         │
    │ - Excitement      │         │ - Recovery          │
    └─────────┬─────────┘         └──────────┬──────────┘
              │                               │
    Effects:  │                     Effects:  │
    • Heart rate ↑                  • Heart rate ↓
    • HRV ↓                         • HRV ↑
    • Sweating ↑                    • Sweating ↓
    • Pupils dilate                 • Pupils constrict
    • Blood pressure ↑              • Digestion ↑
    • Breathing rate ↑              • Breathing rate ↓
```

### 2.2 The Vagus Nerve

The **vagus nerve** is the main parasympathetic pathway:

```
    Brain
      │
      ↓ (Vagus Nerve - 10th Cranial Nerve)
    ┌─┴─────────────────────────────────┐
    │                                   │
    ↓                                   ↓
  Heart                              Gut
  - Slows heart rate               - Stimulates digestion
  - Source of HF-HRV               - Gut-brain connection

    "Vagal Tone" = Parasympathetic activity
    Higher vagal tone = better stress resilience
    Measured by: HF-HRV power, RMSSD
```

### 2.3 Electrodermal Activity (EDA)

Sweat glands are **only controlled by sympathetic** nerves:

```
    Stress → Sympathetic Activation → Sweat Glands → Skin Conductance ↑

    ┌────────────────────────────────────┐
    │     ELECTRODERMAL ACTIVITY         │
    ├────────────────────────────────────┤
    │ Tonic (SCL):                       │
    │   - Slow-changing baseline         │
    │   - Overall arousal level          │
    │   - Range: 1-20 µS (microsiemens)  │
    ├────────────────────────────────────┤
    │ Phasic (SCR):                      │
    │   - Rapid responses to stimuli     │
    │   - Peak in 1-3 seconds            │
    │   - Amplitude: 0.1-1.0 µS          │
    │   - Recovery: 2-10 seconds         │
    └────────────────────────────────────┘

    Measurement locations:
    - Fingers (highest density of sweat glands)
    - Palm
    - Foot sole
```

### 2.4 Stress Physiology

The **stress response** involves multiple systems:

```
    Stressor (physical or psychological)
              │
              ↓
    ┌─────────────────────┐
    │   HYPOTHALAMUS      │ (brain region)
    └─────────┬───────────┘
              │
    ┌─────────┴─────────────────────────────────┐
    │                                           │
    ↓                                           ↓
  Fast Response                            Slow Response
  (seconds)                                (minutes-hours)
    │                                           │
    ↓                                           ↓
  Sympathetic                              HPA Axis
  Nervous System                           (Hypothalamic-Pituitary-Adrenal)
    │                                           │
    ↓                                           ↓
  Adrenaline                               Cortisol
  (Epinephrine)                            (Stress hormone)
    │                                           │
    ↓                                           ↓
  • HR ↑, BP ↑                             • Sustained alertness
  • Sweating                               • Immune suppression
  • Pupil dilation                         • Metabolic changes
  • Quick energy release                   • Long-term adaptation

    Measurable indicators:
    - Heart rate increase (+10-30 bpm)
    - HRV decrease (RMSSD ↓, LF/HF ↑)
    - Skin conductance increase
    - Respiratory rate increase
    - Skin temperature decrease (peripheral)
```

---

## 3. Sleep Physiology

### 3.1 Sleep Architecture

Sleep is not a uniform state—it cycles through **distinct stages**:

```
    Sleep Architecture (typical night)

    Wake ━━━━                                        ━━━━
    REM      ╲    ╱╲    ╱╲    ╱╲    ╱╲    ╱╲    ╱
    N1        ╲  ╱  ╲  ╱  ╲  ╱  ╲  ╱  ╲  ╱  ╲  ╱
    N2         ╲╱    ╲╱    ╲╱    ╲╱    ╲╱    ╲╱
    N3          ╲    ╱╲    ╱
                 ╲__╱  ╲__╱
          ────────────────────────────────────────────→
          11pm   12am   1am   2am   3am   4am   5am  6am

    Sleep Cycle: ~90 minutes each
    Total cycles per night: 4-6
    Deep sleep (N3): More in first half
    REM sleep: More in second half
```

### 3.2 Sleep Stages Defined

```
    ┌─────────────────────────────────────────────────────────────┐
    │                      SLEEP STAGES                           │
    ├────────┬────────────────────────────────────────────────────┤
    │ WAKE   │ • Alert, eyes open                                 │
    │        │ • EEG: Beta waves (13-30 Hz), alpha when relaxed   │
    │        │ • High muscle tone, frequent movement              │
    ├────────┼────────────────────────────────────────────────────┤
    │ N1     │ • Light sleep, easily awakened                     │
    │ (5%)   │ • EEG: Theta waves (4-7 Hz)                        │
    │        │ • Transition stage, "hypnic jerks"                 │
    │        │ • Slow eye movements                               │
    ├────────┼────────────────────────────────────────────────────┤
    │ N2     │ • Intermediate sleep                               │
    │ (45%)  │ • EEG: Sleep spindles (12-14 Hz), K-complexes      │
    │        │ • Memory consolidation begins                      │
    │        │ • Body temperature drops                           │
    ├────────┼────────────────────────────────────────────────────┤
    │ N3     │ • Deep/slow-wave sleep                             │
    │ (25%)  │ • EEG: Delta waves (0.5-4 Hz)                      │
    │        │ • Physical restoration, growth hormone release     │
    │        │ • Difficult to awaken                              │
    │        │ • Lowest heart rate, blood pressure                │
    ├────────┼────────────────────────────────────────────────────┤
    │ REM    │ • "Rapid Eye Movement" / Dream sleep               │
    │ (25%)  │ • EEG: Mixed frequencies (like wake)               │
    │        │ • Muscle paralysis (atonia)                        │
    │        │ • Variable HR and breathing                        │
    │        │ • Memory consolidation, emotional processing       │
    └────────┴────────────────────────────────────────────────────┘
```

### 3.3 Physiological Changes During Sleep

```
    Parameter       Wake    Light   Deep    REM
    ─────────────────────────────────────────────
    Heart Rate      70-80   60-70   50-60   Variable
    HRV (RMSSD)     Low     Medium  High    Variable
    Breathing       Irregular Regular Very    Irregular
                                    Regular
    Movement        High    Low     Very    Very Low
                                    Low     (paralysis)
    Body Temp       Normal  Drops   Lowest  Rising
    EEG Pattern     β/α     θ       δ       Mixed
```

### 3.4 Circadian Rhythm

The **circadian rhythm** is our internal 24-hour clock:

```
    Body Clock (Circadian Rhythm)

    Alertness
        │
      High│      ╭───╮                    ╭───╮
        │     ╱     ╲                  ╱     ╲
    Medium│    ╱       ╲                ╱       ╲
        │   ╱         ╲    ╭─╮      ╱
      Low│──╱           ╲──╯  ╲____╱
        │
        └───────────────────────────────────────→
         6am  9am  12pm  3pm  6pm  9pm  12am 3am

    Key times:
    - 6-9am: Cortisol rise (wake up)
    - 2-3pm: Post-lunch dip (siesta window)
    - 9-11pm: Melatonin release (sleep onset)
    - 3-4am: Lowest body temperature

    Controlled by:
    - Light exposure (main zeitgeber)
    - Suprachiasmatic nucleus (SCN) in brain
    - Melatonin from pineal gland
```

---

## 4. Glucose Metabolism and Diabetes

### 4.1 Blood Glucose Regulation

The body maintains **blood glucose in a narrow range**:

```
    Blood Glucose Regulation

    Glucose (mg/dL)
        │
     180│ ─ ─ ─ ─ ─ Diabetic threshold ─ ─ ─ ─ ─
        │
     140│     ╭─╮       Meal spike
        │    ╱   ╲      (normal response)
     100│───╱─────╲─────────────────────── Normal fasting
        │           ╲
      70│            ╲___╱
        │
      54│ ─ ─ ─ ─ ─ Hypoglycemia ─ ─ ─ ─ ─
        │
        └────────────────────────────────────→
          6am   12pm   6pm   12am   6am

    Normal ranges:
    - Fasting: 70-100 mg/dL (3.9-5.6 mmol/L)
    - 2h after meal: <140 mg/dL (<7.8 mmol/L)

    Diabetes criteria:
    - Fasting: >126 mg/dL (>7.0 mmol/L)
    - 2h post-meal: >200 mg/dL (>11.1 mmol/L)
    - HbA1c: >6.5%
```

### 4.2 Insulin and Glucagon: The Balance

```
    ┌─────────────────────────────────────────────────────────┐
    │           GLUCOSE REGULATION HORMONES                   │
    └─────────────────────────────────────────────────────────┘

    HIGH Blood Glucose (after eating)
              │
              ↓
    ┌─────────────────────┐
    │  PANCREAS (β cells) │
    └─────────┬───────────┘
              │
              ↓ Releases INSULIN
              │
    ┌─────────┴─────────────────────────────────────┐
    │ Insulin effects:                              │
    │ • Cells absorb glucose from blood             │
    │ • Liver converts glucose to glycogen (storage)│
    │ • Fat cells store excess as fat               │
    │ → Blood glucose DECREASES                     │
    └───────────────────────────────────────────────┘

    LOW Blood Glucose (fasting/exercise)
              │
              ↓
    ┌─────────────────────┐
    │  PANCREAS (α cells) │
    └─────────┬───────────┘
              │
              ↓ Releases GLUCAGON
              │
    ┌─────────┴─────────────────────────────────────┐
    │ Glucagon effects:                             │
    │ • Liver releases stored glycogen as glucose   │
    │ • Stimulates gluconeogenesis (making glucose) │
    │ → Blood glucose INCREASES                     │
    └───────────────────────────────────────────────┘
```

### 4.3 Diabetes Types

```
    ┌────────────────────────────────────────────────────────────┐
    │                    DIABETES TYPES                          │
    ├────────────────┬───────────────────────────────────────────┤
    │ TYPE 1         │ • Autoimmune destruction of β cells       │
    │ (5-10%)        │ • No insulin production                   │
    │                │ • Requires insulin injections             │
    │                │ • Usually diagnosed in childhood          │
    │                │ • CGM essential for management            │
    ├────────────────┼───────────────────────────────────────────┤
    │ TYPE 2         │ • Insulin resistance + β cell dysfunction │
    │ (90-95%)       │ • Body doesn't use insulin effectively    │
    │                │ • Often associated with obesity           │
    │                │ • Can be managed with diet/exercise/meds  │
    │                │ • CGM helpful for lifestyle feedback      │
    ├────────────────┼───────────────────────────────────────────┤
    │ GESTATIONAL    │ • Develops during pregnancy               │
    │                │ • Usually resolves after birth            │
    │                │ • Increases risk for Type 2 later         │
    └────────────────┴───────────────────────────────────────────┘
```

### 4.4 Continuous Glucose Monitoring (CGM)

CGM devices measure glucose every few minutes:

```
    CGM System Overview

    ┌─────────────────────────────────────────────────────────┐
    │                 CGM COMPONENTS                          │
    └─────────────────────────────────────────────────────────┘

    1. SENSOR (under skin, typically arm or abdomen)
       ┌──────────────────────────────────────────┐
       │  Electrode with glucose oxidase enzyme   │
       │                                          │
       │  Glucose + O₂ → H₂O₂ → Electric current  │
       │                                          │
       │  Current ∝ Glucose concentration         │
       └──────────────────────────────────────────┘

    2. TRANSMITTER (on skin, attached to sensor)
       - Wireless communication (Bluetooth)
       - Battery powered

    3. RECEIVER/PHONE
       - Displays glucose values
       - Trend arrows (rising/falling)
       - Alerts for high/low

    Measurement location:
    - Interstitial fluid (between cells)
    - NOT direct blood glucose
    - 5-15 minute lag behind blood glucose
```

**CGM Metrics**:

| Metric | Target | Description |
|--------|--------|-------------|
| Time in Range (TIR) | >70% | % time glucose 70-180 mg/dL |
| Time Below Range | <4% | % time glucose <70 mg/dL |
| Time Above Range | <25% | % time glucose >180 mg/dL |
| Glucose Management Indicator (GMI) | <7% | Estimated HbA1c |
| Coefficient of Variation (CV) | <36% | Glucose variability |

### 4.5 Interstitial vs. Blood Glucose

```
    Blood Glucose vs. Interstitial Glucose

    Glucose
    (mg/dL)
        │
     180│                    ╭─╮ Blood glucose
        │                   ╱   ╲
        │                  ╱     ╲    ╭─╮ Interstitial
        │                 ╱       ╲  ╱   ╲ (CGM measures this)
     120│────────────────╱─────────╲╱─────╲───────────
        │              ╱
        │             ╱  ~5-15 min lag
        │            ╱
      70│           ╱
        │
        └────────────────────────────────────────→ Time
                    Meal

    Why the lag matters:
    - CGM may not catch rapid changes
    - Verify with fingerstick when making treatment decisions
    - Calibration may be needed during rapid changes
```

---

## 5. The Respiratory System

### 5.1 Breathing Mechanics

```
    ┌─────────────────────────────────────────────────────────┐
    │               RESPIRATORY SYSTEM                        │
    └─────────────────────────────────────────────────────────┘

    Air flow:
    Nose/Mouth → Trachea → Bronchi → Bronchioles → Alveoli

                        ┌─────────────┐
                        │   Trachea   │
                        └──────┬──────┘
                               │
                    ┌──────────┴──────────┐
                    │                     │
              ┌─────┴─────┐         ┌─────┴─────┐
              │Left Bronchus│       │Right Bronchus│
              └─────┬─────┘         └─────┬─────┘
                    │                     │
               ┌────┴────┐           ┌────┴────┐
               │Bronchioles│         │Bronchioles│
               └────┬────┘           └────┬────┘
                    │                     │
               ┌────┴────┐           ┌────┴────┐
               │ Alveoli │           │ Alveoli │
               │ (gas    │           │ (gas    │
               │exchange)│           │exchange)│
               └─────────┘           └─────────┘

    Breathing muscles:
    - Diaphragm (main muscle)
    - Intercostal muscles (between ribs)

    Inhalation: Diaphragm contracts → Chest expands → Air flows in
    Exhalation: Diaphragm relaxes → Chest contracts → Air flows out
```

### 5.2 Respiratory Parameters

```
    Normal Breathing Parameters:

    ┌────────────────────────┬────────────────────────┐
    │ Parameter              │ Normal Adult Values    │
    ├────────────────────────┼────────────────────────┤
    │ Respiratory Rate (RR)  │ 12-20 breaths/min      │
    │ Tidal Volume           │ ~500 mL/breath         │
    │ Minute Ventilation     │ ~6 L/min               │
    │ Inspiratory Time       │ ~1.5-2 seconds         │
    │ Expiratory Time        │ ~2-3 seconds           │
    │ I:E Ratio              │ 1:2 to 1:3             │
    └────────────────────────┴────────────────────────┘

    Measuring breathing from wearables:
    1. Accelerometer: Chest/abdomen movement
    2. PPG: Respiratory modulation of pulse
    3. Impedance: Chest impedance changes
```

### 5.3 Oxygen Saturation (SpO2)

```
    Blood Oxygen Saturation (SpO2)

    ┌─────────────────────────────────────────────────────────┐
    │ SpO2 measures: % of hemoglobin carrying oxygen         │
    │                                                        │
    │ Normal: 95-100%                                        │
    │ Mild hypoxemia: 90-94%                                 │
    │ Moderate hypoxemia: 80-89%                             │
    │ Severe hypoxemia: <80% (medical emergency)             │
    └─────────────────────────────────────────────────────────┘

    How pulse oximetry works:

    ┌─────────────────────────────────────────────────────────┐
    │                                                        │
    │   LED (Red 660nm + IR 940nm)                           │
    │          │                                             │
    │          ↓ Light shines through tissue                 │
    │   ┌──────────────────┐                                 │
    │   │    Finger/Wrist  │                                 │
    │   │   ┌──────────┐   │                                 │
    │   │   │  Blood   │   │ ← Pulsating arterial blood     │
    │   │   │  vessels │   │                                 │
    │   │   └──────────┘   │                                 │
    │   └──────────────────┘                                 │
    │          │                                             │
    │          ↓ Photodetector measures transmitted light    │
    │                                                        │
    │   Oxygenated hemoglobin: Absorbs more IR               │
    │   Deoxygenated hemoglobin: Absorbs more Red            │
    │                                                        │
    │   SpO2 = f(Red/IR ratio)                               │
    └─────────────────────────────────────────────────────────┘
```

### 5.4 Sleep Apnea

```
    ┌─────────────────────────────────────────────────────────┐
    │                   SLEEP APNEA                           │
    └─────────────────────────────────────────────────────────┘

    Types:
    1. OBSTRUCTIVE (OSA) - Most common
       - Airway physically blocked
       - Soft tissue collapses during sleep

    2. CENTRAL
       - Brain doesn't signal to breathe
       - Often associated with heart failure

    3. MIXED
       - Combination of both

    Detection signals:
    ┌──────────────────────────────────────────────────────┐
    │ SpO2 during normal sleep:                           │
    │ ───────────────────────────────────── 97-98%        │
    │                                                     │
    │ SpO2 with sleep apnea:                              │
    │              ╱╲              ╱╲                      │
    │ ─────────╲  ╱  ╲  ──────╲  ╱  ╲  ─────  Desaturations│
    │           ╲╱    ╲╱        ╲╱    ╲╱      down to 80%  │
    │                                                     │
    │ Accompanied by:                                     │
    │ - Breathing pauses (>10 seconds)                    │
    │ - Heart rate changes (bradycardia then tachycardia) │
    │ - Arousal from sleep                                │
    └──────────────────────────────────────────────────────┘

    Severity (Apnea-Hypopnea Index):
    - Normal: <5 events/hour
    - Mild: 5-14 events/hour
    - Moderate: 15-29 events/hour
    - Severe: >30 events/hour
```

---

## 6. The Musculoskeletal System and Movement

### 6.1 How Movement Creates Sensor Signals

```
    ┌─────────────────────────────────────────────────────────┐
    │              MOVEMENT AND SENSORS                       │
    └─────────────────────────────────────────────────────────┘

    Walking Gait Cycle:

          Heel Strike    Mid-stance    Toe-off     Swing
              │              │            │           │
              ↓              ↓            ↓           ↓
         ┌────┼────┐    ┌────┼────┐  ┌────┼────┐  ┌───┼────┐
         │ ●  │    │    │  ● │    │  │    │ ●  │  │   │  ● │
         │╱   │    │    │ ╱  │    │  │   ╲│    │  │   │╱   │
    ─────┴────┴────┴────┴────┴────┴──┴────┴────┴──┴───┴────┴──

    Accelerometer signal (vertical axis):
          │
          │     ╭╮  Heel strike     ╭╮
          │    ╱  ╲                ╱  ╲
       +1g│   ╱    ╲              ╱    ╲
          │──╱──────╲────────────╱──────╲──
       -1g│          ╲    ╱╲    ╱
          │           ╲──╱  ╲──╱
          │              Toe-off
          └────────────────────────────────→ Time

    Step frequency:
    - Walking: 1.5-2.5 Hz (90-150 steps/min)
    - Running: 2.5-3.5 Hz (150-210 steps/min)
```

### 6.2 Activities and Their Signatures

```
    Activity Signatures (Accelerometer)

    ┌─────────────┬────────────────────────────────────────┐
    │ Activity    │ Signal Characteristics                 │
    ├─────────────┼────────────────────────────────────────┤
    │ Stationary  │ ~1g constant (gravity only)            │
    │             │ Low variance                           │
    ├─────────────┼────────────────────────────────────────┤
    │ Walking     │ Periodic signal, 1.5-2.5 Hz            │
    │             │ Amplitude: 0.2-0.5g                    │
    │             │ Distinct heel strike pattern           │
    ├─────────────┼────────────────────────────────────────┤
    │ Running     │ Periodic signal, 2.5-3.5 Hz            │
    │             │ Amplitude: 0.5-2.0g                    │
    │             │ Stronger vertical component            │
    ├─────────────┼────────────────────────────────────────┤
    │ Cycling     │ Low-frequency oscillation              │
    │             │ Pedaling frequency: 1-2 Hz             │
    │             │ Less vertical impact than walking      │
    ├─────────────┼────────────────────────────────────────┤
    │ Stairs      │ Similar to walking but asymmetric      │
    │             │ Different acceleration during ascent   │
    │             │ vs. descent                            │
    └─────────────┴────────────────────────────────────────┘
```

### 6.3 Energy Expenditure

```
    Energy Expenditure Estimation

    ┌─────────────────────────────────────────────────────────┐
    │ MET (Metabolic Equivalent of Task)                     │
    │ 1 MET = 3.5 mL O₂/kg/min = resting energy expenditure  │
    └─────────────────────────────────────────────────────────┘

    Activity METs:
    - Sleeping: 0.9
    - Sitting: 1.0
    - Light walking: 2.5
    - Brisk walking: 4.0
    - Running (6 mph): 10.0
    - Running (10 mph): 16.0

    Calorie calculation:
    Calories/min = METs × Weight(kg) × 3.5 / 200

    Example: 70 kg person running at 6 mph
    Calories/min = 10 × 70 × 3.5 / 200 = 12.25 cal/min
```

---

## 7. Thermoregulation

### 7.1 Body Temperature Regulation

```
    ┌─────────────────────────────────────────────────────────┐
    │              THERMOREGULATION                           │
    └─────────────────────────────────────────────────────────┘

    Core Temperature: 36.5-37.5°C (97.7-99.5°F)

    Hypothalamus (brain thermostat)
              │
    ┌─────────┴─────────────────────────────────┐
    │                                           │
    ↓ TOO HOT                          TOO COLD ↓
    │                                           │
    ├─ Vasodilation (blood to skin)      ├─ Vasoconstriction
    ├─ Sweating                          ├─ Shivering
    ├─ Behavioral (seek shade)           ├─ Behavioral (seek warmth)
    │                                    ├─ Piloerection (goosebumps)
    ↓                                           ↓
    Heat loss                            Heat conservation

    Fever response:
    - Infection → Immune system releases pyrogens
    - Hypothalamus raises "set point"
    - Body feels cold, shivers to reach new set point
```

### 7.2 Skin vs. Core Temperature

```
    Temperature Measurement Locations

    ┌────────────────┬─────────────────────────────────────┐
    │ Location       │ Characteristics                     │
    ├────────────────┼─────────────────────────────────────┤
    │ Rectal         │ Gold standard for core temperature  │
    │                │ ~37°C (98.6°F)                      │
    ├────────────────┼─────────────────────────────────────┤
    │ Oral           │ ~0.5°C lower than rectal            │
    │                │ Affected by breathing, drinks       │
    ├────────────────┼─────────────────────────────────────┤
    │ Ear (tympanic) │ Close to core, fast response        │
    │                │ Requires proper technique           │
    ├────────────────┼─────────────────────────────────────┤
    │ Forehead       │ ~1°C lower than core                │
    │                │ Affected by ambient temperature     │
    ├────────────────┼─────────────────────────────────────┤
    │ Wrist (skin)   │ 3-5°C lower than core               │
    │                │ Varies with vasodilation            │
    │                │ Used in wearables                   │
    └────────────────┴─────────────────────────────────────┘

    Challenge for wearables:
    - Skin temperature ≠ Core temperature
    - Must estimate core from skin + other signals
    - Circadian pattern: lowest ~3-4am, highest ~5-7pm
```

### 7.3 Temperature in Health Monitoring

```
    Temperature Applications

    ┌─────────────────────────────────────────────────────────┐
    │ Fever Detection                                        │
    │ - Core temp >38°C (100.4°F)                            │
    │ - Wearables: Look for elevation above personal baseline│
    ├─────────────────────────────────────────────────────────┤
    │ Menstrual Cycle Tracking                               │
    │ - Basal body temp rises ~0.3-0.5°C after ovulation     │
    │ - Requires consistent early morning measurement        │
    │ - Oura Ring uses this for cycle prediction             │
    ├─────────────────────────────────────────────────────────┤
    │ Sleep Quality                                          │
    │ - Core temp drops 1-2°C during sleep                   │
    │ - Drop facilitates sleep onset                         │
    │ - Wrist temp rises (vasodilation to radiate heat)      │
    ├─────────────────────────────────────────────────────────┤
    │ Stress Response                                        │
    │ - Peripheral vasoconstriction → skin temp drops        │
    │ - Can complement HRV for stress detection              │
    └─────────────────────────────────────────────────────────┘
```

---

## 8. Learning Resources

### 8.1 Textbooks

#### Physiology Fundamentals
| Book | Author | Level | Focus |
|------|--------|-------|-------|
| **"Guyton and Hall Textbook of Medical Physiology"** | Hall | Comprehensive | Gold standard physiology reference |
| **"Principles of Anatomy and Physiology"** | Tortora & Derrickson | Beginner | Accessible introduction |
| **"Vander's Human Physiology"** | Widmaier et al. | Intermediate | Systems approach |
| **"Cardiovascular Physiology"** | Mohrman & Heller | Intermediate | Heart and circulation focus |

#### Specialized Topics
| Book | Author | Focus |
|------|--------|-------|
| **"Principles and Practice of Sleep Medicine"** | Kryger | Comprehensive sleep reference |
| **"Heart Rate Variability"** | Malik & Camm | HRV science |
| **"Diabetes Technology"** | Garg et al. | CGM and diabetes devices |
| **"ECG Made Easy"** | Hampton | ECG interpretation |

### 8.2 Online Courses

#### Free Courses
| Course | Platform | Link |
|--------|----------|------|
| **Human Anatomy** | Coursera | [coursera.org](https://www.coursera.org/learn/anatomy) |
| **Introductory Human Physiology** | Coursera (Duke) | [coursera.org](https://www.coursera.org/learn/physiology) |
| **Understanding the Heart** | Coursera | [coursera.org](https://www.coursera.org/learn/understanding-the-heart) |
| **Sleep: Neurobiology, Medicine, and Society** | Coursera | [coursera.org](https://www.coursera.org/learn/sleep) |
| **Diabetes - a Global Challenge** | Coursera | [coursera.org](https://www.coursera.org/learn/diabetes) |

### 8.3 YouTube Channels

| Channel | Focus | Best For |
|---------|-------|----------|
| **Ninja Nerd** | Medical education | Comprehensive physiology lectures |
| **Osmosis** | Medical education | Visual explanations |
| **Khan Academy Medicine** | Basic physiology | Foundational concepts |
| **Dr. John Campbell** | Health topics | Current health issues |
| **Armando Hasudungan** | Illustrated physiology | Visual learners |

### 8.4 Reference Websites

| Resource | Description | Link |
|----------|-------------|------|
| **PhysioNet** | Physiological signal databases | [physionet.org](https://physionet.org/) |
| **PubMed** | Medical research database | [pubmed.ncbi.nlm.nih.gov](https://pubmed.ncbi.nlm.nih.gov/) |
| **UpToDate** | Clinical reference (subscription) | [uptodate.com](https://www.uptodate.com/) |
| **HeartMath Institute** | HRV research | [heartmath.org](https://www.heartmath.org/) |
| **Sleep Foundation** | Sleep science | [sleepfoundation.org](https://www.sleepfoundation.org/) |

### 8.5 Key Research Papers

#### Cardiovascular
1. **"Heart rate variability: Standards of measurement"** (Task Force, 1996)
   - Defines HRV analysis standards
   - [European Heart Journal](https://doi.org/10.1093/oxfordjournals.eurheartj.a014868)

2. **"The assessment and analysis of handedness"** (Oldfield, 1971)
   - Foundation paper for laterality studies

#### Sleep
3. **"The AASM Manual for the Scoring of Sleep"** (AASM, 2007)
   - Gold standard sleep staging rules

4. **"Sleep stage prediction with raw acceleration and PPG"** (2019)
   - Wearable sleep staging validation

#### Glucose
5. **"Clinical Targets for CGM Data Interpretation"** (Battelino et al., 2019)
   - CGM metrics and targets
   - [Diabetes Care](https://doi.org/10.2337/dc19-1028)

### 8.6 Interactive Learning

| Tool | Purpose | Link |
|------|---------|------|
| **Biodigital Human** | 3D anatomy exploration | [biodigital.com](https://www.biodigital.com/) |
| **Visible Body** | Anatomy & physiology apps | [visiblebody.com](https://www.visiblebody.com/) |
| **ECG Simulator** | Practice ECG interpretation | Various apps |
| **Complete Anatomy** | Detailed 3D anatomy | [3d4medical.com](https://3d4medical.com/) |

---

## Summary

Understanding the biology behind health monitoring is essential for:

| System | Key Measurements | What We Learn |
|--------|------------------|---------------|
| **Cardiovascular** | HR, HRV, ECG, BP | Heart health, stress, fitness |
| **Autonomic Nervous** | HRV, EDA, skin temp | Stress, recovery, emotional state |
| **Sleep** | Movement, HR, HRV | Sleep quality, disorders |
| **Metabolic** | Glucose, trends | Diabetes management |
| **Respiratory** | RR, SpO2 | Breathing disorders, fitness |
| **Musculoskeletal** | Acceleration | Activity, energy expenditure |
| **Thermoregulation** | Temperature | Fever, cycles, sleep |

**Key Takeaways**:
1. The body is a complex, interconnected system
2. Multiple signals together provide better insights than single measurements
3. Understanding physiology helps interpret algorithm outputs
4. Normal ranges vary by individual—personalization matters
5. Validation against gold standards is essential

---

**Next Tutorial**: [1.3 Computer Science for Embedded Systems →](03_cs_foundations.md)
