# Intro to Classical ML - Complete Notes

## Glossary

| Term | Definition |
|------|-----------|
| **Data Collection** | Process of gathering and measuring information from various sources. |
| **Cloud Storage** | Storing data in an online space, accessible via the internet, such as Google Cloud. |
| **BigQuery** | A Google Cloud-based data warehouse for real-time analytics using SQL. |
| **Exploratory Data Analysis (EDA)** | Initial data analysis to summarize the main characteristics of the data. |
| **Machine Learning** | A subset of AI focused on building systems that learn from data. |
| **Train-Test Split** | Dividing data into two sets — a larger training set and a smaller testing set. |
| **Random Seed** | A starting point used to ensure reproducibility in random processes, commonly set to 42. |
| **Mean Square Error (MSE)** | Average squared differences between estimated values and actual value. |
| **R-Squared** | Statistical measure of how close data are to the fitted regression line. |
| **Regression vs Classification** | Regression predicts continuous values; classification predicts discrete categories. |
| **Supervised Learning** | Machine learning task of inferring a function from labeled training data. |
| **Feature Engineering** | Creating new input features from existing ones to improve model performance. |

## 1. Machine Learning — Introduction

- **Machine Learning (ML)** involves the use of algorithms to parse data, learn from it, and make decisions or predictions based on that data.
- The field continuously evolves and improves — learning is an ongoing process, not a one-time task.

## 2. Understanding Model Failure

### When to Worry vs When to Accept
- **Basic task failure** (simple algorithm fails simple task): Indicates problems with model setup, features, or training — a concern that requires investigation
- **Complex task failure** (hard problem requiring heavy GPU/memory/computation still fails): Expected and acceptable — the problem is genuinely difficult

## 3. Supervised vs Unsupervised Learning

### Supervised Learning
- Learns from a **labeled dataset** (input + correct output provided)
- Primarily used for **prediction tasks**
- Examples: Linear Regression, Logistic Regression, Support Vector Machines

### Unsupervised Learning
- Deals with **unlabeled data** — infers natural structure present within the data
- Examples: K-means Clustering, Hierarchical Clustering

## 4. Loss Functions & Gradient Descent

### Gradient Descent
- Loss function typically follows a **U-shaped curve** in parameter space
- Objective: reach the **bottom (global minima)** where gradient/tangent approaches zero
- In practice, perfect minima is never achieved, but minimizing it remains the goal
- Multiple error metrics are monitored throughout training

### Common Loss Functions
| Type | When to Use |
|------|-------------|
| **MSE (Mean Squared Error)** | Regression; based on Gaussian distribution assumptions |
| **Gaussian distribution-based** | Continuous target variables |
| **Bernoulli loss** | Binary classification problems |
| **Log transform-based** | Skewed data; stabilizes variance |

## 5. Regression vs Classification (Core Distinction via Bucketing)

### When to Classify: The "Pins and Buckets" Concept
- **Classification**: Used when output falls into discrete categories/buckets
  - Examples: lies/not lies, cat/dog, spam/not spam, purchase/no purchase
  - Each prediction maps to exactly one predefined bucket
- **Key insight from interview question (number range 1,2,3,4,5)**:
  - If treated as *classification with 5 fixed buckets*: prediction probability is limited (e.g., only specific categories can be predicted each with ~20% probability weight)
  - If treated as *regression*: model predicts continuous values across the range without artificial category grouping
  - **Choosing classification unnecessarily inflates the number of contributions/classes** the ML must manage for a task that could be simpler via regression

### The Deciding Factor
- **The target handling is under your control, not the data's**: whether predictions should output 2.0, 2.2, 2.3 etc. (regression) or fixed categories (classification) depends on *your business need*
- When asked this in interviews: clarify intent — what does "predict" mean? You set the target type based on the use case

## 6. End-to-End ML Pipeline Overview

```
Business Problem → Data Collection → EDA → Data Cleaning → Feature Engineering → Train/Test Split → Training → Validation → Hyperparameter Tuning → Evaluation → Deployment
```

### Real-World Case Study: Levi's Sales Forecasting

#### Challenge 1 - Massive Scale Data
- Levi's sells across multiple regions: North America, Latin America, Europe, Asia Pacific, China, Pakistan, etc.
- **5 data tables stored in Google Cloud SQL** — one product information table alone is ~**16 TB**
- Loading 16 TB into personal Excel on a standard laptop (Intel i3) would take 10-15 minutes and slow down significantly
- Combining product + transaction + inventory data makes local processing impossible

#### Challenge 2 - Data Privacy & Cloud Processing
- Industry practice: **data NEVER goes to source laptop**
- Direct transmission of raw cloud data to personal laptops risks data leaks (e.g., sharing 100K rows of Excel to Google = exposing Levi's business insights)
- Correct approach: data stays in cloud; only processed results/aggregates are fetched externally

#### Challenge 3 - EDA & Hierarchical Data Structure
Levi's product categorization drill-down:
```
Men/Women → Tops/Bottoms/Shoes/Accessories → Fit type (regular, relaxed, slim, etc.) → Color (unique color-shade codes) → Size
```
- Final prediction target: **sell at the lowest hierarchy level** (specific size & color of exact product)

#### Challenge 4 - Data Cleanup Issues
Discrepancies found during cleaning:
- Products appearing in transaction tables but missing from inventory (where is it being sold?)
- Artificial MRP inflation followed by fake discounts (e.g., MRP inflated to ₹120, then 10% discount → actual sale at ₹108, real profit)
- Treatment: ensure all products exist in product master; clean raw data for the model ("babysitting" it with clean inputs)

#### Challenge 5 - Feature Engineering (Geographic Awareness)
Different geographies require different features:
- **Iceland store**: winter wear (jackets, sweatshirts) — never shorts
- **Pakistan store**: t-shirts and lighter clothing
- **India events**: Diwali, Dussehra, Great Indian Festival → festival-related buying patterns → add event tags as new feature columns
- **US events**: Prime Day, Amazon festivals, public holidays as features
- **FIFA World Cup example**: 
  - Identified stadium venues in US/Mexico/Canada
  - Targeted stores within 5 km radius of each venue
  - Created special inventory features: football-print t-shirts (70% sports, 30% discount ratio)
  - These are *new* event features not present in historical data

## 7. Train vs True Error Theory (Thomas Mitchell's Framework)

### Error Metrics in ML: RMSE vs R-squared (R²)

#### Root Mean Square Error (RMSE)
- Measures the **average magnitude of errors** by taking the square root of the average squared differences between prediction and actual observation
- Considers the mean into account but may not address individual errors as clearly
- Lower RMSE = better model fit

#### R-squared (R²)
- Contrasts the **residual sum of squares** (sum of squared errors) against the **total sum of squares**
- Measures the proportion of variance in the target variable that is predictable from the independent variables
- An R² closer to 1 indicates a better fit

### Two Error Metrics (Mitchell's View)

| Metric | Formula Description | Practical Value |
|--------|--------------------|-----------------|
| **Training Error** | Average loss of model on the training sample itself | Useful for debugging, but misleading for real performance |
| **True Error** | Expected loss over the entire universe (population distribution D) | The only metric that matters — measures performance on unseen data |

### Key Intuition
- **Training error on the same data used for learning**: Like giving someone 100 questions to practice, then asking them the same 100 as a test → guarantees high score but proves nothing
- **True error on unseen data (universe D)**: Tests generalization — this is what machine learning actually cares about

### Mitchell's TEP Framework
- **T**: Task performance
- **E**: Experience
- **P**: Performance on task T improves with experience E

### Why 80/20 Split?
- **80% training = "experience" (E)** — data the model learns from
- **20% testing = true error measurement** — data the model has *never seen*
- Critical: **test data must never leak into training**. Even if only 5% of test labels accidentally appear in training, it's like showing exam questions beforehand

## 8. Code Walkthrough: California Housing Dataset with scikit-learn

### Libraries Used
- `numpy` — mathematical operations
- `pandas` — tabular data handling and visualization (via `.head()`)
- `matplotlib` — plotting/graphs
- `train_test_split` — built-in scikit-learn preprocessing utility

### Random Seed (42)
- Without seeding: shuffles data differently each run, results are non-reproducible
- With seed 42: deterministic shuffle of the *entire* dataset for reproducibility
- Prevents sampling bias (e.g., fixed top-80% rows might contain outliers like a pink shirt selling at ₹1000)

### Dataset Structure
- **fetch_california_housing(dataset="full")** → DataFrame with `as_frame=True`
- Shape: 20,640 rows × 9 columns
- Contains median house values binned at ₹500K intervals in the graph

### Data Leakage Example (Critical Lesson)
Without train_test_split:
```python
model.fit(X, y)  # Model sees ALL data during both training AND testing
```
- Results appear good but model is **memorizing**, not learning
- Industry context: Even **95% accuracy** can be excellent for real-world ML (vs the naive expectation of 100%)

### The Correct Way
```python
train_test_split(X, y, test_size=0.2, random_state=42)
# → Train set: 16,512 rows
# → Test set: 4,128 rows (unseen by model)
```

### Results Interpretation
| Metric | Training | Test |
|--------|----------|------|
| **RMSE** | 720 | 7,465 |
| **R² squared** | 0.162 | 0.567 |

- R² closer to 1.0 = perfect model; below negative = worst possible
- Gap between train and test R² indicates the true performance drop on unseen data (expected behavior)

### Predicted vs Actual Scatter Plot Interpretation
- Points **on/near** the diagonal line: good predictions
- Points **below** the line: **underpredicted** (model underestimated, e.g., ₹100K actual → ₹70K predicted)
- Points **above** the line: **overpredicted** (model overestimated)
- **Business preference in real world**: Slight overprediction is preferable to underprediction due to financial implications

### Feature Engineering Visualization Notes
- When graph shows a spike at ₹500K in California housing data with many houses: this is not real — it's **truncated binning** where values of ₹500K, ₹600K, ₹700K, ₹800K are all aggregated to the top value of ₹500K for computational savings
- **Median income vs house value scatter**: Shows correlation between median household income and median housing value, providing intuition for which features matter

## 9. Interview Tips

### Answering Strategy
1. **Don't give the full 80/20 answer immediately** — interviewers will probe deeper until you make a mistake ("take them 50-50" strategy)
2. Say something like: *"We cannot use test data for training because that's like giving students exam questions as practice"* → lets them ask follow-up, then reveal the 80/20 split in response

### On Train Error Significance
- Training accuracy is **a trap** — it proves memorization, not learning
- Model experience (training) matters only insofar as it enables accurate performance on *unseen* data tomorrow
- True error test measures: "After unlimited experience, what can the model still predict?"

### Aligning Predictive Target with Goal
- Whether you focus on **regression** (predicting a continuous number) or **classification** (predicting categories) should align with the **final business goal** of the training task
- This choice is under your control — clarify intent when asked in interviews: what does "predict" mean for this use case?

## 10. Hyperparameter Tuning, Error Minimization & Overfitting

### Hyperparameter Tuning
- Involves altering model parameters to **optimize performance**
- Important for refining model predictions to achieve the best possible results
- Occurs during the Validation phase of the pipeline (after training, before final evaluation)

### Error Minimization
- The ultimate goal in model training is to achieve an error rate that's tolerable and matches the **pre-defined goals or benchmarks** set at the onset of a model's design

### Avoiding Overfitting
- Danger: model **memorizes** training data and fails to generalize to unseen data
- Solution: balance between fitting the data well and maintaining a **simple model structure**
- Overfitting is confirmed when training error is low but true (test) error is high
