# Peer Grouping of Ontario Municipal Police Services

## Overview

This project develops a **data-driven peer-grouping framework** for Ontario’s 43 municipal police services to support fairer and more meaningful performance benchmarking within the **Policing Performance Measurement Framework (PPMF)**.

Rather than comparing all police services against a single provincial average, this analysis groups services based on similar structural and operational characteristics such as crime environment, population dynamics, geographic context, and policing intensity.

The project uses publicly available data from **Statistics Canada (2021–2022)** and applies statistical learning techniques including:

* Data standardization and transformation
* Principal Component Analysis (PCA)
* Ward Hierarchical Clustering
* Statistical validation testing

The resulting framework identifies **five distinct peer groups** that provide a more equitable basis for evaluating police performance across Ontario.

---

# Project Objectives

The primary objectives of this project are to:

* Develop a transparent and reproducible peer-grouping model
* Improve comparability between Ontario police services
* Support evidence-based performance benchmarking
* Identify structural drivers of policing demand
* Provide insights for policy and resource allocation decisions

---

# Key Findings

## 1. Northern Ontario Services

* Highest crime severity and operational burden
* Large geographic coverage areas
* Population decline and service dispersion challenges
* Higher policing intensity required to maintain coverage

## 2. High-Growth Suburban Services

* Currently lower crime severity
* Rapid population growth creating future demand pressures
* Expanding infrastructure and service requirements

## 3. Small Rural and Small-Town Services

* Lower population density
* Aging demographic structure
* Limited economies of scale
* Resource constraints driven by structural conditions

## 4. Mid- to Large Urban Services

* Similar crime environments despite population size differences
* Demonstrates limitations of size-based benchmarking alone

---

# Methodology

## Data Sources

Publicly available datasets from:

* Statistics Canada (2021–2022)
* Ontario policing and demographic datasets

---

## Variables Analyzed

The analysis includes 15 variables across four major dimensions:

### Demographic Variables

* Population size
* Population growth
* Population density
* Age structure

### Geographic Variables

* Service area size
* Population dispersion
* Urban/rural composition

### Crime Variables

* Crime Severity Index (CSI)
* Violent Crime Severity Index
* Crime rates
* Operational burden indicators

### Policing Variables

* Officers per capita
* Resource intensity
* Staffing measures

---

## Analytical Approach

### 1. Data Preparation

* Missing value assessment
* Variable transformation
* Standardization (z-score scaling)

### 2. Principal Component Analysis (PCA)

PCA was used to reduce dimensionality while preserving the majority of information in the dataset.

* Reduced 15 variables into 4 principal components
* Explained over 91% of total variance

### 3. Hierarchical Clustering

Ward hierarchical clustering was applied using PCA component scores.

Outputs included:

* Dendrogram analysis
* Cluster assignment
* Cluster profiling

### 4. Validation

The clustering solution was validated using:

* ANOVA significance testing
* Cluster separation assessment
* Interpretability evaluation

All major variables were statistically significant across clusters (**p < 0.001**).

---

# Technologies Used

* Python
* Pandas
* NumPy
* Scikit-learn
* SciPy
* Matplotlib
* Seaborn
* Jupyter Notebook

---

# Repository Structure

```bash
├── data/
│   ├── raw/
│   └── processed/
│
├── notebooks/
│   ├── data_cleaning.ipynb
│   ├── exploratory_analysis.ipynb
│   ├── pca_analysis.ipynb
│   └── clustering_analysis.ipynb
│
├── outputs/
│   ├── figures/
│   ├── tables/
│   └── reports/
│
├── src/
│   ├── preprocessing.py
│   ├── pca_model.py
│   ├── clustering.py
│   └── validation.py
│
├── README.md
└── requirements.txt
```

---

# Results

The final clustering solution identified:

* **5 statistically distinct peer groups**
* Clear structural differentiation across policing environments
* Operationally interpretable benchmarking categories

The framework demonstrates that:

* Police services should not be evaluated solely by population size
* Structural context significantly influences policing demand
* Benchmarking must account for geography, demographics, and crime environment

---

# Policy Implications

## Benchmarking

Performance evaluation should occur within structurally comparable peer groups rather than using provincial averages.

## Resource Allocation

Higher staffing levels in Northern Ontario should be interpreted within the context of geographic and operational challenges.

## Future Planning

Rapid-growth suburban services require forward-looking performance metrics that account for projected demand increases.

## Framework Expansion

Future versions of the framework should incorporate:

* Income indicators
* Housing conditions
* Deprivation indices
* Socioeconomic risk factors

---

# Future Improvements

Potential enhancements include:

* Time-series clustering
* Predictive demand modeling
* Integration of socioeconomic indicators
* Interactive dashboards and visualizations
* Geospatial analysis
* Automated annual updates

---

# Reproducibility

This framework is designed to be:

* Transparent
* Reproducible
* Scalable
* Adaptable to future datasets

All methodologies and analytical steps can be replicated using publicly available data.

---

# Author

**Aude Ines Mbonda**

Statistician | Data Analyst | Research & Policy Analytics

---

# License

This project is intended for academic, research, and public policy analysis purposes.

---

# Acknowledgements

* Statistics Canada
* Ontario public-sector data resources
* Policing Performance Measurement Framework (PPMF)
* Open-source Python data science community
