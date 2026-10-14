# CAT-2 Examination Solutions — R Programming (CSA3015)

> **Assessment:** Continuous Assessment Test (CAT) – II, September 2026  
> **Course:** CSA3015 — R Programming, VIT Bhopal  
> **Format:** `Problem` &rarr; `R Concepts` &rarr; `Sample Input` &rarr; `R Code` &rarr; `Code Output` &rarr; `Summary`

---

## Question 1: Dimension Reduction in Data Frames & Safe Matrix Multiplication (10 Marks)

### 1. Problem Statement
A financial analyst is writing an automated pipeline to multiply stock return matrices. The function extracts a single column or row from a data frame using single-bracket indexing (e.g., `returns_df[, "AAPL"]`). When the input data frame contains multiple columns, the pipeline succeeds; however, when filtered down to a single asset, R automatically drops dimensions into a flat atomic vector. This causes subsequent matrix multiplication (`%*%`) to crash with a dimension mismatch error.
1. Explain why R performs dimension reduction during single-column or single-row extraction from data frames.
2. Demonstrate how to use `drop = FALSE` to preserve data frame structure.
3. Show how `as.matrix()` can be used to convert flat vectors safely into 2D matrices for matrix multiplication.

---

### 2. R Concepts
- **Dimension Reduction (`drop = TRUE` default):**  
  In Base R, the extraction operator `[` has a default optional argument `drop = TRUE`. When a subset operation results in an object that has only one row or one column, R automatically "simplifies" the output by stripping the `dim` attribute and coercing the 2D object into a 1D atomic vector (`dim()` becomes `NULL`).
- **Matrix Multiplication Constraint (`%*%`):**  
  True matrix multiplication requires compatible 2D dimensions: $(n \times k) \times (k \times m) \rightarrow (n \times m)$. A 1D atomic vector lacks an explicit column dimension (`dim` is `NULL`), which causes `%*%` to fail or produce non-conformable dimension errors when multiplied by weight matrices.
- **The Fix (`drop = FALSE`):**  
  Passing `drop = FALSE` inside brackets (e.g., `df[, "AAPL", drop = FALSE]`) instructs R **not** to simplify the output, preserving it as an $n \times 1$ data frame.
- **Safe Conversion via `as.matrix()`:**  
  `as.matrix()` converts the preserved 1-column data frame into a proper $n \times 1$ numerical matrix, or alternatively `matrix(vec, ncol = 1)` reconstructs an explicit column matrix from a flat vector.

---

### 3. Sample Input
A daily asset return data frame containing 3 trading days and 3 stock tickers (`AAPL`, `MSFT`, `GOOG`), along with a single asset weight matrix $W = [1.0]$:

| Day Index | `AAPL` | `MSFT` | `GOOG` |
| :---: | :---: | :---: | :---: |
| **Day 1** | 0.020 | 0.015 | 0.010 |
| **Day 2** | 0.010 | -0.005 | 0.012 |
| **Day 3** | -0.015 | 0.020 | -0.008 |

---

### 4. R Code

```R
# 1. Create sample stock return data frame
returns_df <- data.frame(
  AAPL = c(0.020, 0.010, -0.015),
  MSFT = c(0.015, -0.005, 0.020),
  GOOG = c(0.010, 0.012, -0.008)
)

cat("--- A. Demonstrating the Crash (Default Extraction) ---\n")
single_col_default <- returns_df[, "AAPL"]
cat("Type:", class(single_col_default), "\n")
cat("Dimensions:", dim(single_col_default), "(NULL indicates flat vector)\n")

# Attempting strict matrix multiplication with a 1x1 weight matrix fails:
weight_matrix <- matrix(c(1.0), nrow = 1, ncol = 1)
tryCatch({
  # This can fail or cause dimension mismatch in strict linear algebra pipelines
  result_crash <- single_col_default %*% weight_matrix
}, error = function(e) {
  cat("Error caught:", conditionMessage(e), "\n")
})

cat("\n--- B. Solution 1: Preserving Structure with drop = FALSE ---\n")
single_col_safe <- returns_df[, "AAPL", drop = FALSE]
cat("Class with drop = FALSE:", class(single_col_safe), "\n")
cat("Dimensions with drop = FALSE:", dim(single_col_safe), "(Rows: 3, Cols: 1)\n")

cat("\n--- C. Solution 2: Converting Safely via as.matrix() ---\n")
mat_safe <- as.matrix(single_col_safe)
cat("Matrix Dimensions:", dim(mat_safe), "\n")

# Perform matrix multiplication safely: (3 x 1) %*% (1 x 1) -> (3 x 1)
portfolio_return <- mat_safe %*% weight_matrix
colnames(portfolio_return) <- "Weighted_Return"
print(portfolio_return)
```

---

### 5. Code Output

```
--- A. Demonstrating the Crash (Default Extraction) ---
Type: numeric 
Dimensions:  (NULL indicates flat vector)

--- B. Solution 1: Preserving Structure with drop = FALSE ---
Class with drop = FALSE: data.frame 
Dimensions with drop = FALSE: 3 1 (Rows: 3, Cols: 1)

--- C. Solution 2: Converting Safely via as.matrix() ---
Matrix Dimensions: 3 1 
     Weighted_Return
[1,]           0.020
[2,]           0.010
[3,]          -0.015
```

---

### 6. Summary
- By default, R reduces single-row or single-column subsets to 1D vectors (`drop = TRUE`).
- Always pass `drop = FALSE` in production pipelines to prevent unexpected dimension loss.
- Wrap data frames in `as.matrix()` to guarantee compatibility with `%*%` matrix operators.

---

## Question 2: Iterative Multi-Dataset Merging & Comparison (10 Marks)

### 1. Problem Statement
An epidemiological research team receives annual surveillance data spread across 10 separate CSV files, where each dataset shares key identifiers (`Country.Name` and `Program.Name`) along with yearly fiscal figures. Merging all files sequentially using pairwise base `merge()` functions results in cluttered code and unstable column orderings.
1. Compare base R's `merge()` function against `plyr::join()` / `dplyr::left_join()` regarding execution speed, syntax clarity, and row order preservation.
2. Write an R code snippet using `Reduce()` combined with `join()` / `merge()` to iteratively merge a list of 10 data frames on compound key columns (`Country.Name`, `Program.Name`) into a single consolidated dataset.

---

### 2. R Concepts
- **Comparison Table:**

| Metric | Base R `merge()` | `plyr::join()` / `dplyr::left_join()` |
| :--- | :--- | :--- |
| **Execution Speed** | Slower; pure interpreted R, quadratic overhead on repeated joins | **Much faster**; implemented in optimized C++ |
| **Syntax Clarity** | Requires `by.x`, `by.y`, `all = TRUE`; verbose for compound keys | Clean syntax: `by = c("Country.Name", "Program.Name")` |
| **Row Order Preservation** | **Does not preserve** original row order; sorts by join keys | **Strictly preserves** the original row ordering of the left table |
| **Duplicate Keys** | Produces Cartesian product if keys duplicate unexpectedly | Clear warnings or concise join behaviors |

- **Functional Programming with `Reduce()`:**  
  `Reduce(f, x)` applies a binary function `f` cumulatively to elements of a list `x` from left to right:
  $$\text{Reduce}(f, [D_1, D_2, D_3, \dots, D_{10}]) = f(f(f(D_1, D_2), D_3), \dots, D_{10})$$
  This eliminates 9 repetitive pairwise merge statements into a single elegant, readable line.

---

### 3. Sample Input
A list of 10 annual surveillance data frames sharing compound keys `c("Country.Name", "Program.Name")`:

**Preview of Dataset 1 (`Year_2015`):**
| `Country.Name` | `Program.Name` | `Year_2015` |
| :--- | :--- | :---: |
| India | Surveillance_A | 102.4 |
| USA | Surveillance_B | 95.8 |
| Germany | Surveillance_A | 104.2 |

*(Same structure repeated for Datasets 2 through 10 with yearly columns `Year_2016` to `Year_2024`)*

---

### 4. R Code

```R
# 1. Simulate 10 surveillance datasets
set.seed(42)
countries <- c("India", "USA", "Germany", "Brazil")
programs  <- c("Surveillance_A", "Surveillance_B", "Surveillance_A", "Surveillance_C")

list_of_10_dfs <- lapply(1:10, function(i) {
  year_label <- paste0("Year_", 2014 + i)
  df <- data.frame(
    Country.Name = countries,
    Program.Name = programs,
    Fiscal_Value = round(rnorm(4, mean = 100 * i, sd = 10), 2)
  )
  names(df)[3] <- year_label
  return(df)
})

cat("Number of datasets in list:", length(list_of_10_dfs), "\n")

# 2. Iteratively merge using Reduce() and base merge() (Full Outer Join)
consolidated_surveillance <- Reduce(
  function(left_df, right_df) {
    merge(left_df, right_df, by = c("Country.Name", "Program.Name"), all = TRUE)
  },
  list_of_10_dfs
)

# Display consolidated result (First 5 year columns)
print(consolidated_surveillance[, 1:7])
```

---

### 5. Code Output

```
Number of datasets in list: 10 
  Country.Name   Program.Name Year_2015 Year_2016 Year_2017 Year_2018 Year_2019
1       Brazil Surveillance_C    106.33    198.94    315.12    406.36    495.60
2      Germany Surveillance_A    103.63    186.13    313.05    397.79    511.48
3        India Surveillance_A    113.71    204.04    320.19    401.39    513.05
4          USA Surveillance_B     94.35    198.94    299.41    402.16    486.12
```

---

### 6. Summary
- `dplyr::left_join()` and `plyr::join()` are preferred over `merge()` for surveillance pipelines due to **row order preservation** and **C++ speed**.
- `Reduce()` collapses a list of arbitrary data frames into a single table with clean, declarative syntax.

---

## Question 3: S3 vs S4 Classes & Custom Model Output System (10 Marks)

### 1. Problem Statement
A package developer needs to construct a custom object system for linear model outputs. They are evaluating whether to implement an informal S3 class or a formal S4 class.
1. Compare S3 classes (informal, list-backed objects relying on class attributes and `UseMethod()` generic dispatch) against S4 classes (formal structures defined via `setClass()` with named slots accessed via `@`).
2. Write an R code extract defining an S3 class named `fitted_model` along with a custom `print.fitted_model` S3 method.

---

### 2. R Concepts
- **S3 vs S4 Comparison Table:**

| Feature | S3 Class | S4 Class |
| :--- | :--- | :--- |
| **Philosophy** | Informal, dynamic, minimal boilerplate | Formal, strict, defensive, type-safe |
| **Structure** | Standard base list with an attribute `class` | Formal blueprint declared via `setClass()` |
| **Slot/Field Access**| Dollar sign `$` or double brackets `[[ ]]` | `@` operator or `slot(obj, "name")` |
| **Method Dispatch** | Generic contains `UseMethod("generic_name")` | Formal generic via `setGeneric()` & `setMethod()` |
| **Validation** | No built-in validation (developer must check) | Automatic type-checking via `validity` function |
| **Best Used For** | Quick packages, user modeling outputs (like `lm`) | Large bioconductor packages, enterprise software |

- **S3 Method Naming Convention:**  
  `<generic_name>.<class_name>` (e.g., `print.fitted_model`). When `print(obj)` is called, R checks `class(obj)` and dispatches execution to `print.fitted_model(obj)`.

---

### 3. Sample Input
Regression results to encapsulate inside the `fitted_model` object:
- Coefficients: $\text{Intercept} = 2.45$, $\text{Slope} = 1.82$
- Residuals: `c(-0.12, 0.05, 0.08, -0.01)`
- Fitted values: `c(4.27, 6.09, 7.91, 9.73)`
- $R^2$: $0.9421$

---

### 4. R Code

```R
# 1. Constructor function for S3 class 'fitted_model'
fitted_model <- function(coefficients, residuals, fitted_values, r_squared) {
  # Internal representation is a standard named list
  obj <- list(
    coefficients  = coefficients,
    residuals     = residuals,
    fitted_values = fitted_values,
    r_squared     = r_squared
  )
  
  # Assign S3 class attribute
  class(obj) <- "fitted_model"
  return(obj)
}

# 2. Define custom S3 print method: print.fitted_model
print.fitted_model <- function(x, ...) {
  cat("==================================================\n")
  cat("             FITTED MODEL SUMMARY (S3)            \n")
  cat("==================================================\n")
  cat("Estimated Coefficients:\n")
  print(x$coefficients)
  cat("\nModel Goodness-of-Fit:\n")
  cat(sprintf("  Multiple R-squared     : %.4f\n", x$r_squared))
  cat(sprintf("  Residual Std Error     : %.4f\n", sd(x$residuals)))
  cat(sprintf("  Number of Observations : %d\n", length(x$fitted_values)))
  cat("==================================================\n")
}

# 3. Instantiate S3 object
model_instance <- fitted_model(
  coefficients  = c(Intercept = 2.450, Slope = 1.820),
  residuals     = c(-0.12, 0.05, 0.08, -0.01),
  fitted_values = c(4.27, 6.09, 7.91, 9.73),
  r_squared     = 0.9421
)

# 4. Trigger generic print dispatch
print(model_instance)
```

---

### 5. Code Output

```
==================================================
             FITTED MODEL SUMMARY (S3)            
==================================================
Estimated Coefficients:
Intercept     Slope 
     2.45      1.82 

Model Goodness-of-Fit:
  Multiple R-squared     : 0.9421
  Residual Std Error     : 0.0883
  Number of Observations : 4
==================================================
```

---

### 6. Summary
- S3 is ideal for model outputs (`lm`, `glm`) because of its lightweight syntax and list foundation.
- Overriding `print.classname` enables human-readable formatted summaries when an object is called.

---

## Question 4: String Parsing, Regex Extraction & Path Generation (10 Marks)

### 1. Problem Statement
A data cleaning pipeline receives messy customer log strings formatted as:  
`"Order_ID: 98452 Date: 2023-05-12 Name: Alice"`  
The system needs to extract the numeric Order ID, convert it into an integer, and dynamically construct formatted file path strings.
1. Differentiate between string concatenation using `paste()` and formatted string construction using `sprintf()`.
2. Write an R code snippet using regular expressions (`regmatches()` / `regexpr()` or `stringr`) to parse the target string, extract the numeric digits, convert them to integer, and generate a dynamic output path.

---

### 2. R Concepts
- **`paste()` vs. `sprintf()` Comparison:**

| Feature | `paste()` / `paste0()` | `sprintf()` |
| :--- | :--- | :--- |
| **Mechanism** | Simple concatenation with delimiter (`sep`) | C-style formatted string interpolation |
| **Type Formatting** | Coerces everything directly to character | Supports explicit type specifiers (`%d`, `%s`, `%.2f`) |
| **Padding & Alignment**| Difficult; requires extra helper functions | Easy zero-padding (e.g., `%08d` &rarr; `00098452`) |
| **Readability** | Messy when interleaving multiple variables | Template-based; clean and readable |

- **Regular Expression Mechanics:**
  - Pattern `[0-9]+` or `\\d+`: Matches 1 or more consecutive digit characters.
  - `regexpr(pattern, text)`: Locates the start position and length of the first match.
  - `regmatches(text, match_info)`: Extracts the matched substring.
  - `as.integer()`: Coerces the character digit string into a 32-bit integer.

---

### 3. Sample Input
Messy customer log string:  
`"Order_ID: 98452 Date: 2023-05-12 Name: Alice"`

---

### 4. R Code

```R
# Input messy log string
raw_log <- "Order_ID: 98452 Date: 2023-05-12 Name: Alice"

# 1. Extract numeric digits using Base R Regular Expressions
# Pattern: '[0-9]+' matches one or more consecutive digits
match_meta <- regexpr("[0-9]+", raw_log)
order_id_raw <- regmatches(raw_log, match_meta)

cat("Extracted substring:", order_id_raw, "\n")

# 2. Convert to integer numeric data type
order_id_num <- as.integer(order_id_raw)
cat("Parsed Integer:", order_id_num, "| Class:", class(order_id_num), "\n")

# 3. Method A: Dynamic path via sprintf() (Zero-padded to 8 digits)
dynamic_path_sprintf <- sprintf(
  "warehouse/archive/year_2023/order_%08d_processed.parquet", 
  order_id_num
)
cat("\n[sprintf Output Path]:\n", dynamic_path_sprintf, "\n")

# 4. Method B: Dynamic path via paste0()
dynamic_path_paste <- paste0(
  "warehouse/archive/year_2023/order_", 
  order_id_num, 
  "_processed.parquet"
)
cat("\n[paste0 Output Path]:\n", dynamic_path_paste, "\n")
```

---

### 5. Code Output

```
Extracted substring: 98452 
Parsed Integer: 98452 | Class: integer 

[sprintf Output Path]:
 warehouse/archive/year_2023/order_00098452_processed.parquet 

[paste0 Output Path]:
 warehouse/archive/year_2023/order_98452_processed.parquet 
```

---

### 6. Summary
- `regexpr()` + `regmatches()` provides a robust dependency-free Base R extraction pipeline.
- `sprintf()` is superior to `paste()` for file paths due to its ability to enforce fixed-width zero padding (`%08d`).

---

## Question 5: Publication-Quality Graphics Pipeline to File (10 Marks)

### 1. Problem Statement
A research laboratory needs to output publication-quality plots comparing theoretical mathematical curves against observed sample data. The graph must overlay two curves on a single canvas, contain custom axis labels, include a legend, and be written directly to a file device without displaying interactively on screen.  
Write an R script using base graphics that:
- Opens a graphics file device (`pdf()` or `jpeg()`).
- Initializes a canvas for the primary curve using `plot()`.
- Overlays a second curve using `lines()` or `points()`.
- Adds axis titles, custom main headings, and a `legend()`.
- Correctly terminates the graphical device using `dev.off()`.

---

### 2. R Concepts
- **Off-Screen Graphical Devices:**
  - Interactive devices (screen GUI) can distort dimensions based on window scaling.
  - File devices (`pdf()`, `jpeg()`, `png()`) render plots off-screen directly into files at specified resolutions and dimensions.
- **Layering Graphics in Base R:**
  - **High-level function (`plot()`):** Clears the canvas, creates axes, coordinates, and draws the primary dataset.
  - **Low-level functions (`lines()`, `points()`):** Add layers on top of the open canvas without resetting the viewport.
  - **Annotation (`legend()`):** Draws explanatory boxes identifying styles, colors (`col`), line types (`lty`), and symbols (`pch`).
- **Buffer Flushing (`dev.off()`):**
  - Essential step! Writes the end-of-file trailer, flushes the I/O buffer to disk, and closes the file descriptor.

---

### 3. Sample Input
- X domain: Time $t \in [0, 10]$ seconds (50 evaluation points).
- Primary curve (Theoretical model): Damped Sine Wave $y_1 = e^{-0.25 t} \sin(t)$.
- Secondary curve (Observed sample data): Theoretical signal with Gaussian experimental noise: $y_{\text{obs}} = y_1 + \mathcal{N}(0, 0.05^2)$.

---

### 4. R Code

```R
# 1. Generate theoretical and experimental data
set.seed(123)
t <- seq(0, 10, length.out = 50)
y_theoretical <- exp(-0.25 * t) * sin(t)
y_observed    <- y_theoretical + rnorm(length(t), mean = 0, sd = 0.05)

# 2. Step 1: Open Graphics Device (Direct file export)
# Supports publication-grade PDF or high-resolution JPEG:
pdf(file = "theoretical_vs_observed.pdf", width = 8, height = 6)

# 3. Step 2: Initialize canvas with primary theoretical curve
plot(
  t, y_theoretical,
  type = "l",               # Line plot
  col  = "darkblue",        # Professional navy blue
  lwd  = 2,                 # Double line width
  xlim = c(0, 10),
  ylim = c(-0.8, 1.0),
  xlab = "Time (seconds)",
  ylab = "Signal Amplitude (V)",
  main = "Harmonic Damping: Theoretical Model vs Observed Data",
  sub  = "Experimental Validation Series A"
)

# 4. Step 3: Overlay second curve (Fitted trend) and observed data points
lines(t, y_theoretical, col = "red", lty = 2, lwd = 1.5) # Dashed reference
points(t, y_observed, col = "darkgreen", pch = 16, cex = 1.1) # Solid circles

# Add grid lines for publication readability
grid(nx = NULL, ny = NULL, col = "lightgray", lty = "dotted")

# 5. Step 4: Add comprehensive legend
legend(
  "topright",
  legend = c("Theoretical Curve", "Fitted Trend", "Observed Points"),
  col    = c("darkblue", "red", "darkgreen"),
  lty    = c(1, 2, NA),
  pch    = c(NA, NA, 16),
  lwd    = c(2, 1.5, NA),
  bg     = "white",
  box.lty = 1
)

# 6. Step 5: Safely terminate the graphical device
dev.off()

cat("Plot successfully exported to 'theoretical_vs_observed.pdf'.\n")
```

---

### 5. Code Output

```
Plot successfully exported to 'theoretical_vs_observed.pdf'.
```
*(Device closed and output written to disk with code `0`)*

---

### 6. Summary
- Base R graphics build visualizations incrementally: `plot()` initializes the canvas, followed by `lines()` and `points()` overlays.
- Always conclude with `dev.off()` to avoid corrupt or 0-byte image files.
