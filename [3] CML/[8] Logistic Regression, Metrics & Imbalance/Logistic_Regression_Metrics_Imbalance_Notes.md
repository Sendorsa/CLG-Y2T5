# Logistic Regression, Metrics & Imbalance – Detailed Notes

## 1. Why Logistic Regression for Classification?

- **Linear regression** predicts an exact number (unbounded: −∞ to +∞), which is not a probability.
- **Logistic regression** squeezes that unbounded output into **0–1**, representing the **probability of class 1**.
- Core idea: map any real-valued score to a probability between 0 and 1.

---

## 2. Sigmoid Function (Core of Logistic Regression)

### Formula
$$\sigma(z) = \frac{1}{1 + e^{-z}}$$

Where **z = βᵀx = β₀ + β₁x₁ + β₂x₂ + … + βₙxₙ** (linear combination of features and coefficients).

### Key Properties
- **S-shaped curve** bounded between 0 and 1 (never touches 0 or 1 exactly — asymptotic).
- At **z = 0**, σ(z) = **0.5** (decision boundary / threshold).
- z < 0 → probability approaches 0; z > 0 → probability approaches 1.
- The model is **linear in z** but becomes a classifier after applying the sigmoid.

---

## 3. Odds & Logit Function

### Odds
$$\text{Odds} = \frac{p}{1 - p}$$
- **p** = probability of success (class 1).
- Ratio of probability of success to probability of failure.
- Example: if p = 0.8, odds = 0.8 / 0.2 = **4**, meaning class 1 is **4 times more likely** than class 0.

### Logit Function (Log of Odds)
$$\text{logit}(p) = \ln\left(\frac{p}{1 - p}\right) = \beta_0 + \beta_1 x_1 + \beta_2 x_2 + \dots$$

### Derivation Summary
Starting from σ(z) = 1 / (1 + e⁻ᶻ):
1. Reciprocate: 1/p = 1 + e⁻ᶻ
2. Subtract 1: (1−p)/p = e⁻ᶻ
3. Reciprocate again: p/(1−p) = eᶻ  → this is the **odds**
4. Take natural log: ln(p/(1−p)) = z = βᵀx  → this is the **logit function**

### Interpreting β coefficients
- βj > 0 → increasing odds (eᵝʲ > 1).
- βj < 0 → decreasing odds (eᵝʲ < 1).
- The coefficient represents the change in **log-odds** per unit change in xj.

---

## 4. Log Loss Function (Cross-Entropy Loss)

### Formula
$$J(\beta) = -\frac{1}{n} \sum_{i=1}^{n} \left[ y_i \ln(p_i) + (1 - y_i) \ln(1 - p_i) \right]$$

Where:
- **yᵢ** ∈ {0, 1} (actual label)
- **pᵢ** = predicted probability σ(zᵢ)

### How it works per observation
| Actual yᵢ | Active term in loss | Explanation |
|-----------|-------------------|-------------|
| **yᵢ = 1** | −ln(pᵢ) | Penalizes when predicted p is low |
| **yᵢ = 0** | −ln(1 − pᵢ) | Penalizes when predicted p is high |

### Key Insights
- **Only one term is active** per observation (not both simultaneously).
- When wrong with **high confidence**, loss → ∞ (heavily penalized).
- When right with high confidence, loss → 0.
- **"Being confidently wrong is much worse than being uncertainly wrong."**
- The negative sign compensates for log of probabilities in (0,1), which are always negative.

---

## 5. Threshold Selection

- Default threshold = **0.5** (anything ≥ 0.5 → class 1; < 0.5 → class 0).
- Threshold is a **business decision**, not fixed.

### Examples
| Scenario | Threshold | Reason |
|----------|-----------|--------|
| **Cancer screening** | Lower (e.g., 0.1–0.25) | Catch all possible cases; false positives are acceptable vs. missing cancer |
| **Spam detection** | Higher (e.g., 0.8–0.9) | Avoid flagging legitimate emails as spam |

---

## 6. Confusion Matrix

| | Predicted Positive | Predicted Negative |
|---|---|---|
| **Actual Positive** | True Positive (TP) | False Negative (FN) |
| **Actual Negative** | False Positive (FP) | True Negative (TN) |

### Naming mnemonic
- **True/False** = Does prediction match actual?
  - True → yes (matches)
  - False → no (doesn't match)
- **Positive/Negative** = What did the model predict?
  - Positive → predicted class 1
  - Negative → predicted class 0

---

## 7. Evaluation Metrics

### Precision
$$\text{Precision} = \frac{TP}{TP + FP}$$
- Out of all **predicted positives**, how many are truly positive?
- Measures: **"Of the cases I flagged, how many were correct?"**

### Recall (Sensitivity / TPR)
$$\text{Recall} = \frac{TP}{TP + FN}$$
- Out of all **actual positives**, how many did we capture?
- Measures: **"Of the real positives, how many did I catch?"**

### F1 Score (Harmonic Mean of Precision & Recall)
$$F1 = \frac{2 \cdot \text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}}$$
- Good F1 requires **both** precision and recall to be high.
- If either is 0, F1 = 0 (very sensitive to imbalance).

### Accuracy
$$\text{Accuracy} = \frac{TP + TN}{TP + TN + FP + FN}$$
- Overall proportion of correct predictions. ⚠️ **Can be misleading under class imbalance.**

---

## 8. TPR, FPR & ROC Curve

### TPR (True Positive Rate) = Recall
$$\text{TPR} = \frac{TP}{TP + FN}$$

### FPR (False Positive Rate)
$$\text{FPR} = \frac{FP}{FP + TN}$$

### ROC (Receiver Operating Characteristic) Curve
- **Y-axis**: TPR | **X-axis**: FPR
- Shows model performance across all thresholds.
- **AUC (Area Under Curve)**:
  - AUC = 1 → perfect classifier
  - AUC = 0.5 → random guessing
  - Higher AUC → better discrimination ability

### Interpretation of ROC/AUC
- Tells how well the model distinguishes positives from negatives across thresholds.
- Low FPR is critical: we don't want to flag bad cases as good (e.g., flagging cancerous symptoms as healthy).

⚠️ **ROC/AUC can be misleading under extreme class imbalance** because massive TN inflates a deceptively low FPR.

---

## 9. Class Imbalance & Accuracy Trap

### What is Class Imbalance?
- One class has far more observations than the other (e.g., 99% legitimate, 1% fraud).

### The Accuracy Trap
- A model predicting all as majority class gets **99% accuracy** but catches **0% minority**.
- Single fraud case matters despite 99% accuracy → **accuracy is a trap**.

### Solution: Use Precision, Recall & PR-AUC
- **PR-AUC (Precision-Recall AUC)** is better than ROC-AUC under extreme imbalance.
- Always evaluate against a **baseline** (base rate of minority class).

---

## 10. Handling Class Imbalance

| Method | Description |
|--------|-------------|
| **Class weights** (`class_weight='balanced'`) | Cheapest method — penalizes misclassifying minority class more heavily |
| **Threshold tuning** | Lower threshold for high-cost scenarios (e.g., fraud detection) |
| **Resampling** | Oversample minority / undersample majority |
| **SMOTE (Synthetic Minority Oversampling)** | Synthesizes new minority samples using cross-validation (nearest-neighbour interpolation) |

---

## 11. Summary – What to Remember

1. **Why logistic regression?** Squeezes unbounded linear output into [0,1] probabilities via sigmoid.
2. **Sigmoid formula:** σ(z) = 1/(1 + e⁻ᶻ), where z = βᵀx.
3. **Odds** = p/(1−p); **Logit** = ln(odds) = βᵀx (linear in coefficients).
4. **Log loss:** Only one term active per observation; penalizes confident wrong predictions heavily (→ ∞).
5. **Thresholds:** Chosen based on business cost, not fixed at 0.5 always.
6. **Confusion matrix terms:** TP/FP/TN/FN derived from "match + prediction."
7. **Precision** = TP/(TP+FP); **Recall** = TP/(TP+FN); **F1** = harmonic mean of both.
8. **Accuracy can be misleading** under class imbalance; use PR-AUC instead of ROC-AUC in extreme cases.
9. **Handling imbalance:** class weights, threshold tuning, resampling, SMOTE.
