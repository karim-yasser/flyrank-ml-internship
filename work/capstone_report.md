# Capstone Report — Content Decline Prediction

- **Author:** Karim Yasser
- **Lane:** Machine Learning — Content Performance and Search Discoverability
- **Repo:** https://github.com/karim-yasser/flyrank-ml-internship
- **Date:** October 29, 2026

## 0. Abstract

This project investigates whether machine learning can help identify website content that is likely to experience a decline in Google Search impressions. The analysis uses anonymized FlyRank data, with historical observations from April 2025 to May 2026 for training and June 2026 for evaluation. A Random Forest classifier was trained to identify pages whose impressions decreased by at least 20% compared with the previous month. On the June evaluation data, the model achieved 56.53% accuracy, an F1-score of 0.6601, and a ROC-AUC of 0.5635, while a baseline predicting a decline for every page achieved 68.73% accuracy and an F1-score of 0.8147. The output is intended to support human review and content-refresh prioritization, not to make automatic decisions or predict Google's ranking algorithm.

## 1. Problem framing

Website content can lose search visibility over time. Editors and SEO teams need a way to identify pages that may require investigation, so they can prioritize their limited time and resources.

The research question for this project is:

**Can a machine learning model identify pages that are likely to experience a significant month-over-month decline in Google Search impressions?**

The unit of analysis is a page's performance observation for a particular month. The output is a classification and an estimated probability of decline, which can be used to group pages into risk categories.

A FlyRank editor could use the output to create a review queue, investigate pages with declining impressions, and decide whether a content update or further analysis is appropriate.

A false positive may cause an editor to spend time reviewing a page that would not decline. A false negative may cause a page experiencing a decline to be missed. Both errors have a cost, so the model should be evaluated against a simple baseline before it is used in practice.

Machine learning may help identify patterns across many observations, but a simple rule can sometimes perform just as well or better. This project therefore compares the trained model with a baseline instead of assuming that a more complex model is automatically better.

## 2. Data safety

The analysis used the approved, anonymized FlyRank internship data. The warehouse tables used in the analysis included:

- `fact_content_daily_performance`
- `dim_content`
- `dim_clients`
- `fact_content_query_90d`

The training dataset contained 401,299 rows from 40 clients, covering the target months from April 2025 through May 2026. The June 2026 evaluation dataset contained 92,375 rows from the same 40 clients.

The target was defined using the change in impressions between consecutive months. A row was labeled as a decline when the target month's impressions were less than 80% of the previous month's impressions.

To reduce leakage risk, fields that directly describe the target outcome, including `trend_direction` and `trend_pct`, should not be used as model features when they are derived from the same target-period outcome. Pseudonymous client identifiers should be used for grouping or validation where appropriate, not as predictive features.

Only information available before the prediction period should be considered as a valid input. Any fields calculated using future or target-month information must be excluded.

The project uses anonymized data and should not include client names, domains, page URLs, private keywords, or other client-identifying information in the report or committed artifacts
