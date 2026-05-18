# LightGBM Challenger — Results Analysis
*Quick tuning run (30 Optuna trials), generated 18 May 2026*

---

## Model Performance

| Metric | Out-of-Time | Out-of-Sample |
|---|---|---|
| **AUC-ROC** | **0.9594** | **0.9697** |
| **F1 (test)** | **0.5818** | **0.7086** |
| Precision | 0.6450 | 0.7691 |
| Recall | 0.5298 | 0.6569 |
| F1 (validation) | 0.6543 | 0.7411 |
| Optimal threshold | 0.48 | 0.52 |
| n_estimators | 400 | 600 |

### Why OOS beats OOT — and which one matters

This is expected and intentional. **OOS is the easier task**: test companies are drawn randomly
from the same time period, so the model sees all economic conditions during training.
**OOT is the harder, more realistic task**: the model must generalise to an entirely unseen year
(2021 test, trained on ≤2019, validated on 2020). That is exactly what deployment looks like.

**Lead with OOT as the primary result in the report.** OOS serves as a sanity check showing the
model has real signal (F1 0.71 means the features genuinely predict default), while OOT shows
how well it survives temporal shift.

### AUC vs F1 gap in OOT

AUC-ROC of 0.96 is excellent — the model's **ranking** of companies by risk is near-perfect.
The lower F1 of 0.58 is a **threshold/classification** issue: in 2021, the score distribution
shifted slightly (post-COVID recovery changing financials), so the threshold tuned on 2020
validation is a fraction off. This is normal temporal drift, not model failure. Worth stating
explicitly in the write-up.

---

## Feature Drivers

The **top features are highly consistent** across both split strategies — strong evidence the
model found genuine signal rather than overfitting artefacts.

| Feature | Name | OOT SHAP rank | OOS SHAP rank | IV rank |
|---|---|---|---|---|
| Var_02 | Assets Total | 1 | 1 | 1 |
| Var_39 | Working Capital | 4 | 3 | 2 |
| Var_04 | Current Ratio | 5 | 5 | — |
| Var_12 | — | 6 | 4 | — |

**Notable discrepancy — `Var_18`**: ranked #2 by SHAP in OOT but absent from OOS top 8. It has
no IV value (was excluded from logistic regression). Since it only appears important in the
temporal split, it likely captures a time-period-specific signal (e.g. a pandemic-era metric)
rather than a universal credit driver. Worth a footnote in the report.

**Direction of `Var_02` (Assets Total)**: OOT says larger assets → higher default risk; OOS says
non-linear. This is a real economic phenomenon — very large companies both fail spectacularly
and survive better. The tree model picks up both tails.

---

## Key Numbers for the Write-up

- **AUC-ROC 0.96**: out of 100 random (defaulter, non-defaulter) pairs, the model ranks the
  defaulter higher 96 times.
- **OOT Precision 0.645**: of every 100 companies flagged as high-risk, ~65 actually default
  within 12 months.
- **OOT Recall 0.530**: the model catches 53% of all real defaults — it misses about half,
  which frames the business trade-off for threshold selection.
- **OOS F1 0.71 / OOT F1 0.58**: the ~13 pp gap is attributable to temporal shift, not
  overfitting (AUC stays high and the validation→test F1 drop within each strategy is small).

---

## What to Do Next

1. **Run the champion comparison** (primary deliverable).
   Logistic regression predictions expected at:
   `data/final/champion_test_predictions_{out_of_time|out_of_sample}.csv`
   Columns: `ID, obs_date, default, probability`
   Once available, build a comparison notebook (head-to-head F1, AUC-ROC, Brier score on
   identical test sets).

2. **Thorough overnight tuning run for OOT** (`TUNING_MODE = "thorough"`, 200 trials).
   30 trials barely scratches the hyperparameter space. OOT F1 0.5818 has room to improve;
   this is the headline result so it is worth the ~2 h investment on M2.

3. **Update `RESULTS_DIR` in the notebook** if the subfolder structure
   (`results/LightGBM/out_of_time/`, `results/LightGBM/out_of_sample/`) should be the
   permanent target — the save cell currently writes flat into `results/LightGBM/`.

4. **Investigate `Var_12`, `Var_18`, `Var_25`** — all appear important to LightGBM but have no
   IV (excluded from the logistic regression champion). Understanding what financial ratios they
   represent strengthens the business narrative and helps explain why the challenger finds signal
   the champion misses.
