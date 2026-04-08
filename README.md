# A Measurement-Informed Framework for Reliability Evaluation and AI-Assisted Maintenance Planning of a Capacitive Runway Standing Water Sensor

This repository provides an executable framework for **reliability evaluation** and **maintenance planning** of a capacitive **Standing Water Detector (SWD)** for runway applications using two sensing configurations: **Plate 1** and **Plate 3**.

The framework combines:

- **Accelerated environmental exposure** under temperature and humidity chambers,
- **frequency-based degradation monitoring** from streaming sensor signals,
- **time-to-event extraction** using a reproducible Hz-drop-with-dwell rule,
- **censoring-aware Weibull reliability modelling**,
- **TransQ (Transformer + Quantum module) condition assessment**, and
- **RCM/FMEA-based maintenance interpretation** to produce **RPN/AP outputs** and **risk-bounded inspection intervals**.

> Core idea: the classifier is not used as a stand-alone maintenance decision maker.  
> Its **critical-state probability** is treated as **complementary condition evidence** and is fused with the Weibull-based reliability estimate to support maintenance-oriented decision making.

---

## Highlights

- A capacitive runway standing-water sensor is evaluated under accelerated temperature and humidity exposure.  
- A 10% frequency-drop rule with a dwell constraint converts streaming sensor signals into event/censor observations.  
- A censoring-aware Weibull model estimates durability from sparse-failure sensor data.  
- AI-assisted condition assessment is integrated with reliability evidence for FMEA/AP-based maintenance planning.  
- A survival-calibrated field-alignment step reduces misleading direct comparisons between ALT time and field age.  

---

## Method Overview

### 1) Data Inputs
- **Accelerated chamber data**
  - Temperature test: **7 days**
  - Humidity test: **21 days**
  - Sampling interval: **20 s**
- **Durability sequences**
  - fixed-length vectors of length **951**
  - optional valid-length tracking
- **Sensor configurations**
  - **Plate 1**
  - **Plate 3**

### 2) Operational Failure Definition and Time-to-Event Extraction
The SWD produces a continuous frequency output.  
Operational failure is therefore defined from **signal degradation**, not complete hardware shutdown.

A failure event is detected when:

- `frequency < baseline × (1 − drop_pct)`
- `drop_pct = 0.10`
- the threshold crossing persists for a predefined **dwell duration**

This step converts each chamber trajectory into:

- `t_fail_days`
- `event`  
  - `1 = failure`
  - `0 = right-censored`

This provides reliability-ready observations of the form:

$$\[(t_i, \varepsilon_i)\]$$

---

### 3) Reliability Baseline: Weibull Modelling with Censoring
A two-parameter Weibull model is used to estimate the durability baseline from mixed **failure** and **right-censored** observations.

- Weibull shape: $\(\beta\)$
- Weibull scale: $\(\eta\)$

When observed failures are limited, $\(\beta\)$ can be fixed conservatively (for example, $\(\beta = 2\))$ and $\(\eta\)$ is estimated using a censoring-aware likelihood.

This baseline is used for:

- survival estimation,
- horizon risk calculation,
- maintenance-oriented planning.

---

### 4) AI-Assisted Condition Assessment (TransQ)
TransQ is a hybrid 1D sequence model for degradation-stage recognition.

Main components:
- Conv1D patch embedding
- positional embedding
- Transformer blocks
- quantum feature branch
- gated fusion
- softmax classifier

The model outputs degradation-stage probabilities, including the probability of the **critical state**, which is used as complementary evidence in the maintenance decision layer.

---

### 5) Reliability–Condition Fusion for RCM/FMEA
The framework combines:

- Weibull-based horizon risk, and
- AI-based critical-state probability

using a $\(\Delta t\)$-aware hazard adjustment:

$\[p_{\text{fail,fused}} \approx (1+\gamma p_{\text{critical}})\, p_{\text{fail,base}}(t,\Delta t)\]$

These fused outputs are mapped into:

- **Occurrence** $\(O\)$
- **Detection** $\(D\)$
- **RPN** $\(= S \times O \times D\)$
- **Action Priority (AP)**

The framework then recommends a planning action such as:

- monitor
- inspect
- repair
- replace

depending on the selected risk policy.

---

### 6) Risk-Based Inspection Interval
The inspection interval is determined by solving for the maximum interval that satisfies:

$\[p_{\text{fail,fused}} \le \alpha\]$

where $\(\alpha\)$ is the allowable probability of failure before inspection.

This makes the scheduling step explicitly **policy-dependent** and compatible with different operational risk tolerances.

---

### 7) Survival-Calibrated Field Planning
Direct comparison between accelerated-test time and field age can be misleading.  
To address this, the framework includes a **survival-calibrated field-alignment step** that maps field age into an ALT-equivalent planning scenario.

This step is intended as a **planning bridge**, not as a physics-based equivalence model.

---

## Results Summary

### Condition Assessment
The TransQ classifier showed very strong discrimination across the predefined degradation stages on the evaluated dataset.

In this framework, the classifier is used as:
- a **condition-assessment module**
- a source of **critical-state probability**
- **not** a stand-alone maintenance predictor

> Because the dataset includes augmented durability patterns, stricter validation using file-level, run-level, or device-level grouping is recommended for stronger generalization claims.

### Reliability and Maintenance Planning
Using the extracted event/censor observations:

- Temperature exposure produced failure events
- Humidity exposure remained right-censored within the observation window
- A censoring-aware Weibull baseline was estimated for both sensor configurations

Under the selected policy settings:

- **Severity** $\(S = 9\)$
- **Occurrence** $\(O = 4\)$
- **Detection** $\(D = 1\)$
- **RPN = 36**
- **AP4 (Monitor)**

For both Plate 1 and Plate 3, the recommended inspection interval reached:

- **up to 180 days** for $\(\alpha = 0.10\)$

These outputs should be interpreted as **planning-oriented results under the selected modelling and policy assumptions**, rather than universal maintenance prescriptions.

---

## Repository Structure (Suggested)
