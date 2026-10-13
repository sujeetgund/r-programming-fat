# Module 4: OOP, I/O, Strings & Graphics in R — Exam Revision Guide

> **Target:** R Programming Term-End Exam (FAT)  
> **Key Focus:** Crisp syntax, conceptual clarity, comparison tables, and code snippets ready to write on paper or in lab.

---

## Table of Contents
1. [S3 vs S4 Object-Oriented Programming (OOP)](#1-s3-and-s4-classes)
2. [Managing Your Objects & Memory](#2-managing-your-objects)
3. [Input / Output (I/O) & Web Access](#3-input--output-io)
4. [String Manipulation & Regular Expressions](#4-string-manipulation)
5. [Graphics: Creating, Customizing & Exporting](#5-r-graphics)
6. [Top Exam Questions & Quick-Revision Cheatsheet](#6-high-yield-exam-cheat-sheet)

---

## 1. S3 and S4 Classes

R is primarily functional, but supports multiple OOP paradigms. S3 and S4 are the two most tested systems.

### 1.1 Quick Comparison Table (High-Yield Exam Question)

| Feature | S3 Class | S4 Class |
| :--- | :--- | :--- |
| **Philosophy** | Informal, ad-hoc, quick | Formal, strict, type-safe |
| **Class Definition** | No formal definition (`class(x) <- "name"`) | Formal definition via `setClass()` |
| **Object Instantiation** | Created using lists/vectors and assigning class | Created via `new("ClassName", ...)` |
| **Field/Member Access**| Dollar sign `$` or double bracket `[[ ]]` | `@` operator or `slot()` function |
| **Method Dispatch** | `UseMethod("generic_name")` | `setMethod("generic_name", signature, ...)` |
| **Validation** | No built-in validation | Validation via `validity` argument in `setClass()` |
| **Inheritance** | Set vector of class names (e.g., `class(x) <- c("sub", "base")`) | Explicit via `contains` parameter in `setClass()` |

---

### 1.2 S3 Classes (Informal OOP)

- S3 relies on an **attribute** named `class`.
- Functions inspect the class of the first argument and dispatch using `UseMethod()`.

#### Step 1: Create an S3 Object
```R
# An S3 object is just a base object (usually a list) with a class attribute
student <- list(name = "Rahul", reg_no = "21BCE0001", marks = c(85, 90, 78))
class(student) <- "Student"

# Alternative shortcut constructor:
create_student <- function(name, reg_no, marks) {
  obj <- list(name = name, reg_no = reg_no, marks = marks)
  class(obj) <- "Student"
  return(obj)
}
s1 <- create_student("Priya", "21BCE0002", c(92, 88, 95))
```

#### Step 2: Define a Generic Function & Methods
Method naming follows the convention: `<generic_name>.<class_name>`

```R
# 1. Define the generic function
display_info <- function(x) {
  UseMethod("display_info")
}

# 2. Define the specific method for "Student" class
display_info.Student <- function(x) {
  cat("Student Name:", x$name, "\n")
  cat("Reg Number:  ", x$reg_no, "\n")
  cat("Avg Marks:   ", mean(x$marks), "\n")
}

# 3. Define a default fallback method
display_info.default <- function(x) {
  cat("Generic object with no custom display.\n")
}

# Call the method
display_info(s1)
```

#### Step 3: Overriding Standard Base Generics (`print`, `summary`)
```R
print.Student <- function(x) {
  cat("Student:", x$name, "| Reg:", x$reg_no, "\n")
}

# Now simply typing s1 will call print.Student(s1)
print(s1)
```

---

### 1.3 S4 Classes (Formal OOP)

Requires strict declarations using the `methods` package (loaded by default in R).

#### Step 1: Define the Class (`setClass`)
```R
setClass(
  Class = "Employee",
  slots = c(
    name = "character",
    emp_id = "numeric",
    salary = "numeric"
  ),
  prototype = list(
    name = "Unknown",
    emp_id = 0,
    salary = 30000
  ),
  validity = function(object) {
    if (object@salary < 0) {
      return("Salary cannot be negative!")
    }
    return(TRUE)
  }
)
```

#### Step 2: Create an Instance (`new`)
```R
emp1 <- new("Employee", name = "Arun", emp_id = 101, salary = 55000)

# Accessing slots (Use @ or slot())
emp1@name              # Output: "Arun"
slot(emp1, "salary")   # Output: 55000

# Modifying slots
emp1@salary <- 60000
```

#### Step 3: Create Generics & Methods (`setGeneric`, `setMethod`)
```R
# 1. Create a generic
setGeneric("calculateBonus", function(object) {
  standardGeneric("calculateBonus")
})

# 2. Implement method for Employee
setMethod("calculateBonus", signature = "Employee", function(object) {
  bonus <- object@salary * 0.10
  cat("Bonus for", object@name, "is Rs.", bonus, "\n")
  return(bonus)
})

calculateBonus(emp1)
```

#### Step 4: S4 Inheritance (`contains`)
```R
setClass(
  "Manager",
  slots = c(department = "character"),
  contains = "Employee"   # Inherits all slots from Employee
)

mgr1 <- new("Manager", name = "Sneha", emp_id = 102, salary = 95000, department = "IT")
mgr1@name        # Inherited slot
mgr1@department  # Subclass slot
```

---

## 2. Managing Your Objects

### 2.1 Workspace and Environment Functions

| Command | Purpose | Example |
| :--- | :--- | :--- |
| `ls()` / `objects()` | List all active variables/functions | `ls()` |
| `ls(pattern = "^temp")` | List objects matching regex pattern | `ls(pattern = "^df")` |
| `rm(x)` | Remove a specific object | `rm(s1)` |
| `rm(list = ls())` | **Clear entire workspace** (Memory wipe) | `rm(list = ls())` |
| `exists("x")` | Check if variable exists (returns boolean) | `exists("emp1")` |
| `object.size(x)` | View memory consumed by an object | `object.size(emp1)` |
| `gc()` | Garbage collector (frees RAM) | `gc()` |

### 2.2 Saving and Loading Objects

```R
# Method A: Save specific objects into .RData file
save(emp1, s1, file = "my_objects.RData")
load("my_objects.RData")

# Method B: Save the ENTIRE active workspace
save.image(file = "full_workspace.RData")
load("full_workspace.RData")

# Method C: Save single object to serialized RDS (best practice)
saveRDS(emp1, file = "employee.rds")
restored_emp <- readRDS("employee.rds")  # Can assign to any new variable name
```

### 2.3 Search Path & Environments
```R
search()         # Lists attached packages and environments
attach(mtcars)   # Allows direct column access (e.g., mpg instead of mtcars$mpg)
detach(mtcars)   # Always detach after use to prevent namespace collisions
```

---

## 3. Input / Output (I/O)

### 3.1 Accessing Keyboard and Monitor

#### Reading from Keyboard
```R
# 1. readline() — Best for single string / interactive prompts
user_name <- readline(prompt = "Enter your name: ")
age <- as.numeric(readline(prompt = "Enter your age: "))

# 2. scan() — Best for reading numeric vectors directly from console
# Press Enter twice to finish input
numbers <- scan()
```

#### Output to Monitor
```R
# 1. print() — Prints object representation with index prefixes ([1])
print("Hello World")

# 2. cat() — Concatenates and prints raw text (no [1]), supports \n, sep
cat("Hello", user_name, "your age is", age, "\n", sep = " ")

# 3. message() — Sends diagnostic messages to stderr (useful in functions)
message("Processing completed successfully.")

# 4. sprintf() — C-style formatted string output
sprintf("Pi to 3 decimal places: %.3f", pi)
```

---

### 3.2 Reading and Writing Files

#### Tabular Data (CSV, TXT)
```R
# Reading CSV
df <- read.csv("students.csv", header = TRUE, stringsAsFactors = FALSE)

# Writing CSV (Always set row.names = FALSE to avoid extra unnamed column)
write.csv(df, file = "output.csv", row.names = FALSE)

# Generic table reader (for TSV, custom delimiters)
df_tsv <- read.table("data.txt", header = TRUE, sep = "\t")
write.table(df, file = "data_out.txt", sep = ",", quote = FALSE)
```

#### Text Lines (`readLines`, `writeLines`)
```R
# Reading text file line-by-line into a character vector
lines <- readLines("notes.txt", n = 10)  # Read first 10 lines

# Writing character vector to file
writeLines(c("Line 1: Introduction", "Line 2: Analysis"), con = "summary.txt")
```

---

### 3.3 Accessing the Internet

```R
# 1. Reading CSV directly from a URL
url_link <- "https://people.sc.fsu.edu/~jburkardt/data/csv/hw_200.csv"
web_data <- read.csv(url_link)

# 2. Downloading files from the web to local storage
download.file(
  url = "https://raw.githubusercontent.com/datasets/gdp/master/data/gdp.csv",
  destfile = "gdp_data.csv",
  method = "auto"
)

# 3. Reading webpage lines using url() connection
con <- url("https://www.google.com")
page_content <- readLines(con, n = 5)
close(con)
```

---

## 4. String Manipulation

R has powerful built-in text manipulation functions.

### 4.1 Fundamental String Functions

| Function | Description | Example | Output |
| :--- | :--- | :--- | :--- |
| `nchar(x)` | Counts number of characters | `nchar("Exam FAT")` | `8` |
| `paste(...)` | Joins strings with a space default `sep` | `paste("R", "Lab", sep = "-")` | `"R-Lab"` |
| `paste0(...)` | Joins strings with `sep = ""` (no space) | `paste0("Code", 101)` | `"Code101"` |
| `substr(x, start, stop)` | Extracts substring by index | `substr("VITChennai", 1, 3)` | `"VIT"` |
| `substring(x, first, last)` | Similar to `substr`, vector-friendly | `substring("Hello", 2, 4)` | `"ell"` |
| `tolower(x)` | Converts to lowercase | `tolower("DATA")` | `"data"` |
| `toupper(x)` | Converts to uppercase | `toupper("data")` | `"DATA"` |
| `strsplit(x, split)` | Splits string into a list of substrings | `strsplit("A,B,C", split = ",")`| `list(c("A","B","C"))` |

> **Key Distinction:**  
> `paste("A", "B", "C", collapse = "-")` collapses a vector into a single string.  
> `paste("A", "B", sep = "-")` joins corresponding elements across arguments.

---

### 4.2 Regular Expressions & Pattern Matching

| Function | What it does | Return Value | Exam Example |
| :--- | :--- | :--- | :--- |
| `grep(pattern, x)` | Finds matching indices (or values if `value=TRUE`) | Integer index vector | `grep("apple", c("apple", "banana"))` -> `1` |
| `grepl(pattern, x)` | Logical test if pattern exists | Boolean vector (`TRUE`/`FALSE`) | `grepl("cat", c("caterpillar", "dog"))` -> `c(TRUE, FALSE)` |
| `sub(pattern, replacement, x)` | Replaces **first** occurrence | Character vector | `sub("a", "@", "banana")` -> `"b@nana"` |
| `gsub(pattern, replacement, x)` | Replaces **ALL** occurrences | Character vector | `gsub("a", "@", "banana")` -> `"b@n@n@"` |
| `regexpr(pattern, x)` | Finds position & length of first match | Match details vector | `regexpr("[0-9]+", "ID: 450")` |

#### Regular Expression Syntax Cheat Sheet
- `^` : Start of string (`"^A"` = starts with A)
- `$` : End of string (`"z$"` = ends with z)
- `.` : Any single character
- `[0-9]` or `\\d` : Digit
- `[a-zA-Z]` : Alphabets
- `+` : 1 or more occurrences
- `*` : 0 or more occurrences

---

## 5. R Graphics

Base R includes a powerful, flexible graphics engine.

### 5.1 Primary High-Level Plotting Functions

High-level functions start a brand-new plot window.

| Function | Type of Graph | Typical Use Case |
| :--- | :--- | :--- |
| `plot(x, y)` | Scatter plot (or line plot) | Numerical vs Numerical |
| `barplot(height)` | Bar chart | Categorical counts / frequencies |
| `hist(x)` | Histogram | Distribution of a continuous variable |
| `boxplot(x ~ group)` | Box & Whisker plot | Spread, median & outliers across groups |
| `pie(x)` | Pie chart | Relative proportions (use sparingly) |
| `pairs(data)` | Scatterplot matrix | Correlation exploration across multiple columns |

#### Common Graph Examples
```R
# 1. Scatter Plot
x <- 1:10
y <- x^2
plot(x, y, type = "b", main = "Quadratic Growth", xlab = "X-axis", ylab = "Y-axis")
# type options: "p" (points), "l" (lines), "b" (both), "o" (overplotted), "h" (histogram-like)

# 2. Histogram
hist(mtcars$mpg, breaks = 6, col = "skyblue", main = "Distribution of MPG", xlab = "MPG")

# 3. Boxplot
boxplot(mpg ~ cyl, data = mtcars, col = c("gold", "lightgreen", "coral"),
        main = "MPG by Cylinder Count", xlab = "Cylinders", ylab = "MPG")

# 4. Barplot
counts <- table(mtcars$gear)
barplot(counts, col = "steelblue", main = "Gear Frequency", xlab = "Gears", ylab = "Count")
```

---

### 5.2 Customizing Graphs (Parameters & Low-Level Additions)

#### Core Graphical Parameters (`par()` or inside plot function)
- `main` : Main title of the chart
- `sub` : Sub-title below the chart
- `xlab`, `ylab` : X and Y axis labels
- `xlim`, `ylim` : Axis limits, e.g., `xlim = c(0, 100)`
- `col` : Color (e.g., `"red"`, `"#336699"`, `col = 1:4`)
- `pch` : Plotting character symbol (numbers 0 to 25; 16 = filled circle, 19 = solid circle, 17 = triangle)
- `cex` : Character expansion / scaling factor (`cex = 1.5` makes points 50% larger)
- `lty` : Line type (`1` = solid, `2` = dashed, `3` = dotted)
- `lwd` : Line width (`lwd = 2` = double thickness)

#### Low-Level Plotting Functions (Layering over an existing plot)
Do **NOT** clear the existing plot; they add annotations on top:
```R
# Base plot
plot(mtcars$wt, mtcars$mpg, pch = 19, col = "darkgray",
     main = "Car Weight vs MPG", xlab = "Weight (1000 lbs)", ylab = "MPG")

# 1. Add trend line / reference line
abline(lm(mpg ~ wt, data = mtcars), col = "red", lwd = 2, lty = 2) # Linear regression line
abline(h = mean(mtcars$mpg), col = "blue", lty = 3)                # Horizontal line at mean

# 2. Add custom points or lines
points(x = 3.5, y = 20, col = "purple", pch = 17, cex = 2)

# 3. Add text annotation
text(x = 4.5, y = 30, labels = "Important Zone", col = "darkgreen")

# 4. Add legend
legend("topright", legend = c("Data Points", "Regression Line", "Mean MPG"),
       col = c("darkgray", "red", "blue"), pch = c(19, NA, NA), lty = c(NA, 2, 3))

# 5. Add grid
grid(nx = NULL, ny = NULL, col = "lightgray", lty = "dotted")
```

#### Multi-Panel Plots with `par()`
```R
# Arrange multiple plots in 2 rows and 2 columns
par(mfrow = c(2, 2))  # Filled row-wise (use mfcol for column-wise)

plot(mtcars$mpg)
hist(mtcars$wt)
boxplot(mtcars$hp)
barplot(table(mtcars$cyl))

# Reset to single window afterwards!
par(mfrow = c(1, 1))
```

---

### 5.3 Saving Graphs to Files (Graphics Devices)

The golden 3-step workflow tested in almost every practical / written exam:
1. **Open Device:** Call `png()`, `pdf()`, `jpeg()`, etc.
2. **Execute Plotting Code:** Run your `plot()`, `hist()`, customizations.
3. **Close Device:** Call `dev.off()` (Flushes output buffer and saves the file to disk).

#### Examples:
```R
# Example 1: Saving as High-Resolution PNG
png(filename = "scatter_plot.png", width = 800, height = 600, res = 120)
plot(mtcars$wt, mtcars$mpg, col = "blue", pch = 16, main = "Weight vs MPG")
abline(lm(mpg ~ wt, data = mtcars), col = "red")
dev.off() # MANDATORY! If omitted, file will be 0 KB / corrupted.

# Example 2: Saving as Multi-Page Vector PDF
pdf(file = "analysis_report.pdf", width = 7, height = 5)
plot(mtcars$mpg, main = "Page 1: MPG Plot")
hist(mtcars$hp, main = "Page 2: HP Histogram")
dev.off()
```

---

## 6. High-Yield Exam Cheat Sheet

### Common Viva & Written Traps
1. **`sub()` vs `gsub()`**:
   - `sub()` replaces **only the first** match.
   - `gsub()` replaces **all** matches ("global substitute").
2. **`paste()` vs `paste0()`**:
   - `paste(a, b)` has `sep = " "` by default.
   - `paste0(a, b)` has `sep = ""` by default.
3. **`print()` vs `cat()`**:
   - `print()` returns the object invisibly and outputs format with `[1]`.
   - `cat()` converts arguments to character, concatenates them, interprets `\n`, prints without index brackets, and returns `NULL`.
4. **S3 vs S4 Member Access**:
   - S3 uses `$` or `[[ ]]`.
   - S4 uses `@` or `slot()`.
5. **Graphics Device Closure**:
   - Why is `dev.off()` needed? It tells R that plotting is finished, writes the final headers/EOF bytes, and frees the file lock.
6. **`type` argument in `plot()`**:
   - `"p"`: Points
   - `"l"`: Lines
   - `"b"`: Both points and lines
   - `"o"`: Overplotted points and lines
   - `"h"`: Histogram-like vertical lines
   - `"n"`: No plotting (creates empty canvas with coordinate axes)

---

### Quick 5-Minute Practice Code

```R
# 1. Create S3 Class with Custom Print
make_book <- function(title, price) {
  obj <- list(title = title, price = price)
  class(obj) <- "Book"
  obj
}
print.Book <- function(x) {
  cat("Book:", x$title, "| Price: $", x$price, "\n")
}
b <- make_book("R for Data Science", 45)
print(b)

# 2. String Cleaning in one pipe/sequence
raw_text <- "  apple, banana, cherry  "
clean_vec <- unlist(strsplit(gsub(" ", "", raw_text), ","))
clean_vec  # c("apple", "banana", "cherry")

# 3. Export a 2-in-1 Boxplot to JPEG
jpeg("cars_summary.jpg", width = 600, height = 300)
par(mfrow = c(1, 2))
boxplot(mtcars$mpg, main = "MPG Distribution", col = "cyan")
boxplot(mtcars$hp, main = "HP Distribution", col = "salmon")
dev.off()
```
