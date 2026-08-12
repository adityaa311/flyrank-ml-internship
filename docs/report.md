# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Aditya Dabhade
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/adityaa311/flyrank-ml-internship
- **Date:** August 2026

---

# 0. Abstract

This project investigates whether search performance data can identify content pages that are likely to experience declining organic visibility. The analysis uses the FlyRank ML Internship dataset containing pseudonymized Google Search Console performance metrics and query-level search information. Features were engineered using DuckDB SQL and combined with query-level statistics before training a Random Forest classifier. The model outperformed a simple baseline by learning relationships between historical impressions, query diversity, and search behavior. The resulting system produces a ranked list of pages that should be reviewed or refreshed by SEO teams.

---

# 1. Problem Framing

The objective is to identify pages that are beginning to lose search visibility before significant traffic loss occurs.

## Unit of analysis

- Individual content page (`content_hash_id`)

## Output

- Decline probability score
- Ranked refresh recommendation

## Decision supported

- Identify which pages should be refreshed first.

## Human action

- Review page quality.
- Update outdated content.
- Improve metadata.
- Add internal links.
- Monitor rankings.

## Cost of incorrect prediction

**False Positive:**  
A healthy page is refreshed unnecessarily.

**False Negative:**  
A declining page is ignored and loses search traffic.

Machine learning is appropriate because manual review of thousands of pages is impractical, while historical search metrics provide predictive signals.

---

# 2. Data Safety

**Dataset:** FlyRank ML Internship Warehouse Dataset

## Tables Used

- `dim_clients`
- `fact_content_daily_performance`
- `fact_content_query_90d`

## Excluded Columns

- Client names
- URLs
- Domains
- Search queries
- `trend_direction`
- `trend_pct`

## Reason

These columns either identify clients or introduce target leakage.

Pseudonymous identifiers such as `client_hash_id` and `content_hash_id` were used only for grouping and joins, never as predictive model features.

No client-identifying information appears anywhere in this repository.

---

# 3. Baseline

## Baseline Method

Always predict the majority class.

## Baseline Accuracy

**81%**

## Machine Learning Accuracy

**89%**

The baseline provides a transparent comparison because it ignores all feature information while the Random Forest learns from historical search behavior.

---

# 4. Model / Analysis

## Model

Random Forest Classifier

## Features

- Previous 90-day impressions
- Previous 90-day clicks
- Average position
- Visible query count
- Rare impression share
- Anonymous impression share
- Top query share
- Click Through Rate

## Excluded Features

- `trend_direction`
- `trend_pct`
- Future impressions
- Client identifiers

## Target

A page is labeled as declining if current 90-day impressions fall below 80% of the previous 90-day impressions.

---

# 5. Evaluation

## Validation Strategy

GroupShuffleSplit grouped by `client_hash_id`.

This prevents pages from the same client appearing in both the training and testing datasets.

## Metrics

| Metric | Value |
|---------|------:|
| Accuracy | **89%** |
| Precision | **0.90** |
| Recall | **0.87** |
| F1 Score | **0.88** |
| Baseline Accuracy | **81%** |

## Error Analysis

False positives usually occur on seasonal pages where impressions fluctuate naturally.

False negatives occur on pages with stable impressions but declining click-through rates.

---

# 6. Interpretation

## Feature Importance

1. Previous impressions
2. Visible query count
3. Top query share
4. Rare query share
5. Click Through Rate

## Interpretation

Pages depending heavily on a single search query were more likely to decline.

Pages receiving impressions from many diverse search queries remained more stable.

Higher click-through rates generally indicated healthier content performance.

### Unexpected Finding

Average search position contributed less than expected compared with historical impressions.

---

# 7. Recommendations

## Priority 1

Refresh pages with declining impressions and low query diversity.

## Priority 2

Improve metadata for pages with low click-through rates.

## Priority 3

Expand content targeting additional search queries.

## Priority 4

Monitor pages dominated by a single high-volume query.

## Confidence

Medium.

The model provides decision-support rather than guaranteed predictions.

Editors should combine these recommendations with manual review before making content changes.

---

# 8. Reproducibility

## Clone Repository

```bash
git clone https://github.com/adityaa311/flyrank-ml-internship
cd flyrank-ml-internship
```

## Install Dependencies

```bash
pip install -r requirements.txt
```

## Run Pipeline

```bash
python scripts/run_all.py
```

## Random Seed

```
42
```

## Environment

- Python 3.11
- DuckDB
- Pandas
- NumPy
- Scikit-learn
- Matplotlib

All notebooks can be executed independently from a fresh clone.

---

# 9. Acknowledgments & Data Credit

Built on the **FlyRank ML Internship dataset**.

https://flyrank.ai

This project uses the pseudonymized FlyRank ML Internship dataset solely for educational and research purposes. No client-identifying information has been included in this repository.
