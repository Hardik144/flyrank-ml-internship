# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Hardik Patidar
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/Hardik144/flyrank-ml-internship
- **Date:** 2026-09-19

> **Lane note:** This lane is selected because it describes the work implemented in the notebook. Confirm it matches the lane assigned by the internship before submitting.

## 0. Abstract

This project investigates whether historical content and search-performance indicators can help prioritize pages for human review. The analysis uses the anonymized FlyRank ML Internship content-refresh dataset, which the notebook describes as containing 30,000 records and 44 columns. A transparent heuristic score combines a declining-trend indicator, relative content staleness, below-median click-through rate within content type, and historical visibility. The notebook reports a Spearman rank correlation of approximately 0.175 with the historical-impressions ranking and an overlap of 9 records among the two rankings’ top 100. The output is intended to help human reviewers organize investigations and review work; it is not a validated prediction of future performance or evidence that refreshing content causes improvement.

## 1. Problem framing

The project addresses the question: **Can historical content and search-performance signals help identify pages that may merit review?**

- **Unit of analysis:** A content record/page.
- **Output:** A review-priority score, ranking, priority group, and suggested review action.
- **Human action:** A reviewer can inspect high-ranked records, investigate the signals contributing to their scores, and decide whether to refresh, improve, monitor, or leave the content unchanged.
- **Cost of a wrong call:** A poor ranking could divert reviewer time, delay attention to another page, or prompt an unnecessary content change.
- **Why data/ML helps:** A consistent scoring rule can combine several signals and make prioritization easier to inspect. This project uses a heuristic rather than a trained predictive model, so its value is as decision support—not as an automated quality judgment.

## 2. Data safety

The analysis uses the anonymized content-refresh dataset provided for the FlyRank ML Internship. The notebook identifies the source file as `content_refresh_anonymized.csv` and describes the dataset as containing **30,000 rows and 44 columns**.

Fields discussed in the notebook include:

- Content and client identifiers: `content_id`, `client_id`
- Search characteristics: `search_volume`, `competition`, `competition_level`, `main_intent`
- Content characteristics: `content_type`, `word_count`, `char_count`
- Search-performance metrics: `impressions_90d`, `clicks_90d`
- Engagement-related fields, including pageviews and sessions
- Other fields used by the scoring process, including `ctr`, `days_since_last_update`, and `trend_direction`

Identifiers such as `content_id` and `client_id` are for record grouping or internal reference only; they are not scoring features and must not be used to identify clients. No client names, domains, URLs, private queries, credentials, or raw exports should appear in the public paper or public artifacts.

The notebook notes that the exact date range and underlying warehouse release/table names have not been verified. Do not add those details unless they can be confirmed from the supplied documentation.

The analysis uses `trend_direction` as an explicit component of the heuristic, not as an independently verified future outcome label. The report therefore makes no claim of causal impact or validated future prediction.

## 3. Baseline

The baseline ranks records by descending `impressions_90d`, placing records with more historical impressions first.

This is a transparent, easy-to-reproduce historical-visibility rule. It is a useful point of comparison because it differs from the proposed score, which combines trend, staleness, CTR, and visibility signals. Both rankings should be computed on the same set of records.

The baseline is not a trained model and does not measure future content performance.

## 4. Model / analysis

The project uses a manually weighted heuristic score, not a supervised machine-learning model.

The score combines these components:

- **Declining signal (40%):** 1 when `trend_direction` is `"down"`, otherwise 0.
- **Staleness (30%):** The percentile rank of `days_since_last_update`; relatively older records receive a higher value.
- **Low CTR (20%):** 1 when the record’s `ctr` is below the median CTR for its `content_type`, otherwise 0.
- **Visibility (10%):** The percentile rank of `log1p(impressions_90d)`.

The combined score is:

`review_priority_score = 100 × (0.40 × declining_signal + 0.30 × staleness_score + 0.20 × low_ctr_signal + 0.10 × visibility_score)`

Records are ranked by descending score. Priority groups use score cutoffs of 40, 60, and 80, consistent with the corrected grouping thresholds in the notebook.

There is no independently verified target label for successful refreshes in this analysis. The score is therefore a heuristic ranking for review, not a prediction of whether a page will improve.

## 5. Evaluation

The notebook compares the heuristic ranking with the historical-impressions baseline over the same records. It also checks ranking stability under alternative weight configurations.

### Reported ranking comparison

- **Records analyzed:** 30,000
- **Spearman rank correlation with the impressions baseline:** approximately 0.175
- **Top-100 overlap:** 9 records (9%)

These are descriptive ranking-comparison results. The low overlap indicates that the two rules produce different top-ranked lists; it does not establish that either list is more accurate or useful.

### Sensitivity analysis

The notebook reports the following comparisons of alternative weighting configurations against the original ranking:

| Alternative configuration | Top-100 overlap with original | Spearman correlation with original |
|---|---:|---:|
| Decline-focused | 92% | 0.9973 |
| Freshness-focused | 89% | 0.9781 |
| CTR-focused | 92% | 0.9221 |

These figures describe how much the ranking changes under the tested weight variations. They do not prove that the original weights are optimal.

### Evaluation limitations

The notebook does not report a supervised train/test split, predictive accuracy, AUC, or lift against an independently verified outcome. Those metrics are not applicable to the current heuristic without a suitable outcome label and evaluation design. Do not describe this as a sealed holdout evaluation.

## 6. Interpretation

The notebook reports the following priority-group summary:

| Priority group | Records | Average score |
|---|---:|---:|
| Low priority | 10,946 | 23.061 |
| Moderate priority | 7,643 | 51.229 |
| High priority | 7,701 | 71.167 |
| Very high priority | 3,710 | 88.395 |

The reported declining-signal share is 1.0 in both the high- and very-high-priority groups. The very-high-priority group also has a low-CTR share of 1.0 and an average staleness component of approximately 0.783.

These characteristics are partly expected because the same signals are used to construct the score. They are not independent evidence that the pages are low quality or that updating them will improve performance.

The heuristic and impressions baseline have limited top-100 overlap, so the heuristic does not simply reproduce the historical-impressions ordering.

## 7. Recommendation

Use the ranking as a review queue while keeping decisions with human editors.

1. **Very-high-priority records:** Review early. Investigate declining trends, relative CTR, and freshness before deciding whether a change is justified.
2. **High-priority records:** Inspect the score components and assess whether content accuracy, relevance, structure, or presentation needs attention.
3. **Moderate-priority records:** Schedule routine review and monitor for meaningful changes.
4. **Low-priority records:** Continue ordinary monitoring and revisit when new signals or business needs arise.

The notebook’s action playbook includes investigating declining performance, reviewing freshness, examining titles and search intent where CTR is relatively low, and examining high-visibility content.

**Confidence and limits:** The output supports prioritization and investigation only. It does not guarantee future performance, establish business impact, or show that a content change causes improvement. Reviewers should verify the underlying evidence before acting.

## 8. Reproducibility

- **Repository:** https://github.com/Hardik144/flyrank-ml-internship
- **Notebook:** `work/notebooks/capstone.ipynb`
- **Input filename identified in the notebook:** `content_refresh_anonymized.csv`

The notebook uses Python data-analysis and plotting tools, including Pandas, NumPy, and Matplotlib. It includes code for generating rankings, summary tables, visualizations, and exported artifacts.

To reproduce the analysis, clone the repository, follow the project’s documented environment and dataset instructions, make the authorized anonymized dataset available at the expected path, and run the notebook from top to bottom.

Before submission, rerun the notebook and confirm that the outputs match the figures reported here and that the intended artifacts are present in the repository. Exact dependency versions, random seeds, and any required setup commands should be recorded from the actual environment rather than guessed.

This project does not claim a sealed or blind holdout evaluation. No such claim should be made unless the repository contains both the script/cell that constructs the sealed frame and the resulting metrics file.

## 9. Acknowledgments & data credit

I acknowledge FlyRank for providing the opportunity to complete this capstone project and for providing the anonymized dataset used in the analysis.

Built on the [FlyRank ML Internship dataset](https://flyrank.ai).
