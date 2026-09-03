# Bias-Variance & Polynomial Regression — Detailed Notes

## 1. Sources of Model Error (Why does a model perform badly?)

Three main factors:
- **High Bias** → Underfitting
- **High Variance** → Overfitting
- **Noise** → Irreducible error

---

## 2. Noise (Irreducible Error)

- Noise exists in the real world; it is **outside our control**.
- No model can perfectly predict outcomes due to inherent uncertainty in the data.
- Example: Weather forecasts are uncertain regardless of model quality.
- **Cannot be reduced** — we can only try to minimize reducible error (bias + variance).

---

## 3. Bias

### Definition
- High bias means the model's predictions on average are **far from the target**.
- The model is **not fitting** the training data well → **underfitting**.

### Causes
- Model is too simple for the underlying data pattern.
- Low model complexity.

### Reduction Strategies
- **Increase model complexity**: Add more features, use polynomial features.
- Use more expressive models (e.g., decision trees instead of linear models).
- Improve feature engineering / provide better features and data.

---

## 4. Variance

### Definition
- High variance means the model learns the training data **too well** (including noise).
- Performs excellently on training but **fails on testing/validation data**.
- Model is overly sensitive to small fluctuations in the training set → **overfitting**.

### Cause: Sample-to-Sample Stability
- Training data is a **random sample** from the population.
- Different samples produce different trained models → predictions vary (fluctuate).
- Formula intuition: If actual value = 100, different samples might give predictions like 120, 90, 95, 98 — their average gives a stable prediction.

### Reduction Strategies
- **Reduce model complexity**.
- Provide **more diverse / larger training data** so the model doesn't memorize noise.
- Use regularization (covered in next lectures).

---

## 5. Model Complexity

- Controls how flexible the model is.
- Increase complexity → **bias decreases, variance increases**.
- Decrease complexity → **variance decreases, bias increases**.
- Example: Asking a student to study 10 hours instead of 2 hours = increasing "complexity" so they learn more from training data.

---

## 6. Bias-Variance Decomposition

The total expected error is decomposed into three parts:

### Error Decomposition Equation (derived from `E[(y - f̂(x))²]`)

```
Total Expected Error = Noise (Irreducible Error) + Variance + Bias²
```

### Derivation Steps
1. Start with squared error on a given sample: `(y - ŷ)²`
2. Add and subtract expected values `E[y]` (true signal) and `E[ŷ]` (mean prediction):  
   `(y - ŷ) = (y - E[y]) + (E[y] - E[ŷ]) + (E[ŷ] - ŷ)`
3. Square the expression:  
   `[(y - E[y]) + (E[y] - E[ŷ]) + (E[ŷ] - ŷ)]²`
4. Take expectation `E[·]`:
   - The cross-term `E[(y - E[y])(...)]` → **cancels to zero** because `E[y - E[y]] = 0` (mean of deviations from mean is zero).
5. Final result:
   - `E[(y - E[y])²]` → **Noise** / Irreducible error
   - `E[(E[ŷ] - ŷ)²]` → **Variance** = Var(ŷ)
   - `(E[ŷ] - E[y])²` → **Bias²**

### What each term represents
| Term | Meaning | Reducible? |
|------|---------|------------|
| **Noise** = E[(y - ȳ)²] | Error due to inherent uncertainty; cannot be improved by any model | No (irreducible) |
| **Variance** = E[(f̂_s(x) - E[f̂_s(x)])²] | Average spread of predictions from different samples around their mean | Yes — reduce model complexity or add more data |
| **Bias²** = (E[f̂_s(x)] - ȳ)² | How far the average prediction is from the actual target value | Yes — use more complex/expressive models, better features |

---

## 7. Dartboard Analogy (Intuitive Understanding of All 4 Combinations)

| | Low Variance | High Variance |
|---|-------------|---------------|
| **Low Bias** | ✅ **Optimal model** — shots clustered tightly around bullseye | Overfitting — shots near average target but scattered |
| **High Bias** | Underfitting — shots clustered but far from bullseye | ❌ **Worst model** — both high bias & high variance (miserable model) |

### Detailed Breakdown

1. **High Bias, High Variance**: Shots are scattered and far from the target → worst performing model
2. **High Bias, Low Variance**: Shots land together but miss the target consistently → underfitting
3. **Low Bias, High Variance**: Shots cluster around/near the target but widely spread → overfitting (some hit near bullseye!)
4. **Low Bias, Low Variance**: Shots clustered tightly on the bullseye → ideal balanced model

### Key Distinction — "Hit vs Cover" Bullseye
- **Hit the target** = minimize bias; all 10 shots should land at bullseye
- **Cover the target** = high variance (shots spread but average near target)
- Our goal: **hit the bullseye with all shots** (both low bias AND low variance)

---

## 8. Reducible vs Irreducible Error

| Type | What it is | How to handle |
|------|-----------|---------------|
| **Irreducible (Noise)** | Inherent uncertainty in data/environment | Accept it; cannot reduce |
| **Reducible: Bias** | Average prediction far from true target | Increase model complexity, add features, use better models/data |
| **Reducible: Variance** | Predictions fluctuate a lot across samples | Decrease model complexity, add more diverse training data, regularize |

### Quick Diagnosis Questions
- **Q**: "If my model fits training data perfectly every time, what happens to variance?"  
  **A**: High variance (overfitting — learning too much noise)
- **Q**: "If my model always outputs a flat line, what happens to bias and variance?"  
  **A**: Variance = 0 (no fluctuation), Bias is large (flat line far from true target)

---

## 9. Polynomial Regression

### Why Do We Need It?
- Linear regression assumes a linear relationship: `y = β₀ + β₁x`
- This assumption may not hold → model fails to capture complex patterns
- Solution: Use polynomial features to create a **curved relationship** while keeping parameters linear

### How Polynomial Features Work
- We do NOT create new variables — we transform existing features into higher-degree versions.
- Example: Original features `x₁, x₂, x₃` → degree 2 polynomial features include the original terms plus all products: `x₁, x₂, x₃, x₁², x₂², x₃², x₁x₂, x₁x₃, x₂x₃`.
- The **parameters remain linear** (`β₁ * x₁`, `β₂ * x₂²`) — only the relationship becomes non-linear/curved.

### Linear Parameters in Polynomial Regression
```
y = β₀ + β₁x₁ + β₂x₂² + β₃(x₁·x₂) + ...  [each term has a single linear parameter]
```
The **equation is curved** (polynomial), but the **parameters are still linear** — this is called "maintainable linearity."

### Feature Explosion in Polynomial Regression
- Number of polynomial terms grows rapidly with degree `d` and dimension `N`.
- Formula: `(N + d)! / (d! × N!)` → binomial coefficient
- Example: N = 5 features, d = 3 degree → **56 polynomial terms**
- This is why **feature scaling is necessary**: unscaled features cause large numerical values when raised to powers

---

## 10. Learning Curves

Learning curves plot training error and validation error against training set size (or model complexity).

### High Bias Scenario
- Both training and test errors are **high**
- As more data is added: training error increases, test error decreases → they **converge at a high level**
- More data **does not help** beyond convergence point
- **Remedies**: Add more complex features, reduce regularization
- Adding more data beyond the gap = no improvement

### High Variance Scenario
- Large persistent gap between training (low) and test (high) errors → **overfitting**
- Test error scored 70% but training error scored 99% → model memorized training data
- **Remedies**: Reduce model complexity, provide more diverse training data

### Practical Example
> A student scores near-perfectly on unit tests but drops significantly in the final exam. The gap should be minimized — test performance at least matches training performance. Going slightly beyond is acceptable.

---

## 11. Interview Questions & Answers

| Q | A |
|---|---|
| What are the three terms of bias-variance decomposition? | Noise, Variance, Bias² |
| Why do we square in decomposition? | Squaring catches both positive and negative errors (e.g., +10 and -10 would cancel if not squared) |
| Which is reducible? | Both Bias and Variance are reducible; Noise / Irreducible Error is not |
| Does more data reduce bias or variance? | Reduces **variance** by preventing overfitting (doesn't memorize existing data) |
| Why is polynomial parameter linear? | Each polynomial term has a single coefficient `βᵢ` — the parameters remain linear even though features are non-linear |
| When does adding more data NOT help? | During **high bias** — once training and test error converge, extra data provides no benefit |

---

## 12. Summary

- **Total Error = Noise + Variance + Bias²**
- **Bias** → underfitting; fix by increasing complexity
- **Variance** → overfitting; fix by reducing complexity or adding diverse data
- **Noise** → inherent, irreducible
- Use the **dartboard analogy** to quickly identify the bias-variance combination
- **Polynomial regression** creates curved relationships with linear parameters
- Watch for **feature explosion** — scale features before going high-degree
- **Learning curves** help diagnose whether you have bias or variance problems
