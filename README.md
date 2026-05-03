# AquaAgent-OSPy

**A simulation-grounded LLM tool-agent for auditable irrigation decision support**

## Overview

AquaAgent-OSPy is an A4-class code-executing LLM tool-agent for irrigation decision support. The agent orchestrates AquaCrop-OSPy crop-water simulations, NSGA-II multi-objective optimization, GEE-based site-context verification, and memo faithfulness auditing.

**This system advises — it does not control.** All recommendations require human agronomist approval.

The current manuscript target is Computers and Electronics in Agriculture (Elsevier). This public snapshot contains the reproducibility package associated with the submission draft. Internal coordination logs, raw agent scratch outputs, credentials, and private run metadata are intentionally excluded.

## Quick Start

```bash
conda create -n aquaagent python=3.11 && conda activate aquaagent
pip install -r requirements.txt
export ANTHROPIC_API_KEY=your_key_here

# Run scenario matrix
python3 scripts/run_aquacrop_scenarios.py --cases experiments/case_registry.yml

# Generate memo
python3 scripts/generate_action_memo.py \
  --scenario-csv data/processed/scenario_results_confirmed_sites.csv \
  --pareto-csv   data/processed/pareto_front_case1_gimje_2019.csv \
  --weather-csv  data/processed/weather_aquacrop_case1_gimje_2019.csv \
  --case-id case1_gimje --year 2019 \
  --output experiments/memos/memo_case1_gimje_2019.md \
  --trace  data/processed/tool_trace_case1_gimje_2019.json
```

## Key Results

| Finding | Value |
|---|---|
| Validation sites | Gimje (Jeollabuk-do) + Hampyeong (Jeollanam-do), Korea |
| GEE G1 verification | crops_p = 0.571 / 0.506 — both confirmed cropland |
| Evidence scope | auditable DSS workflow validation, not field-calibrated yield prediction |
| Evaluation sites | Gimje + Hampyeong, Korea; 2 sites × 3 years |
| Site-context verification | GEE Dynamic World, Sentinel-2 NDVI, Sentinel-1 SAR |
| Deficit vs fixed comparison | limited to evaluated Korean paddy site-years under shared AquaCrop-OSPy assumptions |
| Memo faithfulness rubric | pilot unit-audit; not a statistical LLM reliability estimate |
| Repository status | public reproducibility snapshot |

## Agentic Classification

| Level | Label | Example |
|---|---|---|
| A2 | Tool-using RAG advisory | CottonBot 2025 |
| **A4** | **Code-executing scientific agent** | **AquaAgent-OSPy (this work)** |
| A5 | Control/actuation agent | aquacrop-gym DRL |

## Data Sources

- Weather: NASA POWER Daily API (no key required)
- Soil: Korean national drainage map (HSG) + SoilGrids v2.0 cross-check
- Site confirmation: GEE Dynamic World V1, Sentinel-2 SR, ERA5-Land
- AquaCrop: AquaCrop-OSPy 3.0.12, PaddyRice parameterization
- Optimizer: pymoo 0.6.1.6, NSGA-2
- LLM agent: Anthropic Claude Sonnet 4.6

## Notes

1. GEE/Sentinel checks are site-context verification, not field validation.
2. Results are within-model irrigation strategy comparisons under shared assumptions.
3. The 200 mm safety flag is an illustrative operational constraint unless local policy evidence is added.
4. Hargreaves-Samani ETo can over-trigger irrigation relative to Penman-Monteith; strategy ranking is more stable than absolute flag classification.

## License

No open-source license is granted in this public snapshot. The repository is provided for manuscript review and non-commercial academic reproducibility. Reuse, redistribution, or commercial use requires written permission from the authors.
