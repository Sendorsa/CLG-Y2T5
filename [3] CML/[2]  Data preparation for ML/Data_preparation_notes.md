# Data Preparation for ML - Class Notes

## 1. Train-Test Split

- **Split data first**: Separate training and test data before any processing
- Apply transformations **only on the training part**, not the entire dataset
- The test part must remain untouched and kept aside throughout the process
- Random train-test split is the standard approach (e.g., 80-20 split)

## 2. Pre-processing Pipeline

- Pre-processing should happen **after** the train-test split
- Fit transformations on training data, then transform both train and test
- Examples: scaling, normalization, feature engineering
- Rule: **"Make sure those operations are only done on the train"**

### Wrong Approach (Data Leakage):
```
Full Data → Pre-processing → Train-Test Split  ❌
```

### Correct Approach:
```
Train-Test Split → Fit on Train → Transform Both  ✅
```

## 3. Types of Data Leakage

### 3.1 Target Leakage
**Target leakage** occurs when the model indirectly uses information that won't be available at prediction time.

#### Common Causes:
- Using features derived from future data
- Including variables that are calculated using the target itself
- Applying transformations that require knowledge unavailable during real-world prediction

#### Example:
> If you calculate `X - 500 units` and that calculation requires knowing the target value, your model will see it during training but not at inference time.

#### Real-World Scenario:
- Predicting sales for a t-shirt store
- Adding a promo code feature = leakage if promo codes aren't known beforehand
- Model gets artificially good results on training data but fails in production

### 3.2 Temporal Leakage
Happens in time-series data when future information is used during model training. Data from the future should not be included in the training dataset if forecasting beyond that point.

### 3.3 Pre-processing Leakage
Occurs when pre-processing operations are applied on the full dataset instead of just the training data. Computing statistics (e.g., mean, median) over the entire dataset leaks information about the test set into the model.

---

### Identifying Data Leakage

- **Single feature performance**: If a model performs exceptionally well with a single feature, it may indicate leakage — one feature providing disproportionate predictive power is suspicious
- **Model intuition and testing**: Analyse and validate regularly to ensure no forecasting data leaks into training

## 4. Handling Missing Data

- Missing data is a real-world concern that must be addressed
- Strategies should be learned and applied consistently between train and test sets
- Do not randomly split time-series data — it breaks temporal order

## 5. Time-Series Data Considerations

- **Do NOT use random train-test split** for time-series data
- Time-series requires sequential/sorted splitting to preserve temporal relationships
- Future data must never leak into training
- If data is time-based, the split should respect chronological ordering

## 6. Key Principles

| Principle | Explanation |
|-----------|-------------|
| No data leakage | Ensure no information from test/future leaks into training |
| Fit on train only | Always fit scalers, encoders, imputers on training data first |
| Test is sacred | Never touch or adjust test data during modeling |
| Real-world simulation | Pre-processing must reflect what happens at prediction time |
| Monitor the process | Watch for bugs that cause pre-processing errors and leakage |

## 7. Practical Rules

1. **Split first, then process** — never process before splitting
2. **Don't use features** that won't be available at prediction time
3. **Check your features** — are they realistic for real-world deployment?
4. **For time-series** — use time-based splits, not random splits
5. **Never adjust test data** — it represents future, unseen data
6. **Be patient and careful** — small mistakes in data prep lead to big model failures

### Practical Example: Future Inventory Leakage
> Predicting sales for a store — if you include inventory levels at the end of October as a feature while trying to predict October sales, your model has access to information that wouldn't exist at prediction time. The future inventory level provides indirect future knowledge of sales, leading to artificial performance during training but failure in production.

## 8. Summary

> "This is the practical rule: keep the train part for transformations and learning, and keep the test part completely aside. If you expose the model to features it shouldn't have access to, you create target leakage. The model may look good during training but fail in production."

---

## 9. Key Terms Glossary

| Term | Definition |
|------|------------|
| **Data Leakage** | Refers to errors resulting from the improper use of information during model training that skews the results. |
| **Prediction Time** | The time at which a model outputs a prediction based on unseen test data. |
| **Training Time** | The phase where a machine learning model learns from labeled training data. |
| **Train-Test Split** | A technique to divide a dataset into two parts: one for training a model and the other for testing its performance. |
| **Feature Engineering** | The process of using domain knowledge to select, modify, or create features that make machine learning algorithms work effectively. |
| **Mean Imputation** | A method of filling missing data by substituting them with the mean of the available entries. |
| **Standard Scaling** | A preprocessing method that rescales the features such that they have the properties of a standard normal distribution with mean 0 and variance 1. |
| **MCAR (Missing Completely At Random)** | A mechanism where the likelihood of data being missing is completely random and independent of any observed or unobserved variable. |
| **Classical Machine Learning** | Encompasses traditional methods like linear regression used for predictive data modeling. |
| **Real World Data Centralization** | The practice of aggregating and maintaining coherent data from various sources to improve modeling accuracy. |

---

*Notes generated from class transcript - Data Preparation for Machine Learning*
