# AI-Agent (TransQ) for AI-Informed RCM of Runway Standing Water Sensors

This repository provides an **executable AI-agent–orchestrated framework** for **reliability prediction** and **maintenance planning** of **runway Standing Water Detector (SWD)** capacitive sensors (Plate 1 and Plate 3).  
The pipeline integrates:

- **Accelerated Life Testing (ALT)** (temperature and humidity chambers),
- **time-to-event extraction** from streaming Hz signals (Hz-drop + dwell rule),
- **censoring-aware Weibull reliability baseline**,
- **TransQ (Transformer–Quantum) degradation stage classifier**, and
- **RCM/FMEA coupling** to produce **RPN/AP** and **risk-bounded inspection intervals**.

> Core idea: the AI model is not used only for classification. The **critical-class probability** is treated as a condition-risk proxy, fused with Weibull horizon risk to generate **actionable RCM outputs**.

---

## Highlights

- **Executable end-to-end AI-agent**: Train → Evaluate → derive `D_fixed` from validation FNR → ALT extraction → Weibull fit → field alignment → RPN/AP → maintenance interval search.
- **ALT-driven reliability**: converts chamber streaming into \((t_i, \varepsilon_i)\) tuples (failure vs right-censored).
- **Field-aligned planning**: supports a **two-year field simulation** using survival-calibrated time scaling.
- **Policy-aware scheduling**: inspection interval is computed to satisfy a target risk tolerance \(\alpha\) (e.g., 0.10 / 0.05 / 0.02 / 0.01).
- **TransQ model**: hybrid **Transformer + Quantum feature branch** with gated fusion for degradation-stage recognition.

---

## Method Overview

### 1) Data Inputs
- **Chamber ALT logs** (Excel):  
  - Temperature test: 7 days  
  - Humidity test: 21 days (often right-censored)
- **Durability dataset** (`.mat`):  
  - fixed-length vectors `durability` with length 951 (zero-padded)  
  - optional `durability_len` for valid-length tracking  
- **Plate configuration**: Plate 1 and Plate 3

### 2) ALT Failure-Time Extraction (Hz Drop)
A failure event is detected when Hz drops below a threshold:
- threshold = baseline × (1 − drop_pct)
- baseline estimated from initial window
- **dwell constraint** avoids transient-trigger failures

Outputs:
- `t_fail_days` and `event` (1 = failure, 0 = censored)

### 3) Reliability Baseline (Weibull with Censoring)
Weibull time-to-event model is fit using censoring-aware likelihood.  
With limited failures, \(\beta\) can be fixed conservatively (e.g., \(\beta = 2\)) and \(\eta\) estimated from mixed failure + censored evidence.

### 4) TransQ Degradation Classifier
TransQ is a 1D Transformer model with:
- Patch embedding (Conv1D stride=patch)
- Positional embedding
- Transformer blocks
- Quantum branch (AngleEmbedding + entangler layers; Pauli-Z expectations)
- Gated fusion + Softmax classifier (5 classes)

### 5) AI-Informed RCM (RPN/AP + Interval)
We fuse Weibull horizon risk with AI critical-state probability using a Δt-aware multiplier:

\[
p_{fail,fused} \approx (1+\gamma p_{critical}) \cdot p_{fail,base}(t,\Delta t)
\]

Then map to:
- Occurrence \(O = f(p_{fail,fused})\)
- Detection \(D = f(\text{FNR critical})\) (deployment uses `D_fixed` from validation)
- RPN = \(S \times O \times D\)
- AP mapping → maintenance action
- interval search: max days such that \(p_{fail,fused} \le \alpha\)

---

## Results (Latest)

### Classification (TransQ)
On the test set, the classifier achieved near-ideal performance:
- Accuracy: **1.0000**
- Macro/Weighted F1: **1.0000**
- Log loss: **0.0003**
- Brier score: **0.000021**
- Critical class (Failure Stage 2) recall: **1.0000** (FNR = 0.0)

> Note: extremely high scores can indicate strong separability **or** correlated samples across splits.  
> For stricter generalization, use **file-/device-level group splitting**.

### Reliability + AI-Informed RCM (2-year field simulation)
- ALT extraction:
  - Temperature: failure observed (event=1)
  - Humidity: right-censored at 21 days (event=0)
- Weibull baseline: \(\beta\) assumed, \(\eta\) estimated with censoring
- For both Plate 1 and Plate 3 under current policy settings:
  - Severity \(S=9\), Occurrence \(O=4\), Detection \(D=1\)
  - **RPN = 36**, **AP4 (Monitor)**
  - Recommended interval up to **180 days** for \(\alpha=0.10\) (search bound)

---

## Repository Structure (Suggested)
