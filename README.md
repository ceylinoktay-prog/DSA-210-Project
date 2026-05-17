DSA-210 Project

## Motivation: 
Giant pandas are among the world's most endangered species, and their survival is tightly bound to a single food source: bamboo. Habitat loss, climate change, and human activity all threaten panda populations — but they do so primarily by reducing bamboo availability. Rather than studying these pressures in isolation, this project traces a **three-stage causal chain**:

> **Environmental & Human Drivers → Bamboo Availability (BAI) → Panda Population**

The goal is to systematically quantify how ecological and human factors shape bamboo accessibility, and whether that accessibility meaningfully predicts giant panda population dynamics.

---

## Main Hypothesis:

- **H₀ (Null):** Bamboo availability has no significant relationship with panda population.
- **H₁ (Alternative):** Bamboo availability has a significant relationship with panda population.

---

## Data Sources:

Six datasets spanning the **1974–2002** study period are integrated into a single analytical framework:

### 1. Bamboo Occurrence Proxy
- **Source:** GBIF occurrence records for *Fargesia* and *Bashania* genera (the two main bamboo species consumed by giant pandas)
- **Processing:** Yearly counts min-max normalised → `bamboo_index` (0–1 scale)

### 2. Climate Data (NOAA)
- **Source:** NOAA NCEI Global Summary of the Year — Chengdu, Xi'an, and Lanzhou stations
- **Variables:** Annual precipitation (inches) and temperature (°F), averaged across the three stations

### 3. Panda Population (WWF / IUCN)
- **Source:** WWF / IUCN National Panda Surveys
- **Anchor years:** 1975 ≈ 2,459 individuals; 1986 ≈ 1,114; 2003 ≈ 1,596
- **Processing:** Linearly interpolated to produce a continuous annual series

### 4. Land-Use & Habitat Fragmentation
- **Source:** ESA CCI Land Cover / China National Land Use Survey
- **Variables:** `forest_area_km²`, `agriculture_pct`, `urban_pct`, `fragmentation_index`

### 5. Human Activity Indicators
- **Source:** World Bank Open Data, UN Population Division, IUCN WDPA
- **Variables:** `population_density`, `gdp_per_capita_usd`, `road_density`, `nature_reserve_area_km²`

### 6. Satellite Vegetation Index (NDVI & LST)
- **Source:** NASA MODIS (MOD13A3, MOD11A2) and Landsat archive (pre-2000)
- **Variables:** `ndvi_mean`, `ndvi_bamboo_zone`, `lst_celsius` (land surface temperature)

---

## Bamboo Availability Index (BAI_full)

A composite **Bamboo Availability Index** is constructed by combining four normalised ecological dimensions, each scaled to [0–1] before weighting:

 Component | Weight | Rationale |
|---|---|---|
| `bamboo_index` (GBIF occurrence) | **35%** | Direct occurrence-based proxy — primary signal |
| `ndvi_bamboo_zone` | **25%** | Satellite-derived vegetation density in bamboo habitat cells |
| `1 − fragmentation_index` | **20%** | Habitat connectivity — lower fragmentation = better access |
| `forest_area_km²` (normalised) | **20%** | Total extent of bamboo-suitable forest |

A higher BAI_full score indicates **greater bamboo availability and accessibility** for giant pandas.

---

## Analysis Pipeline

### 1. Data Preparation
- All six datasets merged on `year` into a single analytical DataFrame (n = 29 years)
- BAI_full computed as a weighted composite of normalised ecological components
- Climate variables averaged across three weather stations

### 2. Exploratory Data Analysis (EDA)
- Dual-axis time series: panda population vs. bamboo index (with 1983–1987 bamboo die-off highlighted)
- 3-year rolling mean on bamboo index to reveal the underlying trend
- Linear warming trend overlaid on temperature, ±1σ uncertainty band on precipitation
- Distribution histograms with fitted normal density curves and Q-Q plots for normality assessment
- Full correlation heatmap across all 12 variables
- Land-use trends, NDVI time series, and scatter plots for all key driver relationships

### 3. Hypothesis Testing

All hypotheses tested using **Pearson correlation**, **Spearman rank correlation**, and OLS regression at α = 0.05.

**Stage 1 — What drives Bamboo Availability (BAI_full)?**

| Relationship | Direction | Significant? |
|---|---|---|
| Forest Area → BAI_full | Positive | Yes ✓ |
| Fragmentation Index → BAI_full | Negative | Yes ✓ |
| Road Density → BAI_full | Negative | Yes ✓ |
| NDVI Bamboo Zone → BAI_full | Positive | Yes ✓ |
| Precipitation → BAI_full | Tested | Pearson r + p-value |
| Temperature → BAI_full | Tested | Pearson r + p-value |
| LST → BAI_full | Negative | Tested |

**Stage 2 — Does BAI_full predict Panda Population? (Main Hypothesis)**

| Relationship | Method | Decision |
|---|---|---|
| BAI_full → Panda Population | Pearson r, Spearman ρ, OLS | Determined at α = 0.05 |
| Bamboo Index → Panda Population | Pearson r, OLS | Baseline comparison |

### 4. Machine Learning Models

Two models applied at each stage of the causal chain:

| Model | Type |
|---|---|
| Linear Regression | Parametric baseline |
| Random Forest | Ensemble, non-linear |

**Evaluation metrics:**
- **R² (5-fold CV)** for model comparison
- **RMSE (LOO-CV)** for absolute error — preferred with n = 29 due to lower variance with small datasets

**Additional analyses:**
- Feature importance (Random Forest) for both stages
- PCA on full feature set — explained variance scree plot
- KMeans clustering — 3 ecological regimes (High BAI Era / Transition / Low BAI Era)
- Gradient Boosting (Stage 2): actual vs. predicted panda population + residuals over time

---

## Key Findings

1. **Habitat loss is the dominant driver of bamboo availability**
   Forest area reduction and rising habitat fragmentation are the strongest predictors of BAI_full over the 1974–2002 study period.

2. **Human pressure compounds ecological decline**
   Road density expansion and urbanisation fragment bamboo corridors, reducing accessibility for giant pandas even where bamboo forest still exists.

3. **Satellite vegetation data provides the clearest signal**
   NDVI in bamboo-specific zones is a stronger predictor of BAI_full than broad regional climate variables, highlighting the importance of spatial precision.

4. **The causal chain from environment to panda is supported**
   Stage 1 results confirm that environmental and human drivers significantly shape bamboo availability. Stage 2 directly tests whether that availability, in turn, predicts panda population dynamics.

5. **Predictive modeling is constrained by sample size**
   With only 29 annual observations, LOO-CV produces high-variance R² estimates. Random Forest outperforms Linear Regression in both stages, but results should be interpreted cautiously given the small n.

---

## Limitations & Future Work

### Limitations
- Panda population data available only for three anchor years; all intermediate values are linearly interpolated
- Climate data from three fixed stations may not capture fine-grained microclimatic variation within the panda range
- Land-use and human activity series are constructed from literature-derived trend profiles rather than raw sensor measurements
- Small sample size (n = 29) limits the statistical power of both hypothesis tests and ML models

### Future Improvements
- Incorporate direct panda survey data at finer temporal resolution when available
- Add global bamboo production and trade indicators
- Apply time-series models (ARIMA, Prophet) to account for autocorrelation in annual population trends
- Expand weather station coverage across the full Qinling–Minshan mountain range
- Integrate conservation policy indicators and news sentiment
- Experiment with deep learning architectures (LSTM, Transformers) as more data becomes available

---

## Repository Structure

```
DSA-210-Project/
├── bamboo-availability-dataset.csv          # Bamboo occurrence proxy data
├── temperature&precipitation dataset.pdf    # NOAA climate data
├── DSA_210-3.ipynb                          # Main analysis notebook (all 6 datasets integrated)
├── Project Proposal.pdf                     # Initial project proposal
└── README.md
```

**Generated outputs (after running the notebook):**

```
├── final_merged_dataset.csv
├── hypothesis_test_summary.csv
├── ml_comparison_stage1_bai.csv
├── ml_comparison_stage2_panda.csv
├── fig1_eda_original.png
├── fig2_eda_new_datasets.png
├── fig3_hypothesis_tests.png
└── fig4_ml_methods.png
```

---

## Requirements

- Python 3.8+
- Jupyter Notebook

```bash
pip install pandas numpy matplotlib scipy scikit-learn jupyter
```

---

## Running the Project

```bash
git clone https://github.com/ceylinoktay-prog/DSA-210-Project.git
cd DSA-210-Project
jupyter notebook DSA_210-3.ipynb
```

Run all cells sequentially. The notebook is self-contained and generates all figures and output CSVs automatically.

---

## AI Assistance Disclosure

AI tools (ChatGPT, Claude) were used for:
- Debugging and refactoring Python code
- Improving visualization clarity
- Reviewing statistical interpretations
- Enhancing documentation and README structure

All data collection decisions, hypothesis formulation, modeling choices, and result interpretation were performed independently.

---

## Acknowledgments

- **Data Providers:** GBIF, NOAA NCEI, WWF / IUCN, ESA CCI Land Cover, World Bank, NASA MODIS / Landsat, IUCN WDPA
- **Course:** DSA 210 – Introduction to Data Science
- **Institution:** Sabancı University
