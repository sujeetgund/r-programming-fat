# Module 3: Data Frames, Factors, Tables, Functions & Simulations — Exam Revision Guide

> **Target:** R Programming Term-End Exam (FAT)  
> **Key References:** Base R Mechanics & *The Art of R Programming* by Norman Matloff  
> **Exam Strategy:** Direct answers for short notes, syntax templates, comparison tables, and code snippets ready to write on paper or run in a lab exam.

---

## Table of Contents
1. [Data Frames: Creation, Operations & Merging](#1-data-frames)
2. [Applying Functions to Data Frames (The `apply` Family)](#2-applying-functions-to-data-frames)
3. [Factors and Tables](#3-factors-and-tables)
4. [Operators & Control Statements](#4-operators--control-statements)
5. [R Functions as First-Class Objects](#5-functions-as-objects--core-concepts)
6. [Scoping, "Writing Upstairs" & Advanced Functions](#6-scoping-writing-upstairs--advanced-functions)
7. [Math & Statistical Simulations in R](#7-math-and-simulations-in-r)
8. [High-Yield Exam Trap Sheet & Frequently Asked Questions](#8-high-yield-exam-trap-sheet)

---

## 1. Data Frames

A **Data Frame** is a 2D tabular data structure where:
- Columns can have **different data types** (unlike matrices).
- All columns must have the **exact same length** (number of rows).
- Under the hood, a data frame is an S3 list of equal-length vectors.

### 1.1 Creating Data Frames
```R
# Creating a data frame from individual vectors
students_df <- data.frame(
  id = 101:104,
  name = c("Arun", "Bhavna", "Chetan", "Divya"),
  marks = c(85.5, 92.0, 78.0, 88.5),
  passed = c(TRUE, TRUE, FALSE, TRUE),
  stringsAsFactors = FALSE  # Keeps characters as strings (default in R >= 4.0)
)

# Inspecting structure
dim(students_df)       # Output: c(4, 4) -> 4 rows, 4 columns
nrow(students_df)      # 4
ncol(students_df)      # 4
str(students_df)       # Data types and preview of each column
summary(students_df)   # Statistical summary
```

---

### 1.2 Matrix-like Operations in Data Frames

Data frames support both **list-style indexing** and **matrix-style 2D indexing**.

#### Indexing & Slicing
```R
# 1. Accessing single columns:
students_df$name                # Returns vector: c("Arun", "Bhavna", ...)
students_df[["name"]]           # Returns vector
students_df[, "name"]           # Returns vector (simplifies by default)
students_df[, "name", drop = FALSE] # Keeps as a 1-column data frame

# 2. Matrix 2D Indexing: df[row_indices, col_indices]
students_df[1, 2]               # Row 1, Col 2 -> "Arun"
students_df[1:2, ]              # Rows 1 to 2, all columns
students_df[, c(1, 3)]          # All rows, columns 1 and 3

# 3. Filtering / Boolean Subsetting:
top_students <- students_df[students_df$marks >= 85, ]
passed_only  <- subset(students_df, passed == TRUE & marks > 80)
```

#### Modifying, Adding & Dropping Columns/Rows
```R
# Add a new column
students_df$grade <- c("A", "O", "B", "A")

# Delete a column by assigning NULL
students_df$grade <- NULL

# Add columns horizontally using cbind()
bonus <- c(5, 4, 2, 5)
students_df <- cbind(students_df, bonus)

# Add rows vertically using rbind()
new_student <- data.frame(id = 105, name = "Esha", marks = 95.0, passed = TRUE, bonus = 5)
students_df <- rbind(students_df, new_student)
```

---

### 1.3 Merging Data Frames (`merge()`)

Similar to SQL Joins between two data frames based on a common key column.

#### Input Data Frames:

**Table A: `df_info`**
| `id` | `name` |
| :---: | :---: |
| 1 | Kavya |
| 2 | Leo |
| 3 | Mohan |
| 4 | Neha |

**Table B: `df_scores`**
| `id` | `score` |
| :---: | :---: |
| 2 | 90 |
| 3 | 85 |
| 4 | 77 |
| 5 | 95 |

```R
df_info <- data.frame(
  id = c(1, 2, 3, 4),
  name = c("Kavya", "Leo", "Mohan", "Neha")
)

df_scores <- data.frame(
  id = c(2, 3, 4, 5),
  score = c(90, 85, 77, 95)
)
```

---

#### 1. Inner Join (Default: `by = "id"`)
Keeps **only rows with matching IDs** present in **both** tables (IDs 2, 3, 4):

```R
res_inner <- merge(df_info, df_scores, by = "id")
print(res_inner)
```

**Console Output:**
```
  id  name score
1  2   Leo    90
2  3 Mohan    85
3  4  Neha    77
```

---

#### 2. Full Outer Join (`all = TRUE`)
Keeps **all rows from both tables**; fills missing values with `NA`:

```R
res_outer <- merge(df_info, df_scores, by = "id", all = TRUE)
print(res_outer)
```

**Console Output:**
```
  id  name score
1  1 Kavya    NA
2  2   Leo    90
3  3 Mohan    85
4  4  Neha    77
5  5  <NA>    95
```
> *ID 1 has no score $\rightarrow$ `NA`. ID 5 has no name $\rightarrow$ `<NA>`.*

---

#### 3. Left Outer Join (`all.x = TRUE`)
Keeps **all rows from the left table (`df_info`)**, matching scores where available:

```R
res_left <- merge(df_info, df_scores, by = "id", all.x = TRUE)
print(res_left)
```

**Console Output:**
```
  id  name score
1  1 Kavya    NA
2  2   Leo    90
3  3 Mohan    85
4  4  Neha    77
```

---

#### 4. Right Outer Join (`all.y = TRUE`)
Keeps **all rows from the right table (`df_scores`)**, matching names where available:

```R
res_right <- merge(df_info, df_scores, by = "id", all.y = TRUE)
print(res_right)
```

**Console Output:**
```
  id  name score
1  2   Leo    90
2  3 Mohan    85
3  4  Neha    77
4  5  <NA>    95
```

---

#### 5. When Key Column Names Differ
If Table A uses `student_id` and Table B uses `roll_no`:
```R
# Syntax:
merge(df_info, df_scores, by.x = "student_id", by.y = "roll_no")
```

---

## 2. Applying Functions to Data Frames

The `*apply` family replaces explicit `for` loops with vectorized functional programming.

```
       ┌────────────────────────┐
       │   The *apply Family    │
       └────────────────────────┘
          │        │         │
          ▼        ▼         ▼
       apply()   lapply()  tapply()
          │        │         │
      (Matrix/DF  (List/DF   (Grouped by
       by Row/Col) returns    Factor)
                    list)
                      │
                      ▼
                   sapply()
                  (Simplified output)
```

### 2.1 The `apply()` Family Comparison

| Function | Primary Input | Margin / Grouping | Output Type | Typical Use Case |
| :--- | :--- | :--- | :--- | :--- |
| `apply(X, MARGIN, FUN)` | Matrix or numeric DF | `1` = Rows, `2` = Columns | Vector or Matrix | Row-wise or Column-wise calculations |
| `lapply(X, FUN)` | List, Vector, or DF | Operates on each column | **Always a List** | Column transformations returning varied lengths |
| `sapply(X, FUN)` | List, Vector, or DF | Operates on each column | **Vector / Matrix** (simplified) | Column-wise calculations returning simple vector |
| `tapply(X, INDEX, FUN)`| Atomic vector | Grouped by **Factor vector** | Array / Table | Group statistics (e.g., mean mark by department) |

---

### 2.2 Input Data & Step-by-Step Code with Output

#### A. Input Data Frame (`df_nums`)

| Row Index | `math` | `science` | `english` |
| :---: | :---: | :---: | :---: |
| **1** | 80 | 95 | 75 |
| **2** | 90 | 70 | 85 |
| **3** | 85 | 88 | 92 |

```R
df_nums <- data.frame(
  math = c(80, 90, 85),
  science = c(95, 70, 88),
  english = c(75, 85, 92)
)
```

---

#### 1. `apply()` with `MARGIN = 1` (Row-wise Calculation)
Calculates the mean score for each student (each row across all subjects):

```R
row_averages <- apply(df_nums, 1, mean)
print(row_averages)
```

**Console Output:**
```
[1] 83.33333 81.66667 88.33333
```
> *(Row 1 avg = 250/3 = 83.33, Row 2 avg = 245/3 = 81.67, Row 3 avg = 265/3 = 88.33)*

---

#### 2. `apply()` with `MARGIN = 2` (Column-wise Calculation)
Calculates the maximum score achieved in each subject (each column across all students):

```R
col_maximums <- apply(df_nums, 2, max)
print(col_maximums)
```

**Console Output:**
```
   math science english 
     90      95      92 
```
> *Returns a named numeric vector matching each column name.*

---

#### 3. `lapply()` (List Apply — Returns a List)
Computes the mean of each column, **always returning a list**:

```R
lap_res <- lapply(df_nums, mean)
print(lap_res)
```

**Console Output:**
```
$math
[1] 85

$science
[1] 84.33333

$english
[1] 84
```
> *Notice the `$colname` list notation — each element is a list entry.*

---

#### 4. `sapply()` (Simplified Apply — Returns a Vector)
Computes the mean of each column and **simplifies the result into a clean named vector**:

```R
sap_res <- sapply(df_nums, mean)
print(sap_res)
```

**Console Output:**
```
    math  science  english 
85.00000 84.33333 84.00000 
```
> *Contrast this with `lapply`: `sapply` simplifies list objects into an atomic vector.*

---

#### 5. `tapply()` (Grouped Summary by Factor)

#### Input Data:
| Index | `salaries` | `dept` (Factor) |
| :---: | :---: | :---: |
| **1** | 45,000 | HR |
| **2** | 52,000 | HR |
| **3** | 75,000 | IT |
| **4** | 80,000 | IT |
| **5** | 48,000 | HR |

```R
salaries <- c(45000, 52000, 75000, 80000, 48000)
dept <- factor(c("HR", "HR", "IT", "IT", "HR"))

mean_sal_by_dept <- tapply(salaries, dept, mean)
print(mean_sal_by_dept)
```

**Console Output:**
```
      HR       IT 
48333.33 77500.00 
```
> *Calculates `mean(45000, 52000, 48000)` = 48,333.33 for HR, and `mean(75000, 80000)` = 77,500.00 for IT.*

---

## 3. Factors and Tables

### 3.1 Factors and Levels
A **Factor** is R's categorical data structure. Internally, R stores factors as an **integer vector** with a character attribute `levels`.

```R
# 1. Nominal (Unordered) Factor — Categories with NO natural ranking
# E.g., Gender, Blood Group, Colors
gender <- factor(c("Male", "Female", "Female", "Male", "Male"))
gender
# Levels: Female Male (alphabetical order by default)

# Inspecting levels
levels(gender)    # c("Female", "Male")
nlevels(gender)   # 2

# Custom level ordering / renaming
status <- factor(c("low", "high", "medium"), levels = c("low", "medium", "high"))

# 2. Ordinal (Ordered) Factor — Categories WITH a strict ranking/hierarchy
# E.g., Shirt Sizes: Small < Medium < Large
# Or Student Grades: C < B < A
```

#### Deep Dive: How Ordered Factors Work (`ordered = TRUE`)

Suppose we have grades for 4 students: `"B"`, `"A"`, `"C"`, `"A"`.

```R
grade <- factor(
  c("B", "A", "C", "A"),
  levels = c("C", "B", "A"),  # Left-to-right defines hierarchy: Lowest (C) to Highest (A)
  ordered = TRUE              # Tells R to enable '<', '<=', '>', '>=' comparisons
)

print(grade)
```

**Console Output:**
```
[1] B A C A
Levels: C < B < A
```
> Notice the **`<` symbol** in `Levels: C < B < A`. R now officially knows that $C < B < A$.

Now let's compare student 1 and student 2:
```R
grade[1]          # Student 1 has grade "B"
grade[2]          # Student 2 has grade "A"

grade[1] < grade[2] 
```

**Console Output:**
```
[1] TRUE
```

**Why is it `TRUE`?**
1. `grade[1]` is `"B"`.
2. `grade[2]` is `"A"`.
3. In `levels = c("C", "B", "A")`, `"B"` comes **before** `"A"`.
4. Therefore, R evaluates `"B" < "A"` as **`TRUE`**!

> **Exam Tip:** If `ordered = FALSE` (default), R will throw a warning:  
> `Warning: '<' not meaningful for factors`  
> because standard factors only allow equality tests (`==`, `!=`), NOT inequality comparisons (`<`, `>`).


---

### 3.2 Common Functions Used with Factors

- `gl(n, k, labels = ...)`: **Generate factor levels** (very common short-answer exam question!).
  ```R
  # Generate 3 levels, each repeated 2 times
  gl(n = 3, k = 2, labels = c("Ctrl", "TreatA", "TreatB"))
  # Output: Ctrl Ctrl TreatA TreatA TreatB TreatB
  ```
- `cut(x, breaks, labels)`: Converts a continuous numeric vector into categorical factor bins.
  ```R
  ages <- c(12, 25, 45, 68, 18, 33)
  age_group <- cut(ages, breaks = c(0, 18, 60, 100), labels = c("Youth", "Adult", "Senior"))
  ```
- `split(x, f)`: Splits a vector or data frame into groups defined by factor `f`.
  ```R
  marks <- c(78, 92, 85, 64)
  sec <- factor(c("A", "B", "A", "B"))
  split(marks, sec)   # Returns a list of 2 vectors: $A and $B
  ```
- `droplevels(f)`: Drops unused factor levels after subsetting.

---

### 3.3 Working with Tables

Tables represent frequency and contingency counts.

```R
# 1. 1D Frequency Table
dept_vec <- c("HR", "IT", "IT", "Finance", "HR", "IT")
t1 <- table(dept_vec)
# Finance  HR  IT 
#       1   2   3

# 2. 2D Contingency Table
gender_vec <- c("M", "F", "M", "F", "F", "M")
t2 <- table(dept_vec, gender_vec)

# 3. Proportions table (Percentages)
prop.table(t1)          # Proportions summing to 1.0 across all cells
prop.table(t2, margin = 1) # Row proportions (each row sums to 1.0)
prop.table(t2, margin = 2) # Column proportions (each column sums to 1.0)

# 4. Margin sums and adding margins
margin.table(t2, 1)     # Row totals
margin.table(t2, 2)     # Column totals
addmargins(t2)          # Table with "Sum" row and column appended

# 5. Formula-based cross-tabulation: xtabs()
df <- data.frame(dept = dept_vec, gender = gender_vec)
xtabs(~ dept + gender, data = df)
```

---

## 4. Operators & Control Statements

### 4.1 Arithmetic & Boolean Operators

#### Arithmetic Operators
- `+`, `-`, `*`, `/` : Standard element-wise math
- `^` or `**` : Exponentiation ($2^3 = 8$)
- `%%` : **Modulo** (Remainder: `7 %% 3` = `1`)
- `%/%` : **Integer Division** (`7 %/% 3` = `2`)

#### Relational Operators
- `==`, `!=`, `<`, `<=`, `>`, `>=`

#### Logical / Boolean Operators: **`&` vs `&&`** (Crucial Exam Concept)

| Operator | Name | Vectorized? | Evaluates | Typical Use |
| :--- | :--- | :--- | :--- | :--- |
| `&` | Element-wise AND | **Yes** | All elements in both vectors | Subsetting data frames: `df[a > 5 & b < 2, ]` |
| `&&` | Short-circuit AND | **No (Scalar only)**| Only the **first** element; stops early if `FALSE` | `if (x > 0 && y > 0)` conditions |
| `\|` | Element-wise OR | **Yes** | All elements in both vectors | Vector filtering |
| `\|\|` | Short-circuit OR | **No (Scalar only)**| Only the **first** element; stops early if `TRUE` | `if (x < 0 \|\| y < 0)` conditions |
| `!` | Logical NOT | **Yes** | Inverts logical values | `!is.na(x)` |

#### Returning Boolean Values: Logical Predicates
```R
any(c(FALSE, FALSE, TRUE))   # TRUE (if at least one element is TRUE)
all(c(TRUE, TRUE, FALSE))    # FALSE (TRUE only if EVERY element is TRUE)
isTRUE(TRUE)                 # TRUE (strictly checks single logical TRUE)
identical(x, y)              # Exact, strict equality test
all.equal(x, y)              # Near-equality test (allowing floating point tolerance)
```

---

### 4.2 Control Statements

#### `if - else` and `ifelse()`
```R
# 1. Standard if-else (works on a single condition)
score <- 85
if (score >= 90) {
  grade <- "A+"
} else if (score >= 80) {
  grade <- "A"
} else {
  grade <- "B"
}

# 2. Vectorized ifelse(condition, yes, no) — Operates over full vectors!
marks_vec <- c(45, 88, 32, 75)
results <- ifelse(marks_vec >= 50, "Pass", "Fail")
# Output: c("Fail", "Pass", "Fail", "Pass")
```

#### `switch()` Statement
```R
calc <- function(a, b, op) {
  switch(op,
    "add" = a + b,
    "sub" = a - b,
    "mul" = a * b,
    "div" = a / b,
    stop("Unknown operation")
  )
}
calc(10, 5, "mul")  # Output: 50
```

#### Loops: `for`, `while`, `repeat`, `break`, `next`
```R
# 1. for loop
for (i in 1:5) {
  if (i == 3) next     # 'next' skips current iteration (like 'continue' in C/Java)
  if (i == 5) break    # 'break' terminates loop immediately
  cat(i, " ")
}
# Output: 1 2 4 

# 2. while loop
count <- 1
while (count <= 3) {
  print(count)
  count <- count + 1
}

# 3. repeat loop (Infinite loop until explicit break)
k <- 1
repeat {
  if (k > 3) break
  k <- k + 1
}
```

---

## 5. Functions as Objects & Core Concepts

In R, **functions are first-class objects** (citizens). This means:
1. Functions can be assigned to variables (`f <- function(...)`).
2. Functions can be passed as arguments to other functions (e.g., in `lapply`, `apply`).
3. Functions can be returned by other functions (function factories / closures).
4. Functions can be inspected and manipulated like any other R object.

### 5.1 The Three Components of a Function Object
Every standard R function consists of 3 parts:
```R
my_func <- function(x, y = 10) {
  z <- x + y
  return(z * 2)
}

formals(my_func)     # The formal arguments and default values: $x (none), $y = 10
body(my_func)        # The actual code block inside the function
environment(my_func) # The enclosing environment where the function was created
```

### 5.2 Default Argument Values & Lazy Evaluation
```R
# 1. Default argument values
greet <- function(name, greeting = "Hello") {
  paste(greeting, name)
}
greet("Kavya")             # "Hello Kavya"
greet("Kavya", "Welcome")  # "Welcome Kavya"

# 2. Lazy evaluation: Arguments are only evaluated when accessed!
test_lazy <- function(a, b) {
  return(a * 2)  # 'b' is NEVER evaluated, so passing an error to 'b' doesn't fail!
}
test_lazy(5, stop("Error!")) # Returns 10 without throwing an error!
```

### 5.3 Anonymous Functions & Tools for Composing Functions
```R
# Anonymous (unnamed) function passed inline:
sapply(1:5, function(x) x^3)

# Short syntax in R >= 4.1:
sapply(1:5, \(x) x^3)

# do.call(): Construct and execute a function call dynamically
args_list <- list(c(10, 20, 30), na.rm = TRUE)
do.call(mean, args_list)  # Equivalent to mean(c(10, 20, 30), na.rm = TRUE)
```

---

## 6. Scoping, "Writing Upstairs" & Advanced Functions

*(Topics emphasized directly from Norman Matloff's syllabus structure)*

### 6.1 Lexical Scoping & Environments
R uses **lexical scoping**: free variables inside a function are looked up in the environment **where the function was defined**, not where it is called.

```R
w <- 100
outer_fn <- function() {
  w <- 50
  inner_fn <- function() {
    print(w)  # Looks upstairs in outer_fn's environment first!
  }
  inner_fn()
}
outer_fn() # Prints 50, not 100
```

---

### 6.2 "Writing Upstairs" (The Super-Assignment Operator `<<-`)
- The standard assignment operator `<-` writes to the **local execution environment**.
- **Writing Upstairs** refers to modifying or creating a variable in a **parent or global environment** from inside a function.
- Accomplished using the **super-assignment operator** `<<-` or the `assign()` function.

```R
# Counter using Writing Upstairs (Closure)
counter_val <- 0

increment_counter <- function() {
  # counter_val <- counter_val + 1  # WRONG: would only create a local variable!
  counter_val <<- counter_val + 1   # "WRITING UPSTAIRS": Modifies parent counter_val!
  return(counter_val)
}

increment_counter()  # 1
increment_counter()  # 2
counter_val          # 2 (Global variable has changed)
```

> **Exam Definition:**  
> *"Writing upstairs in R refers to reassigning a variable in an enclosing or global environment rather than creating a local variable, achieved via the super-assignment operator `<<-`."*

---

### 6.3 Recursion
A function calling itself until reaching a base condition.

```R
# 1. Recursive Factorial
factorial_rec <- function(n) {
  if (n == 0 || n == 1) return(1)  # Base condition
  return(n * factorial_rec(n - 1)) # Recursive step
}
factorial_rec(5)  # 120

# 2. Recursive Fibonacci
fibonacci_rec <- function(n) {
  if (n <= 1) return(n)
  return(fibonacci_rec(n - 1) + fibonacci_rec(n - 2))
}
fibonacci_rec(7)  # 13
```

---

### 6.4 Replacement Functions
In R, syntactical constructs like `names(x) <- c("A", "B")` look like assignments, but are actually **Replacement Functions**.
- Their names always end with `<-`.
- Syntax convention: `"function_name<-" <- function(x, value) { ... return(x) }`

```R
# Creating a custom replacement function that replaces the first element of a vector:
"first<-" <- function(x, value) {
  x[1] <- value
  return(x)  # Must return the modified object!
}

vec <- c(10, 20, 30)
first(vec) <- 999  # Calls `first<-`(vec, 999) behind the scenes!
vec                # Output: c(999, 20, 30)
```

---

### 6.5 The Combine Function `c()`
- `c()` combines arguments into a vector or list.
- All elements coerced to a common type:  
  `Logical` $\rightarrow$ `Integer` $\rightarrow$ `Numeric` $\rightarrow$ `Character`
  ```R
  c(TRUE, 5, "Hello")  # Becomes character vector: c("TRUE", "5", "Hello")
  ```

---

## 7. Math and Simulations in R

### 7.1 Built-in Mathematical Functions

| Function | Operation | Example | Output |
| :--- | :--- | :--- | :--- |
| `abs(x)` | Absolute value | `abs(-15)` | `15` |
| `sqrt(x)` | Square root | `sqrt(49)` | `7` |
| `ceiling(x)` | Smallest integer $\ge x$ | `ceiling(4.2)` | `5` |
| `floor(x)` | Largest integer $\le x$ | `floor(4.8)` | `4` |
| `trunc(x)` | Truncate decimal part | `trunc(4.8)` | `4` |
| `round(x, digits)` | Round to $n$ digits | `round(3.14159, 2)`| `3.14` |
| `cumsum(x)` | Cumulative sum | `cumsum(c(1, 2, 3, 4))`| `c(1, 3, 6, 10)` |
| `diff(x)` | Lagged differences | `diff(c(2, 5, 9, 14))` | `c(3, 4, 5)` |

---

### 7.2 Probability Distributions: The 4 Prefix Rule

For any probability distribution in R (Normal, Binomial, Poisson, Uniform), there are 4 standard prefix functions:

| Prefix | Meaning | What it computes |
| :---: | :--- | :--- |
| **`d`** | **Density / PMF / PDF** | Value of probability density function $f(x)$ or point probability $P(X = x)$ |
| **`p`** | **Cumulative (CDF)** | Cumulative probability $P(X \le x)$ |
| **`q`** | **Quantile (Inverse CDF)** | Finds $x$ such that $P(X \le x) = p$ |
| **`r`** | **Random Generation** | Generates $n$ random observations |

#### Common Distributions
- **Normal:** `dnorm`, `pnorm`, `qnorm`, `rnorm(n, mean = 0, sd = 1)`
- **Uniform:** `dunif`, `punif`, `qunif`, `runif(n, min = 0, max = 1)`
- **Binomial:** `dbinom`, `pbinom`, `qbinom`, `rbinom(n, size, prob)`
- **Poisson:** `dpois`, `ppois`, `qpois`, `rpois(n, lambda)`

---

### 7.3 Random Sampling & Simulations

```R
# 1. set.seed() — Ensures reproducibility of pseudo-random numbers
set.seed(42)

# 2. sample(x, size, replace = FALSE/TRUE, prob = NULL)
# Rolling a fair 6-sided die 10 times (with replacement):
die_rolls <- sample(1:6, size = 10, replace = TRUE)

# Simulating tossing a biased coin (70% Heads, 30% Tails)
coin_tosses <- sample(c("H", "T"), size = 20, replace = TRUE, prob = c(0.7, 0.3))

# 3. Simple Monte Carlo Simulation: Estimating Probability of Sum > 8 with 2 Dice
sim_two_dice <- function(trials = 10000) {
  die1 <- sample(1:6, size = trials, replace = TRUE)
  die2 <- sample(1:6, size = trials, replace = TRUE)
  total <- die1 + die2
  prob_gt_8 <- mean(total > 8)  # Proportion where sum > 8
  return(prob_gt_8)
}

sim_two_dice() # Expected theoretical: 10/36 = 0.2777...
```

---

## 8. High-Yield Exam Trap Sheet

### Top 7 Rapid-Recall Distinctions

1. **`apply` vs `lapply` vs `sapply`**:
   - `apply`: Requires a `MARGIN` (1 for rows, 2 for cols); works on 2D arrays/matrices.
   - `lapply`: **L**ist apply — always returns a list.
   - `sapply`: **S**implified apply — turns list into a vector/matrix when possible.

2. **`&` vs `&&`**:
   - `&`: Element-wise AND across entire vectors (returns logical vector).
   - `&&`: Short-circuit scalar AND; only tests the 1st element (used in `if` statements).

3. **`<-` vs `<<-`**:
   - `<-`: Assigns variable in the **current local** scope.
   - `<<-`: "Writing upstairs" — searches enclosing scopes and assigns to parent/global variable.

4. **`table()` vs `prop.table()`**:
   - `table()` calculates raw counts.
   - `prop.table()` converts counts to relative frequencies/proportions (summing to 1).

5. **`levels()` vs `nlevels()`**:
   - `levels(f)` returns the character vector of categories.
   - `nlevels(f)` returns the integer count of categories.

6. **Replacement Function Signature**:
   - Must be named `"func<-"` and must return the modified object.

7. **`set.seed()`**:
   - Why use it? Generates identical sequence of pseudo-random numbers so results can be verified by exam evaluators.

---

### Quick 5-Minute Practice Code

```R
# Task: Create DF, add status factor, calculate group mean with tapply, and write upstairs
set.seed(123)

df <- data.frame(
  id = 1:6,
  score = c(45, 82, 67, 91, 53, 78)
)

# 1. Categorize using cut()
df$result <- cut(df$score, breaks = c(0, 50, 75, 100), labels = c("Fail", "Average", "Distinction"))

# 2. Group average using tapply()
group_avg <- tapply(df$score, df$result, mean)
print(group_avg)

# 3. Writing upstairs closure
total_calls <- 0
log_call <- function() {
  total_calls <<- total_calls + 1
  cat("Call count upstairs:", total_calls, "\n")
}
log_call()
log_call()
```
