# Credit Default Risk Scorecard

> Predicting the probability of loan default and turning it into an interpretable, cost-aware credit scorecard — built on the German Credit (Statlog) dataset.

A complete, end-to-end credit-risk project: data cleaning, exploratory analysis, feature engineering, Weight-of-Evidence (WoE) transformation, Information Value (IV) screening, a calibrated logistic regression, a cost-sensitive decision cut-off, PDO scorecard scaling, validation, a fair-lending audit, and risk-band segmentation.

The goal is a model that is not just accurate but **explainable** — the kind of scorecard actually used in banks and fintech for approve / review / reject decisions.

---

## Key Results

| Metric | Value | Reading |
|---|---|---|
| AUC | **0.809** | 0.5 = random |
| Gini | 0.619 | 2·AUC − 1 |
| KS | 0.514 | > 0.30 = good separation |
| PSI (train → test) | 0.043 | < 0.10 = stable population |
| Benchmark (HistGradientBoosting) AUC | 0.786 | on par — see note below |

Moving the decision cut-off from a naive 0.50 to a cost-optimal **0.25** cut the expected cost from **220 → 149** and raised the share of true defaulters caught from **58% → 81%**.

> **On the benchmark:** the logistic model (AUC 0.809) is *on par with*, not clearly better than, the gradient-boosting model (AUC 0.786). On a 300-row test set, a ~0.02 AUC gap is within normal variation. The logistic scorecard is preferred because it matches a stronger non-linear model while staying simple to explain.

---

## Dataset

- **Source:** UCI Statlog (German Credit Data) — Hofmann, H. (1994).
- **Size:** 1,000 applicants, with a ~70/30 good/bad (default) split.
- **Mix:** categorical attributes (checking-account status, credit history, loan purpose, savings, employment, property, housing) and numeric attributes (loan duration, credit amount, age, instalment rate).
- **Quality:** no missing values, no duplicate rows.
- **License:** CC BY 4.0.

The class imbalance drives three deliberate choices: accuracy is rejected as the headline metric (in favour of AUC, KS, recall); the logistic model uses balanced class weights; and the threshold is set from a cost rule rather than the default 0.5.

---

## Methodology / Pipeline

1. **Data preparation** — load the space-delimited `german.data` with explicit column names; recode the target so class 2 (bad credit) = 1 (default).
2. **Exploratory analysis** — profile default rates across the strongest drivers (checking-account status, credit history, savings, loan duration) to confirm the signal matches credit intuition.
3. **Feature engineering** — two ratio features: `credit_per_month` (credit ÷ duration) and `amount_to_age` (credit ÷ age). Both rank among the stronger predictors by IV. Pairwise correlations checked below the 0.85 pruning threshold.
4. **WoE + IV** — bin features (shallow decision tree for numeric, native levels for categorical), score by Information Value with Laplace smoothing. **Bins learned on the training fold only**, then applied to the test fold to prevent leakage. Features with IV < 0.02 dropped; WoE features correlated > 0.85 removed.
5. **Modelling & calibration** — 70/30 stratified split; logistic regression with balanced class weights on WoE features; **isotonic calibration** so predicted probabilities line up with observed default rates.
6. **Cost-sensitive decisioning** — using the dataset's official 5:1 cost matrix (a missed default costs 5× a false alarm), choose the threshold that minimises expected cost. Selected cut-off: **0.25**.
7. **Validation & benchmark** — banking-standard metrics (AUC, Gini, KS, PSI) and comparison against a HistGradientBoosting benchmark.
8. **Scorecard (PDO scaling)** — convert calibrated probabilities to points (PDO 20, base score 600, base odds 50:1), with the offset anchored to the population's average log-odds so every risk band stays populated. Scores range 469–938 (mean 593); higher = lower risk.
9. **Risk segmentation** — bucket scores into high / medium / low bands mapped to a three-way lending policy.
10. **Fair-lending audit** — `sex` is excluded from the model entirely and used only to audit outcomes afterward (disparate-impact ratio).

---

## Information Value Ranking (top features)

| Feature | IV | Strength |
|---|---|---|
| checking_status | 0.633 | Strong |
| credit_amount | 0.304 | Strong |
| amount_to_age *(engineered)* | 0.296 | Medium |
| duration_months | 0.277 | Medium |
| credit_history | 0.254 | Medium |
| savings | 0.223 | Medium |
| credit_per_month *(engineered)* | 0.188 | Medium |
| employment_since | 0.140 | Medium |

---

## Risk Bands & Policy (test set)

| Risk band | Applicants | Default rate | Recommended action |
|---|---|---|---|
| High | 77 | 63.6% | Reject / escalate |
| Medium | 109 | 26.6% | Manual review |
| Low | 114 | 10.5% | Approve |

Default rate falls monotonically across bands (~6× drop from high to low), all bands populated.

---

## Fair-Lending Audit

`sex` (derived from the personal-status field) is **not a model input** and is used only to check outcomes. Approval rates: 54.5% (female) vs 62.8% (male) → **disparate-impact ratio 0.87**, above the common 0.80 "four-fifths" threshold. Acceptable as it stands; in production the gap would be monitored over time.

---

## Repository Structure
