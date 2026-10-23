# CampaignIQ — Marketing Uplift & Incrementality Platform

## Business Problem

RetailPulse UK, an e-commerce retailer, ran a randomised email campaign across 64,000 customers split into three groups: a men's catalogue email, a women's catalogue email, and a no-contact control group. The Head of Marketing wants to move beyond aggregate conversion rates and identify *which customers are genuinely persuadable* — those who convert only because of the campaign — so future spend is directed at those customers rather than those who would have bought anyway or those who respond negatively to outreach.

---

## Project Structure

```
CampaignIQ/
├── data/
│   ├── raw/                              # Downloaded dataset (not in Git)
│   └── processed/                        # Cleaned and scored data
├── notebooks/
│   ├── 01_eda_campaign_analysis.ipynb    # EDA, randomisation balance check, aggregate lift
│   ├── 02_statistical_inference.ipynb   # ATE, t-tests, bootstrap CIs, power analysis
│   ├── 03_uplift_model.ipynb            # T-learner, S-learner, Qini curve, AUUC
│   └── 04_targeting_strategy.ipynb      # Budget optimisation, ROI by decile, persuadable profiling
├── app.py                                # Interactive Streamlit dashboard
├── outputs/
│   └── figures/                          # Saved charts (01–13)
├── requirements.txt
└── README.md
```

---

## Key Findings

| Metric | Finding |
|--------|---------|
| Overall visit rate lift (any email) | +5.0pp vs control — statistically significant (p < 0.001) |
| Average Treatment Effect on spend | £0.77/customer incremental spend |
| Estimated total incremental revenue | ~£16,000 from 21,000 treated customers |
| Randomisation balance | All pre-treatment features balanced across groups (p > 0.05) |
| Persuadable customers identified | ~25% of customer base — the only segment worth targeting |
| Uplift model (T-Learner) AUUC | Positive — outperforms random targeting across all deciles |
| Top-decile actual lift | 3–4× higher than bottom-decile lift — model successfully separates high/low response |
| Optimal targeting threshold | Top 20–30% by uplift score captures majority of incremental value at fraction of blanket campaign cost |
| Sleeping Dogs identified | ~10% of customers — email reduces their conversion probability |

---

## Methodology

### 1 — EDA & Campaign Analysis (`Notebook 01`)
- Verified the experimental design: 3-group randomised controlled trial (RCT)
- Ran a randomisation **balance check** across all pre-treatment features to confirm the experiment was valid before any analysis
- Calculated aggregate lift in visit rate, conversion rate, and spend by group
- Segmented lift by location type, acquisition channel, recency, and prior spend tier — showed substantial heterogeneity that motivates the uplift model

### 2 — Statistical Inference (`Notebook 02`)
- Formally tested the null hypothesis (email has no effect) using two-proportion z-tests and two-sample t-tests
- Calculated **bootstrap confidence intervals** (5,000 resamples) around the Average Treatment Effect — robust to the right-skewed spend distribution
- Measured **Cohen's d** effect sizes alongside p-values to distinguish statistical from practical significance
- Ran **Bonferroni correction** across all 6 simultaneous hypotheses (3 outcomes × 2 treatment groups)
- Conducted **power analysis** to confirm the sample size could detect effects of the observed magnitude

### 3 — Uplift Modeling (`Notebook 03`)
- Implemented a **T-Learner** (two-model approach): separate Random Forest classifiers trained on treatment and control groups; individual uplift = P(visit | T=1, X) − P(visit | T=0, X)
- Implemented an **S-Learner** (single model with treatment as feature) as a comparison
- Evaluated both models using the **Qini curve** and **AUUC** — the correct metrics for uplift models (analogous to ROC/AUC for binary classifiers)
- Classified customers into four response types: **Persuadables, Sure Things, Lost Causes, Sleeping Dogs**

### 4 — Targeting Strategy (`Notebook 04`)
- Demonstrated targeting efficiency: the top 30% of customers ranked by uplift score captures the majority of total incremental value
- Built a **marginal ROI framework**: the optimal targeting threshold is where marginal revenue from the next customer targeted equals the cost of outreach
- Profiled Persuadable customers to provide the marketing team with a plain-language description of who to target
- Compared three campaign strategies — blanket, random 30%, and uplift-ranked 30% — on net ROI

---

## Live Dashboard

*(Link added after Streamlit deployment)*

---

## Technical Stack

| Layer | Tools |
|-------|-------|
| Data processing | Python, pandas, numpy |
| Statistical inference | scipy (z-test, t-test, Mann-Whitney, ANOVA, bootstrap) |
| Uplift modeling | scikit-learn (Random Forest T-Learner and S-Learner) |
| Model evaluation | Custom Qini curve and AUUC implementation |
| Visualisation | matplotlib, seaborn, plotly |
| Dashboard | Streamlit |

---

## How to Run

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Download the dataset (see data/raw/README.md for the direct link)
#    Save as: data/raw/hillstrom_email.csv

# 3. Run notebooks in order (Google Colab or Jupyter)
jupyter notebook notebooks/01_eda_campaign_analysis.ipynb

# 4. Launch dashboard
streamlit run app.py
```

---

## Data Source

[Hillstrom Email Marketing Dataset](http://www.minethatdata.com/Kevin_Hillstrom_MineThatData_E-MailAnalytics_DataMiningChallenge_2008.03.20.csv) — 64,000 customers across a randomised A/B/C email campaign, published by Kevin Hillstrom (former VP Analytics at Amazon and Netflix) as an open benchmark challenge in 2008. Widely used in the causal inference and uplift modeling literature.

---

## Key Interview Talking Points

- **Why uplift modeling over response modeling?** Response models predict who will convert. Uplift models predict who will convert *because of the campaign*. Targeting Sure Things (customers who buy regardless) wastes spend without generating incremental revenue.
- **How did you validate the experiment?** Randomisation balance check using chi-square and ANOVA across all pre-treatment features before any lift analysis — if groups were imbalanced, every downstream number would be invalid.
- **What is AUUC?** Area Under the Uplift Curve — the uplift equivalent of AUC-ROC. Measures how much better the model ranks Persuadables above other customers compared to random targeting.
- **What are Sleeping Dogs?** Customers whose conversion probability is *lower* when treated — the campaign actively hurts their conversion. A naive response model would never identify these; an uplift model flags them for suppression.
- **How do you find the optimal targeting threshold?** Marginal ROI: keep adding customers to the target list while marginal incremental revenue exceeds cost per outreach. This gives the profit-maximising threshold without arbitrary decile cutoffs.
