# K-Means Clustering, K-Medoids, Covariance & Regression — Class Notes

> **Source Material:** [K-Means Clustering and other notes.pdf](docs/K-Means%20Clustering%20and%20other%20notes.pdf)  
> **Scope:** Strictly covers the concepts, derivations, formulas, and step-by-step solved numerical problems from all 17 pages of the PDF.

---

## Table of Contents
1. [Covariance (Theory & Solved Problems)](#1-covariance)
2. [Correlation & Correlation Coefficient](#2-correlation--correlation-coefficient)
3. [Linear Regression (Least Squares Method & Solved Problem)](#3-linear-regression)
4. [K-Means Clustering (Step-by-Step 3-Cluster Solved Problem)](#4-k-means-clustering)
5. [K-Medoid Clustering / PAM (Manhattan Distance Solved Problem)](#5-k-medoid-clustering-pam)
6. [Hierarchical Clustering (Single-Link Agglomerative & Dendrogram)](#6-hierarchical-clustering-single-link)
7. [K-Means vs K-Medoids (Comparison, Advantages & Disadvantages)](#7-k-means-vs-k-medoids)

---

## 1. Covariance

### 1.1 Formulas
- **Sample Covariance:**
  $$\text{Cov}(x, y) = \frac{\sum (x_i - \bar{x})(y_i - \bar{y})}{N - 1}$$
- **Population Covariance:**
  $$\text{Cov}(x, y) = \frac{\sum (x_i - \bar{x})(y_i - \bar{y})}{N}$$

---

### 1.2 Problem 1: Stock A vs. Stock B (Page 1)

**Given Data:**
- Stock A ($x$): `2, 3, 5, 4, 6`
- Stock B ($y$): `5, 8, 10, 11, 14`
- Number of observations ($n$): `5`

#### Step 1: Calculate the Mean of Each Stock
- $\bar{x} = \frac{\sum x}{n} = \frac{2 + 3 + 5 + 4 + 6}{5} = \frac{20}{5} = 4$
- $\bar{y} = \frac{\sum y}{n} = \frac{5 + 8 + 10 + 11 + 14}{5} = \frac{48}{5} = 9.6$

#### Step 2: Find Deviations from the Mean
- For A: $(x - \bar{x}) = c(-2, -1, 1, 0, 2)$
- For B: $(y - \bar{y}) = c(-4.6, -1.6, 0.4, 1.4, 4.4)$

#### Step 3: Multiply Corresponding Deviations
- $(-2) \times (-4.6) = 9.2$
- $(-1) \times (-1.6) = 1.6$
- $(1) \times (0.4) = 0.4$
- $(0) \times (1.4) = 0.0$
- $(2) \times (4.4) = 8.8$

#### Step 4: Sum of the Products
$$\sum (x - \bar{x})(y - \bar{y}) = 9.2 + 1.6 + 0.4 + 0 + 8.8 = 20$$

#### Step 5: Calculate Sample Covariance
$$\text{Cov}(x, y) = \frac{\sum (x - \bar{x})(y - \bar{y})}{N - 1} = \frac{20}{5 - 1} = \frac{20}{4} = \mathbf{5}$$

#### Interpretation:
- A **positive covariance** indicates that the two stocks tend to move in the same direction (rise or fall together).
- A **negative covariance** suggests they move in opposite directions.
- In this case, covariance is **positive (+5)**, indicating Stock A and Stock B tend to rise or fall together.

---

### 1.3 Problem 2: Covariance Matrix Table Problem (Page 2)

**Given:**
- $X = (2, 4, 6, 8, 10)$
- $Y = (3, 7, 10, 14, 17)$
- $N = 5$

#### Calculation Table:
| $x$ | $y$ | $(x - \bar{x})$ | $(y - \bar{y})$ | $(x - \bar{x})(y - \bar{y})$ |
| :---: | :---: | :---: | :---: | :---: |
| 2 | 3 | $-4$ | $-7.2$ | $28.8$ |
| 4 | 7 | $-2$ | $-3.2$ | $6.4$ |
| 6 | 10 | $0$ | $-0.2$ | $0.0$ |
| 8 | 14 | $+2$ | $+3.8$ | $7.6$ |
| 10 | 17 | $+4$ | $+6.8$ | $27.2$ |
| **$\sum x = 30$** | **$\sum y = 51$** | **$0$** | **$0$** | **$\sum = 70.0$** |

- $\bar{x} = \frac{30}{5} = 6$
- $\bar{y} = \frac{51}{5} = 10.2$
- $\text{Cov}(x, y) = \frac{\sum (x_i - \bar{x})(y_i - \bar{y})}{N - 1} = \frac{70}{5 - 1} = \frac{70}{4} = \mathbf{17.5}$

---

## 2. Correlation & Correlation Coefficient

### 2.1 Definition & Meaning (Pages 3 & 4)
- **What is Correlation?**
  - A statistical technique used to determine the degree to which two variables are related.
  - Etymology: *"Co"* (Together) + *"Relation"* (Connection).
  - *Example:* Price and demand of a product.
- **Correlation Coefficient ($r$):**
  - Measures the **strength and direction** of the linear relationship between two variables.
  - Scale: Bounded strictly between **$-1$ and $+1$**:
    - **$-1$**: Strong Negative correlation
    - **$0$**: No relationship ($r = 0$)
    - **$+1$**: Strong Positive correlation

```
  -1 ────────────────────── 0 ────────────────────── +1
Strong                   No Linear                 Strong
Negative               Relationship               Positive
```

### 2.2 Types of Correlation
1. **Positive Correlation:**
   - As one variable increases, the other variable also increases (or vice versa; both move in the same direction).
   - *Example:* As the number of trees cut down increases, the probability of soil erosion increases.
2. **Negative Correlation:**
   - Inverse relationship: when one variable increases, the other decreases.
   - *Example:* If a car decreases speed, the travel time to reach the destination increases.

### 2.3 Limitations of Correlation (Page 3)
- Correlation does **not** tell us everything about the data; mean and standard deviation continue to be important.
- The true underlying data pattern may be a more complicated curve than a straight line, which will not show up in the linear calculation of $r$.
- **Real-World Examples:**
  - E-commerce: Unique website visitors vs. total daily sales (Amazon / Flipkart).
  - Healthcare: Measuring correlation between a patient's blood pressure and medication dosage.

---

## 3. Linear Regression

### 3.1 Overview & Least Squares Method (Page 5)
- **Goal:** Perform predictive analysis of the relationship between two variables ($x$ independent, $y$ dependent).
- **Regression Line:** $y = a + bx$ or $y = \beta_0 + \beta_1 x$
  - Intercept ($a$):
    $$a = \frac{(\sum y)(\sum x^2) - (\sum x)(\sum xy)}{n(\sum x^2) - (\sum x)^2}$$
  - Slope ($b$):
    $$b = \frac{n(\sum xy) - (\sum x)(\sum y)}{n(\sum x^2) - (\sum x)^2} = \frac{\sum (x_i - \bar{x})(y_i - \bar{y})}{\sum (x_i - \bar{x})^2}$$
- **Least Squares Principle:**
  - Fits the "line of best fit" by **minimizing the sum of squared vertical deviations** from each data point to the line.
  - For any point fitted accurately on the line, its perpendicular error/deviation is zero.

---

### 3.2 Solved Problem: Fitting Regression Line (Page 6)

#### Given Dataset:
| $x$ | $y$ | $x^2$ | $xy$ |
| :---: | :---: | :---: | :---: |
| 3 | 8 | 9 | 24 |
| 9 | 6 | 81 | 54 |
| 5 | 4 | 25 | 20 |
| 3 | 2 | 9 | 6 |
| **$\sum x = 20$** | **$\sum y = 20$** | **$\sum x^2 = 124$** | **$\sum xy = 104$** |

- Number of observations: $n = 4$

#### Calculating Intercept ($a$):
$$a = \frac{20(124) - 20(104)}{4(124) - (20)^2} = \frac{2480 - 2080}{496 - 400} = \frac{400}{96} \approx \mathbf{4.17}$$

#### Calculating Slope ($b$):
$$b = \frac{4(104) - 20(20)}{4(124) - (20)^2} = \frac{416 - 400}{496 - 400} = \frac{16}{96} \approx \mathbf{0.166}$$

#### Final Regression Equation:
$$y = 4.17 + 0.166 x$$

---

## 4. K-Means Clustering

### 4.1 Problem Statement (Page 7)
Cluster 8 data points into **$K = 3$ clusters** using **Euclidean distance**:
$$d(P_1, P_2) = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}$$

#### Given Points:
- $A_1(2, 10)$
- $A_2(2, 5)$
- $A_3(8, 4)$
- $B_1(5, 8)$
- $B_2(7, 5)$
- $B_3(6, 4)$
- $C_1(1, 2)$
- $C_2(4, 9)$

---

### 4.2 Iteration 1 (Page 7)

#### Initial Centroids:
- $C_1 = A_1(2, 10)$
- $C_2 = B_1(5, 8)$
- $C_3 = C_1(1, 2)$

#### Distance Table & Cluster Assignment:
| Point | Coordinates | Dist to $C_1(2, 10)$ | Dist to $C_2(5, 8)$ | Dist to $C_3(1, 2)$ | Assigned Cluster |
| :---: | :---: | :---: | :---: | :---: | :---: |
| $A_1$ | $(2, 10)$ | **0.00** | 3.61 | 8.06 | **1** |
| $A_2$ | $(2, 5)$ | 5.00 | 4.24 | **3.16** | **3** |
| $A_3$ | $(8, 4)$ | 8.49 | **5.00** | 7.28 | **2** |
| $B_1$ | $(5, 8)$ | 3.61 | **0.00** | 7.21 | **2** |
| $B_2$ | $(7, 5)$ | 7.07 | **3.61** | 6.71 | **2** |
| $B_3$ | $(6, 4)$ | 7.21 | **4.12** | 5.39 | **2** |
| $C_1$ | $(1, 2)$ | 8.06 | 7.21 | **0.00** | **3** |
| $C_2$ | $(4, 9)$ | 2.24 | **1.41** | 7.62 | **2** |

- **Cluster 1:** $\{A_1\}$
- **Cluster 2:** $\{A_3, B_1, B_2, B_3, C_2\}$
- **Cluster 3:** $\{A_2, C_1\}$

---

### 4.3 Iteration 2 (Page 8)

#### Recompute Centroids:
- $C_1 = (2, 10)$
- $C_2 = \left(\frac{8+5+7+6+4}{5}, \frac{4+8+5+4+9}{5}\right) = (6, 6)$
- $C_3 = \left(\frac{2+1}{2}, \frac{5+2}{2}\right) = (1.5, 3.5)$

#### Distance Table to New Centroids $(2, 10), (6, 6), (1.5, 3.5)$:
| Point | Coordinates | Dist to $C_1$ | Dist to $C_2$ | Dist to $C_3$ | New Cluster |
| :---: | :---: | :---: | :---: | :---: | :---: |
| $A_1$ | $(2, 10)$ | **0.00** | 5.66 | 6.52 | **1** |
| $A_2$ | $(2, 5)$ | 5.00 | 4.12 | **1.58** | **3** |
| $A_3$ | $(8, 4)$ | 8.49 | **2.83** | 6.52 | **2** |
| $B_1$ | $(5, 8)$ | 3.61 | **2.24** | 5.70 | **2** |
| $B_2$ | $(7, 5)$ | 7.07 | **1.41** | 5.70 | **2** |
| $B_3$ | $(6, 4)$ | 7.21 | **2.00** | 4.53 | **2** |
| $C_1$ | $(1, 2)$ | 8.06 | 6.40 | **1.58** | **3** |
| $C_2$ | $(4, 9)$ | **2.24** | 3.61 | 6.04 | **1** *(Shifted!)* |

- **Cluster 1:** $\{A_1, C_2\}$
- **Cluster 2:** $\{A_3, B_1, B_2, B_3\}$
- **Cluster 3:** $\{A_2, C_1\}$

---

### 4.4 Iteration 3 & Convergence (Page 9)

#### Recompute Centroids:
- $C_1 = \left(\frac{2+4}{2}, \frac{10+9}{2}\right) = (3, 9.5)$
- $C_2 = \left(\frac{8+5+7+6}{4}, \frac{4+8+5+4}{4}\right) = (6.5, 5.25)$
- $C_3 = (1.5, 3.5)$

#### Distance Table:
- Distances re-evaluated; point $B_1(5, 8)$ moves to Cluster 1.
- **Updated Clusters:**
  - Cluster 1: $\{A_1, B_1, C_2\}$
  - Cluster 2: $\{A_3, B_2, B_3\}$
  - Cluster 3: $\{A_2, C_1\}$

#### Final Centroid Computation:
- $C_1 = \left(\frac{2+5+4}{3}, \frac{10+8+9}{3}\right) = \mathbf{(3.67, 9)}$
- $C_2 = \left(\frac{8+7+6}{3}, \frac{4+5+4}{3}\right) = \mathbf{(7, 4.33)}$
- $C_3 = \left(\frac{2+1}{2}, \frac{5+2}{2}\right) = \mathbf{(1.5, 3.5)}$

On the next distance check, **no points change clusters** (clusters remain identical).  
The algorithm has **converged**!

#### Final Output:
- **Cluster 1:** $\{A_1, B_1, C_2\}$ with centroid $(3.67, 9)$
- **Cluster 2:** $\{A_3, B_2, B_3\}$ with centroid $(7, 4.33)$
- **Cluster 3:** $\{A_2, C_1\}$ with centroid $(1.5, 3.5)$

---

## 5. K-Medoid Clustering (PAM)

### 5.1 Problem Statement (Page 10)
Partition 10 data points into **2 clusters** using **Manhattan Distance**:
$$\text{Mdist} = |x_1 - x_2| + |y_1 - y_2|$$

#### Data Points:
- $x_1(2, 6)$
- $x_2(3, 4)$
- $x_3(3, 8)$
- $x_4(4, 7)$
- $x_5(6, 2)$
- $x_6(6, 4)$
- $x_7(7, 3)$
- $x_8(7, 4)$
- $x_9(8, 5)$
- $x_{10}(7, 6)$

---

### 5.2 Step 1 & 2: Initial Medoid Assignment & Clusters (Page 10)
- **Initial Medoids selected:** $c_1 = x_2(3, 4)$ and $c_2 = x_8(7, 4)$

#### Distance Table:
| Point | $(x, y)$ | Dist to $c_1(3, 4)$ | Dist to $c_2(7, 4)$ | Assigned Cluster |
| :---: | :---: | :---: | :---: | :---: |
| $x_1$ | $(2, 6)$ | $|2-3| + |6-4| = 3$ | $|2-7| + |6-4| = 7$ | **$C_1$** |
| $x_2$ | $(3, 4)$ | **0** | $|3-7| + |4-4| = 4$ | **$C_1$** |
| $x_3$ | $(3, 8)$ | $|3-3| + |8-4| = 4$ | $|3-7| + |8-4| = 8$ | **$C_1$** |
| $x_4$ | $(4, 7)$ | $|4-3| + |7-4| = 4$ | $|4-7| + |7-4| = 6$ | **$C_1$** |
| $x_5$ | $(6, 2)$ | $|6-3| + |2-4| = 5$ | $|6-7| + |2-4| = 3$ | **$C_2$** |
| $x_6$ | $(6, 4)$ | $|6-3| + |4-4| = 3$ | $|6-7| + |4-4| = 1$ | **$C_2$** |
| $x_7$ | $(7, 3)$ | $|7-3| + |3-4| = 5$ | $|7-7| + |3-4| = 1$ | **$C_2$** |
| $x_8$ | $(7, 4)$ | $|7-3| + |4-4| = 4$ | **0** | **$C_2$** |
| $x_9$ | $(8, 5)$ | $|8-3| + |5-4| = 6$ | $|8-7| + |5-4| = 2$ | **$C_2$** |
| $x_{10}$ | $(7, 6)$ | $|7-3| + |6-4| = 6$ | $|7-7| + |6-4| = 2$ | **$C_2$** |

- **Cluster 1:** $\{x_1, x_2, x_3, x_4\}$
- **Cluster 2:** $\{x_5, x_6, x_7, x_8, x_9, x_{10}\}$

---

### 5.3 Calculating Initial Total Cost (Page 11)
$$\text{Cost} = \sum_{i} |c_i - x_i|$$
- **Cost for $C_1$ (Medoid $(3, 4)$):** $3 + 0 + 4 + 4 = 11$
- **Cost for $C_2$ (Medoid $(7, 4)$):** $3 + 1 + 1 + 0 + 2 + 2 = 9$
- **Total Cost:** $11 + 9 = \mathbf{20}$

---

### 5.4 Step 3 & 4: Swapping Medoid Test (Pages 11 & 12)
- Randomly select non-medoid point $O = x_7(7, 3)$ to replace $c_2(7, 4)$.
- **New Medoid pair:** $c_1(3, 4)$ and $O(7, 3)$.

#### Recalculating Cost with New Medoids:
- $C_1$ cost remains $11$.
- New $C_2$ cost w.r.t $O(7, 3)$:
  - to $x_5(6, 2): |6-7| + |2-3| = 2$
  - to $x_6(6, 4): |6-7| + |4-3| = 2$
  - to $x_7(7, 3): 0$
  - to $x_8(7, 4): |7-7| + |4-3| = 1$
  - to $x_9(8, 5): |8-7| + |5-3| = 3$
  - to $x_{10}(7, 6): |7-7| + |6-3| = 3$
  - Cost for $C_2 = 2 + 2 + 0 + 1 + 3 + 3 = 11$.
- **Current Total Cost:** $11 + 11 = \mathbf{22}$.

#### Swap Decision:
$$S = \text{Current Total Cost} - \text{Previous Total Cost} = 22 - 20 = 2 > 0$$

- Since $S > 0$, the swap **increases cost** (makes clustering worse).
- We **scrap the swap** and keep the original medoids!
- **Final Medoids:** $\mathbf{c_1 = (3, 4)}$ and $\mathbf{c_2 = (7, 4)}$.

---

## 6. Hierarchical Clustering (Single-Link)

### 6.1 Problem Definition (Page 13)
Perform **Agglomerative Hierarchical Clustering** using the **Single-Link (minimum distance) technique** and draw the **Dendrogram**.

#### Given 6 Samples:
| Sample | $x$ | $y$ |
| :---: | :---: | :---: |
| $P_1$ | 0.40 | 0.53 |
| $P_2$ | 0.22 | 0.38 |
| $P_3$ | 0.35 | 0.32 |
| $P_4$ | 0.26 | 0.19 |
| $P_5$ | 0.08 | 0.41 |
| $P_6$ | 0.45 | 0.30 |

Distance function: $d(A, B) = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}$

---

### 6.2 Step-by-Step Distance Matrices (Pages 13 – 15)

#### Initial $6 \times 6$ Distance Matrix:
```
       P1     P2     P3     P4     P5     P6
P1      0
P2   0.23      0
P3   0.22   0.14      0
P4   0.37   0.19   0.13      0
P5   0.34   0.14   0.28   0.23      0
P6   0.24   0.24   0.10   0.22   0.39      0
```

1. **Iteration 1:**
   - Minimum value = **$0.10$** between **$P_3$ and $P_6$**.
   - Merge into cluster **$(P_3, P_6)$**.
   - Single-link update rule: $d((P_3, P_6), P_k) = \min(d(P_3, P_k), d(P_6, P_k))$.

2. **Iteration 2 (Matrix $5 \times 5$):**
   - Minimum value = **$0.13$** between **$(P_3, P_6)$ and $P_4$**.
   - Merge into cluster **$\{(P_3, P_6), P_4\}$**.

3. **Iteration 3 (Matrix $4 \times 4$):**
   - Minimum value = **$0.14$** between **$P_2$ and $P_5$**.
   - Merge into cluster **$(P_2, P_5)$**.

4. **Iteration 4 (Matrix $3 \times 3$):**
   - Minimum value = **$0.14$** between **$\{(P_3, P_6), P_4\}$** and **$(P_2, P_5)$**.
   - Merge into cluster **$[\{(P_3, P_6), P_4\}, (P_2, P_5)]$**.

5. **Iteration 5 (Matrix $2 \times 2$):**
   - Last remaining point **$P_1$** merges with the entire group at distance **$0.22$**.

---

### 6.3 Dendrogram Representation (Page 15)

```
Distance
  0.25 ┼─────────────────────────────────┐
       │                                 │
  0.20 ┼                         ┌───────┘ (0.22)
       │                         │       │
  0.15 ┼         ┌───────────────┤       │
       │         │ (0.14)        │ (0.14)│
  0.10 ┼ ┌───────┤         ┌─────┴─────┐ │
       │ │ (0.10)│ (0.13)  │           │ │
     0 ┴─┴───────┴─────────┴───────────┴─┴───
        P3      P6        P4          P2  P5 P1
```

---

## 7. K-Means vs K-Medoids

*(From Pages 16 & 17 of Notes)*

### 7.1 Core Algorithmic Difference
- **K-Means:** Chooses the **mean (average coordinates)** of points in a cluster as the centroid (the centroid is a calculated virtual point, not necessarily a data point).
- **K-Medoids:** Chooses an **actual data point from the dataset** to be the representative center (medoid) of the cluster.

---

### 7.2 K-Medoid Algorithm Steps (Page 16)
1. Initially select $k$ random points as the **medoids** from the $n$ data points.
2. Associate each data point to the closest medoid using a distance metric (e.g., Manhattan or Euclidean).
3. Calculate the total cost $C$ (sum of distances/dissimilarities from assigned medoid):
   $$C = \sum_{C_i} \sum_{P_j \in C_i} |P_j - K_i|$$
4. Swap one medoid point with a non-medoid point and recalculate the cost.
5. If the calculated cost with the new medoid is higher than the previous cost, **undo the swap**; otherwise, keep the swap and repeat until convergence.

---

### 7.3 Advantages of K-Medoids (Page 17)
1. Simple to understand and easy to implement.
2. Fast and converges in a fixed number of steps.
3. **Less sensitive to outliers** than other partitioning algorithms (e.g., K-Means).

### 7.4 Disadvantages of K-Medoids (Page 17)
1. **Not suitable for non-spherical clusters:** Cannot properly cluster arbitrarily shaped groups of objects.
2. **Initial randomness:** May obtain different results across different runs because initial medoids are chosen randomly.
