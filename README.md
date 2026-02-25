# Minimum Density Hyperplanes (MDH)
### Optimization-Based Clustering for High-Dimensional Data

## 📌 Overview
This repository contains a Python implementation of **Minimum Density Hyperplanes (MDH)**, a clustering algorithm proposed by **Pavlidis, Hofmeyr, and Tasoulis (2016)**.

Developed as part of a **Bachelor's Thesis**, this project translates the continuous optimization framework from the original paper into a Scikit-Learn compatible estimator.

### The Problem
In high-dimensional space, standard clustering methods (like K-Means) often fail because distance metrics become unreliable. While other Projection Pursuit methods (like **RLAC**) use random search to find separators, **MDH** treats clustering as a continuous optimization problem.

### The Solution
MDH actively **searches** for the optimal separating hyperplane by minimizing the integral of the probability density function along the separator.
1.  It projects data onto a vector $\mathbf{v}$.
2.  It calculates the density using Kernel Density Estimation (KDE).
3.  It uses **Gradient-Based Optimization** (L-BFGS-B) to rotate the vector $\mathbf{v}$ and shift the bias $b$ until it finds the "thinnest" cut through the data (the Minimum Density).

---

## 🚀 Key Features
*   **Optimization-Based:** Uses BFGS to strictly minimize the density integral, finding precise separators that random search might miss.
*   **Scikit-Learn API:** Fully compatible with `sklearn` pipelines, `GridSearchCV`, and standard fit/predict workflows.
*   **Robustness:** Implements the **Alpha-Continuation Strategy** from the paper to avoid local minima by progressively narrowing the search space.
*   **Penalized Objective:** Includes the "Balancing Constraint" penalty to prevent the algorithm from slicing off tiny groups of outliers.

---

## 📦 Installation & Dependencies
The model requires standard scientific Python libraries:
```bash
pip install numpy scipy scikit-learn matplotlib
```


## 🧠 How It Works (The Math)
The algorithm solves a constrained optimization problem to find a hyperplane **H(v,b)** defined by a normal vector **v** and offset **b**.

### 1. The Objective Function
The goal is to minimize the **Penalized Density Integral**:

$$ \min_{\mathbf{v}, b} \hat{I}(\mathbf{v}, b) + \text{Penalty}(b) $$

Where $\hat{I}(\mathbf{v}, b)$ is the integral of the empirical density along the hyperplane. Minimizing this ensures the cut passes through the region of lowest data density (the "valley" between clusters).

### 2. Projection Pursuit (MDP²)
Instead of optimizing in high-dimensional space directly, **MDH uses Minimum Density Projection Pursuit**:
1. **Project**: Map data X onto a candidate vector v (1D space).
2. **Evaluate**: Compute the KDE profile of the projection.
3. **Optimize**: Adjust the angles of v using gradients to lower the density at the cut point b.

### 3. Hierarchical Splitting
The algorithm works top-down:
1. Start with the whole dataset.
2. Find the best MDH to split the data into two.
3. Recursively apply MDH to the sub-clusters until n_clusters is reached.

---
## 📄 Reference
This implementation is based on the following paper:
* **Pavlidis, N. G., Hofmeyr, D. P., & Tasoulis, S. K. (2016)** [Minimum Density Hyperplanes. Journal of Machine Learning Research, 17(156), 1-33](https://research.lancaster-university.uk/en/publications/minimum-density-hyperplanes/)
