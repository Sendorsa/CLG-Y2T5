# Regularization & Cross-Validation — Class Notes

## Quiz 9 (Section A) — Logistics

### Format
- MCQ + 2 short coding questions
- One mathematical question in MCQ (two options provided)
- Duration: ~30–45 min (may complete in 20–25 min)

### Coding Question Expectations
- Know which libraries/imports to use (reference the first notebook sent out)
- Translate simple formulas into 1–2 lines of code
- No complex problem solving — just awareness of model imports and usage
- Every known regression model should be familiar (what models to import)

### Mathematical Questions
- Test recall of key formulas, not derivation or calculations
- Core idea: remember the formula

### Coverage & Timeline
- Covers ~Sessions 1–8
- MCQ part: all answers expected as correct (no off-topic questions)
- Section A to be completed by next Saturday

---

## Regularization & Cross-Validation Overview

### Context in Course Flow
- Next layer on top of bias-variance (Sessions that introduced bias/variance and their causes/remedies)
- Very short, highly conceptual lecture
- All formulas link back to Session 2 — once you see the hint, previous concepts connect

### Key Distinctions
- **Regularization & Cross-validation are techniques, NOT algorithms**
- Lasso = L1 regularization
- Ridge = L2 regularization

---

## Intuition Behind Regularization

### The Problem: Model Complexity
- When coefficients become very large (due to many features), model complexity explodes
- Example: multiplying a feature by a huge value makes the model overly dependent on it, creating enormous spread in predictions

### Regularization as "Price for Complexity"
- Adds a penalty term — like allowing students to discuss answers but deducting 0.2 marks per discussion
- Maximum score drops; getting extra features is not free
- Core philosophy: trade-off between performance and complexity

### Cost-Benefit Perspective (SpaceX vs NASA Analogy)
- **Spacex**: highly optimized, low cost, scarce resources drive innovation — similar to what regularization aims for
- **NASA**: extremely expensive, not resource-optimized
- Engineers' dream: get close to target with minimal resource utilization
- Regularized models: slightly lower performance but far cheaper computation

### R² vs Adjusted R²
- R² always increases with more features (even irrelevant ones)
- Adjusted R² accounts for feature count — penalizes unnecessary complexity

---

## Loss Function with Penalty

```
Loss = MSE + λ · penalty(β)
```

- Previously: only cared about how well model fits (MSE)
- Now: also care about **how complex** the model is
- **λ (lambda)** = regularization strength / penalty parameter
- Larger λ → smaller coefficients → less complex model
- As λ → ∞, all coefficients approach 0 (but never exactly 0 with Ridge)
- As λ = 0, no regularization applied

---

## L1 Regularization (Lasso)

### Key Property: Feature Selection via Sparsity
- Penalty uses **absolute value** of β: |β₁| + |β₂| + ...
- Can cause coefficients to become **exactly zero** → features get eliminated
- Useful when you want automatic feature selection

### Trade-off
- Some features dropped, others may survive
- Best when many features are irrelevant

---

## L2 Regularization (Ridge)

### Key Property: Shrinkage Without Elimination
- Penalty uses **squared value** of β: β₁² + β₂² + ...
- Coefficients shrink toward zero but **never become exactly zero**
- Never drops any feature — all features remain in the model
- Useful when you want to keep all features but reduce their weight

### Why Squared (Not Absolute Value)?
- With absolute values, positive and negative β can cancel out (e.g., +5 and −5 = 0)
- Squaring captures **magnitude only** — no cancellation possible
- Also makes XᵀX invertible in cases of multicollinearity

### Matrix Formulation Insight
```
β_Ridge = (XᵀX + λI)⁻¹ Xᵀy
```
- λI added to diagonal keeps matrix invertible even with correlated features
- All 10 features remain present but with reduced weightage

---

## Key Differences: Lasso vs Ridge

| | **Lasso (L1)** | **Ridge (L2)** |
|---|---|---|
| Penalty | \|β\| (absolute) | β² (squared) |
| Coefficients | Can be exactly 0 | Shrink toward 0, never 0 |
| Feature selection | Yes | No |
| Use case | Many irrelevant features | All features matter |
| Features dropped | Some features eliminated | All features kept |

### Easy Memorization Tip
- **L1 → Lasso** (1 in lasso)
- **L2 → Ridge** (2 in ridge)

---

## Feature Scaling Before Regularization — Critical!

### Why Scaling Is Required
- Features must be on the **same scale/unit** before regularizing
- Example: comparing km to meters is meaningless → convert to same unit first
- Without scaling, a feature with larger values gets unfairly penalized more

### Normalization / Standardization Range
- Min-max normalization: values range 0–1
- Helps keep λ penalty consistent across features

### Important Rule
- **Scale only on training data** — test data stays aside
- Calculating stats (mean, std) from entire batch before splitting leaks information from test into train

---

## Beta-Not (Intercept / β₀) — Why Not Penalized?

### Intuition: Benchmark / Base Price
- β₀ acts as a **benchmark** or **base level** for predictions
- Like Uber's base fare — covers minimum cost regardless of distance
- Not tied to any feature; it's the starting point of prediction

```
y = β₀ + β₁x₁ + β₂x₂ + ...
     ^^^^   ^^^^^^^
   NOT       PENALIZED
   PENALIZED
```

- If β₀ were penalized to zero, even baseline predictions could be distorted
- It recovers the base cost/value that must always exist

---

## Extreme Values of Lambda

| **Lambda Value** | **Effect** |
|---|---|
| λ → 0 | No penalty, model overfits (high variance) |
| High λ | All coefficients → 0, model becomes flat line (low bias, high bias model — underfitting) |
| Low λ | Weak regularization, more complex model |

- With very high λ: every feature coefficient approaches 10⁻⁰ → no feature has any advantage → model can't understand which features predict the target
- Result: worse predictions than without proper tuning

---

## Cross-Validation (Preview)

- Next topic after regularization techniques
- K-Fold cross-validation: K = number, how many folds?
- Purpose: find optimal hyperparameters while avoiding data leakage

---

*Notes compiled from Class Scaler Academy lecture transcript on Regularization & Cross-Validation.*
