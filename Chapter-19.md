---
title: "Chapter 19"
author: "Julin Maloof"
date: "2023-07-20"
output: 
  html_document: 
    keep_md: yes
---




```r
library(rlang)
library(purrr)
```

```
## 
## Attaching package: 'purrr'
```

```
## The following objects are masked from 'package:rlang':
## 
##     %@%, flatten, flatten_chr, flatten_dbl, flatten_int, flatten_lgl,
##     flatten_raw, invoke, splice
```


# Quasiquotation

## 19.1

quotation is the act of capturing an unevaluated expression

unquotation is the ability to selectively unquote parts of a quoted expression.

Together these are _quasiquoation_.  Quasiquotation enables combining code inherent to the function with that provided by the user.

## 19.2 Motivation

This is a pain:


```r
paste("Good", "morning", "Hadley")
```

```
## [1] "Good morning Hadley"
```

```r
paste("Good", "afternoon", "Alice")
```

```
## [1] "Good afternoon Alice"
```

Nicer to do it without typing all of the quotes


```r
cement <- function(...) {
  args <- ensyms(...)
  paste(purrr::map(args, as_string), collapse = " ")
}

cement(Good, morning, Hadley)
```

```
## [1] "Good morning Hadley"
```

```r
#> [1] "Good morning Hadley"
cement(Good, afternoon, Alice)
```

```
## [1] "Good afternoon Alice"
```

```r
#> [1] "Good afternoon Alice"
```

But what if we want to do it with objects? 


```r
#works with paste:
name <- "Hadley"
time <- "morning"

paste("Good", time, name)
```

```
## [1] "Good morning Hadley"
```

```r
#but not with cement:
cement(Good, time, name)
```

```
## [1] "Good time name"
```

We can solve this by using !! to drop the implicit quotes:


```r
cement(Good, !!time, !!name)
```

```
## [1] "Good morning Hadley"
```

Comparing cement and paste.  paste we have to quote literals.  Cement we have to unquote variable.


```r
paste("Good", time, name)
```

```
## [1] "Good morning Hadley"
```

```r
cement(Good, !!time, !!name)
```

```
## [1] "Good morning Hadley"
```

### 19.2.1 Vocabulary

Distinction between evaluated and quoted arguements.

_evaluated_ follows R's normal evaluation rules
_quoted_ are captured expressions that are treated in some custom way.

paste evaluates all of its arguements, whereas cement quotes all of its arguments.

### 19.2.2 Exercises

#### 1. For each function, identify which arguments are quoted and which are evaluated


```r
library(MASS) # quoted

mtcars2 <- subset(mtcars, cyl == 4) # first evaluated, second quoted

with(mtcars2, sum(vs)) #first is evaluated, second is quoted
```

```
## [1] 10
```

```r
sum(mtcars2$am) #evaluated
```

```
## [1] 8
```

```r
rm(mtcars2) #evaluated(?)
```

#### 2. For each function in the following tidyverse code, identify which arguments are quoted and which are evaluated.


```r
library(dplyr) # quoted
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following object is masked from 'package:MASS':
## 
##     select
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(ggplot2) # quoted

by_cyl <- mtcars %>%
  group_by(cyl) %>% #quoted
  summarise(mean = mean(mpg)) #quoted

ggplot(by_cyl, aes(cyl, mean)) + geom_point() # by_cyl evaluated, cyl, mean quoted
```

![](Chapter-19_files/figure-html/unnamed-chunk-8-1.png)<!-- -->


## 19.3 Quoting

Capturing an expression without evaluating it

Need a pair of functions because the expression can be supplied directly or indirectly via lazily-evaluated function argument.  (Hopefully there is an example...)

### 19.3.1 Capturing expressions

There are four important quoting functions. For interactive use, `expr()` is the most important. It captures its argument exactly as provided.


```r
expr(x + y)
```

```
## x + y
```

```r
expr(1 / 2 / 3)
```

```
## 1/2/3
```

But not great inside a function because it captures what the function developer typed.


```r
f1 <- function(x) expr(x)
f1(a + b + c)
```

```
## x
```

```r
#> x
```

To solve this problem we use another function, `enexpr()`.  This captures what the caller supplied to the. function.


```r
f2 <- function(x) enexpr(x)
f2(a + b + c)
```

```
## a + b + c
```

expr and exprs for capturing expressions we type

enexpr and enexprs for capturing expressions that the function caller types

### 19.3.2 capturing symbols

`ensym` and `ensyms`: similar to `enexpr`, but through an error if given anything byt a symbol (or string)


```r
f <- function(...) ensyms(...)
f(x)
```

```
## [[1]]
## x
```

```r
f("x")
```

```
## [[1]]
## x
```

```r
f(2+3)
```

```
## Error in `sym()`:
## ! Can't convert a call to a symbol.
```

### 19.3.3 with base R

The base R functions do not support unquoting

base equivalent of `expr` is `quote`

base equivalent of `enexpr` is `substitute`

base equivalent of `exprs` is `alist`

also note that `~` is a quoting function that captures the environment.

### 19.3.4 substitution

in addition to quoting, `substitute` will substitute in the values of symbols in the current environment

### 18.3.6 Exercises:

#### 1. how is expr() implemented?


```r
expr
```

```
## function (expr) 
## {
##     enexpr(expr)
## }
## <bytecode: 0x1274da870>
## <environment: namespace:rlang>
```

#### 2 Compare and contrast the following two functions. Can you predict the output before running them?


```r
f1 <- function(x, y) {
  exprs(x = x, y = y)
}
f2 <- function(x, y) {
  enexprs(x = x, y = y)
}
f1(a + b, c + d) # --> "x=x, y=y"
```

```
## $x
## x
## 
## $y
## y
```

```r
f2(a + b, c + d) # -->
```

```
## $x
## a + b
## 
## $y
## c + d
```

```r
#[x]
#a + b
#[y]
#c+d
```

#### 3 What happens if you try to use enexpr() with an expression (i.e.  enexpr(x + y) ? What happens if enexpr() is passed a missing argument?


```r
enexpr(x + y)
```

```
## Error in `enexpr()`:
## ! `arg` must be a symbol
```

```r
enexpr(NA)
```

```
## Error in `enexpr()`:
## ! `arg` must be a symbol
```

Throws errors

#### 4. How are exprs(a) and exprs(a = ) different? Think about both the input and the output.

First one should capture "a".  Second one is setting "a" as the name for the first item in the list.


```r
exprs(a)
```

```
## [[1]]
## a
```

```r
exprs(a=c+d)
```

```
## $a
## c + d
```

#### 5. What are other differences between exprs() and alist()? Read the documentation for the named arguments of exprs() to find out.


```r
?exprs
```

options for whether or not names are assigned to the unamed arguments, what to do with empty arguments, and something about unquoting names that I don't understand.

#### 6. The documentation for substitute() says:

Substitution takes place by examining each component of the parse tree as follows:

* If it is not a bound symbol in env, it is unchanged.
* If it is a promise object (i.e., a formal argument to a function) the expression slot of the promise replaces the symbol.
* If it is an ordinary variable, its value is substituted, unless env is .GlobalEnv in which case the symbol is left unchanged.

Create examples that illustrate each of the above cases.


```r
# 1
x <- 5
substitute(x+y*2, list(y=4))
```

```
## x + 4 * 2
```

```r
#x not substituted, y substituted
```



```r
#2
f6_2 <- function(x)
  substitute(x+y*2)
f6_2(x=a)
```

```
## a + y * 2
```



```r
#3
f6_3 <- function(y) {
  x <- 5 
  substitute(x+y*2)
}
f6_3(y=4)
```

```
## 5 + 4 * 2
```

```r
#compare with # 1
```

