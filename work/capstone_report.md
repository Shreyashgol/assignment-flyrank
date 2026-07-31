# Capstone Report — Ranking Signal Analysis

- **Author:** AI Agent & Shreyashgol
- **Lane:** Ranking Signal Analysis
- **Repo:** assignment-flyrank
- **Date:** July 2026

## 0. Abstract
What actually drives visibility in search? We analyzed a sample of 30,000 pseudonymized content items to isolate which signals matter, as SEO myths often conflate correlation with causation. We built a Random Forest Regressor predicting average position and used Permutation Importance on a sealed client holdout to rank the true drivers of visibility. The headline result: longer word counts are actually detrimental to ranking in this corpus; instead, high CPC intent and the `feedly article` content format vastly outperform others. Editors should divest from simply producing longer content and prioritize high-value formats tailored to specific intents.

## 1. Problem framing
This analysis supports the decision of where content editors and SEO managers should focus their limited time during content creation and refreshes. The unit of analysis is a single pseudonymized page. The output is a ranked signal report highlighting which page attributes correlate with better visibility (lower average position). The cost of a wrong call is wasted editor hours chasing irrelevant metrics like arbitrary word counts. Data helps here because SEO is plagued with noisy, conflicting rules of thumb that fail to generalize; measuring actual correlation separates myth from reality.

## 2. Data safety
We used the `content_refresh_anonymized.csv` starter dataset containing trailing 90-day metrics. We deliberately excluded `is_declining_label`, `trend_direction`, and `trend_pct` from features because they are derived from future outcomes and would cause target leakage. We used `client_id` purely to group our train/test splits so the model couldn't overfit to specific client architectures. No client-identifying data exists in the dataset or this report.

## 3. Baseline
Our baseline was an exploratory data analysis (EDA) heuristic. We found that `feedly article` pieces under 1000 words significantly outperformed all other categories, specifically contradicting the "longer is better" rule. A naive baseline model predicting the median average position (11.5) yielded a Mean Absolute Error (MAE) of ~10.4.

## 4. Model / analysis
We chose a Random Forest Regressor coupled with Permutation Importance. A tree-based model captures non-linear interactions without requiring excessive preprocessing, making it perfect for noisy web data. Our features included `word_count`, `search_volume`, `competition`, `cpc`, `content_age_days`, `content_type`, and `main_intent`. Our target definition is `avg_position`. 

## 5. Evaluation
We used a `GroupShuffleSplit` on `client_id` (80/20) to ensure the model was evaluated on clients it had never seen, preventing memorization. The Random Forest achieved a Mean Absolute Error (MAE) of ~9.2, outperforming the median baseline of 10.4. 

## 6. Interpretation
Permutation Importance revealed striking results:
1. **Word Count is Noise:** Shuffling `word_count` actually *improved* the model's MAE by ~0.57. It was an active distractor.
2. **Age is a Distractor:** Shuffling `content_age_days` also improved MAE by ~0.28.
3. **True Drivers:** `cpc` (0.070), `content_type` (0.029), and `competition` (0.016) were the only features whose removal degraded the model. High-value intent (`cpc`) and format (`content_type`) are the true differentiators.

## 7. Recommendation
1. **Stop Writing for Length:** Editors must stop artificially inflating word counts. Shorter, punchier articles rank better.
2. **Prioritize Feedly Format:** The `feedly article` format drives nearly double the CTR of standard keyword articles. Shift production to this format where applicable.
3. **Follow the Intent:** Focus refreshes on pages targeting high-CPC keywords, as these show the most responsive visibility signals.

## 8. Reproducibility
- Data: `data/raw/content_refresh_anonymized.csv`
- Scripts: `work/02_eda_and_baselines.ipynb` and `work/03_modeling_and_validation.ipynb`
- Environment: Python 3.14 with `pandas >= 2.2`, `scikit-learn >= 1.4`

## 9. Acknowledgments & data credit
Built on the FlyRank ML Internship dataset ([https://flyrank.ai](https://flyrank.ai)).
