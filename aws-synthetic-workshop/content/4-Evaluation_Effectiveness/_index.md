---
title : "Evaluation and Effectiveness"
date: 2025-07-16
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

## 📈 Model Evaluation and Pipeline Effectiveness
After completing the synthetic data generation pipeline, we need to evaluate both the **quality of the generated data** and the **overall efficiency** of the system.  
The evaluation criteria include:

* Similarity of statistical distributions between real and synthetic data
* Ability to remove Personally Identifiable Information (PII)
* Reusability of the pipeline for different datasets
* Time and cost efficiency compared to manual processes

---

## 1. Evaluating the Quality of Synthetic Data

### 🎯 Key Metrics:
- **Histogram Similarity**: Distribution similarity between real and synthetic data for each numeric column.
- **Correlation Similarity**: Comparison of correlation matrices between real and synthetic datasets.
- **Outlier Proportion**: Ratio of outliers in synthetic data compared to the real dataset.
- **Diversity**: Degree of variety in synthetic data (avoiding excessive repetition of values).

### 🧪 Example — Checking Histogram Similarity:
Use the Wasserstein distance (Earth Mover’s Distance) to measure distribution shift:

```python
from scipy.stats import wasserstein_distance

def histogram_similarity(real_series, synthetic_series):
    real_series = real_series.dropna().astype(float)
    synthetic_series = synthetic_series.dropna().astype(float)
    distance = wasserstein_distance(real_series, synthetic_series)
    return max(0, 1 - distance) * 100  # % similarity

similarity_amount = histogram_similarity(df['amount'], synthetic_df['amount'])
print(f"Histogram similarity (amount): {similarity_amount:.2f}%")
```
🎯 Expected Results:
- Histogram similarity: ≥ 90% for key columns (amount, step…)
- Correlation similarity: ≥ 85%
- 100% of records pass PII regex and Macie scan checks

## 2. Evaluating PII Removal Capability
### ✅ Goal:
Ensure that the generated synthetic data contains no sensitive information from the original dataset.

### 🔍 How to Check:
* Automatically validate using regex patterns for emails, account numbers, and phone numbers.
* Run Amazon Macie scan on the synthetic output for confirmation.

```python
import re

patterns = {
    "email": r"[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}",
    "phone": r"\b\d{9,11}\b",
    "acct": r"\b\d{6,20}\b"
}

for col in synthetic_df.columns:
    for name, pat in patterns.items():
        matches = synthetic_df[col].astype(str).str.contains(pat, regex=True, na=False).sum()
        if matches > 0:
            print(f"⚠️ Found {matches} {name} in column {col}")
```
### 🎯 Expected Results:
* No matches in any column
---

## 3. Pipeline Reusability
### 🔄 Criteria:
- Switching datasets should only require updating the S3 path or schema mapping in Glue.
- No need to rewrite processing or ML training code from scratch.
- Pipeline can run successfully for ≥ 2 different datasets.

### 📌 How to Test:
- Re-run the pipeline with a second dataset (same tabular format).
- Measure the time from data upload to synthetic output generation.

### Expected Results:
- Setup time for a new dataset: < 15 minutes
- Core code (TVAE training, evaluation) remains ≥ 90% unchanged
---


## 4. Time and Cost Efficiency
### 4.1. Manual Process vs Automated Pipeline
| Criteria                    | Manual (anonymization + sampling) | Automated Pipeline     |
| --------------------------- | --------------------------------- | ---------------------- |
| Processing time (1 dataset) | 1–2 days                          | 2–3 hours              |
| PII leakage risk            | High                              | Low (Macie + regex)    |
| Data similarity             | Inconsistent                      | ≥ 90% statistical      |
| Cost                        | High (manpower)                   | \~0.45 USD (Free Tier) |


### 4.2. Time Savings
Example:
- First pipeline run: 3 hours (including service setup)
- Subsequent runs (reuse): < 1 hour
→ Saves ~70% of time per rerun

### 4.3. Estimated Cost (AWS Free Tier)
- S3: 0.10 USD
- Glue: 0.15 USD
- Macie: 0.20 USD
- SageMaker Studio Lab: 0 USD
→ Total:  ~0.45 USD / 1 dataset

## 5. Summary of Benefits

### 🎯 From a Technical Perspective:
* ✅ Ensures data safety, free of PII
* ✅ Synthetic data preserves utility for ML/AI
* ✅ Pipeline is easy to maintain and scale


### 💼 From a Business Perspective:
* ⏱️ Accelerates time-to-insight
* 💰Reduces data processing costs
* 📊 Improves cross-team collaboration without breaching compliance
---


