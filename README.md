# 🚔 Ontario Municipal Police Services-Cluster Analysis

## 📋 Table of Contents

1. [Project Overview](#1-project-overview)
3. [Data Sources](#3-data-sources)
4. [Variables](#4-variables)
5. [Methodology](#5-methodology)
   - [5.1 Data Preprocessing](#51-data-preprocessing)
   - [5.2 Pre-Clustering Hypothesis Checks](#52-pre-clustering-hypothesis-checks)
   - [5.3 Feature Selection & Engineering](#53-feature-selection--engineering)
   - [5.4 Standardisation](#54-standardisation)
   - [5.5 Determining Optimal k](#55-determining-optimal-k)
   - [5.6 Ward's Hierarchical Clustering (Primary)](#56-wards-hierarchical-clustering-primary)
   - [5.7 K-Means Validation (Secondary)](#57-k-means-validation-secondary)
6. [Results](#6-results)
   - [6.1 Cluster Profiles](#61-cluster-profiles)
   - [6.2 Cluster Membership](#62-cluster-membership)
   - [6.3 Validation Metrics](#63-validation-metrics)
7. [Figures](#7-figures)
8. [Descriptive Statistics](#8-descriptive-statistics)
9. [Interpretation & Policy Implications](#9-interpretation--policy-implications)

---

## 1. Project Overview

The **Inspectorate of Policing (IoP)** is developing a **Policing Performance Measurement Framework (PPMF)** to support fair and evidence-based performance comparisons across Ontario's 43 municipal police services.

A fundamental challenge in cross-service comparison is that police services operate in vastly different environments — comparing Toronto (population 2.8M) to Deep River (population 4,175) on raw performance metrics is neither fair nor meaningful. This project addresses that challenge by using **statistical clustering** to group police services into **peer groups** that share comparable demographic, socioeconomic, and operational characteristics.

### Key Objectives

- Group Ontario's 43 municipal police services into statistically homogeneous clusters
- Use **15 variables** spanning socioeconomic context and policing operations
- Apply **Ward's hierarchical clustering** as the primary method
- Validate results with **K-Means clustering**
- Produce interpretable cluster profiles to support the PPMF

---

## 3. Data Sources

All data are drawn from the **Statistics Canada** sources specified in the assignment Variables sheet:

| # | Variable Group | Source |
|---|----------------|--------|
| 1–4 | Population, Land Area, Density, Growth | [Census 2021 — Population and Dwelling Counts](https://www12.statcan.gc.ca/census-recensement/2021/dp-pd/prof/index.cfm?Lang=E) |
| 5–9 | Sex & Age Distribution | [Census 2021 — Marital status, age group and gender](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=9810036201) |
| 10–11 | Crime Severity Index, Violent CSI | [StatCan Table 35-10-0061-01](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=3510006101) |
| 12 | Crime Rate per 100,000 | [StatCan Table 35-10-0177-01](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=3510017701) |
| 13–15 | Police Personnel (Actual, Authorised, Civilian) | [StatCan Table 35-10-0077-01](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=3510007701) |

> **Reference year:** 2021 Census; 2022 crime and personnel data.  
> **Population figures** for the 43 services are taken directly from the assignment spreadsheet (Sheet 1: *Municipal Police Service*).

---

## 4. Variables

### Socioeconomic Variables (9)

| Variable | Description | Unit |
|----------|-------------|------|
| `Population` | Total population served (2021 Census) | Count |
| `Land_Area_km2` | Total land area of jurisdiction | km² |
| `Pop_Density` | Population per km² | persons/km² |
| `Pop_Change_5yr_pct` | Population change 2016→2021 | % |
| `Men_pct` | Share of male population | % |
| `Women_pct` | Share of female population | % |
| `Age_0_14_pct` | Share aged 0–14 | % |
| `Age_15_64_pct` | Share aged 15–64 (working-age) | % |
| `Age_65_plus_pct` | Share aged 65+ | % |

### Police Operations Variables (6)

| Variable | Description | Unit |
|----------|-------------|------|
| `Crime_Severity_Index` | Weighted severity of all reported crime | Index (Canada=100) |
| `Violent_CSI` | Weighted severity of violent crime only | Index (Canada=100) |
| `Crime_Rate_per_100k` | Total incidents per 100,000 population | Rate |
| `Actual_Officers` | Sworn police officers deployed | Count |
| `Auth_Officers` | Authorised officer complement | Count |
| `Civilian_Personnel` | Civilian and other support staff | Count |

### Derived Variables (used in clustering)

| Variable | Formula | Rationale |
|----------|---------|-----------|
| `Officers_per_1000` | `Actual_Officers / Population × 1000` | Per-capita operational capacity |
| `Civilian_per_1000` | `Civilian_Personnel / Population × 1000` | Per-capita support capacity |
| `Auth_Ratio` | `Actual_Officers / Auth_Officers` | Staffing fill rate |

---

## 5. Methodology

### 5.1 Data Preprocessing

1. **Load** the 43 police services from the assignment Excel file (Sheet 1)
2. **Collect** the 15 variables from Statistics Canada sources listed in Sheet 2
3. **Verify completeness** — all 43 services have complete data for all variables
4. **Engineer** three per-capita derived variables to eliminate scale distortion from raw counts

```python
df["Officers_per_1000"] = (df["Actual_Officers"] / df["Population"]) * 1000
df["Civilian_per_1000"] = (df["Civilian_Personnel"] / df["Population"]) * 1000
df["Auth_Ratio"]        = df["Actual_Officers"] / df["Auth_Officers"]
```

---

### 5.2 Pre-Clustering Hypothesis Checks

Before applying any clustering algorithm, three statistical checks were performed:

#### A. Shapiro-Wilk Normality Test

Tests whether each feature follows a normal distribution. This informs:
- Whether parametric or non-parametric validation tests are appropriate
- Awareness of skewed features that need careful standardisation

| Variable | W-statistic | p-value | Normal? |
|----------|------------|---------|---------|
| Pop_Density | 0.7048 | < 0.0001 | ❌ No |
| Pop_Change_5yr_pct | 0.9617 | 0.1602 | ✅ Yes |
| Age_0_14_pct | 0.9579 | 0.1162 | ✅ Yes |
| Age_65_plus_pct | 0.9825 | 0.7462 | ✅ Yes |
| Crime_Severity_Index | 0.9612 | 0.1537 | ✅ Yes |
| Violent_CSI | 0.9614 | 0.1560 | ✅ Yes |
| Crime_Rate_per_100k | 0.9555 | 0.0944 | ✅ Yes |
| Officers_per_1000 | 0.9847 | 0.8291 | ✅ Yes |
| Civilian_per_1000 | 0.9377 | 0.0215 | ❌ No |
| Auth_Ratio | 0.8715 | 0.0002 | ❌ No |

> **Implication:** Three variables are non-normal. Z-score standardisation is still appropriate (it does not require normality); however, non-parametric **Kruskal-Wallis** tests are used post-clustering to validate separation rather than ANOVA.

#### B. Correlation / Multicollinearity Check

See [Figure 5](#figure-5-feature-correlation-matrix) for the full heatmap.

Key findings:
- `Crime_Severity_Index` and `Violent_CSI` are highly correlated (r ≈ 0.98) — **both retained** because they are distinct conceptually and specified in the assignment
- `Men_pct` and `Women_pct` are perfectly collinear (r = −1.0) → **`Women_pct` dropped**
- `Age_15_64_pct` = 100 − `Age_0_14_pct` − `Age_65_plus_pct` → **`Age_15_64_pct` dropped** (redundant)
- Raw `Population` and `Land_Area_km2` are subsumed by `Pop_Density` → **dropped from feature set**

#### C. Hopkins Statistic (Clustering Tendency)

The Hopkins statistic measures whether the data has a non-uniform distribution suitable for clustering.

$$H = \frac{\sum u_i}{\sum u_i + \sum w_i}$$

| Hopkins Statistic | Interpretation |
|:-----------------:|----------------|
| **0.7731** | Strong clustering tendency (H > 0.5 confirms non-random structure) |

> A Hopkins value of 0.77 indicates the data is **well-suited** for cluster analysis.

---

### 5.3 Feature Selection & Engineering

**Final feature set (10 variables):**

```
Socioeconomic:    Pop_Density, Pop_Change_5yr_pct, Age_0_14_pct, Age_65_plus_pct
Police Ops:       Crime_Severity_Index, Violent_CSI, Crime_Rate_per_100k,
                  Officers_per_1000, Civilian_per_1000, Auth_Ratio
```

**Rationale for exclusions:**

| Excluded Variable | Reason |
|------------------|--------|
| `Population` (raw) | Captured by `Pop_Density`; raw size should not dominate |
| `Land_Area_km2` | Redundant with density |
| `Women_pct` | Perfect collinearity with `Men_pct` |
| `Age_15_64_pct` | Linear dependency: sums to 100 with other age groups |
| `Actual_Officers` (raw) | Replaced by `Officers_per_1000` (per-capita) |
| `Auth_Officers` (raw) | Replaced by `Auth_Ratio` (fill rate) |
| `Civilian_Personnel` (raw) | Replaced by `Civilian_per_1000` |

---

### 5.4 Standardisation

All features standardised to **Z-scores** (mean = 0, SD = 1) using `sklearn.preprocessing.StandardScaler`:

$$z = \frac{x - \mu}{\sigma}$$

**Why Z-score standardisation?**
- Variables span vastly different scales (e.g., Crime Rate 3,500–11,000 vs. Auth_Ratio 0.86–0.98)
- Without standardisation, high-variance variables dominate Euclidean distance in Ward's method
- Z-scores give each variable equal weight while preserving relative variation structure

---

### 5.5 Determining Optimal k

Three complementary indices were computed for k = 2 to 8:

| k | Silhouette ↑ | CH Index ↑ | DB Index ↓ |
|---|:-----------:|:----------:|:----------:|
| 2 | 0.2710 | 18.3 | 1.4050 |
| 3 | 0.3057 | 20.2 | 1.0415 |
| **4** | **0.2576** | **20.2** | **1.1754** |
| 5 | 0.2555 | 20.2 | 0.9314 |
| 6 | 0.2680 | 20.8 | 0.9419 |
| 7 | 0.2086 | 19.2 | 1.0358 |
| 8 | 0.2086 | 18.4 | 1.0377 |

**Decision: k = 4** — justified by:

1. **Dendrogram** (Figure 1): A clear elbow in fusion distances appears at k = 4; cutting the dendrogram at this level yields four visually distinct, well-separated groups
2. **CH Index**: Stabilises near maximum from k = 3 onward; k = 4 maintains high CH while adding a fourth meaningful group
3. **Policy interpretability**: Four clusters align naturally with observable tiers in Ontario policing — rural/small-town, growing suburbs, mid-size urban, and high-demand service areas — making the PPMF directly actionable
4. **Dendrogram visual**: The dendrogram shows a natural 4-group structure with large inter-cluster distances

> The Silhouette score at k = 3 is marginally higher (0.306 vs 0.258), but k = 3 conflates distinct service environments (growing suburbs with high-crime mid-size cities). The policy gain from k = 4 outweighs the small metric difference.

---

### 5.6 Ward's Hierarchical Clustering (Primary)

**Algorithm:** Agglomerative hierarchical clustering with Ward's minimum variance linkage

**Distance measure:** Euclidean distance on standardised features

**Ward's criterion:** At each step, merge the two clusters that minimise the increase in total within-cluster sum of squares (WSS):

$$\Delta(A, B) = \frac{|A| \cdot |B|}{|A| + |B|} \|\bar{x}_A - \bar{x}_B\|^2$$

**Why Ward's method?**

| Criterion | Ward's | Complete | Average | Single |
|-----------|:------:|:--------:|:-------:|:------:|
| Compact, spherical clusters | ✅ Best | ✅ Good | ✅ Good | ❌ Poor |
| Resistant to chaining | ✅ Best | ✅ Good | ✅ OK | ❌ Poor |
| Handles mixed scale after standardisation | ✅ | ✅ | ✅ | ✅ |
| Recommended for socioeconomic data | ✅ Literature consensus | — | — | — |
| Produces equal-ish cluster sizes | ✅ Tendency | ❌ | ❌ | ❌ |

> Ward's method is the most widely recommended linkage for socioeconomic and public-policy clustering because it minimises within-group heterogeneity at each step, producing compact and interpretable groups — directly aligned with the PPMF's need for fair peer comparisons.

```python
from scipy.cluster.hierarchy import linkage, fcluster
linkage_matrix = linkage(X_scaled, method='ward', metric='euclidean')
labels = fcluster(linkage_matrix, k=4, criterion='maxclust')
```

---

### 5.7 K-Means Validation (Secondary)

K-Means with k = 4 (30 random initialisations, random state = 42) was run as an independent validation.

**Adjusted Rand Index (ARI) between Ward and K-Means:**

$$\text{ARI} = \boxed{0.9281}$$

> An ARI of 0.93 indicates **near-perfect agreement** between the two independent methods. This strongly validates the cluster solution — the groupings are robust and not an artefact of the algorithm chosen.

See [Figure 8](#figure-8-ward-vs-k-means-comparison) for the side-by-side PCA comparison.

---

## 6. Results

### 6.1 Cluster Profiles

Mean standardised (Z-score) values per cluster — see [Figure 4](#figure-4-cluster-profile-heatmap) for visual.

| Feature | C1: Suburban Growth | C2: Rural/Small Town | C3: Large Urban | C4: High-Demand |
|---------|:-------------------:|:--------------------:|:---------------:|:---------------:|
| Pop_Density | +0.02 | −0.41 | +0.72 | −0.28 |
| Pop_Change_5yr_pct | **+1.52** | **−0.56** | +0.33 | −0.74 |
| Age_0_14_pct | **+1.72** | −0.37 | +0.07 | −0.75 |
| Age_65_plus_pct | **−1.59** | **+0.78** | −0.26 | +0.49 |
| Crime_Severity_Index | **−0.91** | **−0.89** | +0.09 | **+1.24** |
| Violent_CSI | **−0.89** | **−0.88** | +0.09 | **+1.23** |
| Crime_Rate_per_100k | **−0.98** | **−0.78** | +0.02 | **+1.26** |
| Officers_per_1000 | −0.96 | −0.64 | +0.35 | +0.74 |
| Civilian_per_1000 | −0.49 | −1.17 | +0.58 | +0.69 |
| Auth_Ratio | +0.67 | **−1.39** | +0.40 | +0.24 |

**Cluster interpretations:**

| Cluster | Label | Key Characteristics |
|---------|-------|---------------------|
| **C1** | 🟦 Suburban Growth Corridors | Fastest population growth (+15%), youngest demographics (Age 0–14 highest), low crime, largest by total population served |
| **C2** | 🟩 Rural & Small Town | Smallest populations, oldest demographics (Age 65+ highest), low crime, lowest officer density |
| **C3** | 🟧 Large & Mid-Size Urban | Moderate-to-high crime, diverse city profiles, higher officer and civilian density |
| **C4** | 🟥 High-Demand / Northern Urban | Highest crime severity (CSI avg 109), aging populations, northern cities + Windsor/Belleville |

---

### 6.2 Cluster Membership

#### 🟦 Cluster 1 — Suburban Growth Corridors (n = 7)

> Fast-growing regional municipalities with young families, high authorisation fill rates, and low crime relative to population.

| Police Service | Population | CSI | VCSI | Density |
|----------------|:-----------:|:---:|:----:|:-------:|
| LaSalle | 32,711 | 41.3 | 31.7 | 1,143 |
| South Simcoe | 86,001 | 41.8 | 33.5 | 144 |
| Waterloo Regional | 587,216 | 65.4 | 55.8 | 429 |
| Halton Regional | 596,840 | 52.6 | 41.9 | 616 |
| Durham Regional | 696,981 | 67.8 | 58.6 | 276 |
| York Regional | 1,174,129 | 47.1 | 38.7 | 666 |
| Peel Regional | 1,373,673 | 72.3 | 62.5 | 1,102 |

---

#### 🟩 Cluster 2 — Rural & Small Town (n = 11)

> Smallest and least dense services, older populations, low crime, fewer officers per capita.

| Police Service | Population | CSI | VCSI | Density |
|----------------|:-----------:|:---:|:----:|:-------:|
| Deep River | 4,175 | 60.8 | 51.7 | 98 |
| Gananoque | 5,383 | 52.4 | 45.1 | 681 |
| Aylmer | 7,666 | 51.7 | 44.2 | 861 |
| Hanover | 8,018 | 53.2 | 41.3 | 431 |
| West Grey | 12,537 | 42.1 | 35.4 | 9 |
| Port Hope | 13,409 | 56.1 | 47.6 | 188 |
| Saugeen Shores | 15,908 | 46.3 | 36.2 | 68 |
| Cobourg | 20,505 | 61.7 | 51.3 | 1,206 |
| Brockville | 21,969 | 75.3 | 62.4 | 1,077 |
| Strathroy-Caradoc | 23,851 | 52.8 | 41.2 | 80 |
| Kawartha Lakes | 27,892 | 64.2 | 54.7 | 9 |

---

#### 🟧 Cluster 3 — Large & Mid-Size Urban (n = 13)

> Established Ontario cities with moderate-to-high crime, higher officer/civilian density, diverse economic profiles.

| Police Service | Population | CSI | VCSI | Density |
|----------------|:-----------:|:---:|:----:|:-------:|
| St. Thomas | 42,868 | 85.4 | 68.1 | 1,197 |
| Stratford | 44,400 | 62.9 | 51.4 | 342 |
| Woodstock | 46,705 | 72.4 | 61.8 | 1,062 |
| Chatham-Kent | 102,798 | 79.8 | 66.5 | 42 |
| Brantford | 104,821 | 101.4 | 88.6 | 1,450 |
| Kingston | 132,507 | 74.5 | 61.2 | 293 |
| Guelph | 143,787 | 67.2 | 52.3 | 1,650 |
| Barrie | 147,832 | 89.6 | 78.2 | 190 |
| London | 422,297 | 93.5 | 82.3 | 1,004 |
| Niagara Regional | 477,835 | 77.2 | 66.4 | 258 |
| Hamilton | 569,410 | 89.7 | 78.9 | 501 |
| Ottawa | 1,017,526 | 73.8 | 64.5 | 365 |
| Toronto | 2,794,307 | 78.5 | 67.8 | 4,433 |

---

#### 🟥 Cluster 4 — High-Demand / Northern Urban (n = 12)

> Highest crime burden, many mid-size northern cities with social complexity, ageing populations, high violent crime severity.

| Police Service | Population | CSI | VCSI | Density |
|----------------|:-----------:|:---:|:----:|:-------:|
| Smiths Falls | 9,421 | 98.3 | 82.1 | 1,178 |
| Owen Sound | 21,601 | 95.6 | 82.4 | 844 |
| Timmins | 41,102 | 122.8 | 107.4 | 14 |
| Cornwall | 47,845 | 101.2 | 89.5 | 792 |
| North Bay | 52,675 | 108.3 | 93.4 | 168 |
| Belleville | 55,071 | 102.5 | 89.4 | 223 |
| Sarnia | 72,695 | 96.4 | 82.7 | 444 |
| Sault Ste. Marie | 73,003 | 119.6 | 103.6 | 327 |
| Peterborough | 96,148 | 87.9 | 75.4 | 62 |
| Thunder Bay | 114,840 | 138.2 | 119.4 | 350 |
| Greater Sudbury | 166,106 | 121.4 | 105.8 | 46 |
| Windsor | 253,219 | 115.7 | 101.3 | 1,731 |

---

### 6.3 Validation Metrics

| Metric | Value | Interpretation |
|--------|:-----:|----------------|
| **Silhouette Score** (Ward, k=4) | 0.258 | Moderate — expected for real-world socioeconomic data |
| **Calinski-Harabasz Index** (Ward, k=4) | 20.20 | Near-maximum across all k tested |
| **Davies-Bouldin Index** (Ward, k=4) | 1.175 | Reasonable separation |
| **Adjusted Rand Index** (Ward vs K-Means) | **0.928** | Near-perfect agreement ✅ |

**Kruskal-Wallis Tests** (cluster separation significance):

| Variable | H-statistic | p-value | Significance |
|----------|:-----------:|:-------:|:------------:|
| Pop_Change_5yr_pct | 24.91 | < 0.0001 | *** |
| Age_0_14_pct | 23.75 | < 0.0001 | *** |
| Age_65_plus_pct | 26.05 | < 0.0001 | *** |
| Crime_Severity_Index | 33.52 | < 0.0001 | *** |
| Violent_CSI | 32.79 | < 0.0001 | *** |
| Crime_Rate_per_100k | 32.56 | < 0.0001 | *** |
| Officers_per_1000 | 19.02 | 0.0003 | *** |
| Civilian_per_1000 | 26.10 | < 0.0001 | *** |
| Auth_Ratio | 26.02 | < 0.0001 | *** |
| Pop_Density | 3.43 | 0.330 | ns |

> 9 of 10 variables show highly significant differences across clusters (p < 0.001). Population density alone is non-significant — this confirms that the clusters capture crime, age, growth, and operational patterns rather than simply separating by geographic density.

---

## 7. Figures

### Figure 1: Ward's Hierarchical Clustering Dendrogram

> The dendrogram visualises the full merge history of the 43 police services. The red dashed line marks the cut point for k = 4 clusters. The clear gap between fusion distances just above the cut confirms four well-separated groups.

![Figure 1 — Dendrogram](figures/fig1_dendrogram.png)

---

### Figure 2: Cluster Validity Indices

> Three independent validity metrics plotted for k = 2 through 8. The Calinski-Harabasz index peaks and stabilises at k = 3–4; the Silhouette score peaks at k = 3 but k = 4 is marginally lower while offering greater interpretability. The vertical red line marks the selected k = 4.

![Figure 2 — Validity Indices](figures/fig2_validity_indices.png)

---

### Figure 3: PCA Scatter Plot — Ward Clusters

> The 43 police services projected onto the first two principal components (PC1 + PC2 explain 77.9% of variance). Points are coloured by Ward cluster. Clear separation between clusters — particularly between the blue (suburban growth) group and the red (high-demand) group — confirms cluster quality.

![Figure 3 — PCA Scatter](figures/fig3_pca_scatter.png)

---

### Figure 4: Cluster Profile Heatmap

> Mean standardised (Z-score) values for each variable across the four clusters. Blue = below average, Red = above average. This allows immediate visual identification of each cluster's defining characteristics.

![Figure 4 — Cluster Heatmap](figures/fig4_cluster_heatmap.png)

---

### Figure 5: Feature Correlation Matrix

> Lower-triangular correlation heatmap of the 10 features used in clustering. High correlation between CSI and Violent CSI is visible (r ≈ 0.98), as is the negative relationship between Age_0_14 and Age_65_plus. Both highly-correlated crime pairs were retained as they are substantively distinct.

![Figure 5 — Correlation Matrix](figures/fig5_correlation.png)

---

### Figure 6: PCA Loadings

> Loadings of each variable on the first three principal components. PC1 (49.8% variance) is dominated by crime variables, capturing overall demand intensity. PC2 (28.0%) contrasts demographic age structure (young vs. aging) and population growth. PC3 (12.6%) primarily captures density and operational capacity differences.

![Figure 6 — PCA Loadings](figures/fig6_pca_loadings.png)

---

### Figure 7: Key Variable Distributions by Cluster

> Box plots for four key variables across clusters. The separation in Crime Severity Index (top-left) is especially stark: Cluster 4 (red) is substantially higher than Clusters 1 and 2. Officers per 1,000 population (bottom-right) shows Cluster 1 (blue) is operationally lean relative to its low crime burden.

![Figure 7 — Boxplots](figures/fig7_boxplots.png)

---

### Figure 8: Ward vs. K-Means Comparison

> Side-by-side PCA scatter plots coloured by Ward (left) and K-Means (right) cluster assignments. The near-identical visual patterns confirm the ARI of 0.928 — the two independent methods produce essentially the same groupings, strongly validating the cluster solution.

![Figure 8 — Ward vs K-Means](figures/fig8_ward_vs_kmeans.png)

---

## 8. Descriptive Statistics

| Variable | Mean | SD | Min | 25th | Median | 75th | Max |
|----------|:----:|:--:|:---:|:----:|:------:|:----:|:---:|
| Pop_Density (p/km²) | 652.2 | 761.5 | 8.7 | 177.6 | 428.9 | 1,032.7 | 4,432.6 |
| Pop_Change_5yr_pct (%) | 6.38 | 5.72 | −3.1 | 2.0 | 5.6 | 10.3 | 18.9 |
| Age_0_14_pct (%) | 16.01 | 2.38 | 11.9 | 14.2 | 15.8 | 17.3 | 22.1 |
| Age_65_plus_pct (%) | 21.41 | 4.60 | 11.0 | 19.0 | 22.0 | 24.1 | 30.3 |
| Crime_Severity_Index | 78.11 | 24.92 | 41.3 | 58.5 | 74.5 | 96.0 | 138.2 |
| Violent_CSI | 66.38 | 22.72 | 31.7 | 49.5 | 62.5 | 82.4 | 119.4 |
| Crime_Rate_per_100k | 6,306.5 | 1,859.8 | 3,512 | 4,974 | 5,923 | 7,633 | 10,834 |
| Officers_per_1000 | 1.532 | 0.178 | 1.17 | 1.42 | 1.53 | 1.67 | 1.96 |
| Civilian_per_1000 | 0.612 | 0.111 | 0.31 | 0.57 | 0.64 | 0.68 | 0.82 |
| Auth_Ratio | 0.949 | 0.027 | 0.86 | 0.94 | 0.96 | 0.97 | 0.98 |

---

## 9. Interpretation & Policy Implications

### How the Clusters Support the PPMF

The four clusters enable the IoP to make **like-for-like performance comparisons** within each group, rather than comparing services operating under fundamentally different conditions. Key implications:

#### 🟦 Cluster 1 — Suburban Growth Corridors
- **Context:** Rapid population growth creates demand surges; demographics are young-family oriented
- **PPMF application:** Compare response times, clearance rates, and resource scaling against other high-growth regional services — not against stable or declining-population areas
- **Watch for:** Capacity lag — authorised complements may not keep pace with population growth

#### 🟩 Cluster 2 — Rural & Small Town
- **Context:** Low crime environments with older populations and large geographic areas
- **PPMF application:** Benchmarking must account for the rural policing cost premium (travel time, coverage per officer) rather than raw crime rates
- **Watch for:** Senior services demand (elderly populations), volunteer emergency support integration

#### 🟧 Cluster 3 — Large & Mid-Size Urban
- **Context:** Ontario's core cities — diverse economies, established crime patterns, large civilian support infrastructure
- **PPMF application:** Rich peer group for comparing major-city services; Toronto fits here despite scale because its CSI and demographics align with urban peers
- **Watch for:** Neighbourhood-level inequality obscured by city-wide averages

#### 🟥 Cluster 4 — High-Demand / Northern Urban
- **Context:** Highest crime burden, significant social challenges, many northern cities with Indigenous population considerations and economic vulnerability
- **PPMF application:** This cluster requires context-sensitive performance standards — raw clearance rates are not comparable to Cluster 1 or 2 services
- **Watch for:** Officer wellness and retention in high-demand environments; Thunder Bay's CSI of 138 is nearly double the provincial average

### Limitations

1. **2021/2022 data:** The clustering reflects a post-pandemic snapshot; repeat analysis with 2024–2025 data is recommended as the PPMF matures
2. **15 variables:** The assignment specifies 15 variables; additional dimensions (e.g., Indigenous population share, median income, housing instability) could refine the clusters
3. **Cluster stability:** With n = 43, small services can shift cluster membership with updated data — monitoring over time is advised
4. **Equal variable weighting:** Z-score standardisation implicitly weights all variables equally; domain-expert weighting could be explored in future iterations

**Data sources:**
- Statistics Canada, Census Profile, 2021 Census of Population, Catalogue no. 98-316-X2021001
- Statistics Canada, Table 35-10-0061-01: Crime severity index and weighted clearance rates, police services in Ontario
- Statistics Canada, Table 35-10-0177-01: Incident-based crime statistics, by detailed violations, police services in Ontario
- Statistics Canada, Table 35-10-0077-01: Police Administration Survey


