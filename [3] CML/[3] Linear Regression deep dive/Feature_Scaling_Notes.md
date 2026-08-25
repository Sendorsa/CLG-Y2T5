# Linear Regression Deep Dive - Feature Scaling Notes

> **Source:** Class transcript (Scaler Academy)
> **Note:** Source text is a continuous block with limited newlines; interpretations follow from context.

---

## 1. Standardization (Z-score Scaling)

- Formula: `z = (x - mean) / std_deviation`
- Each value `x` has the mean subtracted, then divided by standard deviation
- Useful when data has a **high variance** (outliers present)
- Produces a distribution with **mean = 0** and **standard deviation = 1**
- Sensitive to outliers since mean and std dev are affected by extreme values

## 2. Min-Max Scaling (Normalization)

- Formula: `x_scaled = (x - min) / (max - min)`
- Scales values to a boundary between **0 and 1**
- Can shift the boundary slightly left of 0 or right of 1 using a "bias/brain" parameter
- Results in skewed distribution when data points cluster on one side with outliers on the other
- Adding bias shifts boundary beyond [0, 1]; most values cluster left with outliers right (skewed distribution)

## 3. Robust Scaling

- Uses **median** instead of mean to handle outliers
- Formula: `x_scaled = (x - median) / IQR` (where IQR = Interquartile Range)
- Median represents the **50th percentile** — resistant to extreme values
- Preferred when data contains significant outliers
- More reliable than mean-based methods for non-normal distributions

---

## Key Takeaways

| Method         | Central Tendency | Spread Measure | Sensitive to Outliers? | Scaling Range |
|----------------|------------------|----------------|--------------------|---------------|
| Standardization (Z-score) | Mean             | Std Deviation  | Yes                | Unbounded     |
| Min-Max        | Min / Max        | Range          | Yes                | [0, 1]        |
| Robust         | Median           | IQR            | No                 | Variable      |

## When to Use Which

- **Standardization:** Data is approximately normally distributed with moderate outliers
- **Min-Max:** Need bounds (e.g., image processing, neural networks requiring small input values)
- **Robust Scaling:** Data has significant outliers or is heavily skewed

---

## Linear Regression Fundamentals

### Mean Square Error (MSE) and Its Minimization

- MSE is the average of the squares of the errors, where error = predicted value (`ŷ`) − observed value (`y`)
- Core objective: **minimize** MSE to make predictions as accurate as possible
- Analogy: Calculate squared errors for a given prediction; steps prove that taking the mean of values minimizes MSE
- The **conditional mean** provides the best prediction under MSE minimization for normally distributed errors

### Error Perpendicularity

- Minimizing prediction errors by making them perpendicular to the feature space
- Equivalent to finding the shortest perpendicular distance between a point and a line in vector space
- Influences optimization of predictions

### Closed Form Solutions and Normal Equations

- **Normal equation:** `β = (XᵀX)⁻¹ Xᵀy`
- A closed-form solution allows solving for coefficient `β` directly, leading to minimal MSE under linearity assumption
- Computationally efficient for small-to-medium datasets

### Multicollinearity and Its Implications

- Occurs when features are highly correlated → leads to **singular matrices** that cannot be inverted
- Cannot compute the normal equation when multicollinearity is present
- Affects model **interpretability** and **stability** of coefficients

### Squared Errors vs. Absolute Errors

- Squared errors preferred over absolute errors for:
  - Computational ease (differentiability)
  - Accommodating normally distributed errors
  - Enhancement of gradient-based optimization methods

### Beta Coefficients

- Optimal beta coefficients balance feature weights, improving model accuracy
- Found through MSE minimization via the normal equation or gradient descent

---

## Key Takeaways: Linear Regression

| Concept | Description |
|---------|-------------|
| MSE | Measures average squared error; must be minimized |
| Normal Equation | Closed-form solution: `β = (XᵀX)⁻¹ Xᵀy` |
| Multicollinearity | Correlated features → singular matrix → inversion fails |
| Squared vs Absolute Error | Squared errors are differentiable and match normal distributions |
| Optimal Beta | Best coefficient weights via MSE minimization |

## When to Use Normal Equation vs Gradient Descent

- **Normal Equation:** Small-to-medium datasets, no need for iterative optimization; exact solution
- **Gradient Descent:** Large datasets, when `(XᵀX)` inversion is computationally expensive (O(n³))

---

## Glossary of Key Terms

| Term | Definition |
|------|-----------|
| **Normal Equation** | An equation used to solve linear regression directly, expressed as $(X^T X)^{-1} X^T y$. |
| **Mean Squared Error (MSE)** | A measure of the average squared differences between actual and predicted values. |
| **Perpendicular Projection** | A vector projection that minimizes the distance to the actual observation plane. |
| **Conditional Mean** | The expected value of a random variable given a certain condition. |
| **Orthogonal** | Vectors are orthogonal if their dot product is zero, implying perpendicularity. |
| **Collinearity** | A condition where two or more features are highly linearly related. |
| **Singular Matrix** | A square matrix that does not have an inverse. |
| **Closed Form Solution** | A solution to an equation expressed explicitly in terms of known functions or values. |
| **R-Squared** | A statistical measure that represents the proportion of variance for a dependent variable that's explained by an independent variable or variables in a regression model. |
| **Adjusted R-Squared** | An adjusted version of R-squared that accounts for the number of predictors in the model. |
| **Mean** | The average value of a set of numbers, calculated as the sum of all numbers divided by the count of numbers. |
| **Gaussian Distribution** | A symmetric probability distribution, known as normal distribution, commonly used in statistics. |

---

*Generated from class notes — Linear Regression Deep Dive, Scaler Academy*
