# Abstract

---

Irrigation scheduling in agricultural water management requires balancing crop yield, water efficiency, and safety constraints under inter-annual rainfall variability. Existing approaches - fixed irrigation calendars, numerical optimizers, and conversational AI advisors - share a common gap: they do not deliver model-grounded recommendations that are simultaneously traceable, safety-checked, and adaptive to climate conditions. AquaAgent-OSPy is an A4-class code-executing large language model (LLM) tool-agent that orchestrates AquaCrop-OSPy irrigation scenario simulations, non-dominated sorting genetic algorithm II (NSGA-II) multi-objective optimization, Google Earth Engine (GEE) site context verification, and an illustrative seasonal-irrigation safety checker to generate auditable action memos with full tool provenance. The system was evaluated at two confirmed Korean paddy rice sites (Gimje, Jeollabuk-do and Hampyeong, Jeollanam-do) across three contrasting climate years (403-824 mm May-September growing-season precipitation) using a five-item binary faithfulness rubric (F1-F5). As a pilot unit-audit of memo traceability, not a statistical reliability estimate for LLM behavior, four confirmed-site action memos passed all rubric items (20/20 combined). The evaluation produced four findings: (1) the GEE land-cover and Sentinel-1 synthetic aperture radar (SAR) pipeline caught an automated site-selection error and confirmed paddy rice phenology at both sites; (2) in a severe drought year (2015, 403 mm), the fixed weekly schedule triggered the illustrative high-irrigation safety flag (239 mm > 200 mm) and achieved lower water productivity (WP) than the selected low-irrigation threshold strategy under shared AquaCrop-OSPy assumptions; (3) across all four primary evaluation cases, the agent recommended soil-moisture threshold (SMT) scheduling or zero irrigation and never recommended a safety-flagged strategy, while scenario results from all six case-year combinations supported the same within-model water-saving advisory pattern; (4) AquaCrop-OSPy PaddyRice outputs were qualitatively aligned with a published Thailand site (Veerakachen and Raksapatcharawong, 2020; rainfed yield 4.77 t/ha within a reported 3-5 t/ha range), providing a plausibility check rather than Korean field validation. Soil hydraulic parameters were cross-checked against SoilGrids v2.0. AquaAgent-OSPy is intended for agronomist review: it links recommendations to simulation outputs and safety checks, but it does not authorize or actuate irrigation.

**Keywords**: AquaCrop-OSPy; irrigation decision support; four-layer decision-support architecture; large language model tool-agent; agentic AI; water productivity; Google Earth Engine; Sentinel-1 SAR; action memo; Korean paddy rice

---


# 1. Introduction

---

Irrigation scheduling decisions in agricultural water management require balancing competing objectives: maximizing crop yield, minimizing water consumption, satisfying water-use constraints, and responding to weather and soil uncertainty in near-real time. In practice, most irrigation recommendations are delivered through one of three paths: rule-of-thumb fixed calendars prescribed by extension services, optimization tools that require expert setup and interpretation, or conversational AI assistants that retrieve agronomic knowledge but do not execute crop-model simulations. Each path has a characteristic failure mode with measurable consequences. Fixed calendars do not respond to inter-annual rainfall variability: in the severe-drought site-year evaluated here (403 mm May-September growing-season rainfall), a standard weekly irrigation calendar triggered an illustrative seasonal over-irrigation flag (239 mm > 200 mm threshold) and achieved lower water productivity than the selected low-irrigation threshold strategy under the same AquaCrop-OSPy assumptions. Optimization tools produce Pareto-efficient trade-offs but require specialist interpretation to translate into a specific field recommendation. Conversational AI systems generate human-readable advice but cannot guarantee that the advice is grounded in a transparent crop-water simulation, raising both traceability and hallucination concerns.

Recent work has shown that large language model (LLM) agents can call external tools, execute code, observe outputs, and revise their reasoning in multi-turn loops (Zhang et al., 2026). In the agricultural domain, CottonBot (Kandamali et al., 2025) demonstrated that a tool-using LLM can integrate soil-moisture sensor readings, weather forecasts, and a rule-based irrigation decision engine to deliver field-specific recommendations. SPADE (Lee et al., 2025) showed that LLMs can interpret soil-moisture time series and generate readable anomaly reports. These systems represent important advances but share a common gap: they do not execute a crop-growth model during the advisory session, so the recommendations cannot be directly traced to simulated yield-water outcomes.

We address this gap with AquaAgent-OSPy, an A4-class code-executing advisory agent that orchestrates AquaCrop-OSPy (Kelly and Foster, 2021) irrigation scenario simulations as part of its tool-calling loop. The agent is positioned using the A0-A5 taxonomy defined in Section 4.1: it is explicitly not an A2 retrieval-augmented generation (RAG) advisory tool (which does not execute simulations) and not an A5 autonomous irrigation controller (which closes a field actuation loop). It occupies the A4 slot: it executes crop-model code, observes scenario outputs, checks multi-objective optimizer results, flags safety violations, and generates action memos in which every quantitative recommendation is traceable to a simulation row. A human agronomist must approve and execute the recommendation; the agent advises, not controls.

This study does not claim novelty in AquaCrop modeling, NSGA-II optimization, or satellite land-cover classification individually. Its novelty lies in an auditable agricultural LLM tool-agent architecture that integrates crop-model execution, geospatial site verification, Pareto-front inspection, safety screening, provenance logging, and numeric faithfulness auditing to generate traceable irrigation action memos for human review. The unresolved computational problem is not merely running AquaCrop or generating a text memo, but ensuring that a natural-language irrigation recommendation produced by an LLM remains numerically grounded in model outputs, site verification results, optimizer trade-offs, and safety constraints.

This work contributes an operational A4-class LLM tool-agent for AquaCrop-OSPy irrigation advisory, with a five-tool registry, provenance logging, and a binary memo-faithfulness rubric (F1-F5) that distinguishes legitimate derived calculations from hallucinated numbers. It also contributes a GEE-based site context verification pipeline (G1-G4) showing that satellite land-cover and normalized difference vegetation index (NDVI) seasonality checks can serve as a practical data-quality gate, including detection and correction of an automated site-selection error before scenario simulation. The empirical evaluation compares two Korean paddy rice sites across three simulation years per site and shows that low-irrigation threshold strategies, particularly SMT30 or Pareto-selected SMT values near 40-60%, generally reduce irrigation relative to fixed weekly scheduling and often improve water productivity under the evaluated within-model assumptions. Finally, the study reports soil-parameter and missing-data sensitivity checks that define deployment boundaries for the system.

The paper is organized as follows. Section 2 reviews prior work and positions the advisory gap. Section 3 describes the data and site selection procedure. Section 4 presents the system design, methods, and A0-A5 classification framework. Section 5 reports results. Sections 6 and 7 discuss and conclude.

---


# 2. Related Work and Paper Positioning

---

## 2.1 Autonomy Levels in Agricultural Decision Support Systems

Agricultural decision support systems span a wide range of autonomy, from passive knowledge retrieval to autonomous control actuation. The A0-A5 classification ladder used in this paper (defined in full in Section 4.1) is organized around minimum observable system behaviors and is presented alongside the system design rather than as a standalone classification contribution.

The practical gap this paper addresses lies between existing tool-using RAG advisory systems (A2-class: CottonBot, SPADE) and simulation-grounded advisory agents (A4-class): among the systems reviewed here, none executes a crop-growth model during the advisory session, traces each numeric recommendation back to a simulation row, and formally checks a safety constraint within a single auditable tool-call loop. Existing A2 systems retrieve agronomic rules but do not execute crop models. Existing classical multi-objective optimization tools such as NSGA-III wheat scheduling (Lyu et al., 2022) produce Pareto-efficient schedules but delegate natural-language interpretation to specialists. The A4 design in this paper bridges these two capabilities.

---

## 2.2 AquaCrop-OSPy and Irrigation Scheduling

AquaCrop is the Food and Agriculture Organization of the United Nations (FAO) crop-water productivity model for simulating yield response to water (Steduto et al., 2009a, 2009b; Raes et al., 2009). The Python wrapper AquaCrop-OSPy (Kelly and Foster, 2021) makes the model accessible inside Python workflows and has enabled a growing body of irrigation scheduling and optimization research.

Chung (2010) applied FAO-AquaCrop to Korean rice evapotranspiration and yield-response simulations under climate-change conditions. That work provides Korean AquaCrop rice application context, while the present study remains a site-specific advisory workflow evaluation rather than a field-calibrated yield validation.

Lyu et al. (2022) coupled AquaCrop-OSPy with NSGA-III for multi-objective winter wheat irrigation strategy optimization, demonstrating that the yield-water tradeoff Pareto front provides more actionable guidance than single-objective optimization. This work establishes the optimization baseline against which our advisory agent is compared.

Remote-sensing and data-assimilation extensions of AquaCrop are outside the implemented scope of this study. Our GEE verification pipeline checks site context and crop identity but does not implement direct state assimilation — a recognized limitation addressed in the discussion.

Kelly et al. (2024) introduced aquacrop-gym, an OpenAI Gym environment wrapping AquaCrop-OSPy for training and evaluating deep reinforcement learning (DRL) irrigation policies. This is the closest prior A5 work in our domain. AquaAgent-OSPy differs from aquacrop-gym in both architecture (LLM tool-agent vs DRL policy network) and design objective (advisable action memo vs trained irrigation control policy).

---

## 2.3 LLM Agents in Agricultural Applications

CottonBot (Kandamali et al., 2025) is the most closely related LLM irrigation advisory system. It uses Ollama-hosted LLMs, FastAPI, and Flutter to deliver irrigation recommendations from soil-moisture sensor readings and weather forecast APIs. CottonBot does not execute a crop-growth simulation; its recommendations are retrieved from agronomic rules and sensor thresholds. This paper's approach extends CottonBot's A2 architecture to A4 by integrating a crop-model simulation as a callable tool.

The GEE site context verification pipeline in this paper uses the Dynamic World V1 near-real-time land-cover product (Brown et al., 2022) for G1 cropland confirmation. Brown et al. (2022) demonstrate that Dynamic World's crops probability layer provides 10-m ground-resolved land-cover classification updated at roughly 5-day Sentinel-2 revisit frequency, enabling rapid site-specific crop identity checking without downloading and processing raw imagery.

Zhang et al. (2026) define the AgriWorld World Tools Protocol for agricultural reasoning agents, including tools for geospatial queries, remote sensing time series, crop-growth simulation, and predictor models. It is the closest architectural precedent for the A4 design in this work. AquaAgent-OSPy specializes the AgriWorld pattern for AquaCrop-OSPy irrigation advisory with explicit safety checking and a formal memo-faithfulness rubric.

Lee et al. (2025) showed through SPADE that LLM-based pattern recognition over soil-moisture time series can generate meaningful irrigation event reports. The action memo format in our system is informed by SPADE's approach to human-readable soil-water interpretation, but grounds the memo in simulation scenario outputs rather than sensor pattern matching.

---

## 2.4 Knowledge Gap and Paper Positioning

Synthesizing the above, existing AI advisory systems either function as conversational wrappers for static agronomic knowledge (A1-A2-class) or as black-box optimizers that do not produce human-auditable reasoning (classical multi-objective tools). No current published system provides an auditable, code-executing bridge that allows an irrigation advisor to trace a recommended strategy directly to its specific simulated yield-water tradeoff, verify that the agent did not hallucinate numeric claims, and confirm that a safety constraint on seasonal water use was formally checked. The faithfulness rubric (F1-F5) introduced in this paper operationalizes this traceability requirement. The GEE-assisted site verification pipeline (G1-G4) addresses a separate gap: none of the reviewed decision support systems (DSS) include an automated data-quality gate that checks whether the target field coordinates correspond to the crop type assumed by the simulation model.

---


# 3. Study Area, Data, and Site Selection

---

## 3.1 Study Region

This study focuses on transplanted paddy rice cultivation in the Jeolla provinces of southwestern Korea (34.5°–36.5°N, 126°–128°E). The region contains approximately 40% of Korea's total paddy rice area and encompasses a gradient of climate conditions from cooler inland basins in Jeollabuk-do to warmer coastal plains in Jeollanam-do. Growing-season precipitation (May–September) ranges from approximately 500–900 mm in drier years to over 1,200 mm in years with extended summer monsoon, creating year-to-year irrigation requirement variability that tests the agent's adaptive recommendation capability.

Two administrative jurisdictions were selected: Gimje-si, Jeollabuk-do, recognized as one of Korea's most extensive paddy plains (Honam plain); and Hampyeong-gun, Jeollanam-do, a coastal agricultural county with historically lower elevation and higher rainfall.

---

## 3.2 Local Geospatial Data

Three national-scale Korean geospatial layers were used for site screening and soil-context assignment. The first layer was a polygon land-use database that distinguishes consolidated and non-consolidated paddy-rice areas across the Korean Peninsula. The second layer was a national soil-drainage database that reports drainage class and hydrologic soil group information, which was used to assign an initial soil hydraulic class where field-measured hydraulic properties were not available. The third layer was a shallow-soil texture database used to check whether the selected paddy plains were consistent with expected Korean topsoil texture classes. Dominant regional textures in the study area are sandy loam (57% of polygons) and loam (21%).

---

## 3.3 Site Selection Procedure

### 3.3.1 Candidate identification

Candidate paddy sites were identified by spatially filtering national land-use polygons for paddy-rice classes and then converting candidate centroid coordinates to geographic latitude and longitude. Sites were selected from two geographically separated sub-regions (Jeollabuk-do for Case 1, Jeollanam-do for Case 2) to maximize climate contrast.

### 3.3.2 GEE verification and correction

An initial set of candidates derived from large aggregated polygon boundaries (area ≈ 3.8 × 10⁶ m²) failed the G1 land-cover check: European Space Agency (ESA) WorldCover v200 classified one centroid as Grassland (class 30) and another as Water (class 80), indicating that the centroids did not fall within cultivated paddy fields. This result demonstrates that large polygons in the national land-use database represent administrative-level paddy zones rather than individual farm-scale fields, and that automated centroid extraction from such polygons is not a reliable site selection strategy without satellite-based verification.

Candidates were revised by sampling Dynamic World V1 crops-class probability at published Korean rice-plain locations. Two sites achieving crops probability > 0.35 during the growing season (July-October 2022) were confirmed as Case 1 and Case 2; their verification metrics are summarized in the Results.

### 3.3.3 Confirmed sites

**Case 1; Gimje, Jeollabuk-do**: 35.754°N, 126.898°E. Located in the Honam plain, Korea's most productive paddy region. GEE G1 crops probability = 0.571. NDVI peak September 2022 = 0.773, with characteristic low-NDVI transplanting dip in June-July (NDVI approx. 0.12). ERA5-Land vs NASA POWER temperature mean absolute error (MAE) = 0.8°C; precipitation correlation = 0.878.

**Case 2; Hampyeong, Jeollanam-do**: 35.070°N, 126.540°E. Coastal agricultural plain with above-average rainfall. GEE G1 crops probability = 0.506. NDVI peak September 2022 = 0.674. Temperature MAE = 0.92°C; precipitation correlation = 0.667 (G4 warning; see Section 5.1).

Both confirmed sites are shown in Figure 1.

---

## 3.4 External Data Sources

### 3.4.1 Weather

Daily weather for both sites and all three simulation years (2015, 2019, and 2022) was obtained from the NASA POWER Daily API (https://power.larc.nasa.gov), community parameter set AG (agroclimatology). Retrieved variables were daily maximum 2-m temperature, daily minimum 2-m temperature, corrected precipitation, and all-sky surface shortwave radiation. All queries returned complete records with zero missing values across six site-year combinations (2 sites × 3 years = 6 weather records, 365 rows each).

Reference evapotranspiration was computed using the Hargreaves-Samani equation as described in Section 4.2.3. The Hargreaves-Samani method was selected for its minimal input data requirements (Tmax, Tmin, extraterrestrial radiation); the known positive bias under humid conditions is acknowledged in Section 6.3.

Three simulation years were selected to represent the range of growing-season rainfall regimes. Year 2015 was added as a severe-drought test: Gimje received 403 mm May–September precipitation (−40% below the 2015–2022 site average) and Hampyeong received 578 mm. Year 2019 represents a moderate year: Gimje 675 mm (near-balanced precip/ETo ≈ 1.0), Hampyeong 824 mm (wet year). Year 2022 represents a moderate surplus: Gimje 736 mm, Hampyeong 724 mm. This three-year design exposes the advisory system to drought, balanced, and surplus conditions at both sites.

### 3.4.2 Google Earth Engine context

All satellite-derived context checks were performed using the Google Earth Engine (GEE) cloud platform (Gorelick et al., 2017), accessed via the GEE Python API.

In addition to the G1-G4 optical checks, Sentinel-1 C-band SAR VH backscatter time series were extracted for both confirmed and rejected site candidates. The VH backscatter rise from the early-season minimum (May-July, transplanting/standing water period) to the late-season peak (September, mature canopy) provides an independent paddy rice phenological indicator (Nguyen et al., 2016; Torbick et al., 2017). Confirmed sites showed mean VH rises of 8.6 dB (Gimje) and 4.8 dB (Hampyeong), consistent with the documented pattern of low early-season rice backscatter followed by canopy-driven increases. Rejected original sites showed rises of 1.5-2.1 dB, confirming their non-paddy character.

The GEE context verification used Dynamic World V1 for G1 cropland confirmation, Sentinel-2 surface reflectance harmonized imagery for monthly NDVI composites in the G2 phenological check, and ERA5-Land monthly aggregates for the G3 temperature and G4 precipitation cross-check against NASA POWER. These products were used only for site-context verification, not as direct AquaCrop-OSPy state inputs.

All context-verification outputs were archived as machine-readable provenance tables and used only to support site screening and uncertainty reporting.

---

## 3.5 Data Provenance Summary

Input sources, derivation steps, and site metadata were recorded in a machine-readable provenance registry. Weather forcing tables followed the AquaCrop-OSPy input structure (Year, Month, Day, MinTemp, MaxTemp, Precipitation, ReferenceET) and were retained with the simulation package for reproducibility.

---


# 4. System Design and Methods

---

## 4.1 System Overview

The A0-A5 agentic classification ladder (Table 1) positions AquaAgent-OSPy relative to prior work based on minimum observable system behaviors, distinguishing tool-using retrieval systems (A2) from code-executing scientific agents (A4) and autonomous control agents (A5).

**Table 1. A0-A5 agentic classification ladder for agricultural advisory systems.**

| Level | Label | Minimum requirement | Example |
|---:|---|---|---|
| A0 | RAG chatbot | Retrieves documents; no tool calls | Generic agriculture Q&A |
| A1 | LLM analytical framework | LLM interprets structured data; no persistent loop | SPADE (Lee et al., 2025) |
| A2 | Tool-using RAG advisory | LLM calls live APIs or deterministic tools | CottonBot (Kandamali et al., 2025) |
| A3 | Multi-agent advisory | Role-specialized agents coordinate and critique | general multi-agent advisory systems |
| A4 | Code-executing scientific agent | Agent executes code, observes outputs, logs artifacts | AgriWorld (Zhang et al., 2026); **AquaAgent-OSPy (this work)** |
| A5 | Control/actuation agent | Agent directly actuates field irrigation objectives | aquacrop-gym DRL (Kelly et al., 2024) |

AquaAgent-OSPy is an A4-class code-executing advisory agent for irrigation decision support. The system orchestrates AquaCrop-OSPy crop-water simulations, compares irrigation strategies, checks water-use and safety constraints, and generates auditable action memos with full tool provenance. It is explicitly not an A5 irrigation-control agent: all recommendations require human agronomist approval before implementation.

The system architecture is organized around a four-layer decision-support pattern (Figure 2). The data layer assembles weather, geospatial, satellite, soil, site metadata, and provenance-controlled inputs from NASA POWER, Google Earth Engine, Sentinel-1/2, Dynamic World, ERA5-Land, SoilGrids, and local Korean geospatial records. The executable crop-water simulation layer runs AquaCrop-OSPy (v3.0.12; Kelly and Foster, 2021) with open-data weather, crop and soil inputs, irrigation strategies, and NSGA-II optimization in pymoo (v0.6.1.6), functioning as a digital-twin-like representation of irrigation response rather than a fully calibrated field twin. The intelligence layer provides an LLM tool-agent interface with five structured evidence-retrieval categories, inspects scenario and Pareto outputs, checks constraints, and drafts the memo. The DSS output layer presents the resulting recommendation as a safety-checked, auditable action memo for agronomist review, with F1-F5 faithfulness auditing used to test whether the memo stays inside recorded tool evidence.

---

## 4.2 AquaCrop-OSPy Scenario Harness

### 4.2.1 Crop model

AquaCrop is the FAO crop-water productivity model for simulating yield response to water in herbaceous crops (Steduto et al., 2009a). The Python implementation, AquaCrop-OSPy (Kelly and Foster, 2021), enables programmatic scenario execution within Python workflows.

We use the PaddyRice crop parameterization, which reflects transplanted paddy rice with a reference harvest index of 0.43 and a base crop coefficient Kcb = 1.1. We specify transplanting date 25 May, consistent with standard Korean rice management, and a simulation end date of 15 October. This captures the full growing season from transplanting through physiological maturity.

No site-specific AquaCrop calibration was performed. The default PaddyRice parameters represent a generalized Asian lowland paddy rice. Korean AquaCrop rice application context is available from Chung (2010), but it does not provide site-specific calibration for the two fields used here. As an additional plausibility check, we reproduced conditions from the Suphan Buri, Thailand site studied by Veerakachen and Raksapatcharawong (2020) using the same default PaddyRice parameters; the simulated rainfed yield (4.77 t/ha) fell within their reported rainfed yield range (3-5 t/ha), providing external evidence that the default parameters produce agronomically plausible results for transplanted Asian monsoon rice. Results in this paper are interpreted as relative strategy comparisons, not absolute yield predictions; this boundary is revisited in Section 6.

### 4.2.2 Soil

Both case sites were first assigned ClayLoam soil, consistent with Hydrologic Soil Group (HSG) C in the Korean national soil drainage database. The AquaCrop-OSPy ClayLoam profile specifies field-capacity theta-FC = 0.390 m³/m³ and wilting-point theta-WP = 0.230 m³/m³ for all 12 computational compartments (0.1 m spacing to 1.2 m depth). Final scenario tables reported in the Results use SoilGrids-corrected field-capacity values (0.306 for Gimje and 0.322 for Hampyeong), while retaining the same wilting-point value and crop parameterization; the default ClayLoam profile is retained here only as the initial assignment and comparison baseline. Throughout the paper, WP denotes water productivity; soil wilting point is written as theta-WP.

### 4.2.3 Weather inputs

Daily weather was obtained from the NASA POWER Daily API (community AG) for both sites and all three simulation years (2015, 2019, and 2022). Retrieved variables were daily maximum 2-m temperature, daily minimum 2-m temperature, corrected precipitation, and all-sky surface shortwave radiation. Reference evapotranspiration (ETo) was computed using the Hargreaves-Samani (HS) equation with FAO-56 extraterrestrial radiation (Allen et al., 1998): ETo = 0.0023 × (Tmean + 17.8) × (Tmax − Tmin)^0.5 × Ra × 0.408.

No missing values were detected in any of the six weather records (2 sites × 3 years, 365 days each). Growing-season (May–September) precipitation and ETo summaries are reported in Table 2.

**Table 2. Growing-season weather summary (May–September).**

| Site | Year | P (mm) | ETo (mm) | P/ETo | Climate class |
|---|---:|---:|---:|---:|---|
| Gimje | **2015** | **403** | **712** | **0.57** | **Severe drought** |
| Gimje | 2019 | 675 | 678 | 1.00 | Near-balanced |
| Gimje | 2022 | 736 | 673 | 1.09 | Moderate surplus |
| Hampyeong | **2015** | **578** | **485** | **1.19** | **Dry-relative** |
| Hampyeong | 2019 | 824 | 477 | 1.73 | Wet |
| Hampyeong | 2022 | 724 | 475 | 1.52 | Moderate surplus |

### 4.2.4 Irrigation strategy set

Five irrigation management strategies were evaluated for each case as interpretable baselines before Pareto optimization. The rainfed strategy applied no irrigation and represents the lower-input reference condition. The fixed weekly strategy applied irrigation every seven days and refilled the root zone to 80% of total available water, representing a calendar-based management rule that does not respond to rainfall timing or soil-water status. Three threshold strategies then used simulated soil-water depletion as the trigger: deficit SMT30 irrigated when depletion reached 30% of total available water, deficit SMT50 used a 50% depletion trigger, and full SMT80 used an 80% depletion trigger. These strategies were included because they are easy to explain in an advisory memo and provide direct contrasts between no irrigation, calendar irrigation, deficit threshold irrigation, and high-irrigation threshold management.

---

## 4.3 Multi-Objective Optimizer

We formulated an irrigation scheduling optimization problem with two continuous decision variables: soil moisture threshold (SMT, 10-90%) and maximum irrigation depth per event (MaxIrr, 10-80 mm). Two objectives were minimized simultaneously: f1 = −yield, which is equivalent to maximizing yield, and f2 = seasonal irrigation depth, which minimizes water use.

The NSGA-II algorithm (Deb et al., 2002) was used with population size 30 and 30 generations. Each function evaluation runs AquaCrop-OSPy for one complete growing season. The Pareto front was computed for four site-year combinations: Case 1 Gimje 2015 (dry year), Case 1 Gimje 2019 (near-balanced), Case 2 Hampyeong 2015 (dry-relative), and Case 2 Hampyeong 2022 (moderate surplus). The dry-year runs (Gimje 2015, Hampyeong 2015) were added after the initial two-site evaluation to test the optimizer under water-stressed conditions.

---

## 4.4 Site Selection and GEE Context Verification

### 4.4.1 Initial site candidates

Candidate paddy sites were identified by querying the Korean national land-use database for paddy-rice polygons within the Jeollabuk-do and Jeollanam-do provincial boundaries.

### 4.4.2 GEE context verification

GEE-based context checks (G1-G4) were applied to all candidate sites before scenario execution. G1 used Dynamic World V1 growing-season median crops probability from July to October 2022 and required crops probability > 0.35. G2 used Sentinel-2 surface reflectance NDVI monthly composites and required growing-season NDVI peak ≥ 0.40 with a peak at least 0.15 above early-season NDVI. G3 compared ERA5-Land and NASA POWER monthly mean temperature and required MAE < 2.5°C. G4 compared ERA5-Land and NASA POWER monthly precipitation and required Pearson correlation > 0.70.

### 4.4.3 Site correction by G1

Initial centroid coordinates derived from large aggregated polygon boundaries (3.8 × 10⁶ m²) failed G1: ESA WorldCover classified one centroid as Grassland and the other as Water body. Candidate coordinates were revised by iteratively sampling known Korean rice-plain locations until G1 and G2 criteria were satisfied. The final confirmed sites and their G1-G4 results are summarized in the Results.

The G4 warning for Case 2 (Hampyeong, 2022) reflects divergence between ERA5-Land and NASA POWER monthly precipitation during the 2022 monsoon season. NASA POWER, used as the primary forcing, is treated as the reference; the ERA5 discrepancy is noted as a weather uncertainty source (Section 6).

---

## 4.5 LLM Tool-Agent Architecture

### 4.5.1 Agent classification

Following the agentic taxonomy in Section 4.1 (Table 1), AquaAgent-OSPy is classified as an **A4 code-executing scientific agent**: it retrieves external data, executes AquaCrop-OSPy simulations, observes outputs, and revises its analysis through a multi-turn tool-calling loop. It is not an A5 irrigation-control agent: the agent recommends; a human agronomist approves and executes.

In comparison with adjacent prior work, CottonBot (Kandamali et al., 2025) is treated as A2 because it uses RAG plus weather and sensor APIs to advise without crop-model execution. Zhang et al. (2026) and AgriWorld are treated as the closest A4 architectural precedent because they execute agricultural tools including crop-growth simulations. aquacrop-gym (Kelly et al., 2024) is treated as A5 because it implements direct deep reinforcement learning (DRL) irrigation control around an AquaCrop-OSPy Gym environment.

### 4.5.2 Tool registry

Five tool categories are available to the agent: site-context retrieval, growing-season weather summarization, AquaCrop-OSPy scenario-table inspection, NSGA-II Pareto-front summarization, and safety-flag checking for high irrigation or low yield. These tool categories expose structured evidence to the agent without requiring the agent to read raw project files directly.

### 4.5.3 Agent loop

The agent receives a structured prompt specifying the A4 advisory role, tool usage rules, memo format requirements, and citation obligations. The interaction follows a structured tool-calling protocol: the agent requests evidence from the available tool categories, observes the returned structured results, and continues until the memo is finalized.

All tool calls, inputs, and outputs are logged as structured traces for post-hoc verification.

### 4.5.4 Memo faithfulness evaluation

Each generated memo was evaluated using a binary five-item rubric designed to test whether the natural-language advisory memo stayed inside the evidence returned by the simulation and checking tools. F1 tests traceability: every quantitative recommendation must be citeable to a scenario table row or Pareto solution. F2 tests numeric faithfulness: no numeric claim may appear unless it is present in tool output or is a valid arithmetic derivative of tool output. F3 requires at least one explicit uncertainty statement covering weather forcing, soil parameters, model limitations, or site-context uncertainty. F4 checks whether the safety result is correctly surfaced, including either a no-flag confirmation or a description of the triggered high-irrigation flag. F5 checks actionability by requiring a specific irrigation depth and a critical timing window.

The valid-number set for F1/F2 includes all tool output values, arithmetic differences and ratios of those values, and constant thresholds used by the safety tool (200 mm, 3.0 t/ha). This prevents false positives from derived calculations, such as the yield difference between two strategies, while still failing unsupported numeric claims. Section 5.4 reports the memo-level pass/fail results.

---

## 4.6 Sensitivity and Robustness Experiments

### 4.6.1 Soil parameter sensitivity

Field capacity and theta-WP were perturbed by ±15% independently and jointly to quantify yield sensitivity to soil hydraulic uncertainty. The base case (Case 1 Gimje 2019, deficit SMT50) was used for all sensitivity runs.

### 4.6.2 Missing-data stress test

A 14-consecutive-day gap was injected into the Case 1 Gimje 2019 growing-season weather record (July 15–28, peak vegetative growth and flowering). AquaCrop-OSPy was run with the gapped record; yield and irrigation changes relative to the complete record were quantified.

---


# 5. Results

---

## 5.1 GEE Context Verification

The GEE context pipeline identified a site-selection error before any scenario simulation was performed. Initial centroid coordinates derived from large aggregated land-use polygons (area ≈ 3.8 × 10⁶ m²) failed G1: ESA WorldCover v200 classified the Chungnam centroid as Grassland (class 30) and the Jeonnam centroid as Water (class 80). Dynamic World crops probability at both initial centroids was below 0.10.

After iterative candidate search, two confirmed cropland sites were identified (Table 3). The G2 NDVI time series for both sites (Figure 3) shows a characteristic paddy rice phenological signature: NDVI decline in June-July (transplanting period, young sparse canopy over standing water) followed by rapid canopy development and peak in September (0.773 for Gimje, 0.674 for Hampyeong). This seasonal trajectory is consistent with Korean transplanted rice (May planting to October harvest) and provides independent evidence that the selected pixels represent active paddy rice cultivation.

**Table 3. GEE context verification results for confirmed sites.**

| Check | Dataset | Gimje (Case 1) | Hampyeong (Case 2) | Pass threshold |
|---|---|---|---|---|
| G1 Land cover | Dynamic World V1 crops probability | 0.571 (Pass) | 0.506 (Pass) | > 0.35 |
| G2 NDVI peak (Jul-Sep) | Sentinel-2 SR NDVI | 0.773 (Pass) | 0.674 (Pass) | > 0.40 |
| G2 NDVI early (Apr-May) | Sentinel-2 SR NDVI | 0.481 | 0.429 | — |
| G3 Temperature MAE | ERA5-Land vs NASA POWER | 0.8°C (Pass) | 0.92°C (Pass) | < 2.5°C |
| G4 Precipitation corr | ERA5-Land vs NASA POWER | 0.878 (Pass) | 0.667 (Warning) | > 0.70 |

The G4 warning for Hampyeong (2022 growing season) reflects a divergence between ERA5-Land and NASA POWER monthly precipitation totals during the peak monsoon months (June–July). This discrepancy is flagged as a weather forcing uncertainty in Section 6.3.

An extended GEE validation run using Sentinel-1 C-band VH backscatter time series further confirmed the site characterizations. The mean VH backscatter rise from the early-season minimum (May-July, transplanting and standing-water period) to the late-season peak (September) was 8.6 dB at Gimje and 4.8 dB at Hampyeong, matching the documented seasonal SAR pattern for rice fields in which early inundation/establishment has lower backscatter and canopy development increases the signal (Nguyen et al., 2016; Torbick et al., 2017). The originally rejected centroid candidates showed rises of only 1.5-2.1 dB, independently confirming their non-paddy classification.

---

## 5.2 AquaCrop-OSPy Scenario Comparison

Table 4 and Figure 4 present the scenario comparison across both confirmed cases and three simulation years each, using SoilGrids-corrected soil hydraulic parameters (field-capacity theta-FC = 0.306-0.322; see Section 6.3 for comparison with AquaCrop defaults). Korean paddy rice yields ranged from 3.856 t/ha (Gimje 2015 rainfed, severe drought) to 7.432 t/ha (Hampyeong 2022 full irrigation). The Korean national average paddy yield is 5.14 t/ha (Statistics Korea, 2024); the simulated range is higher, reflecting optimal irrigated field conditions and the absence of site-specific calibration. Chung (2010) provides Korean FAO-AquaCrop rice application context. The Thailand RiceSAP benchmark (Veerakachen and Raksapatcharawong, 2020) provides an additional monsoon-rice plausibility check, not site-specific validation for the Korean fields.

**Table 4. AquaCrop-OSPy scenario comparison results (SoilGrids-corrected soil parameters; selected strategies shown).**

| Site | Year | P (mm) | Strategy | Yield | Irr. | WP |
|---|---:|---:|---|---:|---:|---:|
| Gimje | 2015 | 403 | Rainfed | 3.856 | 0 | 0.958 |
| Gimje | 2015 | 403 | **deficit SMT50** (Best WP) | **6.633** | **175** | **1.148** |
| Gimje | 2015 | 403 | fixed weekly (Flag) | 7.009 | 239 | 1.093 |
| Gimje | 2015 | 403 | full SMT80 (Flag) | 7.168 | 287 | 1.040 |
| Gimje | 2019 | 675 | Rainfed | 6.142 | 0 | 0.910 |
| Gimje | 2019 | 675 | **deficit SMT30** (Best WP) | **6.613** | **50** | **0.912** |
| Gimje | 2019 | 675 | deficit SMT50 | 7.018 | 125 | 0.877 |
| Gimje | 2019 | 675 | full SMT80 (Flag) | 7.322 | 205 | 0.832 |
| Gimje | 2019 | 675 | fixed weekly | 7.240 | 144 | 0.884 |
| Gimje | 2022 | 736 | Rainfed | 6.261 | 0 | 0.851 |
| Gimje | 2022 | 736 | deficit SMT50 | 7.220 | 100 | 0.864 |
| Gimje | 2022 | 736 | fixed weekly | 7.252 | 94 | 0.874 |
| Hampyeong | 2015 | 578 | Rainfed | 6.016 | 0 | 1.041 |
| Hampyeong | 2015 | 578 | **deficit SMT30** (Best WP) | **6.362** | **25** | **1.055** |
| Hampyeong | 2015 | 578 | deficit SMT50 | 6.772 | 75 | 1.037 |
| Hampyeong | 2015 | 578 | fixed weekly | 7.138 | 143 | 0.991 |
| Hampyeong | 2019 | 824 | Rainfed | 7.197 | 0 | 0.874 |
| Hampyeong | 2019 | 824 | deficit SMT50 | 7.305 | 50 | 0.836 |
| Hampyeong | 2019 | 824 | fixed weekly | 7.307 | 114 | 0.779 |
| Hampyeong | 2022 | 724 | Rainfed | 7.233 | 0 | 1.000 |
| Hampyeong | 2022 | 724 | deficit SMT50 | 7.233 | 0 | 1.000 |
| Hampyeong | 2022 | 724 | fixed weekly | 7.408 | 89 | 0.912 |

Warning = exceeds 200 mm safety threshold. Best WP = highest water-productivity strategy for that site-year. Full 30-strategy matrix in supplementary data.

Four patterns emerge from the scenario comparison. First, low-irrigation threshold strategies, particularly SMT30 or Pareto-selected SMT values near 40-60%, generally reduced irrigation relative to fixed weekly scheduling and often improved water productivity under the evaluated within-model assumptions. In the near-balanced Gimje 2019 year, the fixed weekly schedule applied 144 mm seasonal irrigation to achieve 7.240 t/ha (WP = 0.884 kg/m³), while deficit SMT30 achieved 6.613 t/ha with only 50 mm, a 65% reduction in irrigation at higher water productivity (WP = 0.912 kg/m³). A more aggressive threshold, deficit SMT50, approached the fixed schedule's yield but did not improve WP in that moderate year, which is why the paper interprets the threshold result as a low-irrigation or Pareto-selected strategy pattern rather than a claim that every deficit threshold is superior to fixed scheduling. The Pareto optimizer independently identified a best-WP solution at SMT ≈ 40% and 49 mm seasonal irrigation (WP = 0.93 kg/m³), closely matching the deficit SMT30 strategy.

Second, threshold strategies correctly applied zero irrigation in surplus years. In Hampyeong 2022 (P/ETo = 1.52), deficit SMT50 applied 0 mm irrigation and achieved rainfed-equivalent yield (7.233 t/ha, WP = 1.000 kg/m³). The fixed weekly schedule, which does not respond to soil moisture state, applied 89 mm irrigation in the same conditions for a 0.175 t/ha yield gain (2.4%) while WP declined from 1.000 to 0.912 kg/m³. Third, high-irrigation strategies triggered the safety threshold when they crossed the illustrative 200 mm screen: in Gimje 2019, full SMT80 applied 205 mm seasonal irrigation and the Pareto max-yield solution reached 212.8 mm under the same forcing. Both are correctly flagged by the safety checker and excluded from the recommended strategy set.

Fourth, the optimal strategy shifted with drought severity. A third simulation year, 2015, was added for Gimje to test a severe drought condition (403 mm May-September, compared to 675 mm in 2019). Under this dry-year forcing, rainfed yield collapsed to 3.856 t/ha (−37% vs 2019), and the irrigation benefit became strongly positive. In the dry year the fixed weekly schedule (7.009 t/ha, 238.9 mm) triggered the high-irrigation safety flag (> 200 mm) and simultaneously achieved lower WP (1.093 kg/m³) than the deficit SMT50 strategy (6.633 t/ha, 175 mm, WP = 1.148 kg/m³). The deficit SMT50 delivered 92.5% of maximum yield with 61% less irrigation than the safety-flagged full SMT80 strategy. The Pareto optimizer confirmed the dry-year sweet spot at SMT ≈ 40%, 116 mm (WP = 1.195 kg/m³). Table 5 summarizes the three-year climate response for Gimje.

**Table 5. Three-year climate response for Case 1 Gimje (deficit SMT50 vs fixed, SoilGrids-corrected soil).**

| Year | P (mm) | Climate | Deficit WP | Fixed WP | Fixed irr. | Flag |
|---:|---:|---|---:|---:|---:|---|
| 2015 | 403 | Dry | **1.148** | 1.093 | 238.9 | High-irrigation |
| 2019 | 675 | Moderate | 0.877 | 0.884 | 143.7 | None |
| 2022 | 736 | Moderate | 0.864 | 0.874 | 93.5 | None |

The three-year analysis reveals a critical pattern absent in single-year evaluations: the fixed schedule is unsafe AND less water-efficient specifically in drought years. An advisory system that recommends fixed scheduling without considering inter-annual rainfall variability will over-irrigate and violate safety constraints precisely when water scarcity is most critical.

---

## 5.3 Pareto Optimizer Results

The NSGA-II optimizer was run for four site-year combinations, one drought year and one moderate or surplus year per site, to capture the range of irrigation trade-offs across contrasting rainfall regimes. In Gimje 2015, the severe-drought year with 403 mm growing-season precipitation, the Pareto front spanned 0-287 mm seasonal irrigation and 3.856-7.168 t/ha yield. The best-WP solution was SMT = 40% and MaxIrr = 39.5 mm, producing 6.2 t/ha yield, 116.2 mm irrigation, and WP = 1.195 kg/m³. The deficit SMT50 strategy (175 mm, 6.633 t/ha, WP = 1.148 kg/m³) lay near the Pareto efficient frontier, while both the fixed weekly (239 mm) and full SMT80 (287 mm) strategies exceeded the 200 mm safety threshold and sat outside the preferred water-productivity portion of the Pareto space.

In Gimje 2019, a moderate year with 675 mm precipitation, the Pareto front spanned 0-212.8 mm seasonal irrigation and 6.142-7.332 t/ha yield (Figure 5). The best-WP solution was SMT = 40.1% and MaxIrr = 16.5 mm, producing 6.740 t/ha yield, 49.4 mm irrigation, and WP = 0.930 kg/m³. The max-yield solution reached 7.332 t/ha but required 212.8 mm irrigation, which triggered the safety flag. The fixed weekly schedule (144 mm, 7.240 t/ha) lay off the Pareto front, while comparable-yield solutions used less than 100 mm irrigation.

In Hampyeong 2015, a dry-relative year with 578 mm precipitation, the Pareto front spanned 0-138.4 mm and no solution exceeded the 200 mm safety threshold. The best-WP solution was SMT = 57.7% and MaxIrr = 25 mm, producing 6.4 t/ha yield, 61.9 mm irrigation, and WP = 1.088 kg/m³. The deficit SMT30 scenario (25 mm, WP = 1.055 kg/m³) fell near this best-WP anchor, supporting the low-irrigation recommendation. In Hampyeong 2022, a moderate-surplus year with 724 mm precipitation, the Pareto front spanned only 0-99.3 mm; the best-WP solution was SMT = 54% and MaxIrr = 30 mm with 7.233 t/ha yield, 0.0 mm irrigation, and WP = 1.000 kg/m³.

Across the four evaluated site-years, the Pareto best-WP solution fell in the SMT 40-60% range, suggesting a useful threshold band within the tested Korean paddy cases. Figure 5 shows the Gimje 2019 Pareto front; the other three Pareto-front CSVs are included as supplementary data to avoid overloading the main figure set.

---

## 5.4 LLM Action Memo Evaluation

Four action memos were generated by the AquaAgent-OSPy tool-agent: one for each confirmed site-year used as a primary evaluation case (Gimje 2019 and 2015; Hampyeong 2022 and 2015). Each memo used five evidence categories in sequence: site context, weather summary, scenario table, Pareto front, and safety flags. This makes the evaluation unit a system trace rather than only an agronomic simulation row: a recommendation is counted only after the site was verified, weather and scenario evidence were inspected, Pareto and safety results were surfaced, and the final memo was checked for numeric faithfulness. The four-memo faithfulness results are summarized in Table 6.

### 5.4.1 Faithfulness rubric

All four memos passed all five F1–F5 rubric items (Table 6). This result supports traceability of the evaluated memo outputs; it is not a statistical reliability estimate for LLM behavior.

**Table 6. Memo faithfulness evaluation (F1–F5 rubric), all four evaluation cases.**

| Item | Gimje 2019 | Gimje 2015 | Hampyeong 2022 | Hampyeong 2015 |
|---|---|---|---|---|
| F1 Traceability | PASS | PASS | PASS | PASS |
| F2 No hallucination | PASS | PASS | PASS | PASS |
| F3 Uncertainty stated | PASS | PASS | PASS | PASS |
| F4 Safety flags | PASS (0 flags) | PASS (2 high-irrigation) | PASS (Pareto high-irrigation) | PASS (0 flags) |
| F5 Actionability | PASS | PASS | PASS | PASS |
| **Pass rate** | **5/5** | **5/5** | **5/5** | **5/5** |

F4 produced contrasting results appropriate to each climate context. In Gimje 2019 (moderate year, no strategy exceeded 200 mm), the safety checker returned 0 flags and the memo correctly stated this. In Gimje 2015 (dry year, 403 mm), the full SMT80 (287 mm) and fixed weekly (239 mm) strategies both triggered high-irrigation safety flag, and the memo correctly flagged both. In Hampyeong 2022, the Pareto max-yield solution (205.8 mm) triggered high-irrigation safety flag and the memo surfaced this warning. In Hampyeong 2015, no strategy exceeded the safety threshold, and the memo correctly returned 0 flags while recommending deficit SMT30 (25 mm) based on its superior WP (1.055 kg/m³).

### 5.4.2 Rubric methodology note

An initial implementation of the F1/F2 check using naive numeric string matching produced false positives: derived calculations (e.g., 114 mm − 75 mm = 39 mm) and weather-tool output values (total precipitation, ETo) were incorrectly flagged as untraced. The final rubric expands the valid-number set to include all values returned by any tool call, arithmetic differences and ratios of those values, and known safety thresholds. This methodological refinement is relevant for automated memo auditing systems because valid derived arithmetic must be accepted without allowing unsupported numeric claims.

### 5.4.3 Adversarial numeric audit

An adversarial unit audit tested whether F1/F2 detect numeric hallucination. Each primary memo was copied and modified with unsupported claims (999.9 mm irrigation, 99.9 t/ha yield, 9.999 kg/m3 WP). The evaluator converted all four modified memos from baseline pass to categorical F1/F2 failure (4/4 detected). This does not replace independent actionability review or large-N prompt-variation testing, but verifies that unsupported quantities are caught.

### 5.4.4 Memo recommendation comparison

The four memos made contrasting recommendations appropriate to their contexts. For Gimje 2019, a near-balanced year, the agent recommended deficit SMT50 (125 mm, 7.018 t/ha, WP = 0.877 kg/m³) as a yield-oriented compromise, while also warning that deficit SMT30 and the Pareto best-WP point provide more water-conservative alternatives for constrained supply. This distinction matters because SMT50 does not always maximize WP relative to fixed scheduling in moderate years; the memo's value is that it surfaces the scenario trade-off rather than claiming universal dominance by one threshold. For Hampyeong 2022, a moderate-surplus year, the agent recommended rainfed to deficit SMT50 management (0 mm, 7.233 t/ha) and explicitly noted the G4 precipitation warning as an uncertainty source before any field decision.

For Gimje 2015, the severe dry year, the agent recommended deficit SMT50 (175 mm, 6.633 t/ha, WP = 1.148 kg/m³) and correctly surfaced two high-irrigation safety flags, fixed weekly at 238.9 mm and full SMT80 at 286.9 mm. It also cited the Pareto best-WP solution (SMT = 40%, 116 mm, WP = 1.195 kg/m³) as an even more water-conservative option for constrained supply. For Hampyeong 2015, a dry-relative year, the agent recommended deficit SMT30 (25 mm, 6.362 t/ha, WP = 1.055 kg/m³) because no strategy exceeded the safety threshold and the fixed weekly schedule applied 143 mm for a lower WP (0.991 kg/m³). Together, the memo set shows system behavior rather than only agronomic outcome reporting: the agent inspects site context, weather, scenario rows, Pareto summaries, and safety flags before selecting a recommendation and attaching uncertainty language.

---

## 5.5 Sensitivity and Robustness

### 5.5.1 Soil parameter sensitivity

Perturbing field-capacity theta-FC and wilting-point theta-WP by ±15% produced a yield sensitivity of ±2.0% (range: 6.663–7.193 t/ha) for the deficit SMT50 strategy in Case 1 Gimje 2019. This indicates the model is robust to soil hydraulic parameter uncertainty at the ±15% level. Rainfed yield was more sensitive (5.691–6.650 t/ha, ±8.2%), because the irrigated strategy compensates for soil water deficits that rainfed cannot.

Irrigation depth was more sensitive to soil parameters than yield: the FC−15% perturbation increased deficit SMT50 seasonal irrigation from 75 mm (base) to 121 mm, because lower field capacity leads to more frequent SMT threshold crossings.

### 5.5.2 Missing-data stress test

Injecting a 14-day gap (July 15–28) into the Case 1 Gimje 2019 weather record produced a rainfed yield reduction of 0.224 t/ha (−3.5%) compared to the complete record, while the deficit SMT50 strategy was essentially unchanged (+0.003 t/ha). The irrigation strategy's robustness under mid-season data gaps arises because the SMT threshold trigger compensates for the modelled soil water deficit during the missing period. This result suggests that threshold-based scheduling is more resilient to short-term weather data outages than rainfed management, and supports the action memo's recommendation for deficit scheduling as the safer choice under forecast uncertainty.

### 5.5.3 ETo estimation method sensitivity

Hargreaves-Samani ETo is known to overestimate relative to Penman-Monteith (PM) under humid conditions. To assess whether this affects strategy rankings, ETo for the Gimje 2019 primary case was recomputed using the FAO-56 PM equation with NASA POWER temperature, solar radiation, 2-m wind speed (WS2M), and 2-m relative humidity (RH2M) inputs. PM-ETo for the growing season was 569 mm versus HS-ETo at 674 mm, an 18% reduction. Re-running all five strategies under PM-ETo preserved the WP ranking: deficit SMT30 achieved higher WP than fixed scheduling under both formulations. Absolute irrigation depths were 25-44 mm lower under PM-ETo across irrigated strategies, and full SMT80 fell from 208 mm under HS forcing to 165 mm under PM forcing, dropping below the 200 mm safety threshold. The strategy ranking was unchanged under the two ETo formulations tested, but absolute irrigation depth and high-irrigation safety-flag classification can be forcing-sensitive. The safety checker is therefore best interpreted as an operational constraint screen rather than a field-validated regulatory threshold.

---


# 6. Discussion

---

## 6.1 What This System Contributes

AquaAgent-OSPy demonstrates an auditable agricultural LLM tool-agent system for decision support rather than a claim that LLMs were merely applied to Korean paddy rice. The system integrates crop-model execution, multi-objective optimization, satellite-based site verification, safety screening, provenance logging, and post-hoc numeric faithfulness auditing in one reproducible advisory workflow. Unlike CottonBot, an A2 RAG plus weather API advisory system (Kandamali et al., 2025), or SPADE, an A1 LLM soil-moisture interpretation framework (Lee et al., 2025), the recommendation is derived from simulated yield-water trade-offs that are inspected during the advisory session. Unlike a standalone optimizer, the system also records which evidence categories were consulted, drafts a human-readable action memo, and tests whether the memo introduced unsupported numeric claims. The GEE verification layer is part of that system contribution because it caught an automated site-selection error before simulation, functioning as an active quality gate rather than decorative provenance.

The system does not close an irrigation control loop (A5). It advises; humans decide and act. This framing is intentional: for the agricultural deployment contexts targeted by this work (irrigation consulting, extension services, water management agencies), advisory systems that provide traceable recommendations are both more immediately deployable and more legally defensible than autonomous controllers.

More generally, AquaAgent-OSPy can be interpreted as a domain-specific instantiation of a four-layer environmental decision-support architecture: provenance-controlled data inputs, an executable physical-system representation, agentic intelligence for evidence synthesis and constraint checking, and human-reviewed DSS output. The present paper evaluates this architecture in the bounded case of Korean paddy-rice irrigation rather than claiming a fully general digital-twin platform.

**Architectural value over deterministic selection.** The system contribution is the traceability layer that binds an LLM-generated irrigation memo to crop-model evidence rather than simply passing through an optimizer's best-WP value. Three mechanisms create that layer: (1) the GEE G1-G4 site context verification, which is not part of any deterministic optimizer and which caught a real data error in this study; (2) the natural-language memo that frames the recommendation in terms of the site's specific climate context, safety flags, and uncertainty caveats for human review; and (3) the F1-F5 faithfulness audit mechanism, which verifies that the natural-language output does not introduce claims beyond the simulation evidence. A deterministic script can select a Pareto point, but it does not by itself record site-verification evidence, synthesize climate and safety caveats, or test whether a natural-language recommendation remains numerically grounded in model outputs and safety constraints. The agent's computational cost for a single advisory session (5 tool calls and approximately 8,000 input tokens; order-of-cents under typical low-cost LLM API pricing) is negligible relative to the water savings identified: in the Gimje 2019 test case, switching from fixed to deficit SMT30 saves approximately 94 mm of seasonal irrigation, equivalent to approximately 940 m³/ha — a water saving that exceeds the advisory inference cost by orders of magnitude under ordinary water-pricing assumptions.

Multi-crop extension is deliberately outside the main claim. A supplementary non-rice portability smoke test (Supplementary Data S3) verified that the software path can ingest a US crop/weather setup, while also showing why the evidentiary claim in this paper should stay focused on Korean paddy rice rather than uncalibrated multi-crop advice.

---

## 6.2 Irrigation Strategy Insights

The scenario comparison yields three advisory insights for the evaluated Korean paddy site-years. Low-irrigation threshold strategies, particularly SMT30 or Pareto-selected SMT values near 40-60%, generally reduced irrigation relative to fixed weekly scheduling and often improved WP under shared AquaCrop-OSPy assumptions. In the near-balanced Gimje 2019 year, deficit SMT30 (50 mm, WP = 0.912 kg/m³) exceeded the fixed weekly schedule (144 mm, WP = 0.884 kg/m³) by 3.2% in WP while using 65% less irrigation. The Pareto optimizer independently identified a best-WP solution at SMT ≈ 40%, 49 mm (WP = 0.930 kg/m³), closely tracking the deficit SMT30 strategy. This should not be read as a claim that every deficit threshold dominates fixed scheduling: Table 5 shows that deficit SMT50 has slightly lower WP than fixed scheduling in the 2019 and 2022 Gimje rows. The decision-relevant pattern is narrower and more defensible: the low-irrigation threshold or Pareto-selected option can reduce water use substantially and improve the WP trade-off when the advisory objective prioritizes water productivity.

The 200 mm safety threshold is an illustrative operational constraint screen. Full SMT80 at Gimje 2019 and the Pareto max-yield optimizer solution both triggered the 200 mm flag under HS forcing. Both represent strategies that applied large irrigation volumes for marginal yield gains over more conservative options. The action memo rejected these in favor of a lower-irrigation recommendation. The PM sensitivity run shows that absolute flag classification can change with forcing, so the threshold should be replaced by locally justified water-allocation rules before field deployment.

Year-to-year climate variation changes the optimal strategy. Between the two Gimje moderate years (2019 and 2022, P/ETo ratio 1.00 vs 1.09), deficit SMT30 irrigation changed from 50 mm to 0 mm, while deficit SMT50 shifted from 125 mm to 100 mm under the SoilGrids-corrected parameterization. In the severe drought year (2015), the agent moved to deficit SMT50 with 175 mm irrigation and explicit safety warnings for fixed and full-irrigation alternatives. An advisory system that updates recommendations seasonally is therefore more valuable than one that prescribes a fixed annual irrigation plan.

---

## 6.3 Scope and Deployment Boundaries

The present evaluation intentionally targets system-level auditability rather than field-scale agronomic validation. The validation target is the integrity of the advisory workflow: whether site identity, weather forcing, crop-model execution, optimizer output, safety flags, and memo claims remain traceable end to end. Field-scale yield prediction is a subsequent agronomic validation problem and is not required to test this system-level contribution. Within this scope, GEE, Sentinel-2 NDVI, and Sentinel-1 SAR are used as site-context verification tools that reduce the risk of simulating the wrong crop or wrong location; they are not presented as substitutes for field trials.

Input uncertainty is treated as a deployment-boundary variable rather than as a hidden assumption. The default PaddyRice parameterization provides a generalized transplanted Asian lowland rice setup, while the reported scenario tables use SoilGrids-corrected field-capacity values derived from site-specific texture estimates. Because local soil maps and SoilGrids do not resolve field-scale hydraulic properties, site-calibrated pedotransfer estimates remain necessary for operational deployment. This does not invalidate the architecture tested here: the agent workflow can ingest updated soil parameters, rerun AquaCrop-OSPy scenarios, and regenerate safety-checked memos under the same provenance and faithfulness-audit structure.

Weather forcing and ETo formulation are handled in the same way. The Hampyeong 2022 G4 precipitation warning marks a mismatch between ERA5-Land and NASA POWER monthly precipitation, and the PM-ETo sensitivity test shows that absolute irrigation depth and safety-flag classification can vary with forcing choice. These effects define operational configuration requirements for local deployment. They do not change the system contribution, which is to expose such uncertainty, propagate it into the memo, and keep each recommendation traceable to the forcing and scenario outputs actually used.

The 200 mm high-irrigation threshold is an illustrative safety screen, not a regulatory water-allocation rule. In deployment it should be replaced by locally justified limits from irrigation districts, water managers, or agronomists. The point of the present implementation is that a configurable safety rule is checked explicitly before the agent recommends a strategy, and that flagged strategies remain visible in the audit trail instead of being silently omitted.

The F1-F5 memo evaluation is a numeric traceability unit audit. It verifies that the four generated memos did not introduce unsupported quantitative claims and that adversarially inserted false numbers were detected by the evaluator. It is not presented as a large-N reliability estimate for all prompts, sites, models, or crops. Larger prompt-variation tests, independent agronomist actionability review, site-calibrated crop parameters, and additional crop-region cases are therefore deployment extensions. They extend the operational evidence base, but they are separate from the architectural contribution evaluated in this paper.

---


# 7. Conclusion

---

AquaAgent-OSPy is an auditable LLM tool-agent system for simulation-grounded irrigation decision support. Its contribution is the integrated agricultural DSS architecture: crop-model execution, satellite-based site verification, multi-objective optimization, safety screening, provenance logging, and memo faithfulness auditing are combined in one advisory workflow. In a pilot unit-audit, not a statistical reliability estimate, the system generated four irrigation action memos whose quantitative recommendations were traceable to specific simulation outputs under the 5-item F1-F5 faithfulness rubric.

The experiments showed that GEE G1 context verification caught an automated site-selection error before scenario simulation, demonstrating that satellite-based land-cover checking is a practical data-quality gate rather than a decorative provenance step. Low-irrigation threshold strategies, particularly SMT30 or Pareto-selected SMT values near 40-60%, generally reduced irrigation relative to fixed weekly scheduling and often improved the water-productivity trade-off under the evaluated within-model assumptions; in the near-balanced Gimje 2019 scenario, deficit SMT30 achieved 6.613 t/ha with 50 mm irrigation, while the fixed weekly schedule required 144 mm for 7.240 t/ha, or 2.9× the seasonal water use of deficit SMT30 for a 9.5% yield gain. The Pareto optimizer independently identified the best water-productivity strategy at SMT ≈ 40%, 49 mm (WP = 0.930 kg/m³). The A4 advisory agent also adapted its recommendations across contrasting conditions, recommending deficit SMT50 in the dry Gimje 2015 case, deficit SMT30 or Pareto-style water-saving schedules in moderate years, and low or zero irrigation in wetter Hampyeong years. The 14-day missing-weather stress test further supported threshold scheduling because deficit SMT50 yield was essentially unchanged (+0.003 t/ha), while rainfed yield declined by 0.224 t/ha.

The contribution is intentionally scoped but not merely defensive: AquaAgent-OSPy provides traceable, simulation-grounded irrigation advice for human review, with explicit separation between site-context verification, crop-model simulation, optimizer output, and memo-faithfulness checking. The next deployment steps are concrete: complete an agronomist actionability review, archive the reproducibility package for review, and add site-calibrated crop parameters where field observations are available.

---


# Declarations

Funding. This work was supported by the National Research Foundation of Korea (NRF) grant funded by the Korea government (Ministry of Science and ICT, MSIT) under Grant RS-2023-002772264.

Declaration of competing interest. The authors are affiliated with GeoAI Alignment, Inc., which develops AI-agent systems for geospatial and environmental decision support. The authors declare that this affiliation did not influence the study design, analysis, interpretation, or reporting. The authors declare no other competing financial or non-financial interests relevant to the content of this article.

Ethics approval and consent to participate. Not applicable, this study uses only publicly available environmental, satellite, soil, weather, and crop-model datasets; no human subjects or animal experiments are involved.

Consent for publication. Not applicable.

Data availability. This study used public open-access environmental datasets retrieved from the NASA POWER Daily API, the Google Earth Engine data catalog, Sentinel-1/Sentinel-2 derived products, ISRIC SoilGrids v2.0, and published literature benchmarks. Processed scenario tables, Pareto fronts, tool traces, action memos, and evaluation artifacts are included in the reproducibility package or can be regenerated from the released scripts and documented data sources.

Code availability. The AquaAgent-OSPy scenario harness, NSGA-II optimizer, GEE verification workflow, memo generator, faithfulness evaluator, and manuscript source are hosted in the public project GitHub repository: https://github.com/egpark-knu/aquaagent-ospy. A public archival release of the submission-associated version can be provided according to journal policy.

Materials availability. Not applicable, no new physical materials were generated.

CRediT authorship contribution statement. E.P.: Conceptualization, Methodology, Software, Formal analysis, Visualization, Writing - original draft, Writing - review and editing, Funding acquisition. T.K.: Software validation, Formal analysis, Writing - review and editing. J.P.: Software validation, Formal analysis, Writing - review and editing. All authors read and approved the final manuscript.

---


# References

Allen, R.G., Pereira, L.S., Raes, D., & Smith, M. (1998). *Crop evapotranspiration: Guidelines for computing crop water requirements.* FAO Irrigation and Drainage Paper 56. Rome: FAO. https://www.fao.org/4/x0490e/x0490e00.htm

Brown, C.F., Brumby, S.P., Guzder-Williams, B., et al. (2022). Dynamic World, Near real-time global 10 m land use land cover mapping. *Scientific Data*, 9, 251. https://doi.org/10.1038/s41597-022-01307-4

Chung, S.O. (2010). Simulating evapotranspiration and yield responses of rice to climate change using FAO-AquaCrop. *Journal of The Korean Society of Agricultural Engineers*, 52(3), 57-64. https://doi.org/10.5389/KSAE.2010.52.3.057

Deb, K., Pratap, A., Agarwal, S., & Meyarivan, T. (2002). A fast and elitist multiobjective genetic algorithm: NSGA-II. *IEEE Transactions on Evolutionary Computation*, 6(2), 182–197. https://doi.org/10.1109/4235.996017

Gorelick, N., Hancher, M., Dixon, M., Ilyushchenko, S., & Thau, D. (2017). Google Earth Engine: Planetary-scale geospatial analysis for everyone. *Remote Sensing of Environment*, 202, 18–27. https://doi.org/10.1016/j.rse.2017.06.031

Kandamali, D.F., Porter, W.M., Porter, E., McLemore, A., & Rains, G.C. (2025). CottonBot: An AI-driven cotton farming assistant and irrigation advisor using LLM-RAG and agentic AI tools. *Smart Agricultural Technology*, 12, 101640. https://doi.org/10.1016/j.atech.2025.101640

Kelly, T.D., & Foster, T. (2021). AquaCrop-OSPy: Bridging the gap between research and practice in crop-water modeling. *Agricultural Water Management*, 254, 106976. https://doi.org/10.1016/j.agwat.2021.106976

Kelly, T.D., Foster, T., & Schultz, D.M. (2024). Assessing the value of deep reinforcement learning for irrigation scheduling. *Smart Agricultural Technology*, 7, 100403. https://doi.org/10.1016/j.atech.2024.100403

Lee, Y., Chen, R.Q., Oboamah, J., Su, P.N., Liang, W.Z., Shi, Y., Gan, L., Chen, Y., Qiao, X., & Li, J. (2025). SPADE: A large language model framework for soil moisture pattern recognition and anomaly detection in precision agriculture. arXiv:2509.18123. https://arxiv.org/abs/2509.18123

Lyu, J., Jiang, Y., Xu, C., Liu, Y., Su, Z., Liu, J., & He, J. (2022). Multi-objective winter wheat irrigation strategies optimization based on coupling AquaCrop-OSPy and NSGA-III. *Science of The Total Environment*, 843, 157104. https://doi.org/10.1016/j.scitotenv.2022.157104

Nguyen, D.B., Gruber, A., & Wagner, W. (2016). Mapping rice extent and cropping scheme in the Mekong Delta using Sentinel-1A data. *Remote Sensing Letters*, 7(12), 1209–1218. https://doi.org/10.1080/2150704X.2016.1225172

Raes, D., Steduto, P., Hsiao, T.C., & Fereres, E. (2009). AquaCrop: The FAO crop model to simulate yield response to water: II. Main algorithms and software description. *Agronomy Journal*, 101(3), 438–447. https://doi.org/10.2134/agronj2008.0140s

Saxton, K.E., & Rawls, W.J. (2006). Soil water characteristic estimates by texture and organic matter for hydrologic solutions. *Soil Science Society of America Journal*, 70(5), 1569–1578. https://doi.org/10.2136/sssaj2005.0117

Statistics Korea. (2024). *Rice production in 2024*. Press release, released 15 November 2024. https://kostat.go.kr/board.es?act=view&bid=11712&list_no=434049&mid=a20101000000

Steduto, P., Hsiao, T.C., Raes, D., & Fereres, E. (2009). AquaCrop: The FAO crop model to simulate yield response to water: I. Concepts and underlying principles. *Agronomy Journal*, 101(3), 426–437. https://doi.org/10.2134/agronj2008.0139s

Torbick, N., Chowdhury, D., Salas, W., & Qi, J. (2017). Monitoring rice agriculture across Myanmar using time series Sentinel-1 assisted by Landsat-8 and PALSAR-2. *Remote Sensing*, 9(2), 119. https://doi.org/10.3390/rs9020119

Veerakachen, W., & Raksapatcharawong, M. (2020). RiceSAP: An efficient satellite-based AquaCrop platform for rice crop monitoring and yield prediction. *Agronomy*, 10(6), 858. https://doi.org/10.3390/agronomy10060858

Zhang, Z., Zhang, J., Liu, H., Lv, Q., Yang, J., Cai, K., & Wang, K. (2026). AgriWorld: A World Tools Protocol framework for verifiable agricultural reasoning with code-executing LLM agents. arXiv:2602.15325. https://arxiv.org/abs/2602.15325
