---
title: Capstone Report — Content Decline Prediction
permalink: /capstone-report/
---
- **Author:** Karim Yasser
- **Lane:** Machine Learning — Content Performance and Search Discoverability
- **Repo:** https://github.com/karim-yasser/flyrank-ml-internship
- **Date:** October 29, 2026

## 0. Abstract

This project investigates whether machine learning can help identify website content that is likely to experience a decline in Google Search impressions. The analysis uses anonymized FlyRank data, with historical observations from April 2025 to May 2026 for training and June 2026 for evaluation.

A Random Forest classifier was trained to identify pages whose impressions decreased by at least 20% compared with the previous month. On the June evaluation data, the model achieved 56.53% accuracy, an F1-score of 0.6601, and a ROC-AUC of 0.5635. A baseline predicting a decline for every page achieved 68.73% accuracy and an F1-score of 0.8147.

The model did not outperform the baseline on accuracy or F1-score. Therefore, it should be treated as an experimental decision-support tool rather than a reliable prediction system. Further improvements and validation are needed before using it in real content-refresh decisions.

## 1. Problem Framing

Website content can lose search visibility over time. Editors and SEO teams need a way to identify pages that may require investigation so they can prioritize their time and resources.

The main research question is:

**Can a machine learning model identify pages that are likely to experience a significant month-over-month decline in Google Search impressions?**

The unit of analysis is a page's performance observation for a particular month. The model produces a classification and an estimated probability of decline, which can be used to group pages into risk categories.

A FlyRank editor could use the output to create a review queue, investigate pages with declining impressions, and decide whether a content update or further analysis is appropriate.

A false positive may cause an editor to spend time reviewing a page that would not decline. A false negative may cause a page experiencing a decline to be missed. Both errors have a cost, so the model should be evaluated against a simple baseline.

## 2. Data Safety

The analysis used anonymized FlyRank internship data. The warehouse tables used included:

- `fact_content_daily_performance`
- `dim_content`
- `dim_clients`
- `fact_content_query_90d`

The training dataset contained 401,299 rows from 40 clients, covering target months from April 2025 through May 2026. The June 2026 evaluation dataset contained 92,375 rows from the same 40 clients.

The target was defined using the change in impressions between consecutive months. An observation was labeled as a decline when the target month's impressions were less than 80% of the previous month's impressions.

To reduce leakage risk, fields that directly describe the target outcome, including `trend_direction` and `trend_pct`, should not be used as model features when they are derived from the same target-period outcome.

Only information available before the prediction period should be considered a valid input. Fields calculated using future or target-month information must be excluded.

The project uses anonymized data and should not include client names, domains, page URLs, private keywords, or other client-identifying information in the report or committed artifacts. Restricted datasets should not be committed to the repository.

## 3. Baseline

The baseline predicts that every page will experience a decline. Although this is a simple rule, it provides a useful comparison because declines are common in the evaluation data.

The baseline results on the June 2026 evaluation set were:

| Metric | Baseline |
|---|---:|
| Accuracy | 0.6873 |
| Precision | 0.6873 |
| Recall | 1.0000 |
| F1-score | 0.8147 |
| ROC-AUC | Not applicable |

The decline rate in the evaluation set was 68.7286%. Because the baseline predicts a decline for every observation, its accuracy equals the proportion of actual declines, and its recall is 100%.

This baseline is important because accuracy alone can be misleading when one class is much more common than the other. A more complex model should provide useful improvements to justify its additional complexity.

## 4. Model and Analysis

The selected model was a Random Forest classifier. It was used to test whether a tree-based model could learn useful relationships between historical page-performance information and the decline label.

The model configuration was:

- `n_estimators = 100`
- `max_depth = 15`
- `min_samples_leaf = 50`
- `class_weight = "balanced"`
- `random_state = 42`

The target was whether impressions decreased by at least 20% compared with the previous month.

The model used historical performance and other eligible input features prepared in the capstone analysis. Features derived directly from the target outcome were excluded to avoid leakage.

The model produces a probability score that can be used to order pages for investigation. These probabilities are model estimates and should not automatically be interpreted as perfectly calibrated real-world probabilities.

The exact feature columns and preprocessing steps should match the executed capstone notebook.

## 5. Evaluation

The model was trained using historical data through May 2026 and evaluated on June 2026 data. This time-based evaluation tests the model on a later period than the main training period.

The evaluation dataset contained 92,375 rows, and 68.7286% of the observations were labeled as declines.

### Model Results

| Metric | Random Forest | Baseline |
|---|---:|---:|
| Accuracy | 0.5653 | 0.6873 |
| Precision | 0.7134 | 0.6873 |
| Recall | 0.6142 | 1.0000 |
| F1-score | 0.6601 | 0.8147 |
| ROC-AUC | 0.5635 | Not applicable |
| Average Precision | 0.7365 | Not reported |

The Random Forest achieved higher precision than the baseline, meaning that its positive predictions were more selective. However, it missed a substantial number of actual declines and had lower recall, accuracy, and F1-score.

The ROC-AUC of 0.5635 indicates limited ability to distinguish between declining and non-declining observations. Average Precision should also be interpreted alongside the high positive-class rate rather than treated as proof of strong predictive performance.

The model did not outperform the baseline on accuracy or F1-score. This is an important negative result: the experiment does not currently justify replacing the simple baseline with the Random Forest for general decline detection.

Further error analysis could examine false positives and false negatives by client, page characteristics, and historical performance. These differences should be measured before making claims about where the model works best.

## 6. Interpretation

The main finding is that the Random Forest did not provide a strong improvement over the baseline on the June 2026 evaluation set.

The model's precision was 0.7134, compared with 0.6873 for the baseline. However, the improvement in precision came with a reduction in recall and a lower F1-score. The model's ROC-AUC also suggests that its ability to rank declining observations above non-declining observations is limited.

The risk groups provide another way to inspect the model's output:

| Risk Group | Predicted Probability | Rows | Actual Decline Rate | Mean Predicted Probability |
|---|---|---:|---:|---:|
| Very Low | Below 0.30 | 2,591 | 45.04% | 26.26% |
| Low | 0.30–0.50 | 35,127 | 66.41% | 42.69% |
| Medium | 0.50–0.70 | 43,212 | 69.58% | 59.24% |
| High | 0.70–0.90 | 11,394 | 77.93% | 75.86% |
| Very High | Above 0.90 | 51 | 96.08% | 90.92% |

The observed decline rate generally increases across the risk groups, but the differences are modest for several groups. The Very High group contains only 51 rows, so its observed decline rate should be treated cautiously.

These results suggest that the model's scores may help organize an initial review, but they are not sufficient to establish a reliable ranking system. The findings are observational and do not show that any particular page characteristic causes a decline in search impressions.

## 7. Recommendations

The current model should be treated as an experimental decision-support tool rather than a production-ready system.

Recommended actions, in priority order:

1. **Keep the baseline as a reference.** Any new model should be compared with the same baseline and evaluation period.
2. **Investigate the model's errors.** Review false positives and false negatives to understand where the model fails.
3. **Improve the features carefully.** Test eligible historical performance signals while excluding target-derived fields and future information.
4. **Validate on additional future periods.** A single evaluation month is not enough to establish stable performance.
5. **Use human review for decisions.** Editors should check page and search-performance evidence before deciding to refresh content.

A FlyRank editor could use the current risk groups to organize exploratory investigations, but should not automatically prioritize or update pages based on the score alone.

The next version should only be recommended for broader use if it demonstrates consistent and meaningful improvements on properly separated evaluation periods.

## 8. Reproducibility

The main analysis was performed in Google Colab using the repository's full-release analysis notebook:

`notebooks/03_working_with_the_full_release.ipynb`

The capstone report and related analysis should be kept in the repository's `work/` directory. The analysis should be reproducible from the approved data source and the notebook's documented setup instructions.

The recorded Random Forest configuration uses `random_state = 42`. The preprocessing steps, feature columns, data filters, and evaluation calculations should match the executed notebook.

To reproduce the work:

1. Clone the repository.
2. Read the repository setup and data-use instructions.
3. Request access to the approved warehouse dataset if required.
4. Open `notebooks/03_working_with_the_full_release.ipynb` in Google Colab.
5. Follow the notebook's setup instructions and run the analysis cells in order.
6. Compare the newly generated metrics with the results recorded in this report.
7. Commit the report and permitted metric artifacts without committing restricted datasets.

The results should be refreshed from a clean run before final submission. Differences caused by changes in data, dependencies, preprocessing, or model configuration should be documented.

## 9. Acknowledgments and Data Credit

This project was completed as part of the FlyRank ML Internship using the anonymized internship dataset.

The work follows the internship's data-use requirements and presents the results as measured findings for decision support. It does not claim to predict Google's ranking algorithm or establish causal relationships.

**Data source:** [FlyRank](https://flyrank.ai)

---

## Final Claims Checklist

- Results are presented as measured findings, not causal claims.
- The model is described as a decision-support experiment, not a guaranteed prediction system.
- The high decline base rate is reported alongside the evaluation metrics.
- The report does not claim that the model outperforms the baseline.
- Before final submission, verify the exact feature columns, preprocessing, and reproduction steps against the executed notebook and committed artifacts.
