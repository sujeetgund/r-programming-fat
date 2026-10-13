# Module 5: Interfacing R, Parallelism & Statistical Modeling — Exam Revision Guide

> **Target:** R Programming Term-End Exam (FAT)  
> **Key References:** Base R Mechanics & *The Art of R Programming* by Norman Matloff  
> **Exam Strategy:** Direct answers for short notes, syntax templates, comparison tables, and code snippets ready to write on paper or run in a lab exam.

---

## Table of Contents
1. [Interfacing R to Other Languages (C, C++, Python)](#1-interfacing-r-to-other-languages)
2. [Parallel R (`parallel` Package & Multi-Core Processing)](#2-parallel-r)
3. [Basic Statistics & Hypothesis Testing](#3-basic-statistics--hypothesis-testing)
4. [Linear Models (LM: Simple & Multiple Regression)](#4-linear-models-lm)
5. [Generalized Linear Models (GLM: Logistic & Poisson)](#5-generalized-linear-models-glm)
6. [Non-linear Models (`nls` & Polynomials)](#6-non-linear-models)
7. [Time Series & Auto-correlation (ACF / PACF)](#7-time-series-and-auto-correlation)
8. [Clustering (K-Means & Hierarchical)](#8-clustering)
9. [High-Yield Exam Trap Sheet & Frequently Asked Questions](#9-high-yield-exam-trap-sheet)

---

## 1. Interfacing R to Other Languages

### 1.1 Why Interface R with Other Languages?
- **Speed & Efficiency:** R is interpreted and can be slow for intensive loops and recursions. Calling C/C++ provides a massive 10x–100x performance boost.
- **Code Reuse:** Allows leveraging existing high-performance legacy codebases in C, C++, Fortran, or Python machine learning libraries.

---

### 1.2 Interfacing with C and C++

#### Approach A: Base R Interfaces (`.C()` vs `.Call()`)
| Feature | `.C()` Interface | `.Call()` Interface |
| :--- | :--- | :--- |
| **Object Handling** | Copies R data into standard C pointers (`int*`, `double*`) | Directly manipulates internal R pointers (`SEXP`) |
| **Memory Overhead** | Higher (full copies are made) | Zero-copy / very low memory overhead |
| **Complexity** | Simple, limited to atomic vectors | More complex, requires knowledge of R internals |
| **Safety** | Safer for beginners | Requires explicit memory protection (`PROTECT`, `UNPROTECT`) |

#### Approach B: Modern C++ with `Rcpp` (Highest-Yield Exam Topic)
`Rcpp` automates the bridge between C++ and R without manual pointer handling.

```R
library(Rcpp)

# 1. Inline C++ Function via cppFunction()
cppFunction('
  int addNumbers(int a, int b) {
    return a + b;
  }
')
addNumbers(15, 25)  # Output: 40

# 2. Fast Fibonacci in C++ called directly from R
cppFunction('
  int fibCpp(int n) {
    if (n <= 1) return n;
    return fibCpp(n - 1) + fibCpp(n - 2);
  }
')
fibCpp(10)  # Output: 55
```

---

### 1.3 Interfacing with Python (`reticulate`)
The `reticulate` package embeds a Python session inside R:

```R
library(reticulate)

# 1. Import standard Python modules
np <- import("numpy")
math_py <- import("math")
math_py$sqrt(144)    # Output: 12

# 2. Execute Python code block directly in R
py_run_string("
def multiply(x, y):
    return x * y
")
py$multiply(6, 7)    # Output: 42
```

---

## 2. Parallel R

By default, R executes on a **single CPU core**. The `parallel` package (built into Base R) distributes tasks across multiple processor cores.

### 2.1 The Two Approaches to Parallelism

| Mechanism | Primary Function | Platform Support | Shared Memory? |
| :--- | :--- | :--- | :--- |
| **Forking (POSIX)** | `mclapply()` | Linux & macOS only | **Yes** (Copy-on-write, fastest) |
| **Socket Clusters (Background R sessions)** | `parLapply()`, `makeCluster()` | **Cross-platform (Windows, Mac, Linux)** | **No** (Variables must be explicitly exported) |

---

### 2.2 Cross-Platform / Windows Workflow (`makeCluster`)

The standard 4-step workflow tested in lab and written exams:

```R
library(parallel)

# Step 1: Detect available CPU cores
num_cores <- detectCores()
cat("Available cores:", num_cores, "\n")

# Step 2: Create a local socket cluster (leave 1 core free for OS)
cl <- makeCluster(num_cores - 1)

# Step 3: Export variables or functions to worker nodes if needed
base_multiplier <- 10
clusterExport(cl, varlist = c("base_multiplier"))

# Step 4: Run parallel apply (parLapply or parSapply)
input_data <- 1:5
parallel_results <- parLapply(cl, input_data, function(x) {
  x * base_multiplier
})
print(unlist(parallel_results))

# Step 5: ALWAYS stop the cluster to free system memory!
stopCluster(cl)
```

**Console Output:**
```
[1] 10 20 30 40 50
```

---

## 3. Basic Statistics & Hypothesis Testing

### 3.1 Descriptive Statistics
```R
scores <- c(45, 62, 78, 85, 90, 92, 55, 78)

mean(scores)       # Arithmetic mean: 73.125
median(scores)     # Median: 78
var(scores)        # Sample variance
sd(scores)         # Sample standard deviation
quantile(scores)   # 0%, 25%, 50%, 75%, 100% quantiles
IQR(scores)        # Interquartile Range: Q3 - Q1
summary(scores)    # Min, 1st Qu, Median, Mean, 3rd Qu, Max
```

---

### 3.2 Hypothesis Testing Cheat Sheet

#### 1. Student's t-Test (`t.test`)
Tests differences between means of continuous numeric variables.

```R
# A. One-sample t-test: Is true mean equal to 70?
t.test(scores, mu = 70)

# B. Two-sample independent t-test: Comparing Group A vs Group B
grpA <- c(82, 85, 90, 78, 88)
grpB <- c(65, 70, 72, 68, 75)
t.test(grpA, grpB, var.equal = TRUE)

# C. Paired t-test: Before vs After treatment on same subjects
pre  <- c(120, 130, 125, 140)
post <- c(115, 122, 120, 132)
t.test(pre, post, paired = TRUE)
```

#### 2. Chi-Square Test (`chisq.test`)
Tests association/independence between **categorical variables** or goodness-of-fit.

```R
# Contingency table: Treatment vs Improvement
obs_table <- matrix(c(30, 10, 15, 25), nrow = 2,
                    dimnames = list(c("Drug", "Placebo"), c("Improved", "No_Change")))
chisq.test(obs_table)
```

#### 3. One-Way ANOVA (`aov`)
Tests difference in means across **3 or more groups**.

```R
# H0: Mean MPG is identical across 4, 6, and 8 cylinder cars
anova_model <- aov(mpg ~ factor(cyl), data = mtcars)
summary(anova_model)
```

> **The Golden Decision Rule for Exams:**  
> If **$\text{p-value} < 0.05$** $\longrightarrow$ **Reject $H_0$** (Statistically significant difference).  
> If **$\text{p-value} \ge 0.05$** $\longrightarrow$ **Fail to reject $H_0$**.

---

## 4. Linear Models (LM)

A Linear Model expresses a dependent response variable $Y$ as a linear combination of predictors $X$:
$$Y = \beta_0 + \beta_1 X_1 + \dots + \beta_k X_k + \epsilon$$

### 4.1 Fitting Linear Models (`lm()`)

```R
# 1. Simple Linear Regression: MPG predicted by Weight (wt)
fit_simple <- lm(mpg ~ wt, data = mtcars)

# 2. Multiple Linear Regression: MPG predicted by wt and horsepower (hp)
fit_multi <- lm(mpg ~ wt + hp, data = mtcars)

# 3. Model with interaction term (wt * hp expands to wt + hp + wt:hp)
fit_inter <- lm(mpg ~ wt * hp, data = mtcars)

# 4. Regress against all other variables in dataframe
fit_all <- lm(mpg ~ ., data = mtcars)
```

---

### 4.2 Inspecting and Extracting Model Outputs

```R
summary(fit_simple)
```

**Key Summary Statistics to Explain in Exams:**
1. **Coefficients (Estimate):**
   - Intercept ($\beta_0$): Expected $Y$ when $X = 0$.
   - Slope ($\beta_1$): Expected change in $Y$ per 1 unit increase in $X$.
2. **$p$-value (`Pr(>|t|)`):** Tests $H_0: \beta_i = 0$. If $p < 0.05$, the predictor is statistically significant.
3. **Multiple $R^2$ / Adjusted $R^2$:** Proportion of variance in $Y$ explained by the model (ranges from 0 to 1).
4. **F-Statistic:** Tests if at least one predictor has a non-zero coefficient.

#### Helper Extraction Functions
```R
coef(fit_simple)       # Extract coefficients: Intercept and wt slope
fitted(fit_simple)     # Fitted / predicted values for training data (Y_hat)
residuals(fit_simple)  # Residuals: actual - fitted (e = Y - Y_hat)
confint(fit_simple)    # 95% Confidence intervals for coefficients

# Predicting on new data:
new_cars <- data.frame(wt = c(2.5, 3.8))
predict(fit_simple, newdata = new_cars)
```

#### Diagnostic Plots
```R
# Generates 4 diagnostic plots:
# 1. Residuals vs Fitted (linearity & homoscedasticity)
# 2. Normal Q-Q (normality of residuals)
# 3. Scale-Location (homoscedasticity)
# 4. Residuals vs Leverage (influential outliers & Cook's distance)
par(mfrow = c(2, 2))
plot(fit_simple)
par(mfrow = c(1, 1))
```

---

## 5. Generalized Linear Models (GLM)

### 5.1 Why GLM? (LM vs GLM Comparison)

| Feature | Linear Model (`lm`) | Generalized Linear Model (`glm`) |
| :--- | :--- | :--- |
| **Response Variable $Y$** | Continuous, unbounded, Normal | Non-normal (Binary, Counts, Proportions) |
| **Error Distribution** | Normal ($Y \sim \mathcal{N}(\mu, \sigma^2)$) | Exponential Family (Binomial, Poisson, Gamma) |
| **Relationship** | Linear: $\mu = X\beta$ | Transformed via **Link Function**: $g(\mu) = X\beta$ |

Three components of GLM:
1. **Random Component:** Probability distribution of $Y$.
2. **Systematic Component:** Linear predictor $\eta = X\beta$.
3. **Link Function $g(\mu)$:** Connects expected value $\mu = E(Y)$ to linear predictor $\eta$.

---

### 5.2 Logistic Regression (Binary Classification: 0 or 1)
- Distribution: **Binomial**
- Link function: **Logit** ($\log\left(\frac{p}{1-p}\right)$)

```R
# Predict transmission type: am (0 = Automatic, 1 = Manual) using wt and hp
logit_model <- glm(am ~ wt + hp, data = mtcars, family = binomial(link = "logit"))
summary(logit_model)

# Making probability predictions on new data:
# NOTE: type = "response" is mandatory to get probabilities (0 to 1) instead of log-odds!
prob_preds <- predict(logit_model, newdata = data.frame(wt = 3.0, hp = 150), type = "response")
print(prob_preds)
```

---

### 5.3 Poisson Regression (Count Data)
- Distribution: **Poisson**
- Link function: **Log** ($\log(\mu) = X\beta$)
- Used for count of events (e.g., website visits, defects, accidents $\ge 0$).

```R
# Model number of forward gears (gear) or carburetor counts (carb)
poisson_model <- glm(carb ~ wt + hp, data = mtcars, family = poisson(link = "log"))
summary(poisson_model)
```

---

## 6. Non-linear Models

### 6.1 Non-linear Least Squares (`nls()`)
When the relationship between variables cannot be transformed into a linear form:
$$Y = f(X, \theta) + \epsilon$$

> **Critical Exam Rule:** Unlike `lm()`, `nls()` **requires a starting estimate** (`start = list(...)`) for all unknown parameters to perform numerical optimization (Gauss-Newton algorithm).

```R
# Example: Exponential Growth model: y = a * exp(b * x)
x_vals <- 1:10
y_vals <- 2.5 * exp(0.3 * x_vals) + rnorm(10, sd = 0.5)
sim_df <- data.frame(x = x_vals, y = y_vals)

# Fit non-linear model
fit_nls <- nls(y ~ a * exp(b * x), data = sim_df, start = list(a = 1, b = 0.1))
summary(fit_nls)
```

### 6.2 Polynomial Regression via `I()` Operator
To include powers of predictors inside `lm()`, wrap terms in `I()` to prevent R formula interpretation:
```R
# Quadratic regression: y = beta_0 + beta_1 * x + beta_2 * x^2
quad_model <- lm(mpg ~ wt + I(wt^2), data = mtcars)
summary(quad_model)
```

---

## 7. Time Series and Auto-correlation

### 7.1 Creating Time Series Objects (`ts()`)
A time series is an ordered sequence of data points recorded at uniform time intervals.

```R
# Monthly sales over 2 years (24 months)
monthly_data <- c(112, 118, 132, 129, 121, 135, 148, 148, 136, 119, 104, 118,
                  115, 126, 141, 135, 125, 149, 170, 170, 158, 133, 114, 140)

# frequency = 12 for monthly, 4 for quarterly, 1 for yearly
ts_sales <- ts(monthly_data, start = c(2024, 1), frequency = 12)
print(ts_sales)
plot(ts_sales, main = "Monthly Sales Time Series", ylab = "Units Sold")
```

---

### 7.2 Time Series Decomposition
Breaks time series into 3 core components:  
$Y_t = \text{Trend } (T_t) + \text{Seasonal } (S_t) + \text{Random / Irregular } (I_t)$

```R
decomp <- decompose(ts_sales, type = "additive") # or type = "multiplicative"
plot(decomp)
```

---

### 7.3 Auto-correlation: ACF and PACF

| Function | Full Form | What it Measures |
| :--- | :--- | :--- |
| **`acf(x)`** | Auto-Correlation Function | Correlation between time series $Y_t$ and its lagged values $Y_{t-k}$ (includes indirect correlations) |
| **`pacf(x)`**| Partial Auto-Correlation Function | Direct correlation between $Y_t$ and $Y_{t-k}$, **controlling for** all intermediate lags |

```R
# Plot correlograms
acf(ts_sales, main = "Autocorrelation (ACF)")
pacf(ts_sales, main = "Partial Autocorrelation (PACF)")
```
> **Exam Note:** Spikes crossing the horizontal blue dashed confidence bounds ($\pm \frac{1.96}{\sqrt{N}}$) indicate statistically significant autocorrelation.

---

## 8. Clustering

Clustering is an **unsupervised learning** technique that partitions unlabelled observations into homogeneous groups.

### 8.1 K-Means Clustering (`kmeans()`)

#### Algorithm Steps:
1. Select $K$ initial cluster centroids randomly.
2. Assign each observation to the nearest centroid (Euclidean distance).
3. Recompute centroid coordinates as the mean of assigned points.
4. Repeat until centroids no longer move (convergence).

#### Code & Output:
```R
# Prepare standardized data (e.g., using first 4 columns of iris dataset)
data_cluster <- scale(iris[, 1:4])

# Fit K-Means with K = 3
set.seed(42)
km_res <- kmeans(data_cluster, centers = 3, nstart = 20)

# Inspect results
print(km_res$size)     # Number of points in each of the 3 clusters
print(km_res$centers)  # Coordinates of the 3 cluster centers
print(km_res$tot.withinss) # Total within-cluster sum of squares (lower is more compact)

# View cluster assignments (1, 2, or 3)
head(km_res$cluster)
```

```R
# Visualizing K-Means clusters
plot(data_cluster[, 1:2], col = km_res$cluster, pch = 19,
     main = "K-Means Clustering (K = 3)", xlab = "Sepal Length", ylab = "Sepal Width")
points(km_res$centers[, 1:2], col = 1:3, pch = 8, cex = 2, lwd = 3) # Plot centroids
```

---

### 8.2 Hierarchical Clustering (`hclust()`)

Builds a bottom-up tree hierarchy (Dendrogram).

#### The 3-Step Workflow:
1. **Compute Distance Matrix:** `dist(data, method = "euclidean")`
2. **Build Hierarchy:** `hclust(d, method = "complete")` (linkage methods: `"complete"`, `"single"`, `"average"`, `"ward.D2"`)
3. **Cut Tree into $K$ Clusters:** `cutree(tree, k)`

```R
# 1. Distance matrix
d <- dist(data_cluster[1:15, ], method = "euclidean")

# 2. Hierarchical clustering
hc <- hclust(d, method = "complete")

# 3. Plot Dendrogram
plot(hc, main = "Dendrogram of First 15 Samples", xlab = "Sample Index")

# 4. Cut dendrogram into 3 clusters
cluster_groups <- cutree(hc, k = 3)
print(cluster_groups)
```

---

## 9. High-Yield Exam Trap Sheet

### Top 7 Rapid-Recall Distinctions

1. **`.C()` vs `.Call()` vs `Rcpp`**:
   - `.C()` copies memory into standard C pointers; `.Call()` works with raw R `SEXP` objects.
   - `Rcpp` provides high-level C++ classes without manual memory management.
2. **`mclapply()` on Windows**:
   - `mclapply()` uses POSIX forking; on Windows, it runs sequentially on 1 core!
   - On Windows, always use `makeCluster()` and `parLapply()`.
3. **`predict()` in Logistic Regression**:
   - If `type = "response"` is omitted, R returns **log-odds** ($\eta$), NOT probabilities.
   - Always set `type = "response"` to obtain probabilities $\in [0, 1]$.
4. **`lm()` vs `glm()`**:
   - `lm()` assumes Normal errors with identity link.
   - `glm()` supports Binomial (logistic), Poisson, Gamma via user-specified family and link functions.
5. **`nls()` Starting Values**:
   - `nls()` crashes with an error if initial `start` estimates are omitted.
6. **ACF vs PACF**:
   - ACF measures total correlation including intervening lags.
   - PACF isolates the direct correlation after filtering out intermediate lag effects.
7. **K-Means `nstart` Parameter**:
   - Setting `nstart = 20` runs K-Means with 20 distinct random starting centroids and keeps the best result, preventing poor local optima.
