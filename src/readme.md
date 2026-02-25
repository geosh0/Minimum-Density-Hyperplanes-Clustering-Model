# 🧠 Code Structure & Logic Guide

## Under the Hood of `mdh.py`
This document explains the internal logic of the Minimum Density Hyperplanes implementation. Unlike random-search models, this algorithm uses continuous optimization to actively "hunt" for the best separating hyperplane.

---

### 1. The Coordinate System (The "Sphere" Trick)
- **Functions:** `spherical_to_cartesian`, `cartesian_to_spherical`

Optimization is difficult when you have a strict constraint like "The vector length must be exactly 1" ($||v|| = 1$). Standard optimizers struggle with this.

* **The Solution:** We convert the problem into Spherical Coordinates (angles).
* **How it works:** Instead of optimizing $D$ Cartesian coordinates $(x, y, z, ...)$, we optimize $D-1$ angles $(\theta_1, \theta_2, ...)$.
* **Benefit:** The angles are unconstrained. We can rotate them freely, and converting them back to Cartesian coordinates always guarantees a unit vector.

### 2. The Core Math (Density Estimation)
- **Function:** `kde_hyperplane_density_integral`

This is the "eyes" of the model. It measures how dense the data is at any specific point along a line.

* **Input:** Projected 1D data points and a specific location $b$.
* **Logic:** It uses Gaussian Kernel Density Estimation (KDE). It sums up Gaussian bell curves centered at every data point to estimate the probability density at $b$.
* **Goal:** We want to find a $b$ where this value is lowest (a valley).

### 3. The Objective Function (The "Cost")
- **Function:** `f_cl_penalized_density`

This is the function the optimizer tries to minimize. It combines two things:
1. **The Density:** The value from the KDE (we want this low).
2. **The Penalty:** A mathematical wall that prevents the cut from drifting too far to the edges.

* **Why the penalty?** Without it, the "lowest density" is always at infinity (where there is no data). The penalty forces the cut to stay near the middle of the data, ensuring we don't just slice off one outlier.

### 4. The Optimization Loop (The "Brain")
- **Functions:** `find_optimal_b_for_v`, `_fk_mdh_py_internal_alpha_loop`

This is the most complex part of the code. It uses a **Nested Optimization** strategy:

#### A. Inner Loop (`find_optimal_b`)
* **Job:** For a fixed angle (vector $v$), find the best cut point $b$.
* **Method:** It first does a grid search to find the general area of the valley, then uses `scipy.optimize.minimize_scalar` to find the exact bottom.

#### B. Outer Loop (`alpha_loop`)
* **Job:** Find the best viewing angle (vector $v$).
* **Method:** It uses L-BFGS-B (a gradient-based optimizer) to rotate the vector until the density integral is minimized.

#### The "Alpha" Strategy:
The code solves the problem multiple times.
1. It starts with a tight constraint (small $\alpha$) to find the general center.
2. It progressively relaxes the constraint (larger $\alpha$), using the previous solution as the starting point.
3. This prevents the optimizer from getting stuck in bad local minima.

### 5. The Safety Check (Relative Depth)
- **Function:** `calculate_relative_depth_criterion`

Before accepting a split, the code performs a final quality control check:
1. It looks at the density profile.
2. It identifies the "peaks" (clusters) and the "valley" (cut point).
3. It calculates the **Relative Depth**:

$$ \text{Depth} = \frac{\text{Density(Peak)} - \text{Density(Valley)}}{\text{Density(Valley)}} $$

If the valley isn't deep enough relative to the peaks, the split is rejected.

### 6. The Hierarchy (The Wrapper)
- **Function:** `hierarchical_mdh_clustering` & **Class:** `MDH`

This wraps the math into a usable tool.
* **Start:** Treat the whole dataset as one group.
* **Iterate:** 
  1. Pick the largest group.
  2. Run the MDH optimization to find the best cut.
  3. Split the data into two new groups.
* **Stop:** When we reach `n_clusters`.

### 7. Dependencies
The code relies heavily on:
* **NumPy:** For vector math and projections.
* **SciPy** (`optimize`, `signal`): For the BFGS optimizer and finding peaks in the density curve.
* **Scikit-Learn:** For data scaling (`StandardScaler`) and PCA initialization.
