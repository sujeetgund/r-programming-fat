# Module 1: Introduction to R & Vectors — Exam Revision Guide

> **Target:** R Programming Term-End Exam (FAT)  
> **Key References:** Base R Mechanics & *The Art of R Programming* (Chapters 1 & 2) by Norman Matloff  
> **Exam Strategy:** Direct answers for short notes, syntax templates, comparison tables, and code snippets ready to write on paper or run in a lab exam.

---

## Table of Contents
1. [Introduction & Core Philosophy of R](#1-introduction--core-philosophy-of-r)
2. [R Help Functions](#2-r-help-functions)
3. [Vectors, Scalars & Declarations](#3-vectors-scalars--declarations)
4. [The Recycling Rule](#4-the-recycling-rule)
5. [Vectorized Operations & Built-in Functions](#5-vectorized-operations--built-in-functions)
6. [Using `all()` and `any()`](#6-using-all-and-any)
7. [`NA` vs `NULL` (High-Yield Comparison)](#7-na-vs-null)
8. [Filtering (Vector Subsetting) & `which()`](#8-filtering-vector-subsetting--which)
9. [Vectorized If-Then-Else (`ifelse`)](#9-vectorized-if-then-else-ifelse)
10. [Vector Equality: `==` vs `identical()` vs `all.equal()`](#10-vector-equality)
11. [Vector Element Names](#11-vector-element-names)
12. [High-Yield Exam Trap Sheet](#12-high-yield-exam-trap-sheet)

---

## 1. Introduction & Core Philosophy of R

- **Interpreted & Dynamically Typed:** Code executes line-by-line; variables do not require explicit type declarations.
- **Case-Sensitive:** `MyVar` and `myvar` are completely distinct variables.
- **Functional & Vector-Oriented:** Operations are designed to work on entire vectors at once rather than individual elements via manual loops.
- **Primary Data Structures:**
  1. **Atomic Vectors:** 1D homogeneous (all elements same type).
  2. **Matrices:** 2D homogeneous.
  3. **Arrays:** $n$-dimensional homogeneous.
  4. **Lists:** 1D heterogeneous (can hold mixed types, functions, other lists).
  5. **Data Frames:** 2D heterogeneous tabular structures (columns are equal-length vectors).
  6. **Factors:** Categorical data represented by integer codes + character levels.

---

## 2. R Help Functions

High-frequency viva and 2-mark theory questions:

| Function / Command | Purpose | Example |
| :--- | :--- | :--- |
| `?func_name` / `help(func_name)` | Direct documentation for an exact function | `?mean` or `help("seq")` |
| `??keyword` / `help.search("keyword")`| Searches documentation across all installed packages | `??regression` |
| `args(func_name)` | Displays the formal arguments and default values | `args(round)` |
| `example(func_name)` | Runs and outputs the runnable examples in the docs | `example(plot)` |
| `demo(topic)` | Runs interactive demonstration scripts | `demo(graphics)` |

---

## 3. Vectors, Scalars & Declarations

### 3.1 There Are No Scalars in R!
> **Exam Core Concept:** In R, **scalars do not exist as distinct data types**. A single number or character string is simply an **atomic vector of length 1**.

```R
x <- 42
is.vector(x)   # TRUE
length(x)      # 1
```

### 3.2 Dynamic Declarations & Atomic Types
Variables are declared automatically upon assignment (`<-`):

```R
# 6 Atomic Types in R:
v_log <- TRUE               # 1. Logical
v_int <- 15L                # 2. Integer (explicit 'L' suffix)
v_num <- 15.5               # 3. Numeric / Double (default for numbers)
v_chr <- "Hello"            # 4. Character
v_cpl <- 2 + 3i             # 5. Complex
v_raw <- charToRaw("A")     # 6. Raw (byte representation: 41)
```

### 3.3 Automatic Coercion Rule (The Hierarchy)
When mixing types in `c()`, R automatically coerces all elements to the least restrictive type:

$$\text{Logical} \;\longrightarrow\; \text{Integer} \;\longrightarrow\; \text{Double (Numeric)} \;\longrightarrow\; \text{Character}$$

```R
c(TRUE, 2L)          # Coerced to Integer: c(1, 2)
c(TRUE, 2.5)         # Coerced to Numeric: c(1.0, 2.5)
c(10, "Apple", TRUE) # Coerced to Character: c("10", "Apple", "TRUE")
```

---

## 4. The Recycling Rule

When two vectors of **unequal length** undergo an arithmetic or logical operation, R automatically **recycles (repeats)** the elements of the shorter vector until its length matches the longer vector.

### Case 1: Shorter length is an EXACT factor of longer length
*No warning emitted.*

```R
v1 <- c(10, 20, 30, 40)  # Length 4
v2 <- c(1, 2)            # Length 2 (4 is a multiple of 2)

res <- v1 + v2
print(res)
```

**Console Output:**
```
[1] 11 22 31 42
```
> **Explanation:** `v2` is recycled as `c(1, 2, 1, 2)`.  
> Addition: `(10+1)=11`, `(20+2)=22`, `(30+1)=31`, `(40+2)=42`.

---

### Case 2: Shorter length is NOT an exact factor
*R performs recycling, but throws a Warning.*

```R
v_long  <- c(10, 20, 30, 40, 50)  # Length 5
v_short <- c(1, 2)                # Length 2 (5 is NOT a multiple of 2)

res_warn <- v_long + v_short
print(res_warn)
```

**Console Output:**
```
[1] 11 22 31 42 51
Warning message:
In v_long + v_short :
  longer object length is not a multiple of shorter object length
```
> **Explanation:** `v_short` repeats as `c(1, 2, 1, 2, 1)`.

---

## 5. Vectorized Operations & Built-in Functions

Vectorization means functions operate **element-wise** over entire vectors without writing explicit `for` loops.

### 5.1 Common Vector Generators & Operations
```R
# Generating sequences
1:5                     # c(1, 2, 3, 4, 5)
seq(from = 1, to = 9, by = 2)  # c(1, 3, 5, 7, 9)
seq(0, 1, length.out = 5)      # c(0.00, 0.25, 0.50, 0.75, 1.00)

# Replicating elements (rep)
rep(5, times = 3)              # c(5, 5, 5)
rep(c(1, 2), times = 2)        # c(1, 2, 1, 2)
rep(c(1, 2), each = 2)         # c(1, 1, 2, 2)  <-- Common exam distinction!

# Basic operations
x <- c(4, 16, 25)
sqrt(x)                 # c(2, 4, 5)  (Vectorized!)
x * 2                   # c(8, 32, 50)
sum(x)                  # 45
prod(x)                 # 1600
length(x)               # 3
rev(x)                  # c(25, 16, 4)
sort(c(9, 2, 7))        # c(2, 7, 9)
```

---

## 6. Using `all()` and `any()`

Both functions test logical conditions across entire vectors and return a single boolean:
- **`any()`**: Returns `TRUE` if **at least one** element satisfies the condition.
- **`all()`**: Returns `TRUE` if **every single** element satisfies the condition.

```R
scores <- c(82, 95, 74, 88, 45)

any(scores < 50)   # TRUE  (Because 45 < 50)
all(scores >= 50)  # FALSE (Because 45 is not >= 50)
all(scores > 0)    # TRUE  (All scores are positive)
```

---

## 7. `NA` vs `NULL`

This is one of the most frequently asked viva and 2-mark questions.

### 7.1 Comparison Table

| Feature | `NA` (Not Available) | `NULL` (Empty Object) |
| :--- | :--- | :--- |
| **Meaning** | Missing or unknown data value | Represents non-existence or an empty object |
| **Length** | `length(NA)` is **`1`** | `length(NULL)` is **`0`** |
| **Type** | Has a type (e.g., `NA_integer_`, `NA_character_`) | Has its own special type (`"NULL"`) |
| **In Operations** | **Propagates:** `5 + NA` evaluates to `NA` | **Disappears:** `c(5, NULL)` evaluates to `c(5)` |
| **Test Function** | `is.na(x)` | `is.null(x)` |
| **Use in deletion**| Cannot delete vector elements | Assigning `NULL` deletes list/dataframe items |

### 7.2 Code Proof

```R
# Length comparison
length(NA)      # [1] 1
length(NULL)    # [1] 0

# Arithmetic propagation
5 + NA          # [1] NA
5 + NULL        # numeric(0)

# Vector combination behavior
vec_na   <- c(10, NA, 30)
length(vec_na)  # [1] 3

vec_null <- c(10, NULL, 30)
length(vec_null)# [1] 2  (NULL vanishes!)

# Handling NAs in summary functions (na.rm argument)
sum(c(10, 20, NA))               # [1] NA  (Default na.rm = FALSE)
sum(c(10, 20, NA), na.rm = TRUE) # [1] 30
```

---

## 8. Filtering (Vector Subsetting) & `which()`

R provides 4 distinct ways to filter vector elements.

### 8.1 The 4 Indexing Modes
```R
v <- c(10, 20, 30, 40, 50)

# 1. Positive integers: Select specific positions
v[c(1, 4)]             # [1] 10 40

# 2. Negative integers: Exclude specific positions
v[-c(2, 5)]            # [1] 10 30 40

# 3. Logical vector (Filtering by condition)
v[v > 25]              # [1] 30 40 50

# 4. Element names (if named)
names(v) <- c("a", "b", "c", "d", "e")
v[c("a", "c")]         # a: 10, c: 30
```

### 8.2 Using `which()`
`which(condition)` returns the **integer positions (indices)** where the condition is `TRUE`, rather than the values themselves.

```R
marks <- c(45, 88, 92, 33, 76)

# Direct condition returns TRUE/FALSE vector:
marks > 75              # FALSE  TRUE  TRUE FALSE  TRUE

# which() returns the indices:
which(marks > 75)       # [1] 2 3 5

# which.min() and which.max()
which.min(marks)        # [1] 4 (index of 33)
which.max(marks)        # [1] 3 (index of 92)
```

---

## 9. Vectorized If-Then-Else (`ifelse`)

Standard `if (...) { ... }` in R only tests a **single condition** (a scalar). For vectors, use `ifelse()`.

### Syntax:
`ifelse(test, yes, no)`

#### Input Data:
| Student Index | `score` | Condition: `score >= 50` | Result |
| :---: | :---: | :---: | :---: |
| 1 | 82 | `TRUE` | "Pass" |
| 2 | 34 | `FALSE` | "Fail" |
| 3 | 90 | `TRUE` | "Pass" |
| 4 | 48 | `FALSE` | "Fail" |

```R
scores <- c(82, 34, 90, 48)

result <- ifelse(scores >= 50, "Pass", "Fail")
print(result)
```

**Console Output:**
```
[1] "Pass" "Fail" "Pass" "Fail"
```

---

## 10. Vector Equality

How to compare vectors properly in R:

| Method | What it checks | Return Value | Exam Example |
| :--- | :--- | :--- | :--- |
| `x == y` | **Element-wise** equality | Logical vector of `TRUE`/`FALSE` | `c(1, 2) == c(1, 3)` $\rightarrow$ `c(TRUE, FALSE)` |
| `identical(x, y)` | Exact byte-for-byte identity & identical types | Single boolean (`TRUE`/`FALSE`) | `identical(2, 2L)` $\rightarrow$ `FALSE` (double vs integer) |
| `all.equal(x, y)` | Nearly equal within floating-point tolerance ($10^{-8}$) | `TRUE` if equal; or text string describing difference | `all.equal(sqrt(2)^2, 2)` $\rightarrow$ `TRUE` |

```R
# Floating point comparison trap:
(sqrt(2))^2 == 2          # FALSE (Floating point rounding imprecision!)
all.equal((sqrt(2))^2, 2) # TRUE  (Safe for numerical analysis)
```

---

## 11. Vector Element Names

Vectors can have optional name attributes for self-documenting code.

```R
# 1. Assigning names during creation
salaries <- c(Rahul = 50000, Sneha = 75000, Amit = 62000)

# 2. Assigning / changing names via names() replacement function
scores <- c(88, 92, 79)
names(scores) <- c("Math", "Physics", "Chemistry")
print(scores)
```

**Console Output:**
```
     Math   Physics Chemistry 
       88        92        79 
```

```R
# 3. Accessing by name
scores["Physics"]       # Physics: 92

# 4. Removing names with unname()
clean_vec <- unname(scores)
print(clean_vec)        # [1] 88 92 79
```

---

## 12. High-Yield Exam Trap Sheet

1. **`rep(..., times = 2)` vs `rep(..., each = 2)`**:
   - `times = 2`: `c(1, 2, 1, 2)`
   - `each = 2`: `c(1, 1, 2, 2)`
2. **`NA` vs `NULL`**:
   - `NA` is a missing value with length 1.
   - `NULL` is an empty object with length 0.
3. **`identical(1, 1L)` is `FALSE`**:
   - `1` is a `numeric` (double).
   - `1L` is an `integer`.
4. **Recycling Rule Warning**:
   - Only warns if the longer vector length is **not** an integer multiple of the shorter vector length.
5. **`which()` vs Boolean Indexing**:
   - `v > 10` returns boolean vector `c(FALSE, TRUE)`.
   - `which(v > 10)` returns index integer `c(2)`.
