# SiO₂ Dry Etch Process Optimization – SPC & DoE Study

**A simulated Fab process study applying Statistical Process Control (SPC) and Design of Experiments (DoE) to optimize a reactive ion etch (RIE) process for silicon dioxide.**

---

## Overview

This project demonstrates a structured, data-driven approach to process optimization in a semiconductor manufacturing context. A SiO₂ RIE process is simulated in Python with realistic process noise (~3% of response), unknown true model parameters, and a hidden optimum — mimicking conditions in a real Fab environment.

The study follows the methodology used in semiconductor process engineering:

1. **Baseline SPC** – assess process stability and capability at initial operating conditions
2. **DoE (CCD/RSM)** – identify significant factors and locate the process optimum
3. **Verification SPC** – confirm improved capability at optimized settings

**Result:** Cpk increased from 1.22 → 1.70, exceeding the semiconductor industry standard of Cpk ≥ 1.33.

---

## Process Description

| Parameter | Value |
|---|---|
| Process | SiO₂ Dry Etch (Reactive Ion Etching) |
| Response variable | Etch Rate R [nm/min] |
| LSL / USL | 85 / 125 nm/min |
| Capability target | Cpk ≥ 1.33 |
| Control chart type | Individuals & Moving Range (IMR) – one wafer per batch |

**Process factors studied:**

| Factor | Symbol | Unit | Range |
|---|---|---|---|
| Bias Power | x₁ | W | 12.8 – 50.0 |
| Chamber Pressure | x₂ | mTorr | 15.9 – 23.3 |
| RF Power | x₃ | W | 430.9 – 500.0 |
| CHF₃ Flow | x₄ | sccm | 135.6 – 172.0 |

Factor ranges reflect validated process boundaries based on prior process knowledge (operating outside these ranges results in unstable etch behavior).

> **Screening history:** A prior 2⁶⁻² fractional factorial screening design (Resolution IV) reduced the original 6-factor space to these 4 factors. Two factors showed no significant effect and were eliminated before this study.

---

## Phase 1 – Baseline SPC

**Settings:** x = [35 W, 18 mTorr, 450 W, 150 sccm] — within the validated operating window but not optimized.

**IMR Chart – Phase 1**

![Phase 1 IMR Chart](plots/spc_phase1.png)

*→ No control limit violations. No patterns. Process is statistically in control.*

**Capability – Phase 1:**

| Metric | Value | Assessment |
|---|---|---|
| X̄ | 90.5 nm/min | Below spec center (105 nm/min) |
| Cp | 4.07 | High intrinsic capability |
| **Cpk** | **1.22** | **< 1.33 → Not capable** |
| Pp | 3.65 | — |
| Ppk | 1.09 | — |

**Conclusion:** Process is stable but not capable. The mean (90.5 nm/min) is shifted toward LSL, indicating suboptimal factor settings. DoE required to identify significant factors and shift the mean toward the spec center.

---

## Phase 2 – Design of Experiments (DoE)

### Experimental Design

A **Central Composite Design (CCD, face-centered, α = 1)** was used, implemented via sequential experimentation:

| Stage | Runs | Purpose |
|---|---|---|
| 2⁴ full factorial + 4 center points | 20 | First-order model + curvature test |
| 8 axial points | 8 | Second-order (RSM) model |
| **Total** | **28** | **Full CCD** |

Face-centered design (α = 1) was chosen to keep all experimental points within the validated process boundaries.

### Sequential Experimentation

**Step 1 – Curvature Test (first-order model on factorial + center points):**

> Lack-of-Fit p = 0.0212 < 0.05 → Significant curvature detected → Quadratic terms required

Decision: proceed to full CCD by running the 8 axial points.

**Step 2 – Full RSM Model (all 28 runs):**

A full second-order model was fitted, then reduced by removing non-significant terms (backward elimination), with Lack-of-Fit as the primary stopping criterion:

- Chamber Pressure (x₂): main effect not significant → removed
- Interaction x₁:x₃: not significant → removed
- **Stopping criterion: Lack-of-Fit p = 0.066 > 0.05** (model structure adequate)

**Note on Lack-of-Fit:** After removing x₂ from the model, R reports a 
> significant LoF (p << 0.05) — however, this is a statistical artifact of 
> model reduction, not a genuine model failure. By removing x₂, points that 
> differ only in x₂ are treated as replicates, inflating the Pure Error 
> degrees of freedom (13 df) artificially. The resulting LoF F-test is no 
> longer validly interpretable.
> 
> The relevant benchmark is the LoF from the full model with genuine Pure 
> Error (4 center points, 3 df): **p = 0.066 > 0.05 → no significant 
> lack-of-fit**. Prediction accuracy at the optimum confirms model utility: 
> predicted 102.34 nm/min vs. observed X̄ = 98.93 nm/min (Δ = 3.4%, within 
> process noise).
> 
**Significant effects identified:** x₁ (Bias Power), x₃ (RF Power), x₄ (CHF₃ Flow) and their interactions.

### Response Surface

**Heatmap: RF Power vs. CHF₃ Flow** (x₁, x₂ fixed at optimum)

![Response Surface](plots/rsm_heatmap.png)

*Maximum etch rate at high RF Power and high CHF₃ Flow. Clear gradient across the design space.*

### Optimal Process Settings

Canonical analysis confirmed a maximum within the design space. Ridge analysis + constrained optimization identified:

| Factor | Coded | Real value |
|---|---|---|
| Bias Power (x₁) | −0.11 | **29.4 W** |
| Chamber Pressure (x₂) | 0.00 | **19.6 mTorr** |
| RF Power (x₃) | +1.00 | **482.8 W** |
| CHF₃ Flow (x₄) | +0.82 | **156.7 sccm** |
| **Predicted etch rate** | | **102.34 nm/min** |

---

## Phase 3 – Verification SPC

**Settings:** Optimized parameters from DoE applied. n = 15 batches monitored.

**IMR Chart – Phase 2**

![Phase 2 IMR Chart](plots/spc_phase2.png)

*→ Process remains in control. Tighter clustering around X̄ ≈ 98.9 nm/min.*

**Capability – Phase 2:**

| Metric | Phase 1 | Phase 2 | Δ |
|---|---|---|---|
| X̄ | 90.5 nm/min | 98.9 nm/min | +8.4 |
| Cp | 4.07 | 2.45 | — |
| **Cpk** | **1.22** | **1.70** | **+0.48** |
| Pp | 3.65 | 2.28 | — |
| Ppk | 1.09 | 1.59 | +0.50 |

**Conclusion:** Process in control and capable. Cpk = 1.70 > 1.33 — semiconductor industry standard met. Mean shifted from 90.5 → 98.9 nm/min, closer to spec center (105 nm/min).

---

## Summary

```
Baseline:   Cpk = 1.22  →  in control, NOT capable
                              ↓
            DoE (CCD, 28 runs, RSM)
            Significant factors: Bias Power, RF Power, CHF₃ Flow
                              ↓
Optimized:  Cpk = 1.70  →  in control, CAPABLE  ✓
```

---

## Tools & Methods

| Tool | Purpose |
|---|---|
| Python | Process simulation, data generation, Excel export |
| R (`rsm` package) | CCD design, RSM modeling, canonical analysis, ridge analysis |
| Excel | IMR control charts, process capability indices (Cp, Cpk, Pp, Ppk) |

**Statistical methods applied:**
- Individuals & Moving Range (IMR) control charts
- Process capability analysis (Cp, Cpk, Pp, Ppk)
- Central Composite Design (CCD, face-centered)
- Response Surface Methodology (RSM)
- Curvature test (Lack-of-Fit)
- Sequential experimentation principle
- Canonical analysis & ridge analysis
- Backward model reduction with Lack-of-Fit stopping criterion

---

## Repository Structure

```
├── simulation/
│   └── process_simulator.py     # SiO₂ RIE process simulation (Python)
├── doe/
│   └── doe_analysis.R           # CCD design, RSM, optimization (R)
├── spc/
│   └── spc_workbook.xlsx        # IMR charts, capability indices (Excel)
├── plots/
│   ├── spc_phase1.png
│   ├── rsm_heatmap.png
│   └── spc_phase2.png
└── README.md
```

---

## Background & Motivation

This project was developed to apply SPC and DoE methodology in a semiconductor Fab context, bridging a background in process engineering (mass balancing, process optimization, BASF Schwarzheide) with the data-driven process control methods used in high-volume semiconductor manufacturing.

The simulation intentionally hides the true model parameters and optimum — reflecting real Fab conditions where the engineer must infer process behavior from experimental data alone.

*Complementary coursework: DoE (Udemy), Statistical Process Control (Udemy, accredited), Cleanroom Fundamentals & Semiconductor Technologies (Coursera).*
