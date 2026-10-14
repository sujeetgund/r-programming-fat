# R Programming — Final Assessment Test (FAT) Revision Notes

Comprehensive, exam-oriented revision notes and code snippets for R Programming term-end examinations.

## 📚 Syllabus Modules

- **[Module 1: Introduction to R & Vectors](module1.md)**
  - Introducing to R & Data Structures
  - Help Functions (`?`, `??`, `args`, `example`)
  - Vectors, Scalars (Length 1 Vectors), Declarations
  - Recycling Rule & Common Vector Operations
  - `all()` and `any()`
  - Vectorized Operations
  - `NA` vs `NULL`
  - Filtering & `which()`
  - Vectorized if-then-else (`ifelse()`)
  - Vector Equality (`==`, `identical()`, `all.equal()`)
  - Vector Element Names

- **[Module 2: Matrices, Arrays and Lists](module2.md)**
  - Creating Matrices (Column-Major Storage, `byrow`)
  - Matrix Operations (Element-wise vs `%*%`, `solve()`, `t()`, `det()`)
  - Applying Functions to Rows & Columns (`apply()`, `rowSums()`, `colMeans()`)
  - Adding and Deleting Rows & Columns (`rbind()`, `cbind()`)
  - Vector/Matrix Distinction (`dim` attribute)
  - Avoiding Dimension Reduction (`drop = FALSE`)
  - Higher-Dimensional Arrays (`array()`)
  - Lists & General Operations
  - Accessing Components (`[` vs `[[` vs `$`)
  - Applying Functions to Lists (`lapply()`, `sapply()`)
  - Recursive Lists & `unlist()`

- **[Module 3: Data Frames, Factors, Tables, Functions & Simulations](module3.md)**
  - Creating Data Frames & Matrix-like Subsetting
  - Merging Data Frames (`merge()`: Inner, Outer, Left, Right joins)
  - The `apply()` Family (`apply()`, `lapply()`, `sapply()`, `tapply()`)
  - Factors and Levels (`gl()`, `cut()`, `split()`, Ordered factors)
  - Tables & Contingency Counts (`table()`, `prop.table()`, `addmargins()`)
  - Control Statements & Operators (`&` vs `&&`, `%%`, `%/%`, `switch`)
  - Functions as First-Class Objects (`formals()`, `body()`, `environment()`)
  - Lexical Scoping & "Writing Upstairs" (`<<-`)
  - Recursion & Replacement Functions (`"func<-"`)
  - Math Functions & Statistical Simulations (`rnorm`, `sample`, `set.seed`)

- **[Module 4: S3 & S4 Classes, I/O, Strings & Graphics](module4.md)**
  - S3 Classes (Informal OOP, `UseMethod()`, generic dispatch)
  - S4 Classes (Formal OOP, `setClass()`, `new()`, `setGeneric()`, `setMethod()`, `@` slot access)
  - Managing Objects (`ls()`, `rm()`, `save()`, `load()`, `saveRDS()`, `readRDS()`)
  - Input/Output: Keyboard (`readline()`, `scan()`) & Monitor (`cat()`, `print()`)
  - Reading and Writing Files (`read.csv()`, `write.csv()`, `readLines()`, `writeLines()`)
  - Accessing the Internet (`download.file()`, URL connections)
  - String Manipulation (`nchar()`, `paste()`, `paste0()`, `substr()`, regex: `grep()`, `grepl()`, `sub()`, `gsub()`)
  - Graphics (Plotting, customization parameters, low-level annotations, `dev.off()`)

- **[Module 5: Interfacing R, Parallelism & Statistical Modeling](module5.md)**
  - Interfacing R to Other Languages (`.C()`, `.Call()`, `Rcpp`, `reticulate`)
  - Parallel R (`parallel` package, `detectCores()`, `makeCluster()`, `parLapply()`)
  - Basic Statistics & Hypothesis Testing (`t.test()`, `chisq.test()`, `aov()`)
  - Linear Models (`lm()`, regression diagnostics, formula syntax)
  - Generalized Linear Models (`glm()`, Logistic & Poisson regression, link functions)
  - Non-linear Models (`nls()`, parameter starting values, polynomial models)
  - Time Series & Autocorrelation (`ts()`, `decompose()`, ACF, PACF)
  - Clustering (K-Means with `kmeans()`, Hierarchical with `hclust()`, `cutree()`)

## 📝 Lecture & Solved Problem Notes

- **[Logistic Regression Notes](logistic_regression_notes.md)**
  - Linear vs. Logistic Regression & Binary Classification
  - Sigmoid / S-curve & Decision Thresholds
  - Derivation of Odds, Log-Odds & Sigmoid Formula
  - Solved Numerical Problems (Entrance Exam Selection & Pass/Fail Study Hours)

- **[K-Means, K-Medoids, Covariance & Hierarchical Clustering Notes](kmeans_and_other_notes.md)**
  - Covariance (Sample vs. Population, Solved Stock Problem & Matrix)
  - Correlation Coefficient ($r$), Scale, Properties & Limitations
  - Linear Regression by Least Squares (Formulas for $a$ & $b$, Solved 4-point problem)
  - K-Means Clustering (Complete 3-Cluster Solved Iterations & Convergence)
  - K-Medoids Clustering / PAM (Manhattan Distance, Medoid Swapping & Cost)
  - Agglomerative Hierarchical Clustering (Single-Link Distance Matrix Updates & Dendrogram)
  - K-Means vs. K-Medoids (Centroid vs. Medoid, Outlier Robustness, Advantages & Disadvantages)
