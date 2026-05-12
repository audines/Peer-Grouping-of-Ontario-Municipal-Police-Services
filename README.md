# A Statistical Framework for Peer Grouping Ontario Police Services
### Equitable Performance Benchmarking for the Policing Performance Measurement Framework (PPMF)

## Table of Contents

1. [Project Overview](#project-overview)
2. [Motivation](#motivation)
3. [Data Sources](#data-sources)
4. [Variables](#variables)
5. [Methodology](#methodology)
   - [Pre-Processing Pipeline](#pre-processing-pipeline)
   - [Principal Component Analysis](#principal-component-analysis)
   - [Clustering](#clustering)
   - [Validation](#validation)
6. [Results: The Five Peer Groups](#results-the-five-peer-groups)
7. [Policy Implications](#policy-implications)
8. [Limitations](#limitations)


---

## Project Overview

Ontario's 43 municipal police services operate in fundamentally different environments — different crime burdens, population dynamics, geographic scales, and resource pressures. Evaluating all services against a single provincial benchmark is both analytically misleading and operationally unfair.

This project develops a **data-driven peer-grouping framework** to support the Ontario Inspectorate of Policing's **Policing Performance Measurement Framework (PPMF)**. Using publicly available Statistics Canada data (2021–2022), we apply Principal Component Analysis (PCA) and Ward hierarchical clustering to identify **five distinct, statistically validated peer groups** of structurally similar police services.

The result is a transparent, reproducible foundation for **like-for-like performance benchmarking** across Ontario's policing system.

---

## Motivation

A well-established challenge in public-sector performance measurement is ensuring that comparisons are contextually fair. Consider two real cases from this dataset:

- **Thunder Bay** (Cluster 3): Population ~115,000, Crime Severity Index of **132.6**, population *declining* at −3.2% per year, geographically isolated.
- **Halton Regional** (Cluster 5): Population ~597,000, Crime Severity Index of **48.6**, population *growing* at +13.7% over five years, low-density suburban.

Benchmarking these two services on identical KPIs — response times, staffing ratios, case clearance rates — would penalise Thunder Bay for its structural operating environment and reward Halton for conditions largely outside its control.

**Peer grouping shifts measurement from ranking to contextualised benchmarking: comparing like with like.**

This approach is standard in analogous public-sector domains: school inspectorates, hospital performance frameworks, and local government benchmarking systems all use peer grouping. This project brings the same rigour to Ontario policing.

---

## Data Sources

All data are publicly available from Statistics Canada. No proprietary or restricted data were used.

| Source | Table | Year | Variables |
|--------|-------|------|-----------|
| Statistics Canada, 2021 Census | `98-400-X2021002` | 2021 | Population, land area, density, 5-yr growth, age structure, sex distribution |
| Statistics Canada | `35-10-0026-01` | 2022 | Crime Severity Index (CSI), Violent Crime Severity Index (VCSI) |
| Statistics Canada | `35-10-0177-01` | 2022 | Incident-based crime rate per 100,000 population |
| Statistics Canada | `35-10-0077-01` | 2022 | Actual officers, authorised strength, civilian/other personnel |

---

## Variables

Fifteen variables were selected across three theoretical dimensions:

### Socioeconomic / Demographic (8 variables)

| Variable | Pre-processing | Rationale |
|----------|---------------|-----------|
| Population | `log(·)` | Captures overall service demand; log-transformed due to right-skew (skewness = 3.27) |
| Land Area (km²) | `log(·)` | Proxies geographic coverage and patrol complexity |
| Population Density | `log(·)` | Measures urbanisation; higher density correlates with service demand |
| 5-Year Population Change % | None | Captures growth dynamics and forward-looking demand pressure |
| Male % / Female % | None | Demographic structure; associated with crime patterns |
| Age 0–14 % | None | Proxy for family-oriented communities |
| Age 15–64 % | None | Working-age population; associated with crime exposure |
| Age 65+ % | None | Ageing populations; different service needs, typically lower crime |

### Crime Indicators (3 variables)

| Variable | Pre-processing | Rationale |
|----------|---------------|-----------|
| Crime Severity Index (CSI) | None | Measures overall crime burden, weighted by offence seriousness |
| Violent Crime Severity Index (VCSI) | None | Isolates violent crime; key driver of resource intensity |
| Crime Rate per 100,000 | None | Captures incident frequency; complements severity-based measures |

### Policing Resources (3 variables)

| Variable | Pre-processing | Rationale |
|----------|---------------|-----------|
| Officers per 100,000 | Rate conversion | Policing intensity relative to population |
| Authorised Officers per 100,000 | Rate conversion | Planned staffing capacity |
| Civilian Personnel per 100,000 | Rate conversion | Organisational support capacity |

> **Note on rate conversion:** Raw officer counts were rejected in favour of per-100,000 rates. Using raw counts would introduce a size confound — larger services would cluster together simply because they employ more personnel, not because of structural similarity.

---

## Methodology

### Pre-Processing Pipeline

The analysis follows a four-stage pre-processing pipeline before clustering:

#### Stage 1 — Log-Transformation

Shapiro–Wilk tests and skewness statistics confirm that Population, Land Area, and Population Density are strongly right-skewed (skewness > 1, SW p < 0.001). Log-transformation substantially reduces skewness and improves suitability for Euclidean-distance clustering.

| Variable | Skew (raw) | SW p (raw) | Skew (log) | SW p (log) |
|----------|-----------|-----------|-----------|-----------|
| Population | +3.27 | < 0.001 | +0.27 | 0.31 |
| Land Area (km²) | +1.19 | < 0.001 | −0.26 | 0.45 |
| Population Density | +3.23 | < 0.001 | −0.77 | 0.09 |

#### Stage 2 — Correlation Analysis

Pearson correlation analysis reveals three major redundancy blocks:
- **Crime indicators**: CSI, VCSI, and Crime Rate are near-perfectly intercorrelated
- **Size/geography**: log(Population), log(Land Area), log(Density) are structurally linked via D = P/A
- **Personnel**: Officers/100k, Authorised Officers/100k, and Civilian/100k are moderately intercorrelated

#### Stage 3 — Multicollinearity Check (VIF)

Variance Inflation Factors confirm extreme multicollinearity that would distort distance-based clustering if unaddressed:

| Variable | VIF | Interpretation |
|----------|-----|----------------|
| Male % / Female % | ∞ | Exact collinearity (compositional) |
| Age groups (3 vars) | ∞ | Exact collinearity (compositional) |
| log(Population) | 2.3 × 10⁶ | Extreme — linked to density |
| log(Land Area) | 2.9 × 10⁶ | Extreme — linked to density |
| CSI | 1,774 | Strong — linked to VCSI/rate |
| Crime Rate | 1,748 | Strong — linked to CSI |
| Officers/100k | 148 | High |
| Pop. Change % | 8.1 | Acceptable |

PCA is applied to resolve this multicollinearity.

#### Stage 4 — Z-Score Standardisation

All variables are standardised to zero mean and unit variance prior to PCA and clustering. This ensures that high-magnitude features (e.g., raw population counts) do not dominate Euclidean distance calculations.

---

### Principal Component Analysis

PCA transforms the 15 correlated variables into orthogonal components ordered by explained variance.

**Retention criterion:** Cumulative variance ≥ 90% (Kaiser criterion: eigenvalue > 1).

| Component | Eigenvalue | Variance (%) | Cumulative (%) | Interpretation |
|-----------|-----------|-------------|---------------|----------------|
| PC1 | 6.05 | 40.3 | 40.3 | **Crime Environment** — CSI (+0.36), Crime Rate (+0.36), VCSI (+0.35) |
| PC2 | 3.84 | 25.6 | 65.9 | **Age & Urbanisation** — Age 15–64% (+0.48), Age 65+% (−0.43), log(Pop) (+0.41) |
| PC3 | 2.47 | 16.4 | 82.3 | **Geographic Dispersion** — log(Land Area) (+0.53), Female% (−0.43), Male% (+0.43) |
| PC4 | 1.34 | 8.9 | **91.3** | **Policing Intensity** — Officers/100k (+0.41), Male% (+0.38), Female% (−0.38) |
| PC5 | 0.67 | 4.5 | 95.8 | Age 0–14% / Land Area (noise) |
| ... | ... | ... | ... | ... |

**Decision: Retain PC1–PC4.** These four components capture 91.3% of total variance. PC5 onward adds < 5% marginal variance and introduces noise.

---

### Clustering

**Method:** Ward's minimum variance hierarchical clustering, applied to the four PCA scores for each of the 43 services.

Ward linkage minimises the increase in total within-cluster variance at each merge step:

$$\Delta(A, B) = \frac{n_A n_B}{n_A + n_B} \|\bar{x}_A - \bar{x}_B\|^2$$

**Why Ward linkage?**
- Produces compact, variance-minimising clusters suitable for policy categories
- Robust for small samples (n = 43) — avoids chaining artefacts common in single/average linkage
- Standard method in public-sector demographic clustering applications

**Number of clusters:** Evaluated k = 2 to 8 using three internal validation metrics:

| Metric | k = 5 value | Direction | Verdict |
|--------|------------|-----------|---------|
| Silhouette Coefficient | **0.293** (peak) | ↑ better | k = 5 ✓ |
| Calinski–Harabasz Index | **17.2** (maximum) | ↑ better | k = 5 ✓ |
| Davies–Bouldin Index | **1.097** (near minimum) | ↓ better | k = 5 ✓ |

All three metrics converge on **k = 5**. The dendrogram also shows a clear structural gap at the five-cluster cut.

---

### Validation

#### Comparison with Alternative Linkage Methods

| Metric | Ward | Complete |
|--------|------|----------|
| Silhouette Coefficient | **0.293** | 0.250 |
| Calinski–Harabasz Index | **17.2** | 14.4 |
| Davies–Bouldin Index | **1.097** | 1.312 |
| Adjusted Rand Index (vs Ward) | 1.000 | 0.273 |

Ward linkage outperforms complete linkage on all metrics. The ARI of 0.273 confirms the two methods produce substantially different solutions — Ward's is the more coherent one.

#### Bootstrap Jaccard Stability (B = 300)

| k | Mean Jaccard | Std. Dev. |
|---|-------------|----------|
| 4 | 0.233 | 0.031 |
| **5** | **0.223** | **0.029** |
| 6 | 0.215 | 0.027 |

Moderate absolute stability is expected and normal for n = 43. k = 5 shows the most consistent structure relative to neighbouring solutions.

#### Kruskal–Wallis Tests

Non-parametric tests confirm statistically significant differences across all key variables (p < 0.01):

| Variable | H-statistic | p-value | Significance |
|----------|------------|---------|-------------|
| Population | 27.09 | < 0.001 | *** |
| Population Density | 16.43 | 0.003 | ** |
| 5-yr Pop. Change % | 27.04 | < 0.001 | *** |
| Age 0–14 % | 22.44 | < 0.001 | *** |
| Age 65+ % | 28.04 | < 0.001 | *** |
| Crime Severity Index | 25.33 | < 0.001 | *** |
| Violent CSI | 24.99 | < 0.001 | *** |
| Crime Rate/100k | 25.48 | < 0.001 | *** |
| Officers/100k | 23.73 | < 0.001 | *** |
| Civilian Personnel/100k | 23.06 | < 0.001 | *** |

---

## Results: The Five Peer Groups

### Summary Statistics (Median Values)

| Cluster | n | Median Pop. | Pop Chg % | CSI | VCSI | Crime Rate | Officers/100k |
|---------|---|------------|-----------|-----|------|-----------|--------------|
| C1 Small Southern Towns | 10 | 16,957 | +1.2% | 67.6 | 57.2 | 5,120 | 151.4 |
| C2 Small Rural Services | 4 | 19,880 | +6.3% | 49.7 | 40.2 | 3,880 | 119.5 |
| C3 Northern Ontario Services | 6 | 62,839 | −3.2% | 112.4 | 95.2 | 8,480 | 176.9 |
| C4 Mid/Large Urban Services | 16 | 138,147 | +5.8% | 81.6 | 69.4 | 6,130 | 166.2 |
| C5 High-Growth Suburban Services | 7 | 596,840 | +13.7% | 48.6 | 38.2 | 3,680 | 143.1 |

---

### Cluster 1 — Small Southern Towns (n = 10)

> Aylmer · Brockville · Cobourg · Gananoque · Hanover · Owen Sound · Port Hope · Smiths Falls · St. Thomas · Stratford

Small, stable, slow-growing municipalities with moderate crime levels and ageing populations. Policing demand is steady rather than volatile. These services face balanced operational pressures with limited fiscal expansion and traditional small-town service environments.

**Key characteristics:** Population ~17,000 · CSI ~67.6 · Growth ~+1.2% · Predominantly southern Ontario geography

---

### Cluster 2 — Small Rural Services (n = 4)

> Kawartha Lakes · Saugeen Shores · Strathroy-Caradoc · West Grey

Low crime, sparse populations, and large geographic coverage areas. Staffing intensity is the lowest in the province. The primary challenge is maintaining service accessibility and efficiency across dispersed communities rather than responding to high crime demand.

**Key characteristics:** Population ~20,000 · CSI ~49.7 · Lowest officers/100k (119.5) · Large land areas · Ageing demographics

---

### Cluster 3 — Northern Ontario Services (n = 6)

> Deep River · Greater Sudbury · North Bay · Sault Ste. Marie · Thunder Bay · Timmins

The most structurally demanding cluster in the dataset. Northern services combine the **highest crime severity in the province** with geographic isolation and declining populations. These conditions produce structurally high service costs and sustained operational strain. High officer ratios here reflect necessity, not inefficiency.

**Key characteristics:** CSI ~112.4 (province high) · Population declining −3.2% · Highest officers/100k (176.9) · Geographically isolated

---

### Cluster 4 — Mid/Large Urban Services (n = 16)

> Barrie · Belleville · Brantford · Chatham-Kent · Cornwall · Guelph · Hamilton · Kingston · London · Niagara Regional · Ottawa · Peterborough · Sarnia · Toronto · Windsor · Woodstock

The largest and most heterogeneous cluster, spanning from Woodstock (pop. ~55,000) to Toronto (pop. ~2.8 million). Cluster similarity is driven primarily by **crime structure** rather than population size — demonstrating the limitations of size-based benchmarking. Complexity arises from population density, service diversity, and workload scale.

**Key characteristics:** CSI ~81.6 · Diverse population sizes · Moderate-to-high crime complexity · Crime environment as primary grouping driver

---

### Cluster 5 — High-Growth Suburban Services (n = 7)

> Durham Regional · Halton Regional · LaSalle · Peel Regional · South Simcoe · Waterloo Regional · York Regional

Defined by **rapid population growth, low crime rates, and younger demographic profiles**. These services currently face relatively low crime pressure but are experiencing the fastest demand growth in the province. Forward-looking capacity planning is essential — reactive policing models will be insufficient.

**Key characteristics:** CSI ~48.6 (province low) · Population growth +13.7% · Youngest demographics · Future demand risk

---

## Policy Implications

This framework has four direct applications for the PPMF:

### 1. Differentiated Benchmarking
Performance evaluation should be conducted **within peer groups**. Response time benchmarks, staffing ratios, and case clearance rates should be defined relative to cluster medians, not provincial averages. Cross-cluster comparisons are not analytically appropriate given the substantial heterogeneity in crime structure, population dynamics, and geographic context.

### 2. Fair Resource Interpretation
Variation in staffing intensity across clusters reflects underlying service environments, not managerial efficiency. Higher officer-to-population ratios in Northern Ontario are consistent with elevated crime severity, geographic isolation, and dispersed service areas. The PPMF should flag **deviation from peer-group norms**, not deviation from provincial averages.

### 3. Forward-Looking Planning for High-Growth Services
High-growth suburban services (Cluster 5) require performance metrics that incorporate **projected population and demand trajectories**, not just current conditions. At +13.7% growth per five-year period, their resource requirements will look very different within a decade.

### 4. Tailored KPI Weighting
Performance indicators should be weighted according to cluster context:
- **Northern Ontario (C3):** Crime-severity-weighted metrics
- **Small Rural (C2):** Coverage, accessibility, and response-dispersion metrics
- **High-Growth Suburban (C5):** Growth-adjusted capacity and forward demand metrics
- **Urban (C4):** Workload complexity and service diversity metrics

---

## Limitations

1. **Cluster 4 heterogeneity.** The urban cluster spans a wide population range (Woodstock ~55k to Toronto ~2.8M). Crime structure unifies them, but future work could explore splitting into medium and large urban sub-groups.

2. **Cross-sectional data.** The analysis is based on a 2021–2022 snapshot. Service profiles evolve, particularly in high-growth regions. **Re-estimation every 3 years is recommended.**

3. **Missing socioeconomic variables.** Income levels, housing affordability, and deprivation indices are known predictors of crime and service demand but were unavailable at the service level. Their inclusion in future iterations would improve structural validity.

4. **Small sample constraints.** With n = 43, moderate bootstrap Jaccard stability is expected. Multi-year panel data would strengthen robustness and allow longitudinal cluster tracking.

5. **Equal variable weighting.** Z-score standardisation implicitly weights all variables equally. Policy-informed weighting schemes could be explored to reflect PPMF priorities.

---




