# Gradient Descent - Key Concepts

> *Notes derived from lecture transcript at Scaler Academy*

## Overview
- Gradient descent is one of the top/optimal optimization factors in machine learning
- Core concept tied to function optimization and finding optimal solutions

## Key Ideas Discussed

### 1. Optimization & Functions
- The goal is to find a specific solution that minimizes the cost/loss function
- This concept is critical for understanding how models learn and improve

### 2. Gradient Descent Mechanics
- Involves iterative updates in the opposite direction of the gradient (steepest descent)
- Parameters: θ = θ - η * ∇J(θ)  
  where η is the learning rate and ∇J(θ) is the gradient of the cost function
- The "chain" reference relates to chain rule used in backpropagation

### 4. Learning Rate
- Determines the step size during each parameter update
- Must be carefully chosen:
  - Too large → divergence (overshoots minimum)
  - Too small → slow convergence
- Learning rate strategies:
  - Fixed constant learning rate
  - Adaptive methods (Adam, RMSprop, Adagrad)
  - Learning rate decay/scheduling

### 5. Variants of Gradient Descent
- Batch GD: uses entire dataset per update — accurate but slow for large data
- Stochastic GD (SGD): uses single sample per update — fast but noisy
- Mini-batch GD: uses small batches per update — balances speed and stability; most commonly used

### 6. Convergence
- Model should approach convergence (optimal point)
- Loss curve monitoring is important to verify training progress
- Helps ensure the model is learning effectively

## Takeaways
- Gradient descent is fundamental to training ML models
- Understanding the optimization process is essential
- Monitoring the loss curve helps validate that updates are working correctly
- Hyperparameter tuning (especially learning rate) plays a critical role

---
*Source: Gradient_Descent_-_Class_Scaler_Academy.txt*
