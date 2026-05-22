# A Statistical Framework for Peer Grouping Ontario Police Services

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

Ontario's 43 municipal police services operate in fundamentally different environments: different crime burdens, population dynamics, geographic scales, and resource pressures. Evaluating all services against a single provincial benchmark is both analytically misleading and operationally unfair.

This project develops a **data-driven peer-grouping framework** to support the Ontario Inspectorate of Policing's **Policing Performance Measurement Framework (PPMF)**. Using publicly available Statistics Canada data (2021–2022), we apply Principal Component Analysis (PCA) and Ward hierarchical clustering to identify **five distinct, statistically validated peer groups** of structurally similar police services.

The result is a transparent, reproducible foundation for **like-for-like performance benchmarking** across Ontario's policing system.

---

## Motivation

A well-established challenge in public-sector performance measurement is ensuring that comparisons are contextually fair. Consider two real cases from this dataset:

- **Thunder Bay** (Cluster 3): Population ~115,000, Crime Severity Index of **132.6**, population *declining* at −3.2% per year, geographically isolated.
- **Halton Regional** (Cluster 5): Population ~597,000, Crime Severity Index of **48.6**, population *growing* at +13.7% over five years, low-density suburban.

Benchmarking these two services on identical KPIs — response times, staffing ratios, case clearance rates — would penalise Thunder Bay for its structural operating environment and reward Halton for conditions largely outside its control.

**Peer grouping shifts measurement from ranking to contextualised benchmarking: comparing like with like.**

---

## Data Sources

All data are publicly available from Statistics Canada. No proprietary or restricted data were used.

| Source | Table | Year | Variables |
|--------|-------|------|-----------|
| Statistics Canada, 2021 Census | `98-400-X2021002` | 2021 | Population, land area, density, 5-yr growth, age structure, sex distribution |
| Statistics Canada | `35-10-0026-01` | 2022 | Crime Severity Index (CSI), Violent Crime Severity Index (VCSI) |
| Statistics Canada | `35-10-0177-01` | 2022 | Incident-based crime rate per 100,000 population |
| Statistics Canada | `35-10-0077-01` | 2022 | Actual officers, authorised strength, civilian/other personnel |

All 43 Ontario municipal police services are included.

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

> **Note on rate conversion:** Raw officer counts were rejected in favour of per-100,000 rates to avoid a size confound.

---

## Methodology

### Pre-Processing Pipeline

Four-stage pipeline before clustering:

**Stage 1 — Log-Transformation**

Shapiro–Wilk tests confirm strong right-skew in three variables before transformation:

| Variable | Skew (raw) | SW p (raw) | Skew (log) | SW p (log) |
|----------|-----------|-----------|-----------|-----------|
| Population | +3.27 | < 0.001 | +0.27 | 0.31 |
| Land Area (km²) | +1.19 | < 0.001 | −0.26 | 0.45 |
| Population Density | +3.23 | < 0.001 | −0.77 | 0.09 |

**Stage 2 — Correlation Analysis**

Strong redundancy blocks are visible across crime, size/geography, and personnel variables.
![Pearson Correlation Matrix](https://github.com/audines/Peer-Grouping-of-Ontario-Municipal-Police-Services/blob/main/fig1_correlation_matrix.png)
*Fig 1 — Pearson correlation matrix of the 15 analysis variables after log-transformation and rate conversion. Strong blocks of correlation are visible among crime, size, and personnel variables.*

**Stage 3 — Multicollinearity Check (VIF)**

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

**Stage 4 — Z-Score Standardisation:** all variables standardised to zero mean and unit variance.

---

### Principal Component Analysis

PCA transforms the 15 correlated variables into orthogonal components. The first four components explain **91.3% of total variance**, satisfying the ≥ 90% retention criterion.

![PCA Scree Plot](https://github.com/audines/Peer-Grouping-of-Ontario-Municipal-Police-Services/blob/main/fig2_scree_plot.png)
*Fig 2 — Scree plot (left) and cumulative explained variance (right). PC1–PC4 explain 91.3% of variance; the elbow is clear at PC4.*

| Component | Variance (%) | Cumulative (%) | Interpretation |
|-----------|-------------|---------------|----------------|
| PC1 | 40.3 | 40.3 | **Crime Environment** — CSI (+0.36), Crime Rate (+0.36), VCSI (+0.35) |
| PC2 | 25.6 | 65.9 | **Age & Urbanisation** — Age 15–64% (+0.48), Age 65+% (−0.43), log(Pop) (+0.41) |
| PC3 | 16.4 | 82.3 | **Geographic Dispersion** — log(Land Area) (+0.53) |
| PC4 | 8.9 | **91.3** | **Policing Intensity** — Officers/100k (+0.41) |

![PCA Biplot](https://github.com/audines/Peer-Grouping-of-Ontario-Municipal-Police-Services/blob/main/fig3_validation_metrics.png)
*Fig 3 — PCA biplot (PC1 vs PC2). Services are coloured by cluster; loading arrows show the direction of key variable influence. Northern Ontario services (green) score high on PC1 (crime); High-Growth Suburban (red) score high on PC2 (growth/urbanisation).*

---

### Clustering

**Method:** Ward's minimum variance hierarchical clustering on the 4 PCA scores.

**Number of clusters:** All three internal validation metrics converge on **k = 5**:

![Cluster Validation Metrics](https://github.com/audines/Peer-Grouping-of-Ontario-Municipal-Police-Services/blob/main/fig4_dendrogram.png)
*Fig 4 - Internal validation metrics (Silhouette, Calinski–Harabasz, Davies–Bouldin) across k = 2–8. All three peak or bottom at k = 5.*

The dendrogram confirms the five-cluster structure with a clear structural gap at the k = 5 cut:

![Ward Linkage Dendrogram](https://github.com/audines/Peer-Grouping-of-Ontario-Municipal-Police-Services/blob/main/fig5_silhouette_plot.png)
*Fig 5 - Ward linkage dendrogram on 4 PCA scores. The red dashed line marks the k = 5 cut. Services are colour-coded by cluster. The large gap above the cut confirms five groups is the natural partition.*

---

### Validation

#### Silhouette Analysis

![Silhouette Plot](https://github.com/audines/Peer-Grouping-of-Ontario-Municipal-Police-Services/blob/main/fig6_pca_biplot.png)
*Fig 6 — Silhouette plot for Ward k = 5. C5 (High-Growth Suburban) and C2 (Small Rural) show the tightest internal cohesion. Overall mean silhouette = 0.293.*

#### Comparison with Alternative Linkage Methods

| Metric | Ward | Complete |
|--------|------|----------|
| Silhouette Coefficient | **0.293** | 0.250 |
| Calinski–Harabasz Index | **17.2** | 14.4 |
| Davies–Bouldin Index | **1.097** | 1.312 |
| Adjusted Rand Index | 1.000 | 0.273 |

Ward linkage outperforms complete linkage across all metrics.

#### Bootstrap Jaccard Stability (B = 300)

| k | Mean Jaccard | Std. Dev. |
|---|-------------|----------|
| 4 | 0.233 | 0.031 |
| **5** | **0.223** | **0.029** |
| 6 | 0.215 | 0.027 |

#### Kruskal–Wallis Tests

All key variables show statistically significant differences across clusters (p < 0.01):

![Kruskal-Wallis Results](https://github.com/audines/Peer-Grouping-of-Ontario-Municipal-Police-Services/blob/main/fig9_kruskal_wallis.png)
*Fig 7 — Kruskal–Wallis H-statistics for key variables. All exceed the p < 0.001 critical threshold (dotted line), confirming the clustering reflects genuine structural differences.*

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

### Cluster Profiles

The radar chart below shows each cluster's normalised median profile across six key dimensions:

![Radar Chart](https://github.com/audines/Peer-Grouping-of-Ontario-Municipal-Police-Services/blob/main/fig7_radar_chart.png)
*Fig 8 - Radar chart of normalised median cluster profiles (scaled 0-1 across clusters). C3 Northern Ontario dominates the crime axes; C5 High-Growth Suburban dominates the population growth axis; C2 Small Rural leads on Age 65+%.*

The heatmap shows each cluster's z-score profile versus the all-service mean:

![Cluster Heatmap](https://github.com/audines/Peer-Grouping-of-Ontario-Municipal-Police-Services/blob/main/fig8_cluster_heatmap.png)
*Fig 9 — Cluster profile heatmap. Green = above all-service average; red = below. Annotations show raw median values. C3 (Northern Ontario) is strongly green on crime indicators; C5 (Suburban) is strongly red on crime and green on growth.*

---

### Cluster 1- Small Southern Towns (n = 10)

> Aylmer · Brockville · Cobourg · Gananoque · Hanover · Owen Sound · Port Hope · Smiths Falls · St. Thomas · Stratford

Small, stable, slow-growing municipalities with moderate crime levels and ageing populations. Steady policing demand, traditional small-town service environments.

---

### Cluster 2- Small Rural Services (n = 4)

> Kawartha Lakes · Saugeen Shores · Strathroy-Caradoc · West Grey

Low crime, sparse populations, large geographic coverage. Lowest staffing intensity in the province. Primary challenge is service accessibility across dispersed communities.

---

### Cluster 3 — Northern Ontario Services (n = 6)

> Deep River · Greater Sudbury · North Bay · Sault Ste. Marie · Thunder Bay · Timmins

The most structurally demanding cluster. Highest crime severity (CSI median 112.4), declining population (−3.2%), geographic isolation. High officer ratios reflect structural necessity, not inefficiency.

---

### Cluster 4 — Mid/Large Urban Services (n = 16)

> Barrie · Belleville · Brantford · Chatham-Kent · Cornwall · Guelph · Hamilton · Kingston · London · Niagara Regional · Ottawa · Peterborough · Sarnia · Toronto · Windsor · Woodstock

Spans Woodstock to Toronto. Cluster unity is driven by **crime structure**, not population size — demonstrating the limitations of size-based benchmarking.

---

### Cluster 5 — High-Growth Suburban Services (n = 7)

> Durham Regional · Halton Regional · LaSalle · Peel Regional · South Simcoe · Waterloo Regional · York Regional

Lowest crime (CSI median 48.6), fastest population growth (+13.7%). Demand is rising rapidly; forward-looking capacity planning is essential.

---

## Policy Implications

1. **Differentiated Benchmarks** - Set KPI targets within peer groups, not against provincial averages.
2. **Fair Resource Interpretation** -Northern Ontario's high officer ratios reflect structural necessity; flag deviation from peer norms, not provincial norms.
3. **Forward-Looking Planning** - Suburban services need growth-adjusted capacity metrics, not just current-state measurement.
4. **Tailored KPI Weighting** - Weight performance indicators by cluster context (crime-severity for North; coverage for Rural; growth-capacity for Suburban).

---

## Limitations

1. **Cluster 4 heterogeneity** - Spans Woodstock to Toronto; medium/large urban sub-groups could be explored.
2. **Cross-sectional data** - 2021–22 snapshot; re-estimation every 3 years recommended.
3. **Missing socioeconomic variables** - Income, housing affordability, and deprivation indices not available at service level; inclusion would improve structural validity.
4. **Small sample (n = 43)** - Moderate bootstrap stability is expected; multi-year panel data would strengthen robustness.
5. **Equal variable weighting** - Standardisation treats all variables equally; policy-informed weighting could be explored.

