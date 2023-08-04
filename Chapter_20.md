---
title: "Chapter 20"
author: "Julin Maloof"
date: "2023-08-03"
output: 
  html_document: 
    keep_md: yes
---




```r
library(rlang)
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
## env:  0x1458cdf38
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
## env:  0x1269342b0
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
## env:  0x126ad0588
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
## env:  0x126dc99a8
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

