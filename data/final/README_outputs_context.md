# Final Outputs Context

Notebook output routing follows `data/` subfolders:

- `data/raw/`: source input files.
- `data/processing/`: intermediate processed/modeling datasets.
- `data/logs/`: diagnostics and feature-selection logs.
- `data/final/`: final modeling deliverables.

## data/processing

- `data_split_out_of_sample_train_test.csv`: ID-level out-of-sample split assignment (`train`/`test`).
- `data_split_out_of_time_last_year_train_test.csv`: out-of-time split assignment (last year as test).
- `company_best_case_once.csv`: one representative pre-default record per company with health score.
- `feature_univariate_stats.csv`: univariate feature diagnostics by target class.
- `high_correlation_pairs.csv`: high-correlation feature pairs over threshold.
- `woe_train_features.csv`: WoE-transformed training features.
- `woe_test_features.csv`: WoE-transformed test features.

## data/logs

- `data_split_summary.csv`: split composition by year/type with default rates.
- `vif_final_table.csv`: final VIF table after multicollinearity filtering.
- `vif_drop_log.csv`: iterative VIF drop trace.
- `woe_iv_summary.csv`: WoE/IV summary per feature.
- `optbinning_failed_features.csv` (optional): failed binning diagnostics.
- `iv_selected_features.csv`: features passing IV threshold.
- `selected_features_for_logit.csv`: final selected feature table used for logit input.

## data/final

- `selected_features_for_logit.json`: machine-readable selected features and settings.
- `logit_selected_woe_dataset_out_of_sample.csv`: final logit-ready dataset for out-of-sample evaluation.
- `logit_selected_woe_dataset_out_of_time.csv`: final logit-ready dataset for out-of-time evaluation.

Logit selected output rule: always exactly two files in `data/final/`, one per split strategy.
