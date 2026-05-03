# Supplementary Data S3 — Nebraska Maize Software Portability Test

*Supplementary to: AquaAgent-OSPy: An Auditable LLM Tool-Agent for Irrigation Decision Support*

---

## Purpose

This supplementary provides detailed results for the AquaAgent-OSPy software portability test on Nebraska maize. This test was conducted to confirm that the pipeline is not paddy-rice-specific at the code level. It does **not** constitute a quantitative crop model benchmark or an agronomic generalization claim.

---

## Site and Configuration

| Parameter | Value |
|---|---|
| Site | Lancaster County, Nebraska, USA (41.0°N, 96.7°W) |
| Crop | AquaCrop-OSPy `Crop('Maize')` — default uncalibrated parameters |
| Soil | AquaCrop-OSPy `Soil('SiltLoam')` — default |
| Initial water content | Field Capacity (FC) |
| Planting date | 05/01 |
| Simulation end | 10/20 |
| Weather source | NASA POWER Daily API (same as Korea block) |
| ETo method | Hargreaves-Samani (same as Korea block) |
| Years | 2015, 2019, 2022 (same years as Korea block) |

---

## Weather Summary

| Year | May–Sep Precip (mm) | May–Sep ETo (mm) | Precip/ETo | Classification |
|---:|---:|---:|---:|---|
| 2015 | 650 | 706 | 0.92 | near-balanced |
| 2019 | 620 | 669 | 0.93 | near-balanced |
| 2022 | 344 | 817 | 0.42 | drought year |

---

## Scenario Results

| Year | Strategy | Yield (t/ha) | Irrigation (mm) | Note |
|---:|---|---:|---:|---|
| 2015 | rainfed | 14.27 | 0 | near-balanced |
| 2015 | deficit_SMT30 | 14.27 | 0 | soil stays above threshold |
| 2015 | deficit_SMT50 | 14.27 | 0 | |
| 2015 | full_SMT80 | 14.27 | 75 | marginal gain |
| 2015 | fixed_7d_80pct ⚠️ | 14.27 | 220 | 220mm for 0 yield gain |
| 2019 | rainfed | 14.50 | 0 | |
| 2019 | deficit_SMT30 | 14.50 | 0 | |
| 2019 | deficit_SMT50 | 14.50 | 0 | |
| 2019 | full_SMT80 | 14.50 | 25 | |
| 2019 | fixed_7d_80pct | 14.50 | 175 | 175mm for 0.003 t/ha |
| 2022 | rainfed | 14.36 | 0 | dry year |
| 2022 | deficit_SMT30 | 14.67 | 75 | +0.31 t/ha |
| 2022 | deficit_SMT50 | 14.71 | 150 | |
| 2022 | full_SMT80 ⚠️ | 14.69 | 323 | HIGH_IRR flag |
| 2022 | fixed_7d_80pct ⚠️ | 14.69 | 389 | HIGH_IRR flag |

⚠️ = HIGH_IRRIGATION flag triggered (> 200 mm threshold)

---

## USDA NASS Comparison

| Year | Simulated rainfed (t/ha) | NASS Lancaster all-practice (t/ha) | Over-prediction |
|---:|---:|---:|---:|
| 2019 | 14.50 | 9.93 (158.2 bu/ac) | +46% |
| 2022 | 14.36 | 7.04 (112.1 bu/ac) | +104% |

*NASS sources: USDA NASS 2019 Nebraska Corn Yield county map (Lancaster = 158.2 bu/ac); USDA NASS 2022 Nebraska Corn Yield county map (Lancaster = 112.1 bu/ac). Conversion: 1 bu/ac = 0.0628 t/ha.*

Note: USDA NASS discontinued irrigated/non-irrigated practice separation for Nebraska county estimates beginning with the 2019 crop year. The above figures are all-practice averages.

---

## Known Calibration Issues

1. **Initial water content**: AquaCrop simulations start at Field Capacity by default. A deep SiltLoam starting at FC has sufficient stored water to buffer much of the 2022 drought, explaining the near-zero rainfed drought sensitivity (14.36 vs. 14.50 t/ha) while NASS shows a 29% decline (7.04 vs. 9.93 t/ha).

2. **Crop parameters**: AquaCrop default Maize is calibrated for global average conditions. Nebraska rainfed maize under continental climate and sandy loam-to-silt loam soils requires cultivar-specific HI0, canopy development, and growing degree day parameters.

3. **Soil parameters**: The AquaCrop default SiltLoam does not reflect SSURGO-mapped soil profiles for Lancaster County, which include variability in available water holding capacity, bulk density, and saturated hydraulic conductivity.

4. **Calibration pathway**: Calibration would require: (a) SSURGO/gSSURGO soil profile extraction; (b) cultivar-specific canopy and phenology parameters from published Nebraska AquaCrop studies or AmeriFlux Mead site observations; (c) comparison to AmeriFlux Mead or OpenET seasonal ET observations as an additional constraint.

---

## What This Test Confirms (Portability) vs. What It Does Not Confirm

| Claim | Status |
|---|---|
| AquaAgent-OSPy scenario harness runs on a non-rice crop configuration | CONFIRMED as software portability |
| AquaCrop Maize crop name resolves | ✅ CONFIRMED |
| Safety-flag mechanism can be evaluated in the maize run | CONFIRMED as software behavior |
| Qualitative scheduling patterns | DIAGNOSTIC ONLY — not agronomically validated because rainfed drought sensitivity is weak |
| Absolute yield levels match USDA NASS county data | ❌ NOT CONFIRMED — over-predicts by 46–104% |
| Rainfed drought sensitivity captured | ❌ NOT CONFIRMED — near-zero sensitivity vs. 29% NASS decline |
| Quantitative maize advisory applicable | ❌ NOT APPLICABLE — requires site-specific calibration |

---

*Data file: `data/processed/scenario_results_nebraska_maize.csv` (15 rows: 3 years × 5 strategies)*  
*Pareto file: `data/processed/pareto_front_nebraska_maize_2022.csv` (30 solutions, 2022 dry year only)*  
*Benchmark details are summarized in this supplementary note and in the processed CSV files included with the reproducibility package.*
