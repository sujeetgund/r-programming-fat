# Module 2: Matrices, Arrays and Lists — Exam Revision Guide

> **Target:** R Programming Term-End Exam (FAT)  
> **Key References:** Base R Mechanics & *The Art of R Programming* (Chapters 3 & 4) by Norman Matloff  
> **Exam Strategy:** Direct answers for short notes, syntax templates, comparison tables, and code snippets ready to write on paper or run in a lab exam.

---

## Table of Contents
1. [Creating Matrices & Column-Major Order](#1-creating-matrices)
2. [Matrix Operations (Arithmetic vs Matrix Math)](#2-matrix-operations)
3. [Applying Functions to Rows and Columns](#3-applying-functions-to-matrix-rows-and-columns)
4. [Adding and Deleting Rows & Columns](#4-adding-and-deleting-rows-and-columns)
5. [The Vector/Matrix Distinction](#5-the-vectormatrix-distinction)
6. [Avoiding Dimension Reduction (`drop = FALSE`)](#6-avoiding-dimension-reduction-drop--false)
7. [Higher-Dimensional Arrays](#7-higher-dimensional-arrays)
8. [Lists: Creation & General Operations](#8-lists-creation--general-operations)
9. [Accessing List Components: `[` vs `[[` vs `$`](#9-accessing-list-components--vs--vs-)
10. [Applying Functions to Lists (`lapply` & `sapply`)](#10-applying-functions-to-lists)
11. [Recursive (Nested) Lists & `unlist()`](#11-recursive-nested-lists)
12. [High-Yield Exam Trap Sheet & Frequently Asked Questions](#12-high-yield-exam-trap-sheet)

---

## 1. Creating Matrices

A **Matrix** is a 2D data structure in R that is **homogeneous** (all elements must share the exact same atomic type, typically numeric).

### 1.1 Column-Major Storage (Default)
By default, R stores and populates matrices **column-by-column** (Column-Major order), unless `byrow = TRUE` is explicitly specified.

```R
# 1. Default Column-by-Column (byrow = FALSE)
m1 <- matrix(1:6, nrow = 2, ncol = 3)
print(m1)
```

**Console Output:**
```
     [,1] [,2] [,3]
[1,]    1    3    5
[2,]    2    4    6
```

```R
# 2. Row-by-Row population (byrow = TRUE)
m2 <- matrix(1:6, nrow = 2, ncol = 3, byrow = TRUE)
print(m2)
```

**Console Output:**
```
     [,1] [,2] [,3]
[1,]    1    2    3
[2,]    4    5    6
```

### 1.2 Matrix with Named Dimensions (`dimnames`)
```R
m_named <- matrix(
  c(85, 90, 78, 88), 
  nrow = 2, 
  dimnames = list(c("Student1", "Student2"), c("Math", "Physics"))
)
print(m_named)
```

**Console Output:**
```
         Math Physics
Student1   85      78
Student2   90      88
```

---

## 2. Matrix Operations

### 2.1 Element-wise vs Mathematical Matrix Operations (Top Exam Question)

| Operation | R Syntax | Mathematical Meaning | Requirements |
| :--- | :--- | :--- | :--- |
| **Element-wise Multiplication** | `A * B` | Multiplies $a_{ij} \times b_{ij}$ | $A$ and $B$ must have identical dimensions |
| **Matrix Multiplication** | `A %*% B` | Dot product of rows of $A$ and columns of $B$ | Columns of $A$ == Rows of $B$ |
| **Transpose** | `t(A)` | Flips rows and columns ($A^T$) | Any matrix |
| **Determinant** | `det(A)` | Evaluates scalar $|A|$ | Must be square matrix ($n \times n$) |
| **Matrix Inverse** | `solve(A)` | Finds $A^{-1}$ such that $A A^{-1} = I$ | Must be square and non-singular ($\det \ne 0$) |
| **Linear System Solver** | `solve(A, b)` | Solves linear equation system $A x = b$ | $A$ square, $b$ matching vector |
| **Diagonal Extraction / Creation**| `diag(A)` / `diag(v)` | Extracts diagonal or builds diagonal matrix | Square matrix or vector |
| **Eigenvalues / Eigenvectors** | `eigen(A)` | Computes spectrum $\lambda$ and vectors $v$ | Square matrix |

---

### 2.2 Concrete Code Examples & Outputs

```R
A <- matrix(c(1, 2, 3, 4), nrow = 2, byrow = TRUE)
B <- matrix(c(5, 6, 7, 8), nrow = 2, byrow = TRUE)
```

#### Matrix A and Matrix B:
$$\mathbf{A} = \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix}, \quad \mathbf{B} = \begin{bmatrix} 5 & 6 \\ 7 & 8 \end{bmatrix}$$

#### 1. Element-wise Multiplication (`A * B`)
```R
print(A * B)
```
**Console Output:**
```
     [,1] [,2]
[1,]    5   12
[2,]   21   32
```
> *(Row 1: $1 \times 5 = 5$, $2 \times 6 = 12$; Row 2: $3 \times 7 = 21$, $4 \times 8 = 32$)*

---

#### 2. True Matrix Multiplication (`A %*% B`)
```R
print(A %*% B)
```
**Console Output:**
```
     [,1] [,2]
[1,]   19   22
[2,]   43   50
```
> *Calculation: Row 1 Col 1 = $(1 \times 5) + (2 \times 7) = 5 + 14 = 19$.*

---

#### 3. Matrix Inverse & Solving Systems (`solve()`)
```R
# Matrix Inverse
inv_A <- solve(A)
print(inv_A)
```
**Console Output:**
```
     [,1] [,2]
[1,] -2.0  1.0
[2,]  1.5 -0.5
```

```R
# Solving system: 1x + 2y = 8  and  3x + 4y = 18
b <- c(8, 18)
solution <- solve(A, b)
print(solution)
```
**Console Output:**
```
[1] 2 3
```
> *(Solution: $x = 2, y = 3$)*

---

## 3. Applying Functions to Matrix Rows and Columns

### 3.1 Using `apply()`
- `MARGIN = 1` $\rightarrow$ Evaluates along **Rows**.
- `MARGIN = 2` $\rightarrow$ Evaluates along **Columns**.

```R
M <- matrix(1:9, nrow = 3, byrow = TRUE)
#      [,1] [,2] [,3]
# [1,]    1    2    3
# [2,]    4    5    6
# [3,]    7    8    9

# Row sums using apply:
apply(M, 1, sum)  # [1]  6 15 24

# Column means using apply:
apply(M, 2, mean) # [1] 4 5 6
```

### 3.2 High-Performance Built-in Row/Col Functions
R provides specialized, pre-compiled C functions that run significantly faster than `apply`:
```R
rowSums(M)   # [1]  6 15 24
colSums(M)   # [1] 12 15 18
rowMeans(M)  # [1] 2 5 8
colMeans(M)  # [1] 4 5 6
```

---

## 4. Adding and Deleting Rows and Columns

### 4.1 Binding Rows (`rbind`) and Columns (`cbind`)
```R
mat <- matrix(1:4, nrow = 2)
#      [,1] [,2]
# [1,]    1    3
# [2,]    2    4

# Add a new row at the bottom
mat_row <- rbind(mat, c(10, 20))

# Add a new column to the right
mat_col <- cbind(mat, c(100, 200))
```

### 4.2 Deleting Rows and Columns via Negative Indexing
```R
# Delete row 1:
mat[-1, ]

# Delete column 2:
mat[, -2]
```

---

## 5. The Vector/Matrix Distinction

> **Exam Key Concept (Norman Matloff):** In R, **a matrix is NOT a separate primitive data type**. A matrix is simply an atomic vector that has a `dim` (dimension) attribute!

### 5.1 Verification
```R
x <- 1:6
is.matrix(x)   # FALSE

# Convert vector to matrix simply by adding dim attribute:
dim(x) <- c(2, 3)
is.matrix(x)   # TRUE
attributes(x)  # $dim: [1] 2 3
```

### 5.2 Flattening a Matrix back to a Vector
```R
as.vector(x)   # [1] 1 2 3 4 5 6
c(x)           # [1] 1 2 3 4 5 6
```

---

## 6. Avoiding Dimension Reduction (`drop = FALSE`)

### 6.1 The Problem: Automatic Dimension Reduction
In R, when you extract a single row or single column from a matrix, R **automatically collapses (reduces)** it into a 1D vector. This can cause subtle bugs inside loops or functions that expect a 2D matrix structure.

```R
M <- matrix(1:6, nrow = 2, ncol = 3)

# Extract row 1:
sub1 <- M[1, ]
class(sub1)    # "integer" (Vector)
is.matrix(sub1)# FALSE  <-- Dimension was reduced!
dim(sub1)      # NULL
```

### 6.2 The Solution: `drop = FALSE`
To preserve the 2D matrix structure when subsetting a single row or column, pass `drop = FALSE`:

```R
sub2 <- M[1, , drop = FALSE]
class(sub2)    # "matrix" "array"
is.matrix(sub2)# TRUE   <-- Preserved as a 1 x 3 matrix!
dim(sub2)      # [1] 1 3
```

| Syntax | Return Type | `dim()` | Use Case |
| :--- | :--- | :--- | :--- |
| `M[1, ]` | Atomic Vector | `NULL` | When you just need raw values |
| `M[1, , drop = FALSE]` | 2D Matrix | `c(1, 3)` | When passing to functions requiring matrix input |

---

## 7. Higher-Dimensional Arrays

An **Array** is the generalized $n$-dimensional extension of matrices.

### 7.1 Creating an Array
`array(data, dim = c(nrow, ncol, nlayers))`

```R
# Creating two 2x3 tables stacked in 3 dimensions:
arr <- array(1:12, dim = c(2, 3, 2))
print(arr)
```

**Console Output:**
```
, , 1

     [,1] [,2] [,3]
[1,]    1    3    5
[2,]    2    4    6

, , 2

     [,1] [,2] [,3]
[1,]    7    9   11
[2,]    8   10   12
```

### 7.2 Subsetting 3D Arrays
`arr[row_index, col_index, layer_index]`

```R
arr[1, 2, 1]   # Row 1, Col 2, Layer 1 -> 3
arr[, , 2]     # Complete 2nd layer matrix
```

---

## 8. Lists: Creation & General Operations

A **List** is R's most flexible data structure:
- **Heterogeneous:** Can contain elements of different types and different lengths.
- Can store numbers, strings, vectors, matrices, data frames, and even functions.

### 8.1 Creating a List
```R
student_record <- list(
  name = "Kavya",
  age = 20,
  scores = c(88, 92, 95),
  passed = TRUE
)
```

### 8.2 General List Operations
```R
length(student_record)    # 4 (Number of top-level components)
names(student_record)     # c("name", "age", "scores", "passed")

# 1. Add a new component
student_record$city <- "Chennai"

# 2. Delete a component (Assign NULL)
student_record$passed <- NULL

# 3. Concatenate two lists using c()
extra_info <- list(gpa = 9.1, major = "CSE")
full_profile <- c(student_record, extra_info)
```

---

## 9. Accessing List Components: `[` vs `[[` vs `$`

> **High-Yield Exam Question:** Explain the difference between single brackets `[ ]` and double brackets `[[ ]]` when accessing lists.

### 9.1 The Analogy
* **`[` (Single Bracket):** Like cutting out a slice of a train. It returns a **smaller list containing the element**.
* **`[[` (Double Bracket):** Like opening the cargo container and unpacking what's inside. It returns the **actual object** (vector, numeric, etc.).
* **`$` (Dollar sign):** Shorthand for `[[ ]]` when accessing elements by name.

---

### 9.2 Code Demonstration & Output

```R
my_list <- list(
  name = "Rahul",
  marks = c(85, 90, 95)
)
```

#### 1. Single Bracket `my_list[2]`
```R
res1 <- my_list[2]
class(res1)
print(res1)
```
**Console Output:**
```
$marks
[1] 85 90 95

class: "list"
```
> *`res1` is STILL a list of length 1. You cannot do `mean(res1)` because it is wrapped inside a list!*

---

#### 2. Double Bracket `my_list[[2]]`
```R
res2 <- my_list[[2]]
class(res2)
print(res2)
```
**Console Output:**
```
[1] 85 90 95

class: "numeric"
```
> *`res2` is the actual numeric vector! Now `mean(res2)` works perfectly (returns 90).*

---

#### 3. Dollar Sign `my_list$marks`
```R
my_list$marks  # Equivalent to my_list[["marks"]]
```

### Summary Comparison Table:
| Syntax | What it returns | Can run vector functions on it directly? |
| :--- | :--- | :--- |
| `my_list[1]` | A sub-list of length 1 | **No** (throws type error) |
| `my_list[[1]]` | The underlying object stored inside | **Yes** |
| `my_list$name` | The underlying object by name | **Yes** |

---

## 10. Applying Functions to Lists

### 10.1 `lapply()` vs `sapply()`

```R
exam_scores <- list(
  Alice = c(80, 90, 85),
  Bob = c(70, 75, 80),
  Charlie = c(95, 92, 98)
)

# lapply: Returns results as a list
lap_out <- lapply(exam_scores, mean)
print(lap_out)
```

**Console Output:**
```
$Alice
[1] 85

$Bob
[1] 75

$Charlie
[1] 95
```

```R
# sapply: Simplifies results into a named vector
sap_out <- sapply(exam_scores, mean)
print(sap_out)
```

**Console Output:**
```
  Alice     Bob Charlie 
     85      75      95 
```

---

## 11. Recursive (Nested) Lists

A **Recursive List** is a list that contains other lists as elements.

### 11.1 Creating and Accessing Nested Lists
```R
nested_org <- list(
  dept = "Computer Science",
  head = "Dr. Sharma",
  teams = list(
    ai_team = c("Aman", "Pooja"),
    web_team = c("Vikram", "Tara")
  )
)

# Accessing nested elements:
nested_org$teams$ai_team        # [1] "Aman" "Pooja"
nested_org[["teams"]][["ai_team"]][1] # [1] "Aman"
```

### 11.2 Flattening with `unlist()`
`unlist()` takes a nested/recursive list and converts it into a single flat atomic vector:

```R
flat_teams <- unlist(nested_org$teams)
print(flat_teams)
```

**Console Output:**
```
ai_team1 ai_team2 web_team1 web_team2 
  "Aman"  "Pooja"  "Vikram"    "Tara" 
```

---

## 12. High-Yield Exam Trap Sheet

1. **`A * B` vs `A %*% B`**:
   - `*` is element-wise multiplication.
   - `%*%` is matrix multiplication ($O(n^3)$ dot products).
2. **Matrix Storage Default**:
   - R stores matrices in **column-major order** (`byrow = FALSE` by default).
3. **Why use `drop = FALSE`?**:
   - Prevents R from automatically coercing a 1-row or 1-column matrix slice into a 1D vector.
4. **`[` vs `[[` on Lists**:
   - `[` extracts a list slice (type remains `list`).
   - `[[` extracts the raw underlying object inside.
5. **Deleting elements in a list**:
   - `lst$item <- NULL` removes the element completely and reduces list length.
6. **Matrix is a Vector**:
   - A matrix in R is simply an atomic vector with a `dim` attribute.
