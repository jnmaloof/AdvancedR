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

### 19.3.6 Exercises:

#### 1. how is expr() implemented?


```r
expr
```

```
## function (expr) 
## {
##     enexpr(expr)
## }
## <bytecode: 0x1151521d0>
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

substitute(x+y*2, .GlobalEnv)
```

```
## x + y * 2
```

```r
substitute(x+y*2, list(y=4, x=x))
```

```
## 5 + 4 * 2
```

```r
substitute(x+y*2, as.list(.GlobalEnv))
```

```
## 5 + y * 2
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

f6_3 <- function(y) {
  x <- 5 
  substitute(x+y*2, .GlobalEnv)
}
f6_3(y=4)
```

```
## x + y * 2
```

## 19.4

### 19.4.1 unquote one argument

Use `!!`

### 19.4.2 unquoting a function

You can also use `!!` but pay attention to operatior precedence.  You need an extra set of parentheses


```r
# correct way to only unquote f
x <- 1
y <- 2
f <- expr(foo)
expr((!!f)(x, y))
```

```
## foo(x, y)
```

```r
#> foo(x, y)
#> 
#> #compare to
 expr(!!f(x,y)) # error!
```

```
## Error in f(x, y): could not find function "f"
```

Works when f is a call but perhaps cleaner to use rlang::call2


```r
f <- expr(pkg::foo)
expr((!!f)(x, y))
```

```
## pkg::foo(x, y)
```

```r
#> pkg::foo(x, y)


call2(f, expr(x), expr(y))
```

```
## pkg::foo(x, y)
```

### 19.4.3 unquoting a missing argument


```r
arg <- missing_arg()
expr(foo(!!arg, !!arg))
```

```
## Error in enexpr(expr): argument "arg" is missing, with no default
```

```r
#> Error in enexpr(expr): argument "arg" is missing, with no default
```

Instead

```r
expr(foo(!!maybe_missing(arg), !!maybe_missing(arg)))
```

```
## foo(, )
```

### 19.4.4 Unquoting in special forms

infix function `$` causes problems.  Workaround is to use the pre-fix version:


```r
x <- expr(x)
expr(`$`(df, !!x))
```

```
## df$x
```

```r
#> df$x
```

### 19.4.5 Unquoting many arguments

Use `!!!`, takes a list of expressions, pastes them together and inserts them at the !!!


```r
xs <- exprs(1, a, -b)
expr(f(!!!xs, y))
```

```
## f(1, a, -b, y)
```

```r
#> f(1, a, -b, y)

# Or with names
ys <- set_names(xs, c("a", "b", "c"))
expr(f(!!!ys, d = 4))
```

```
## f(a = 1, b = a, c = -b, d = 4)
```

```r
#> f(a = 1, b = a, c = -b, d = 4)
```

`!!!` can be used anywhere that `...` is accepted.  Helpful in `call2`


```r
call2("f", !!!xs, expr(y))
```

```
## f(1, a, -b, y)
```

```r
#> f(1, a, -b, y)
```

### 19.4.6 fiction of !!

These arent' real operators.  Can only use in Rlang aware situations

### 19.4.7 misleading ASTs

beware inline functions.  expr_print or lobstr::ast can help resolve confusion.

### 19.4.8 Exercises

Given the following components:


```r
xy <- expr(x + y)
xz <- expr(x + z)
yz <- expr(y + z)
abc <- exprs(a, b, c)
```

Use quasiquotation to construct the following calls:



```r
(x + y) / (y + z)
-(x + z) ^ (y + z)
(x + y) + (y + z) - (x + y)
atan2(x + y, y + z)
sum(x + y, x + y, y + z)
sum(a, b, c)
mean(c(a, b, c), na.rm = TRUE)
foo(a = x + y, b = y + z)
```


```r
expr(!!xy /!!xz)
```

```
## (x + y)/(x + z)
```

```r
expr(-(!!xy)  ^ (!!xz))
```

```
## -(x + y)^(x + z)
```

```r
expr((!!(xy)) + !!yz - !!xy) # how to get parentheses?
```

```
## x + y + (y + z) - (x + y)
```

```r
expr(atan2(!!xy, !!yz))
```

```
## atan2(x + y, y + z)
```

```r
expr(sum(!!xy, !!xy, !!yz))
```

```
## sum(x + y, x + y, y + z)
```

```r
expr(sum(!!!abc))
```

```
## sum(a, b, c)
```

```r
expr(mean(c(!!!abc), na.rm = TRUE)) #was I supposed to do this some more complicated way?
```

```
## mean(c(a, b, c), na.rm = TRUE)
```

```r
expr(foo(a=!!xy, b=!!yz))
```

```
## foo(a = x + y, b = y + z)
```


#### 2 The following two calls print the same, but are actually different:


```r
(a <- expr(mean(1:10)))
```

```
## mean(1:10)
```

```r
#> mean(1:10)
(b <- expr(mean(!!(1:10))))
```

```
## mean(1:10)
```

```r
#> mean(1:10)
identical(a, b)
```

```
## [1] FALSE
```

```r
#> [1] FALSE
```

What is the difference and which is more natural?

b is going to go ahead and evaluate 1:10, while a won't.  Why not use a?


```r
expr_print(a)
```

```
## mean(1:10)
```

```r
expr_print(b)
```

```
## mean(<int: 1L, 2L, 3L, 4L, 5L, ...>)
```

## 19.5 Non-quoting

Base R mostly selectively turns off quoting as needed, rather than unquoting.  This is called non-quoting.

## 19.6 ...

### 19.6 Exercises

#### 1. One way to implement exec() is shown below. Describe how it works. What are the key ideas?


```r
exec <- function(f, ..., .env = caller_env()) {
  args <- list2(...)
  do.call(f, args, envir = .env)
}
```

Captures the calling environment, but allows user to set something else

uses `list2` to deal with the possibility that `...` is a list or not.

#### 2. Carefully read the source code for interaction(), expand.grid(), and par(). Compare and contrast the techniques they use for switching between dots and list behaviour.


```r
interaction
```

```
## function (..., drop = FALSE, sep = ".", lex.order = FALSE) 
## {
##     args <- list(...)
##     narg <- length(args)
##     if (narg < 1L) 
##         stop("No factors specified")
##     if (narg == 1L && is.list(args[[1L]])) {
##         args <- args[[1L]]
##         narg <- length(args)
##     }
##     for (i in narg:1L) {
##         f <- as.factor(args[[i]])[, drop = drop]
##         l <- levels(f)
##         if1 <- as.integer(f) - 1L
##         if (i == narg) {
##             ans <- if1
##             lvs <- l
##         }
##         else {
##             if (lex.order) {
##                 ll <- length(lvs)
##                 ans <- ans + ll * if1
##                 lvs <- paste(rep(l, each = ll), rep(lvs, length(l)), 
##                   sep = sep)
##             }
##             else {
##                 ans <- ans * length(l) + if1
##                 lvs <- paste(rep(l, length(lvs)), rep(lvs, each = length(l)), 
##                   sep = sep)
##             }
##             if (anyDuplicated(lvs)) {
##                 ulvs <- unique(lvs)
##                 while ((i <- anyDuplicated(flv <- match(lvs, 
##                   ulvs)))) {
##                   lvs <- lvs[-i]
##                   ans[ans + 1L == i] <- match(flv[i], flv[1:(i - 
##                     1)]) - 1L
##                   ans[ans + 1L > i] <- ans[ans + 1L > i] - 1L
##                 }
##                 lvs <- ulvs
##             }
##             if (drop) {
##                 olvs <- lvs
##                 lvs <- lvs[sort(unique(ans + 1L))]
##                 ans <- match(olvs[ans + 1L], lvs) - 1L
##             }
##         }
##     }
##     structure(as.integer(ans + 1L), levels = lvs, class = "factor")
## }
## <bytecode: 0x1155272b8>
## <environment: namespace:base>
```


```r
expand.grid
```

```
## function (..., KEEP.OUT.ATTRS = TRUE, stringsAsFactors = TRUE) 
## {
##     nargs <- length(args <- list(...))
##     if (!nargs) 
##         return(as.data.frame(list()))
##     if (nargs == 1L && is.list(a1 <- args[[1L]])) 
##         nargs <- length(args <- a1)
##     if (nargs == 0L) 
##         return(as.data.frame(list()))
##     cargs <- vector("list", nargs)
##     iArgs <- seq_len(nargs)
##     nmc <- paste0("Var", iArgs)
##     nm <- names(args)
##     if (is.null(nm)) 
##         nm <- nmc
##     else if (any(ng0 <- nzchar(nm))) 
##         nmc[ng0] <- nm[ng0]
##     names(cargs) <- nmc
##     rep.fac <- 1L
##     d <- lengths(args)
##     if (KEEP.OUT.ATTRS) {
##         dn <- vector("list", nargs)
##         names(dn) <- nmc
##     }
##     orep <- prod(d)
##     if (orep == 0L) {
##         for (i in iArgs) cargs[[i]] <- args[[i]][FALSE]
##     }
##     else {
##         for (i in iArgs) {
##             x <- args[[i]]
##             if (KEEP.OUT.ATTRS) 
##                 dn[[i]] <- paste0(nmc[i], "=", if (is.numeric(x)) 
##                   format(x)
##                 else x)
##             nx <- length(x)
##             orep <- orep/nx
##             if (stringsAsFactors && is.character(x)) 
##                 x <- factor(x, levels = unique(x))
##             x <- x[rep.int(rep.int(seq_len(nx), rep.int(rep.fac, 
##                 nx)), orep)]
##             cargs[[i]] <- x
##             rep.fac <- rep.fac * nx
##         }
##     }
##     if (KEEP.OUT.ATTRS) 
##         attr(cargs, "out.attrs") <- list(dim = d, dimnames = dn)
##     rn <- .set_row_names(as.integer(prod(d)))
##     structure(cargs, class = "data.frame", row.names = rn)
## }
## <bytecode: 0x115703bb0>
## <environment: namespace:base>
```

```r
par
```

```
## function (..., no.readonly = FALSE) 
## {
##     .Pars.readonly <- c("cin", "cra", "csi", "cxy", "din", "page")
##     single <- FALSE
##     args <- list(...)
##     if (!length(args)) 
##         args <- as.list(if (no.readonly) 
##             .Pars[-match(.Pars.readonly, .Pars)]
##         else .Pars)
##     else {
##         if (all(unlist(lapply(args, is.character)))) 
##             args <- as.list(unlist(args))
##         if (length(args) == 1) {
##             if (is.list(args[[1L]]) || is.null(args[[1L]])) 
##                 args <- args[[1L]]
##             else if (is.null(names(args))) 
##                 single <- TRUE
##         }
##     }
##     value <- .External2(C_par, args)
##     if (single) 
##         value <- value[[1L]]
##     if (!is.null(names(args))) 
##         invisible(value)
##     else value
## }
## <bytecode: 0x103f5dd38>
## <environment: namespace:graphics>
```

interaction checks to see if the first item in ... is a list, and if so it uses that.  expand.grid seems functionally equivalent

#### 3 Explain the problem with this definition of set_attr()


```r
set_attr <- function(x, ...) {
  attr <- rlang::list2(...)
  #print(attr)
  attributes(x) <- attr
  x
}
set_attr(1:10, x = 10)
```

```
## Error in attributes(x) <- attr: attributes must be named
```

```r
#> Error in attributes(x) <- attr: attributes must be named
```
Problem is that we have a named argument `x`.  When we call the function, x ends up being 10 and ... is the (unamed) vector `1:10`

```r
set_attr <- function(x, ...) {
  attr <- rlang::list2(..., .named=TRUE)
  attributes(y) <- attr
  x
}
set_attr(1:10, y = 10)
```

```
##  [1]  1  2  3  4  5  6  7  8  9 10
```

### 14.7 exercises

#### 1. In the linear-model example, we could replace the expr() in reduce(summands, ~ expr(!!.x + !!.y)) with call2(): reduce(summands, call2, "+"). Compare and contrast the two approaches. Which do you think is easier to read?


```r
linear <- function(var, val) {
  var <- ensym(var)
  coef_name <- map(seq_along(val[-1]), ~ expr((!!var)[[!!.x]]))

  summands <- map2(val[-1], coef_name, ~ expr((!!.x * !!.y)))
  summands <- c(val[[1]], summands)

  reduce(summands, ~ expr(!!.x + !!.y))
}

linear(x, c(10, 5, -4))
```

```
## 10 + (5 * x[[1L]]) + (-4 * x[[2L]])
```


```r
linear <- function(var, val) {
  var <- ensym(var)
  coef_name <- map(seq_along(val[-1]), ~ expr((!!var)[[!!.x]]))

  summands <- map2(val[-1], coef_name, ~ expr((!!.x * !!.y)))
  summands <- c(val[[1]], summands)

  reduce(summands, call2, "+")
}

linear(x, c(10, 5, -4))
```

```
## Error in `fn()`:
## ! Can't create call to non-callable object
```

#### 2. Re-implement the Box-Cox transform defined below using unquoting and new_function():


```r
bc <- function(lambda) {
  if (lambda == 0) {
    function(x) log(x)
  } else {
    function(x) (x ^ lambda - 1) / lambda
  }
}

bc(0)
```

```
## function(x) log(x)
## <environment: 0x12064fd48>
```

```r
bc(0.5)
```

```
## function(x) (x ^ lambda - 1) / lambda
## <bytecode: 0x107484f98>
## <environment: 0x120714ad8>
```


```r
bc2 <- function(lambda) {
  if (lambda == 0) {
    new_function(exprs(x=), expr(log(x) ))
  } else {
    new_function(exprs(x=), expr((x ^ !!lambda - 1) / !!lambda))
  }
}

bc2(0)
```

```
## function (x) 
## log(x)
## <environment: 0x110063620>
```

```r
bc2(0.5)
```

```
## function (x) 
## (x^(0.5 - 1))/0.5
## <environment: 0x1043a03b8>
```
#### 3 Re-implement the simple compose() defined below using quasiquotation and new_function():


```r
compose <- function(f, g) {
  function(...) f(g(...))
}

compose(mean, median)
```

```
## function(...) f(g(...))
## <environment: 0x1155cd248>
```



```r
compose2 <- function(f, g) {
  f <- enexpr(f)
  g <- enexpr(g)
  new_function(exprs(...=), expr((!!f)((!!g)(...))))
}

compose2(mean, median)
```

```
## function (...) 
## mean(median(...))
## <environment: 0x11588c6d8>
```

