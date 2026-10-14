# Logistic Regression — Class Notes & Derivations

> **Source Material:** [Logistic Regression.pdf](docs/Logistic%20Regression.pdf)  
> **Scope:** Strictly covers the concepts, mathematical derivations, and numerical problems presented in the PDF notes.

---

## 1. Introduction & Motivation

- **Limitation of Linear Regression:**
  - Linear regression predicts a **continuous numerical response**.
  - It is **not suitable** for predicting categorical variables because predicted values can fall below 0 or exceed 1.
- **Classification Problem:**
  - When categorical variables are involved, the problem is termed a **classification problem**.
- **Role of Logistic Regression (LR):**
  - Logistic Regression is suitable for **binary classification problems** (responses with two possible outcomes).
  - *Example:* Is an email **Spam** or **Not Spam**? (**Yes / No**).

---

## 2. Working of the Algorithm & The Sigmoid Curve

- **Scenario:**
  - Predicting whether an employee will get a promotion / salary hike based on their performance rating.
- **Why a Linear Graph Fails:**
  - A straight regression line extends infinitely in both directions ($-\infty$ to $+\infty$), making it invalid for probabilities.
- **Transition to Sigmoid Curve:**
  - We clip the linear prediction at 0 and 1 by transforming it into a **Sigmoid curve (S-curve)**.
  - The S-curve ensures that all outputs strictly lie in the interval `[0, 1]`.
- **Threshold Rule:**
  - A decision threshold (commonly **threshold = 0.5**) is used to assign binary classes:
    - If `Probability >= 0.5` → **Class 1 (Yes / Promotion)**
    - If `Probability < 0.5` → **Class 0 (No / No Promotion)**

```
Probability (p)
    1.0 ┼─────────────────────────╭───────
        │                        ╭╯
   0.75 ┼                       ╭╯
        │                      ╭╯
    0.5 ┼─────────────────────╭╯   (Threshold = 0.5)
        │                   ╭╯
   0.25 ┼                 ╭╯
        │               ╭╯
      0 ┼───────╭───────╯─────────────────
        └───────┬───────┬───────┬───────┬───> Employee Rating (x)
                1       2       3       4
```

---

## 3. Mathematical Foundations: Odds & Log-Odds

### 3.1 Odds of Success ($\Omega$)
$$\text{Odds} = \frac{\text{Probability of an event happening } (p)}{\text{Probability of an event not happening } (1 - p)}$$

$$\text{Odds} = \frac{p}{1 - p}$$

* **Range of Odds:** Extends from **$0$ to $\infty$**.
* **Range of Probability ($p$):** Bound strictly between **$0$ and $1$**.

---

### 3.2 Setting Up the Logistic Model
In Linear Regression, the equation is:
$$y = \beta_0 + \beta_1 x$$
Where:
- $\beta_0$ = Intercept
- $\beta_1$ = Coefficient of regression

In Logistic Regression, we model the **logarithm of the odds** (the **Logit** function) as a linear combination:
$$\log\left(\frac{p(x)}{1 - p(x)}\right) = \beta_0 + \beta_1 x$$

---

## 4. Complete Step-by-Step Derivation of the Sigmoid Function

Starting from the log-odds equation:
$$\ln\left(\frac{p(x)}{1 - p(x)}\right) = \beta_0 + \beta_1 x$$

### Step 1: Exponentiate both sides
$$e^{\ln\left(\frac{p(x)}{1 - p(x)}\right)} = e^{\beta_0 + \beta_1 x}$$

Using the identity $e^{\ln(x)} = x$:
$$\frac{p(x)}{1 - p(x)} = e^{\beta_0 + \beta_1 x}$$

---

### Step 2: Algebraic Simplification
Let $y = e^{\beta_0 + \beta_1 x}$. The equation becomes:
$$\frac{p(x)}{1 - p(x)} = y$$

Multiply both sides by $(1 - p(x))$:
$$p(x) = y(1 - p(x))$$

Expand the right-hand side:
$$p(x) = y - y \cdot p(x)$$

Group all terms containing $p(x)$ on the left-hand side:
$$p(x) + y \cdot p(x) = y$$

Factor out $p(x)$:
$$p(x)(1 + y) = y$$

Solve for $p(x)$:
$$p(x) = \frac{y}{1 + y}$$

---

### Step 3: Substitute $y$ back
Substitute $y = e^{\beta_0 + \beta_1 x}$:
$$p(x) = \frac{e^{\beta_0 + \beta_1 x}}{1 + e^{\beta_0 + \beta_1 x}}$$

---

### Step 4: Divide Numerator and Denominator by $e^{\beta_0 + \beta_1 x}$
$$\frac{\frac{e^{\beta_0 + \beta_1 x}}{e^{\beta_0 + \beta_1 x}}}{\frac{1 + e^{\beta_0 + \beta_1 x}}{e^{\beta_0 + \beta_1 x}}} = \frac{1}{\frac{1}{e^{\beta_0 + \beta_1 x}} + 1}$$

Using the index rule $\frac{1}{e^u} = e^{-u}$:
$$p(x) = \frac{1}{1 + e^{-(\beta_0 + \beta_1 x)}}$$

This is the standard **Sigmoid / Logistic Function**:
$$S(x) = \frac{1}{1 + e^{-x}}$$

---

## 5. Numerical Problems (From PDF)

### Problem 1: Entrance Exam Selection

* **Context:** A student dataset has entrance marks based on past history of selection (selected vs. not selected).
* **Given Parameters:**
  - Learned parameters: $\beta_0 + \beta_1 x$ evaluated for score $x = 60$.
  - Formula:
    $$p(x) = \frac{1}{1 + e^{-(\beta_0 + \beta_1 x)}}$$
* **Calculation:**
  - Evaluated probability:
    $$p(x) = 0.44$$
* **Classification Decision:**
  - Threshold ($\text{th}$) = $0.5$
  - Since $0.44 < 0.5$:
  - **Resultant Class:** **$0$ (Not Selected)**.

---

### Problem 2: Pass / Fail Classification Based on Study Hours

#### Dataset:
| Hours of Study ($x$) | Result: Pass (1) / Fail (0) |
| :---: | :---: |
| 29 | 0 |
| 15 | 0 |
| 33 | 1 |
| 28 | 1 |
| 39 | 1 |

* **Model provided by optimizer for log-odds of passing:**
  $$\log(\text{odds}) = z = -64 + 2 \times \text{hours}$$
* **Sigmoid Function:**
  $$p = S(z) = \frac{1}{1 + e^{-z}}$$

---

#### Question 1:
**Calculate the probability of passing for a student who studies 33 hours.**

**Solution:**
1. Calculate $z$ for $\text{hours} = 33$:
   $$z = -64 + 2(33)$$
   $$z = -64 + 66 = 2$$

2. Calculate probability $p$:
   $$p = \frac{1}{1 + e^{-z}} = \frac{1}{1 + e^{-2}}$$

3. Evaluate $e^{-2}$:
   $$e^{-2} \approx 0.1353$$
   $$p = \frac{1}{1 + 0.1353} = \frac{1}{1.1353} \approx 0.88$$

* **Conclusion:**  
  There is an **88% chance** ($p = 0.88$) that a student who studies 33 hours will pass the examination.

---

#### Question 2:
**At least how many hours should a student study to pass the course with a probability of more than 95% ($p = 0.95$)?**

**Solution:**
1. Set probability $p = 0.95$:
   $$p = \frac{1}{1 + e^{-z}} = 0.95$$

2. Solve for $e^{-z}$:
   $$0.95 \times (1 + e^{-z}) = 1$$
   $$0.95 + 0.95 e^{-z} = 1$$
   $$0.95 e^{-z} = 1 - 0.95 = 0.05$$
   $$e^{-z} = \frac{0.05}{0.95} = 0.0526$$

3. Take natural logarithm ($\ln$) on both sides:
   $$\ln(e^{-z}) = \ln(0.0526)$$
   *(Property: $\ln(e^x) = x$)*
   $$-z = \ln(0.0526) = -2.94$$
   $$z = 2.94$$

4. Substitute $z = 2.94$ into the model equation ($z = -64 + 2 \times \text{hours}$):
   $$2.94 = -64 + 2 \times \text{hours}$$
   $$2 \times \text{hours} = 2.94 + 64$$
   $$2 \times \text{hours} = 66.94$$
   $$\text{hours} = \frac{66.94}{2} = 33.47 \text{ hours}$$

* **Conclusion:**  
  The student should study at least **33.47 hours** to achieve a passing probability greater than 95%.
