# Linear Regression Assumptions — Detailed Notes

## 1. Six Core Assumptions of Linear Regression

| # | Assumption | Description | What Violation Looks Like in Residuals |
|---|-----------|-------------|----------------------------------------|
| 1 | **Linearity** | Relationship between features and target is linear | Curved pattern in residuals (e.g., parabola, U-shape) |
| 2 | **Normality of Errors** | Residuals follow a normal distribution | Deviations from straight line on Q-Q plot; Shapiro-Wilk test |
| 3 | **Homoscedasticity** | Constant variance of errors across all predicted values | Funnel/fan shape in residual plot (spread increasing or decreasing) |
| 4 | **Independence of Errors** | No autocorrelation between consecutive residuals (especially in time series) | Patterned errors: +1, −5, +1, −6 or +2, +3, +4, +5 instead of random |
| 5 | **No Multicollinearity** | Features should not share overlapping/redundant information | High VIF; inflated standard errors |
| 6 | **No Influential Outliers** | No single data point unduly influences the regression line | Extreme leverage + high residual changes model shape if removed |

---

## 2. Linearity

- Assumes systematic bias is zero — predictions fluctuate equally above and below the actual values.
- A good model predicts around the true line (e.g., ±2 units in both directions), not always under-predicting or over-predicting.
- **Industry nuance**: Slight over-prediction vs. under-prediction depends on use-case:
  - *Retail*: slight over-prediction acceptable (stockouts cost more).
  - *FMCG / Food industry*: under-prediction safer (expired products waste money).
  - *Medical*: cannot compromise — predictions must be accurate regardless.

**Remedy for non-linearity**: Polynomial regression, transform features, or switch to non-linear models (decision trees, SVM with kernels, etc.). Noted that Session 6–7 covers feature engineering; Session 8 moves to logistic regression.

---

## 3. Normality of Errors (Residuals)

- Residuals should be normally distributed: mean = 0, variance is constant.
- **Normality means**: errors like −2, −1, 0, +1, +2 — centered at zero with symmetric spread.
- A model that always under-predicts (e.g., actual = 50, predicted = 47, 48, 49) still makes close predictions but the *model is compromised*.
- Better: errors fluctuate around the true value (51, 52 for actual = 50), even if farther — broader coverage.

**Tests**:
- **Q-Q Plot / Quantile-Quantile**: Compare theoretical quantiles vs. sample quantiles. Straight diagonal line = good.
- **KDE (Kernel Density Estimate)**: Overlaid normal curve should match histogram of residuals.
- Q-Q and KDE are interchangeable for normality testing.

**Impact of violation**: Model quality degrades, but predictions may remain *close* to actual values. The model itself is in a bad state.

---

## 4. Homoscedasticity (Constant Variance)

- **Homoscedasticity**: Residuals have constant spread across all predicted values (like light air flowing evenly).
- **Heteroscedasticity**: Spread increases or decreases with predictions (like a funnel shape). Large fluctuations at one end, small at the other.

**Visual patterns in residual plots**:

| Pattern | Interpretation |
|---------|---------------|
| Even strip of residuals around zero line | Good — homoscedasticity holds |
| Funnel/fan widening or narrowing | Heteroscedasticity present |
| Extreme values with one global value | Coin-shaped/error cluster pattern — bad |
| Residuals converging at origin for certain point | Data may be incomplete |

**Remedy**: Log transformation of features/target.

---

## 5. Independence of Errors

- Errors should not follow a pattern/sequence: e.g., +2, +3, +4, +5 indicates autocorrelation.
- Critical in time-series data — consecutive observations may share information.
- Each residual should be independent: today's error shouldn't predict tomorrow's.

**Durbin-Watson (DW) Test**:
- DW ≈ **2** → No autocorrelation (good).
- DW ≈ **0** → Strong positive correlation (bad).
- DW ≈ **4** → Strong negative correlation (problematic).

**Key distinction**: Normality checks whether errors are close to zero; Independence checks whether errors fluctuate randomly (not following a pattern like 5,4,3,2,1,0,4,5,6,7,8,9).

---

## 6. Multicollinearity

- Occurs when two or more features share overlapping information (e.g., *age* and *experience* for salary prediction — a 30-year-old likely has more experience than a 21-year-old).
- Not about the target variable Y; it's about features X₁, X₂, X₃ sharing variance.

**Detection — VIF (Variance Inflation Factor)**:

VIF is computed per feature by regressing that feature against all other features:
- Compute R² for each feature regressed on others.
- **VIF = 1 / (1 − R²ᵢ)**

| R²ᵢ | VIF | Interpretation |
|-----|-----|---------------|
| 0% | 1 | No multicollinearity |
| 90% | 10 | High multicollinearity |
| 95% | 20 | Very high — problem |
| 100% | ∞ | Perfect collinearity (bad) |

- **VIF > 5** or **VIF > 10** is generally considered problematic.
- Higher VIF = higher variance inflation = worse.

**Remedies**:
- Combine correlated features into a new one (e.g., derive "professional experience" from age + experience).
- Use **Ridge Regression** or **Lasso Regression** (covered in Session 7):
  - *Lasso* can eliminate weak features entirely (coefficient → 0).
  - *Ridge* shrinks coefficients but keeps features.

### Multicollinearity — Additional Diagnostics

| Diagnostic | Formula / Concept | Interpretation |
|-----------|------------------|---------------|
| **Tolerance** | 1 − R²ᵢ (complement of VIF) | Tolerance < 0.1 → problem |
| **Condition Number (k)** | Ratio of largest to smallest singular value of X | k > 30 → problem; from `numpy.linalg.cond(X)` |
| **Correlation Matrix** | Pairwise correlation heatmap | Correlation ≥ ±0.8 between any pair → suspect multicollinearity |

- Tolerance = 1/VIF. Low tolerance = high VIF. Both flag the same problem from different angles.
- Condition Number checks ALL features simultaneously (not pairwise like VIF).

---

## 7. Outliers & Influential Points

### Types of anomalous points:

| Type | Location | Effect if Removed |
|------|----------|-------------------|
| **High Leverage** | Far from x-axis only (unusual feature value, but follows trend) | X-axis shifts; model tilt changes slightly |
| **Outlier** | Far from y-axis only (large residual) | Y-axis affected; may reduce if legitimate error |
| **Influential Point** | Unusual in BOTH x AND y | Model shape fundamentally changes — most dangerous |

**Key rule**: Investigate outliers first before removing. An outlier may contain valuable information:
- Example: A Ferrari/Mercedes product with an extremely high price is a legitimate data point, not an error. Removing luxury items skews the category.
- Example: A new product in month 1 with unusual pricing — worth studying, not discarding.

**Remedies for outliers**:
1. Investigate and understand the source first.
2. Feature transformation (log transform).
3. Min-Max scaling (0 to 1) or Standard Scaling.
4. Only remove if truly erroneous.

---

## 8. Gauss-Markov Theorem

- Under assumptions of linearity, independence, homoscedasticity, and no perfect multicollinearity, OLS is **BLUE** — Best Linear Unbiased Estimator:
  - *Best* = smallest variance among all linear unbiased estimators.
  - *Linear* = estimator is a linear function of Y.
  - *Unbiased* = expected value equals the true parameter.
- Normality is NOT required for BLUE property; it's only needed for hypothesis testing (t-tests, F-tests) and confidence intervals in small samples.

---

## 8b. What "Perfect Multicollinearity" Actually Does

- When features are perfectly correlated (e.g., X₃ = 2×X₁ + X₂), the matrix (XᵀX)⁻¹ cannot be computed — coefficients become **undefined**.
- Software (R, sklearn) automatically drops one of the perfectly collinear features.
- Rule of thumb: correlation between features ≥ 0.8–0.9 warrants VIF check.

---

## 8c. Sample Size Considerations

- Small samples + non-normality → inference (p-values, CIs) is unreliable.
- Large samples (>30–50, depending on context) → Central Limit Theorem helps; inference becomes more robust even if errors aren't perfectly normal.
- Rule of thumb: need at least **10–20 observations per feature** for stable estimates.

---

## 8d. Breusch-Pagan Test (Additional Heteroscedasticity Test)

- While Durbin-Watson tests autocorrelation, the **Breusch-Pagan test** tests for heteroscedasticity statistically.
- Null hypothesis: homoscedasticity holds.
- p-value < 0.05 → reject null → heteroscedasticity is present.
- Complemented by visual residual vs. fitted plot (funnel pattern).

---

## 8e. Normality Assumption — When It Actually Matters

| Goal | Normality Required? |
|------|--------------------|
| Point predictions | No — OLS estimates remain unbiased |
| Confidence intervals for predictions | Yes (especially small samples) |
| Hypothesis tests (t-tests, F-tests on coefficients) | Yes |
| P-values for feature significance | Yes |

- With large datasets → CLT makes inference approximately valid even without normal errors.
- For small samples → non-normal residuals → p-values/CIs may be misleading.

---

## 8f. Feature Scaling — Why It Matters (But Not an Assumption)

- **OLS coefficients are invariant to feature scaling** in terms of prediction accuracy, but:
  - Interpretation changes: standardized coefficients allow comparing effect sizes across features measured on different scales.
  - Regularization (Ridge/Lasso) is sensitive to scaling — must standardize before applying.
- Standard formula: z = (x − μ) / σ

---

## 8g. Overfitting vs Model Assumptions

- Non-linearity, multicollinearity, and outliers affect assumption validity.
- **Overfitting** is a separate concern: model fits training data perfectly but fails on new data.
- Detected via: train vs. test performance gap, cross-validation (k-Fold).
- Remedies: regularization (Ridge/Lasso/ElasticNet), feature selection, simpler models, more data.

---

## 7b. Cook's Distance — Detecting Influential Points

| Cook's D Value | Interpretation |
|---------------|----------------|
| < 0.5 | Unlikely to be influential |
| 0.5–1 | Investigate further |
| > 1 | Likely influential — check carefully |
| > 4/n | Threshold rule of thumb (n = sample size) |

**Differences from leverage:**
- Leverage alone ≠ influential (high leverage but follows trend = not influential).
- Influential points have BOTH high leverage AND large residual.

---

## 7c. Other Outlier Detection Metrics

| Metric | What It Measures | Threshold |
|--------|-----------------|-----------|
| **Studentized Residuals** | Residual divided by its estimated std error | |res| > 3 → outlier |
| **DFBETAS** | How much each coefficient changes when a point is removed | |DFBETA| > 2/√n |
| **Cook's Distance (D)** | Combined leverage + residual influence | D > 1 or 4/n |

---

## 7d. Robust Regression Alternatives

When assumptions are severely violated:

| Method | Handles | Best For |
|--------|---------|----------|
| Huber Regressor | Outliers, mild heteroscedasticity | General use |
| RANSAC | Large number of outliers / anomalies | Noisy real-world data |
| Theil-Sen | Outliers in x-direction (feature space) | Robust slope estimation |
| Quantile Regression | Heteroscedastic errors / non-normal residuals | Asymmetric error structures |

---

## 8h. Bias-Variance Tradeoff and Assumptions

- **Underfitting** → model too simple → violates linearity assumption (high bias).
- **Overfitting** → model too complex → fits to noise, not signal (high variance).
- Assumption checking helps find the right model balance.
- Cross-validation is essential alongside diagnostic checks.

---

## 8i. Model Comparison Metrics

| Metric | Formula / Concept | Interpretation |
|--------|------------------|---------------|
| **MSE** | Mean Squared Error: Σ(y − ŷ)²/n | Lower is better; penalizes large errors |
| **RMSE** | √MSE | In same units as target Y |
| **MAE** | Mean Absolute Error: Σ|y − ŷ|/n | Less sensitive to outliers than MSE |
| **R² (Adjusted)** | 1 − [(1−R²)(n−1)/(n−p−1)] | Penalizes adding irrelevant features; only metric that decreases when bad feature added |
| **AIC** | 2k − 2ln(L) | Lower is better; balances fit and complexity for model selection |
| **BIC** | ln(n)·k − 2ln(L) | Stronger penalty than AIC for additional parameters |

- Adjusted R² = R²ₐ𝒹ⱼ; standard R² always increases with more features even if useless.

---

## 8. OLS — The Underlying Assumption

Ordinary Least Squares (OLS), solved via the Normal Equation **β = (XᵀX)⁻¹ Xᵀy**, is the best linear regression method **only under ideal conditions**.

**Underlying assumptions of OLS:**
- Errors have mean zero: E(ε) = 0
- Errors have constant variance: Var(ε) = σ²I (homoscedasticity + independence)
- Errors are uncorrelated with features: Cov(X, ε) = 0 (exogeneity)

When assumptions hold → OLS is BLUE.
When assumptions are violated → consider Ridge, Lasso, robust regression, or generalized least squares (GLS).

---

## 9. Quick Reference Summary

| Problem | Diagnostic Metric/Plot | Threshold/Criteria | Remedy |
|---------|----------------------|-------------------|--------|
| Non-linearity | Residual plot | Curved pattern | Polynomial regression, transform features / use non-linear model |
| Non-normality | Q-Q plot, Shapiro-Wilk | Not straight line on Q-Q | Log transform data, robust methods |
| Heteroscedasticity | Residual vs. fitted plot | Funnel/fan shape | Weighted least squares, log transformation |
| Autocorrelation | Durbin-Watson test | DW ≠ 2 (closer to 0 or 4 = bad) | Time-series models, add lag features |
| Multicollinearity | VIF | VIF > 5–10 | Remove/combine features, PCA; Ridge/Lasso |
| Influential outliers | Leverage vs. residual plot | Both x and y far from center | Cook's distance, investigate before removing |

---

## 10. Key Takeaways

- **Data quality is foundational** — assumptions start with how the data looks.
- Always inspect the **shape of the raw data** first, then check residual patterns.
- **Predictions may still be close to actual values** even when model assumptions are violated — but the *model itself* is compromised.
- Industry context matters: over-prediction vs. under-prediction depends on the domain (retail vs. food vs. medical).
- VIF detects multicollinearity by asking: "How much of one feature can be explained by other features?"
- **Outliers ≠ errors** — investigate before discarding; they may carry critical information.
- OLS is optimal only when all assumptions hold; use regularization (Ridge/Lasso) or alternative models otherwise.

---

## 11. Summary from Live Class

Throughout the class, various analogies and examples were provided to help visualize these concepts, showing the importance of understanding how each assumption impacts the model and how deviations can lead to inaccurate predictions. Understanding and testing these assumptions allows for effective application and refinement of linear regression models.

---

## 12. Additional Glossary

| Term | Definition |
|------|-----------|
| **R-squared** | A statistical measure representing the proportion of the variance for a dependent variable that's explained by an independent variable (or variables). |
| **Cook's Distance** | A measure to identify influential points in a regression analysis. Values > 1 (or > 4/n) suggest influential observations. |
