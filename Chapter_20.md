---
title: "Chapter 20"
author: "Julin Maloof"
date: "2023-08-03"
output: 
  html_document: 
    keep_md: yes
---




```r
library(tidyverse)
```

```
## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
## ✔ dplyr     1.1.0     ✔ readr     2.1.4
## ✔ forcats   1.0.0     ✔ stringr   1.5.0
## ✔ ggplot2   3.4.1     ✔ tibble    3.1.8
## ✔ lubridate 1.9.2     ✔ tidyr     1.3.0
## ✔ purrr     1.0.1     
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
## ℹ Use the ]8;;http://conflicted.r-lib.org/conflicted package]8;; to force all conflicts to become errors
```

```r
library(rlang)
```

```
## 
## Attaching package: 'rlang'
## 
## The following objects are masked from 'package:purrr':
## 
##     %@%, flatten, flatten_chr, flatten_dbl, flatten_int, flatten_lgl,
##     flatten_raw, invoke, splice
```


Evaluation is the developer equivalent of unquotation...it allows the user to evaluate quoted expressions

Two concepts: _quosure_, captures an expression and its environment, and _data masking_, which allows ealuation in the context of a data frame.

quasiquotation, quosures, and data maksing make up _tidy evaluation_

## 20.2 evaluation basics

`eval` takes two arguments, an expression and an enviornment (calling environment is the default)


```r
x <- 10
eval(expr(x))
```

```
## [1] 10
```

```r
#> [1] 10

y <- 2
eval(expr(x + y))
```

```
## [1] 12
```

```r
#> [1] 12
#> 
#
```



```r
eval(expr(x + y), env(x = 1000))
```

```
## [1] 1002
```

```r
#> [1] 1002
```

### 20.2.1 applications (local)

Local allows you to create temporary variables:


```r
# Clean up variables created earlier
rm(x, y)

foo <- local({
  x <- 10
  y <- 200
  x + y
})

foo
```

```
## [1] 210
```

```r
#> [1] 210
x
```

```
## Error in eval(expr, envir, enclos): object 'x' not found
```

```r
#> Error in eval(expr, envir, enclos): object 'x' not found
y
```

```
## Error in eval(expr, envir, enclos): object 'y' not found
```

```r
#> Error in eval(expr, envir, enclos): object 'y' not found
```

capture expression, create new environment with calling environment as parent:


```r
local2 <- function(expr) {
  env <- env(caller_env())
  eval(enexpr(expr), env)
}

foo <- local2({
  x <- 10
  y <- 200
  x + y
})

foo
```

```
## [1] 210
```

```r
#> [1] 210
x
```

```
## Error in eval(expr, envir, enclos): object 'x' not found
```

```r
#> Error in eval(expr, envir, enclos): object 'x' not found
y
```

```
## Error in eval(expr, envir, enclos): object 'y' not found
```

```r
#> Error in eval(expr, envir, enclos): object 'y' not found
```

### 20.2.2 Application: source()


```r
source2 <- function(path, env = caller_env()) {
  file <- paste(readLines(path, warn = FALSE), collapse = "\n")
  exprs <- parse_exprs(file)

  res <- NULL
  for (i in seq_along(exprs)) {
    res <- eval(exprs[[i]], env)
  }

  invisible(res)
}
```

### 20.2.3 function()

beware if using this to create functions.  use `new_function()` or set the src_ref attribute to  NULL

### 20.2.4 Exercises

#### 1. Carefully read the documentation for source(). What environment does it use by default? What if you supply local = TRUE? How do you provide a custom environment?

default is global environment; local=TRUE os calling environment.  local=env sets environment to env

#### 2. Predict the results of the following lines of code:


```r
eval(expr(eval(expr(eval(expr(2 + 2)))))) # 4
```

```
## [1] 4
```

```r
eval(eval(expr(eval(expr(eval(expr(2 + 2))))))) # 4
```

```
## [1] 4
```

```r
expr(eval(expr(eval(expr(eval(expr(2 + 2))))))) # eval(expr(eval(expr(eval(expr(2 + 2))))))
```

```
## eval(expr(eval(expr(eval(expr(2 + 2))))))
```

#### 3. Fill in the function bodies below to re-implement get() using sym() and eval(), and assign() using sym(), expr(), and eval(). Don’t worry about the multiple ways of choosing an environment that get() and assign() support; assume that the user supplies it explicitly.


```r
# name is a string

myenv <- env(x=10,y=3)

get2 <- function(name, env) {
  name <- sym(name)
  eval(name, env=env)
}


get("y", myenv)
```

```
## [1] 3
```

```r
get2("y", myenv)
```

```
## [1] 3
```



```r
myenv <- env(x=10,y=3)


assign2 <- function(name, value, env) {
  name <- sym(name)
  #eval(expr(name <- value))
  eval(expr(!!name <- !!value), envir = env)
}

assign2("z", 5, myenv)

myenv$z
```

```
## [1] 5
```

#### 4 Modify source2() so it returns the result of every expression, not just the last one. Can you eliminate the for loop?


```r
writeLines("4+3
10*10
a <- 'apple'
toupper(a)",
           "sourceme20.R")

source2 <- function(path, env = caller_env()) {
  file <- paste(readLines(path, warn = FALSE), collapse = "\n")
  exprs <- parse_exprs(file)

  res <- NULL
  for (i in seq_along(exprs)) {
    res <- eval(exprs[[i]], env)
  }

  invisible(res)
}


source3 <- function(path, env = caller_env()) {
  file <- paste(readLines(path, warn = FALSE), collapse = "\n")
  exprs <- parse_exprs(file)

  res <- list()
  for (i in seq_along(exprs)) {
    res[[i]] <- eval(exprs[[i]], env)
  }

  invisible(res)
}

s1 <- source("sourceme20.R")
cat("S1\n")
```

```
## S1
```

```r
s1
```

```
## $value
## [1] "APPLE"
## 
## $visible
## [1] TRUE
```

```r
s2 <- source2("sourceme20.R")
cat("\n-------\nS2\n")
```

```
## 
## -------
## S2
```

```r
s2
```

```
## [1] "APPLE"
```

```r
s3 <- source3("sourceme20.R")
cat("\n-------\nS3\n")
```

```
## 
## -------
## S3
```

```r
s3
```

```
## [[1]]
## [1] 7
## 
## [[2]]
## [1] 100
## 
## [[3]]
## [1] "apple"
## 
## [[4]]
## [1] "APPLE"
```

### 20.3 Quosures

Contain both an expression and an environment

Use `enquo` and `enquos` to capture user provided expressions


```r
foo <- function(x) enquo(x)
foo(a + b)
```

```
## <quosure>
## expr: ^a + b
## env:  global
```

Or `quo` and `quos` but this isn't recommended

Or `new_quosure`


```r
new_quosure(expr(x + y), env(x = 1, y = 10))
```

```
## <quosure>
## expr: ^x + y
## env:  0x1209ee510
```

Evaluate with `eval_tidy`


```r
q1 <- new_quosure(expr(x + y), env(x = 1, y = 10))
eval_tidy(q1)
```

```
## [1] 11
```

### 20.3.6 Exercises

#### 1. Predict what each of the following quosures will return if evaluated.


```r
q1 <- new_quosure(expr(x), env(x = 1))
q1
```

```
## <quosure>
## expr: ^x
## env:  0x12142bb30
```

```r
eval_tidy(q1)
```

```
## [1] 1
```

```r
## 1

q2 <- new_quosure(expr(x + !!q1), env(x = 10))
q2
```

```
## <quosure>
## expr: ^x + (^x)
## env:  0x1215ec070
```

```r
eval_tidy(q2)
```

```
## [1] 11
```

```r
## 11

q3 <- new_quosure(expr(x + !!q2), env(x = 100))
q3
```

```
## <quosure>
## expr: ^x + (^x + (^x))
## env:  0x121996940
```

```r
eval_tidy(q3)
```

```
## [1] 111
```

```r
## 111
```


#### 2. Write an enenv() function that captures the environment associated with an argument. (Hint: this should only require two function calls.)


```r
enenv <- function(x) {
  get_env(enquo(x))
}

enenv(10)
```

```
## <environment: R_EmptyEnv>
```

```r
Z <- 10

enenv(z)
```

```
## <environment: R_GlobalEnv>
```

## 20.4

data masks

a data fram argument provided to tidy_eval where variables are looked for.

If neded, can specify .data$x and .env$x to be explicit about where variables should be looked for

### Exercises

#### 1. Why did I use a for loop in transform2() instead of map()? Consider `transform2(df, x = x * 2, x = x * 2).`

Because map will create a list with 2 columns of the same name, and they will not be evaluated recursively (for map, both operations work on the original x, I think)

#### 2. Compare subset2 and 3


```r
subset2 <- function(data, rows) {
  rows <- enquo(rows)
  rows_val <- eval_tidy(rows, data)
  stopifnot(is.logical(rows_val))

  data[rows_val, , drop = FALSE]
}

subset3 <- function(data, rows) {
  rows <- enquo(rows)
  eval_tidy(expr(data[!!rows, , drop = FALSE]), data = data)
}

df <- data.frame(x = 1:3)
subset3(df, x == 1)
```

```
##   x
## 1 1
```

I would think that these would both work equally well.  subset2 is clearer coding and also will maybe give a better error message.  Also can get into trouble if the df has a column "data"

#### 3. The following function implements the basics of dplyr::arrange(). Annotate each line with a comment explaining what it does. Can you explain why !!.na.last is strictly correct, but omitting the !! is unlikely to cause problems?


```r
arrange2 <- function(.df, ..., .na.last = TRUE) {
  args <- enquos(...) ## get the ... expressions and quote them

  order_call <- expr(order(!!!args, na.last = !!.na.last)) ## screate an expression that will call the function "order" with the arugments (which will be column names)

  ord <- eval_tidy(order_call, .df) # now run the order call, using the data frame as the environment
  stopifnot(length(ord) == nrow(.df)) # make sure nothing went wrong

  .df[ord, , drop = FALSE] # actually reorder the df and return it
}
```

!!.na.last will give the value of na.last as an argument in the expression, but without this it will be evaluated later and this should be ok becuase na.last will be in the calling environment.

## 20.5

#### 1. I’ve included an alternative implementation of threshold_var() below. What makes it different to the approach I used above? What makes it harder?


```r
threshold_var <- function(df, var, val) {
  var <- ensym(var)
  subset2(df, `$`(.data, !!var) >= !!val)
}
```


Here we are using the inilne version of the `$` function, and that makes the code harder to read.

## 20.6

#### 1. Why does this function fail?


```r
lm3a <- function(formula, data) {
  formula <- enexpr(formula)

  lm_call <- expr(lm(!!formula, data = data))
  eval(lm_call, caller_env())
}
lm3a(mpg ~ disp, mtcars)$call
```

It fails because `data` does not exist in `caller_env()`.  So we need to quote it and then unquote it in the call, as is done in the `lm3` given in the book

#### 2. When model building, typically the response and data are relatively constant while you rapidly experiment with different predictors. Write a small wrapper that allows you to reduce duplication in the code below.


```r
lm(mpg ~ disp, data = mtcars)
```

```
## 
## Call:
## lm(formula = mpg ~ disp, data = mtcars)
## 
## Coefficients:
## (Intercept)         disp  
##    29.59985     -0.04122
```

```r
lm(mpg ~ I(1 / disp), data = mtcars)
```

```
## 
## Call:
## lm(formula = mpg ~ I(1/disp), data = mtcars)
## 
## Coefficients:
## (Intercept)    I(1/disp)  
##       10.75      1557.67
```

```r
lm(mpg ~ disp * cyl, data = mtcars)
```

```
## 
## Call:
## lm(formula = mpg ~ disp * cyl, data = mtcars)
## 
## Coefficients:
## (Intercept)         disp          cyl     disp:cyl  
##    49.03721     -0.14553     -3.40524      0.01585
```

What is the idea?  give the formulas in ... and loop through them?

A second approach would be to have one arugment for the response, and have the predictors in ...

Start with idea one.


```r
lms_1 <- function(..., data, env=caller_env()) {
  formulas <- enexprs(...)
  
  data <- enexpr(data)
  
  res <- list()
  
  for(i in seq_along(formulas)) {
    lm_call <- expr(lm(!!formulas[[i]], data=!!data))
    expr_print(lm_call)
    res[[i]] <- eval(lm_call, env=env)
    cat("----------------\n\n")
  }
  
  res
  
}

lms_1(mpg ~ disp, mpg ~ I(1/disp), mpg ~ disp * cyl, data=mtcars)
```

```
## lm(mpg ~ disp, data = mtcars)
## ----------------
## 
## lm(mpg ~ I(1 / disp), data = mtcars)
## ----------------
## 
## lm(mpg ~ disp * cyl, data = mtcars)
## ----------------
```

```
## [[1]]
## 
## Call:
## lm(formula = mpg ~ disp, data = mtcars)
## 
## Coefficients:
## (Intercept)         disp  
##    29.59985     -0.04122  
## 
## 
## [[2]]
## 
## Call:
## lm(formula = mpg ~ I(1/disp), data = mtcars)
## 
## Coefficients:
## (Intercept)    I(1/disp)  
##       10.75      1557.67  
## 
## 
## [[3]]
## 
## Call:
## lm(formula = mpg ~ disp * cyl, data = mtcars)
## 
## Coefficients:
## (Intercept)         disp          cyl     disp:cyl  
##    49.03721     -0.14553     -3.40524      0.01585
```

How about a list of predictors?


```r
lms_2 <- function(response, ..., data, env=caller_env()) {
  response <- enexpr(response)
  
  predictors <- enexprs(...)
  
  data <- enexpr(data)
  
  res <- list()
  
  for(i in seq_along(predictors)) {
    lm_call <- expr(lm(!!response ~ !!predictors[[i]], data=!!data))
    expr_print(lm_call)
    res[[i]] <- eval(lm_call, env=env)
    cat("----------------\n\n")
  }
  
  res
  
}

lms_2(mpg, disp, I(1/disp), disp * cyl, data=mtcars)
```

```
## lm(mpg ~ disp, data = mtcars)
## ----------------
## 
## lm(mpg ~ I(1 / disp), data = mtcars)
## ----------------
## 
## lm(mpg ~ disp * cyl, data = mtcars)
## ----------------
```

```
## [[1]]
## 
## Call:
## lm(formula = mpg ~ disp, data = mtcars)
## 
## Coefficients:
## (Intercept)         disp  
##    29.59985     -0.04122  
## 
## 
## [[2]]
## 
## Call:
## lm(formula = mpg ~ I(1/disp), data = mtcars)
## 
## Coefficients:
## (Intercept)    I(1/disp)  
##       10.75      1557.67  
## 
## 
## [[3]]
## 
## Call:
## lm(formula = mpg ~ disp * cyl, data = mtcars)
## 
## Coefficients:
## (Intercept)         disp          cyl     disp:cyl  
##    49.03721     -0.14553     -3.40524      0.01585
```

#### 3. Another way to write resample_lm() would be to include the resample expression (data[sample(nrow(data), replace = TRUE), , drop = FALSE]) in the data argument. Implement that approach. What are the advantages? What are the disadvantages?

advantage: user control.  disadvantage: user has to rewrite it if there df is not called "data".  Uglier.  lm output is uglier.


```r
resample_lm3 <- function(formula, data=data[sample(nrow(data), replace = TRUE), , drop = FALSE], env = caller_env()) {
  formula <- enexpr(formula)
  data = enexpr(data)
  
  lm_call <- expr(lm(!!formula, data = !!data))
  expr_print(lm_call)
  eval(lm_call, env)
}

df <- data.frame(x = 1:10, y = 5 + 3 * (1:10) + round(rnorm(10), 2))

resample_lm3(y ~ x, data=df[sample(nrow(df), replace = TRUE), , drop = FALSE])
```

```
## lm(y ~ x, data = df[sample(nrow(df), replace = TRUE), , drop = FALSE])
```

```
## 
## Call:
## lm(formula = y ~ x, data = df[sample(nrow(df), replace = TRUE), 
##     , drop = FALSE])
## 
## Coefficients:
## (Intercept)            x  
##       2.770        3.327
```

