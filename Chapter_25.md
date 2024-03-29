---
title: "Chapter 25"
author: "Julin Maloof"
date: "2023-10-04"
output: 
  html_document: 
    keep_md: yes
---




```r
library(Rcpp)
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

# 25.2 Getting STarted with C++

use cppFunction() to write a C++ function in R:


```r
cppFunction('int add(int x, int y, int z) {
  int sum = x + y + z;
  return sum;
}')
# add works like a regular R function
add
```

```
## function (x, y, z) 
## .Call(<pointer: 0x1076b1580>, x, y, z)
```

```r
#> function (x, y, z) 
#> .Call(<pointer: 0x107536a00>, x, y, z)
add(1, 2, 3)
```

```
## [1] 6
```

```r
#> [1] 6
```



## 25.2.1 No inputs, scalara output


```r
#R
one <- function() 1L
one()
```

```
## [1] 1
```

```r
one
```

```
## function() 1L
```

```r
#Cpp
cppFunction('int one() {
  return 1;
}')
one()
```

```
## [1] 1
```

```r
one
```

```
## function () 
## .Call(<pointer: 0x1076d0740>)
```

## 25.2.2 Scalar input, scalar output


```r
signR <- function(x) {
  if (x > 0) {
    1
  } else if (x == 0) {
    0
  } else {
    -1
  }
}

cppFunction('int signC(int x) {
  if (x > 0) {
    return 1;
  } else if (x == 0) {
    return 0;
  } else {
    return -1;
  }
}')
```

## 25.2.3 Vector inpur, scalar output


```r
sumR <- function(x) {
  total <- 0
  for (i in seq_along(x)) {
    total <- total + x[i]
  }
  total
}

cppFunction('double sumC(NumericVector x) {
  int n = x.size();
  double total = 0;
  for(int i = 0; i < n; ++i) {
    total += x[i];
  }
  return total;
}')
```


```r
x <- rnorm(10e6)

bench::mark(sumR(x),
            sum(x),
            sumC(x))
```

```
## # A tibble: 3 × 6
##   expression      min   median `itr/sec` mem_alloc `gc/sec`
##   <bch:expr> <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
## 1 sumR(x)    139.02ms 139.47ms      7.16    26.6KB        0
## 2 sum(x)       14.1ms   14.4ms     69.5         0B        0
## 3 sumC(x)      8.56ms   8.68ms    115.      2.49KB        0
```


## 25.2.5 Vector input, Vector output


```r
pdistR <- function(x, ys) {
  sqrt((x - ys) ^ 2)
}

cppFunction('NumericVector pdistC(double x, NumericVector ys) {
  int n = ys.size();
  NumericVector out(n);

  for(int i = 0; i < n; ++i) {
    out[i] = sqrt(pow(ys[i] - x, 2.0));
  }
  return out;
}')

y <- runif(1e6)
bench::mark(
  pdistR(0.5, y),
  pdistC(0.5, y)
)[1:6]
```

```
## # A tibble: 2 × 6
##   expression          min   median `itr/sec` mem_alloc `gc/sec`
##   <bch:expr>     <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
## 1 pdistR(0.5, y)   1.64ms   2.06ms      477.    7.63MB     117.
## 2 pdistC(0.5, y) 356.74µs 594.21µs     1644.    7.63MB     212.
```

```r
#> # A tibble: 2 x 6
#>   expression          min   median `itr/sec` mem_alloc `gc/sec`
#>   <bch:expr>     <bch:tm> <bch:tm>     <dbl> <bch:byt>    <dbl>
#> 1 pdistR(0.5, y)   6.31ms   6.75ms      145.    7.63MB     72.4
#> 2 pdistC(0.5, y)   2.31ms   2.77ms      380.    7.63MB    192.
```

## 25.2.6 Exercises
### 1 With the basics of C++ in hand, it’s now a great time to practice by reading and writing some simple C++ functions. For each of the following functions, read the code and figure out what the corresponding base R function is. You might not understand every part of the code yet, but you should be able to figure out the basics of what the function does.

* f1: mean()
* f2: cumsum()
* f3: any()
* f4: ???
* f5: pmin()




```cpp
#include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
double f1(NumericVector x) {
  int n = x.size();
  double y = 0;
  
  for(int i = 0; i < n; ++i) {
    y += x[i] / n;
  }
  return y;
}

NumericVector f2(NumericVector x) {
  int n = x.size();
  NumericVector out(n);
  
  out[0] = x[0];
  for(int i = 1; i < n; ++i) {
    out[i] = out[i - 1] + x[i];
  }
  return out;
}

bool f3(LogicalVector x) {
  int n = x.size();
  
  for(int i = 0; i < n; ++i) {
    if (x[i]) return true;
  }
  return false;
}

int f4(Function pred, List x) {
  int n = x.size();
  
  for(int i = 0; i < n; ++i) {
    LogicalVector res = pred(x[i]);
    if (res[0]) return i + 1;
  }
  return 0;
}

NumericVector f5(NumericVector x, NumericVector y) {
  int n = std::max(x.size(), y.size());
  NumericVector x1 = rep_len(x, n);
  NumericVector y1 = rep_len(y, n);
  
  NumericVector out(n);
  
  for (int i = 0; i < n; ++i) {
    out[i] = std::min(x1[i], y1[i]);
  }
  
  return out;
}
```

### To practice your function writing skills, convert the following functions into C++. For now, assume the inputs have no missing values.

all().

cumprod(), cummin(), cummax().

diff(). Start by assuming lag 1, and then generalise for lag n.

range().

var(). Read about the approaches you can take on Wikipedia. Whenever implementing a numerical algorithm, it’s always good to check what is already known about the problem.

#### all

```cpp
# include <Rcpp.h>
using namespace Rcpp;

//[[Rcpp::export]]
bool allC(LogicalVector x) {
  int n = x.size();
  
  for (int i = 0; i < n; ++i) {
    if(!x[i]) return false;
  }
  return true;
}
```


```r
all(c(T,T,T))
```

```
## [1] TRUE
```

```r
all(c(T,T,F,T))
```

```
## [1] FALSE
```

```r
allC(c(T,T,T))
```

```
## [1] TRUE
```

```r
allC(c(T,T,F,T))
```

```
## [1] FALSE
```
#### cumX


```cpp
# include <Rcpp.h>
using namespace Rcpp;

//[[Rcpp::export]]
NumericVector cumprodC(NumericVector x) {
  int n = x.size();
  
  NumericVector res(n);
  
  res[0] = x[0];
  
  for(int i=1; i < n; ++i) {
    res[i] = x[i]*res[i-1];
  }
  
  return res;
}

//[[Rcpp::export]]
NumericVector cumminC(NumericVector x) {
  int n = x.size();
  
  NumericVector res(n);
  
  res[0] = x[0];
  
  for(int i=1; i < n; ++i) {
    if (x[i] < res[i-1]) 
      res[i] = x[i];
    else
      res[i] = res[i-1];
  }
  
  return res;
}

//[[Rcpp::export]]
NumericVector cummaxC(NumericVector x) {
  int n = x.size();
  
  NumericVector res(n);
  
  res[0] = x[0];
  
  for(int i=1; i < n; ++i) {
    if (x[i] > res[i-1]) 
      res[i] = x[i];
    else
      res[i] = res[i-1];
  }
  
  return res;
}

```


```r
cumprod(2:10)
```

```
## [1]       2       6      24     120     720    5040   40320  362880 3628800
```

```r
cumprodC(2:10)
```

```
## [1]       2       6      24     120     720    5040   40320  362880 3628800
```

```r
cummin(c(3:1, 2:0, 4:2))
```

```
## [1] 3 2 1 1 1 0 0 0 0
```

```r
cumminC(c(3:1, 2:0, 4:2))
```

```
## [1] 3 2 1 1 1 0 0 0 0
```

```r
cummax(c(3:1, 2:0, 4:2))
```

```
## [1] 3 3 3 3 3 3 4 4 4
```

```r
cummaxC(c(3:1, 2:0, 4:2))
```

```
## [1] 3 3 3 3 3 3 4 4 4
```
### diff


```cpp
# include <Rcpp.h>
using namespace Rcpp;

//[[Rcpp::export]]
NumericVector diffC(NumericVector x, int lag = 1) {
  int n = x.size();
  
  NumericVector res(n-lag);
  
  for(int i = lag; i < n; ++i) {
    res[i-lag] = x[i] - x[i-lag];
  }
  
  return res;
}
```


```r
diff((1:10)^2)
```

```
## [1]  3  5  7  9 11 13 15 17 19
```

```r
diffC((1:10)^2)
```

```
## [1]  3  5  7  9 11 13 15 17 19
```

```r
diff((1:10)^2, 2)
```

```
## [1]  8 12 16 20 24 28 32 36
```

```r
diffC((1:10)^2, 2)
```

```
## [1]  8 12 16 20 24 28 32 36
```
#### range

```cpp
# include <Rcpp.h>
using namespace Rcpp;

//[[Rcpp::export]]
NumericVector rangeC(NumericVector x){
  int n = x.size();
  NumericVector result(2);
  
  result[0] = result[1] = x[0];
  
  for(int i = 1; i < n; ++i){
    result[0] = std::min(result[0], x[i]);
    result[1] = std::max(result[1], x[i]);
  }
  
  return(result);
}
```


```r
x <- sample(1:20)
x
```

```
##  [1] 16 17 20  6  8  7  2 18  4 19 14 10 13 12  5 11 15  3  9  1
```

```r
range(x)
```

```
## [1]  1 20
```

```r
rangeC(x)
```

```
## [1]  1 20
```

#### var

Okay wikipedia says that some pitfalls (e.g. cancellation) can be avoided by shifting the numbers to be centered around their mean, (or even anything in teh range) so I will try that.  There are more exotic algorithms but this at least seems simple.


```cpp
# include <Rcpp.h>
using namespace Rcpp;

//[[Rcpp::export]]
double varC(NumericVector x){
  int n = x.size();
  double K = x[0];
  double sumx = 0;
  double sumsqx = 0;
  
  for(int i = 0; i < n; ++i) {
    sumx += (x[i] - K);
    sumsqx += pow(x[i] -K, 2);
  }
  
  double var = (sumsqx - pow(sumx, 2)/n) / (n-1);
  
  return(var);
}
```


```r
x <- rnorm(100)
var(x)
```

```
## [1] 1.163644
```

```r
varC(x)
```

```
## [1] 1.163644
```

# 25.3 Other Classes

## 23.3.1 Lists and Dataframes
A bunch of other classes.  The most important are Lists and DataFrames.  These are often most useful for output, since they can contain arbitraty classes on input.  But if the structure is known it can be used as input.  For example, lm

Function to extract mean percentage error

```cpp
#include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
double mpe(List mod) {
  if (!mod.inherits("lm")) stop("Input must be a linear model");
  
  NumericVector resid = as<NumericVector>(mod["residuals"]);
  NumericVector fitted = as<NumericVector>(mod["fitted.values"]);
  
  int n = resid.size();
  double err = 0;
  for(int i = 0; i < n; ++i) {
    err += resid[i] / (fitted[i] + resid[i]);
  }
  return err / n;
}
```



```r
mod <- lm(mpg ~ wt, data = mtcars)
mpe(mod)
```

```
## [1] -0.01541615
```

```r
#> [1] -0.0154
```

## 25.3.2 Functions

Can pass an R function to C++ so that it can be called from your C++ code.  Because we don't know what it will return, the return class is "RObject"


```cpp
#include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
RObject callWithOne(Function f) {
  return f(1);
}
```


```r
callWithOne(function(x) x + 1)
```

```
## [1] 2
```

```r
#> [1] 2
callWithOne(paste)
```

```
## [1] "1"
```

```r
#> [1] "1"
```


If you need to pass names arguments to an R function from C++, the syntax is `f(_["x"] = "y", _["value"] = 1);`

## 25.3.4 Attributes

See below for setting names and attributes


```cpp
#include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
NumericVector attribs() {
  NumericVector out = NumericVector::create(1, 2, 3);
  
  out.names() = CharacterVector::create("a", "b", "c");
  out.attr("my-attr") = "my-value";
  out.attr("class") = "my-class";
  
  return out;
}
```

But what if you don't use ::create?


```cpp
#include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
NumericVector attribs2(NumericVector x) {
  
  x.names() = CharacterVector::create("a", "b", "c");
  x.attr("my-attr") = "my-value";
  x.attr("class") = "my-class";
  
  return x;
}
```


```r
attribs2(4:6)
```

```
## a b c 
## 4 5 6 
## attr(,"my-attr")
## [1] "my-value"
## attr(,"class")
## [1] "my-class"
```
works

# 25.4 Missing Values

Need to know how missing values work in scalars and vectors

## 25.4.1 Scalars


```cpp
#include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
List scalar_missings() {
  int int_s = NA_INTEGER;
  String chr_s = NA_STRING;
  bool lgl_s = NA_LOGICAL;
  double num_s = NA_REAL;
  
  return List::create(int_s, chr_s, lgl_s, num_s);
}
```


```r
scalar_missings()
```

```
## [[1]]
## [1] NA
## 
## [[2]]
## [1] NA
## 
## [[3]]
## [1] TRUE
## 
## [[4]]
## [1] NA
```
Whoa, look out for the boolean...

Other wierdness

### 25.4.1.1 Integers: 

C++ stores an NA as the smallest integer, so if you do an operation on it, then the value can change.


```r
evalCpp('NA_INTEGER + 1')
```

```
## [1] -2147483647
```

**yikes**

Solution: use an integer vector of length 1 or be very careful!

### 25.4.1.2 Doubles

somewhat better:


```r
evalCpp("NAN == 1")
```

```
## [1] FALSE
```

```r
#> [1] FALSE
evalCpp("NAN < 1")
```

```
## [1] FALSE
```

```r
#> [1] FALSE
evalCpp("NAN > 1")
```

```
## [1] FALSE
```

```r
#> [1] FALSE
evalCpp("NAN == NAN")
```

```
## [1] FALSE
```

```r
#> [1] FALSE
evalCpp("NAN + 1")
```

```
## [1] NaN
```

However...


```r
evalCpp("NAN && TRUE")
```

```
## [1] TRUE
```

```r
#> [1] TRUE
evalCpp("NAN || FALSE")
```

```
## [1] TRUE
```

```r
#> [1] TRUE
```

But math is OK


```r
evalCpp("NAN + 1")
```

```
## [1] NaN
```

```r
#> [1] NaN
evalCpp("NAN - 1")
```

```
## [1] NaN
```

```r
#> [1] NaN
evalCpp("NAN / 1")
```

```
## [1] NaN
```

```r
#> [1] NaN
evalCpp("NAN * 1")
```

```
## [1] NaN
```

```r
#> [1] NaN
```

## 25.4.2 Strings
OK

## 25.4.3 Boolean
C++ only has true and false, no NA.  `int` however can have true, false, or NA

## 25.4.4 Vectors

With vectors, you need to use a missing value specific to the type of vector, NA_REAL, NA_INTEGER, NA_LOGICAL, NA_STRING:


```cpp
#include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
List missing_sampler() {
  return List::create(
    NumericVector::create(NA_REAL),
    IntegerVector::create(NA_INTEGER),
    LogicalVector::create(NA_LOGICAL),
    CharacterVector::create(NA_STRING)
  );
}
```


```r
str(missing_sampler())
```

```
## List of 4
##  $ : num NA
##  $ : int NA
##  $ : logi NA
##  $ : chr NA
```

## 25.4.5 Exercises

### 1 Rewrite any of the functions from the first exercise of Section 25.2.6 to deal with missing values. If na.rm is true, ignore the missing values. If na.rm is false, return a missing value if the input contains any missing values. Some good functions to practice with are min(), max(), range(), mean(), and var().


```cpp
# include <Rcpp.h>
using namespace Rcpp;

//[[Rcpp::export]]
double minC(NumericVector x, bool na_rm = false) {
  int n = x.size();
  
  double result=x[0];
  
  for(int i=0; i < n; ++i) {
    if (std::isnan(x[i])) {
      if (na_rm) continue;
      else return(NA_REAL);
    }
    result = std::min(result, x[i]);
  }
  
  return(result);
}
```


```r
x <- rnorm(100)
min(x)
```

```
## [1] -2.83912
```

```r
minC(x)
```

```
## [1] -2.83912
```

```r
minC(x, TRUE)
```

```
## [1] -2.83912
```


```r
x[50] <- NA
min(x)
```

```
## [1] NA
```

```r
min(x, na.rm = TRUE)
```

```
## [1] -2.83912
```

```r
minC(x)
```

```
## [1] NA
```

```r
minC(x, na_rm = TRUE)
```

```
## [1] -2.83912
```
#### Range


```cpp
# include <Rcpp.h>
using namespace Rcpp;

//[[Rcpp::export]]
NumericVector rangeC(NumericVector x, bool na_rm = false){
  int n = x.size();
  NumericVector result(2);
  
  int start = 0;
  
  if(isnan(x[0])) {
    if(na_rm) ++start; // Skip the NA in position 0
    else return(NA_REAL);
  }
  
  result[0] = result[1] = x[start];
  ++start;
  
  for(int i = start; i < n; ++i){
    if(isnan(x[i])) {
      if(na_rm) continue; 
      else return(NumericVector::create(NA_REAL, NA_REAL));
    }
    result[0] = std::min(result[0], x[i]);
    result[1] = std::max(result[1], x[i]);
  }
  
  return(result);
}
```


```r
x <- rnorm(100)
range(x)
```

```
## [1] -2.107926  2.254143
```

```r
rangeC(x)
```

```
## [1] -2.107926  2.254143
```

```r
x[50] <- NA
range(x)
```

```
## [1] NA NA
```

```r
rangeC(x)
```

```
## [1] NA NA
```

```r
range(x, na.rm=TRUE)
```

```
## [1] -2.107926  2.254143
```

```r
rangeC(x, na_rm = TRUE)
```

```
## [1] -2.107926  2.254143
```


#### mean

we could either run through the vector twice, first to detect/discard NAs and then compute the mean, or we could run through it once, keeping a separate counter.  Second way seems better.


```cpp
# include <Rcpp.h>
using namespace Rcpp;

//[[Rcpp::export]]
double meanC(NumericVector x, bool na_rm = false){
  int n = x.length();
  int count = 0;
  double sum = 0;
  
  for(int i = 0; i < n; ++i) {
    if(isnan(x[i])) {
      if(na_rm) continue; // Skip the NA in position 0
      else return(NA_REAL);
    }
    sum += x[i];
    ++count;
  }
  
  return(sum/count);
  
}
```


```r
x <- rnorm(100)
mean(x)
```

```
## [1] 0.01707162
```

```r
meanC(x)
```

```
## [1] 0.01707162
```

```r
x[30] <- NaN
mean(x)
```

```
## [1] NaN
```

```r
meanC(x)
```

```
## [1] NA
```

```r
mean(x, na.rm = TRUE)
```

```
## [1] 0.01333287
```

```r
meanC(x, na_rm = TRUE)
```

```
## [1] 0.01333287
```
Alternate, using na_omit


```cpp
# include <Rcpp.h>
using namespace Rcpp;

//[[Rcpp::export]]
double meanC2(NumericVector x, bool na_rm = false){
  
  if(is_true(any(is_na(x)))) {
    if(na_rm) x = na_omit(x);
    else return(NA_REAL);
  }
  
  int n = x.length();
  double sum = 0;
  
  for(int i = 0; i < n; ++i) {
    sum += x[i];
  }
  
  return(sum/n);
}
```


```r
x <- rnorm(100)
mean(x)
```

```
## [1] -0.08023769
```

```r
meanC2(x)
```

```
## [1] -0.08023769
```

```r
x[30] <- NaN
mean(x)
```

```
## [1] NaN
```

```r
meanC2(x)
```

```
## [1] NA
```

```r
mean(x, na.rm = TRUE)
```

```
## [1] -0.08518587
```

```r
meanC2(x, na_rm = TRUE)
```

```
## [1] -0.08518587
```
### 2. Rewrite cumsum() and diff() so they can handle missing values. Note that these functions have slightly more complicated behaviour.

#### cumsum

From `cumsum()` help file: _An NA value in x causes the corresponding and following elements of the return value to be NA, as does integer overflow in cumsum (with a warning)._


```cpp
# include <Rcpp.h>
using namespace Rcpp;

//[[Rcpp::export]]
NumericVector cumsumC(NumericVector x) {
  bool na_found = false;
  int n = x.size();
  NumericVector out(n);
  
  out[0] = x[0];
  for(int i = 1; i < n; ++i) {
    if(na_found) {
      out[i] = NA_REAL;
      continue;
    } 
    if(NumericVector::is_na(x[i])) {
      na_found = true;
      out[i] = NA_REAL;
      continue;
    }
    out[i] = out[i - 1] + x[i];
  }
  return out;
}
```



```r
x <- rnorm(20, 10)
x[15] <- NA
cumsum(x)
```

```
##  [1]   9.711845  18.774889  28.258956  38.674568  48.384743  57.613057
##  [7]  68.312107  78.699230  89.994927  98.957327 108.815375 120.587027
## [13] 131.812228 143.045316         NA         NA         NA         NA
## [19]         NA         NA
```

```r
cumsumC(x)
```

```
##  [1]   9.711845  18.774889  28.258956  38.674568  48.384743  57.613057
##  [7]  68.312107  78.699230  89.994927  98.957327 108.815375 120.587027
## [13] 131.812228 143.045316         NA         NA         NA         NA
## [19]         NA         NA
```

### diff


```cpp
# include <Rcpp.h>
using namespace Rcpp;

//[[Rcpp::export]]
NumericVector test() {
  NumericVector v = {2,4,6,8,10};
  return(v[NumericVector::create(2,4)]);
}
```


```r
test()
```

```
## [1]  6 10
```


```cpp
# include <Rcpp.h>
using namespace Rcpp;

//[[Rcpp::export]]
NumericVector diffC(NumericVector x, int lag = 1) {
  int n = x.size();
  
  NumericVector res(n-lag);
  
  for(int i = lag; i < n; ++i) {
    if(NumericVector::is_na(x[i]) | NumericVector::is_na(x[i-lag])) {
      res[i-lag] = NA_REAL;
    } else {
      res[i-lag] = x[i] - x[i-lag];
    }
  }
  
  return res;
}
```


```r
x <- rnorm(20, 10)
x[15] <- NA
diff(x)
```

```
##  [1]  2.91753541 -1.50333086 -0.44096225  2.03388136 -1.26879839  0.65236981
##  [7] -0.59576361 -0.20961635 -0.89545853  1.53088402 -0.42058809  0.09334168
## [13]  0.21846647          NA          NA -0.80105816 -0.10431032  0.89954748
## [19]  1.23913199
```

```r
diffC(x)
```

```
##  [1]  2.91753541 -1.50333086 -0.44096225  2.03388136 -1.26879839  0.65236981
##  [7] -0.59576361 -0.20961635 -0.89545853  1.53088402 -0.42058809  0.09334168
## [13]  0.21846647          NA          NA -0.80105816 -0.10431032  0.89954748
## [19]  1.23913199
```
# 25.5 Standard Template Library

## 25.5.1 Using Iterators

* Advance with `++`
* dereference with `*`
* compare with `==`

For example:


```cpp
#include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
double sum3(NumericVector x) {
  double total = 0;
  
  NumericVector::iterator it;
  for(it = x.begin(); it != x.end(); ++it) {
    Rcout << *it << " ";
    total += *it;
  }
  return total;
}
```


```r
sum3(10:1)
```

```
## 10 9 8 7 6 5 4 3 2 1
```

```
## [1] 55
```

Or using accumulate


```cpp
#include <numeric>
#include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
double sum5(NumericVector x) {
  return std::accumulate(x.begin(), x.end(), 0.0);
}
```


```r
sum5(10:1)
```

```
## [1] 55
```

## 25.5.2 Algorithims.

The algorithm header provides a large number of iterator functions.  Consider:


```cpp
#include <Rcpp.h>
#include <algorithm>
using namespace Rcpp;

// [[Rcpp::export]]
IntegerVector findInterval2(NumericVector x, NumericVector breaks) {
  IntegerVector out(x.size());

  NumericVector::iterator it, pos;
  IntegerVector::iterator out_it;

  for(it = x.begin(), out_it = out.begin(); it != x.end(); 
      ++it, ++out_it) {
    pos = std::upper_bound(breaks.begin(), breaks.end(), *it);
    *out_it = std::distance(breaks.begin(), pos);
    Rcout << *pos << " " << *out_it << std::endl;
  }

  return out;
}

```


```r
findInterval2(1:10, c(0,3,6,10))
```

```
## 3 1
## 3 1
## 6 2
## 6 2
## 6 2
## 10 3
## 10 3
## 10 3
## 10 3
## 2.91774e-314 4
```

```
##  [1] 1 1 2 2 2 3 3 3 3 4
```

## 25.5.3 Data structures.  

There are a bunch of them.  Perhaps most useful are `vector`, `unordered_set`, and `unordered_map`

## 25.5.4 Vectors

can access with vect[].  Can add to the end with vect.push_back()


```cpp
#include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
List rleC(NumericVector x) {
  std::vector<int> lengths;
  std::vector<double> values;

  // Initialise first value
  int i = 0;
  double prev = x[0];
  values.push_back(prev);
  lengths.push_back(1);

  NumericVector::iterator it;
  for(it = x.begin() + 1; it != x.end(); ++it) {
    if (prev == *it) {
      lengths[i]++;
    } else {
      values.push_back(*it);
      lengths.push_back(1);

      i++;
      prev = *it;
    }
  }

  return List::create(
    _["lengths"] = lengths, 
    _["values"] = values
  );
}
```


## set

## map

a map has a key and value pair


```cpp
#include <Rcpp.h>
#include <algorithm>

using namespace Rcpp;

// [[Rcpp::export]]
std::map<double, int> tableC(NumericVector x) {
  std::map<double, int> counts;

  int n = x.size();
  for (int i = 0; i < n; i++) {
    counts[x[i]]++;
  }

  return counts;
}
```


```r
tableC(sample(1:10,100,TRUE))
```

```
##  1  2  3  4  5  6  7  8  9 10 
##  7  5 11 11  6 12  9  8 16 15
```

# 25.5.7 Exercises
To practice using the STL algorithms and data structures, implement the following using R functions in C++, using the hints provided:

## 1. median.default() using partial_sort.

First let's get a handle on partial sort

```cpp
#include <Rcpp.h>
#include <algorithm>

using namespace Rcpp;

// [[Rcpp::export]]
NumericVector psortC(NumericVector x) {
  std::partial_sort(x.begin(), x.begin() + 5, x.end());
  return(x);
}
```


```r
x <- sample(1:20)
x
```

```
##  [1] 17 18 16  5 11  8 15  2  4  6 10 13  3  7 20 19 12  1  9 14
```

```r
psortC(x)
```

```
##  [1]  1  2  3  4  5 18 17 16 15 11 10 13  8  7 20 19 12  6  9 14
```

okay, so we can just sort the first half of the vector.

note that std::partial_sort modified in place


```cpp
#include <Rcpp.h>
#include <algorithm>

using namespace Rcpp;

// [[Rcpp::export]]
double medianC(NumericVector x) {
  int n = x.size();
  
  std::partial_sort(x.begin(), x.begin() + (n/2) + 1, x.end());
  
  if(n%2 == 0) { // even sized vector, need to get two values and average
    --n; // because vectors are zero indexed
    return( (x[n/2] + x[n/2+1]) / 2 );
  } else {
    --n; // because vectors are zero indexed
    return(x[(n+1)/2]);
  }
}

```


```r
median(1:4)
```

```
## [1] 2.5
```

```r
medianC(1:4)
```

```
## [1] 2.5
```

```r
median(1:5)
```

```
## [1] 3
```

```r
medianC(1:5)
```

```
## [1] 3
```


## 2. %in% using unordered_set and the find() or count() methods.


```cpp
#include <Rcpp.h>
#include <algorithm>

using namespace Rcpp;

// [[Rcpp::export(name = `%inC%`)]]
LogicalVector inC(CharacterVector x, CharacterVector table) {
  
  int n = x.size();

  LogicalVector results(n);
  std::unordered_set<String> table_set(table.begin(), table.end(), table.size());
  
  for(int i = 0; i < n; ++i) {
    // Rcout << i << " " << x[i] << " " << (table_set.count(x[i]) > 0) << std::endl;
    results[i] = (table_set.count(x[i]) > 0);
  }

  return(results);
  }
```


```r
x <- LETTERS[1:10]
table <- LETTERS[5:20]
x %in% table
```

```
##  [1] FALSE FALSE FALSE FALSE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
```

```r
x %inC% table
```

```
##  [1] FALSE FALSE FALSE FALSE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
```



## 3. unique() using an unordered_set (challenge: do it in one line!).


```cpp
#include <Rcpp.h>
#include <algorithm>

using namespace Rcpp;

// [[Rcpp::export]]
NumericVector uniqueC(NumericVector x){
  int n = x.size();
  std::vector<double> uniq ;
  std::unordered_set<double> seen;
  
  for(int i = 0; i < n; ++i) {
    if(seen.insert(x[i]).second) {
      uniq.push_back(x[i]);
    }
  }
    return(wrap(uniq));
}
```


```cpp
#include <Rcpp.h>
#include <algorithm>

using namespace Rcpp;

// [[Rcpp::export]]
NumericVector uniqueC1L(NumericVector x){
  std::unordered_set<double> uniq(x.begin(), x.end(), x.size()); 
  return(wrap(uniq));
}
```


```r
x <- sample(1:10, 20, TRUE)
x
```

```
##  [1]  1  5  3  9  3  3  7  8  1  7  3  7  5  3  7  8  7 10  8  4
```

```r
unique(x)
```

```
## [1]  1  5  3  9  7  8 10  4
```

```r
uniqueC(x)
```

```
## [1]  1  5  3  9  7  8 10  4
```

```r
uniqueC1L(x) 
```

```
## [1]  4 10  8  7  9  3  5  1
```

## 4. min() using std::min(), or max() using std::max().


```cpp
#include <Rcpp.h>

using namespace Rcpp;

// [[Rcpp::export]]
double minC(NumericVector x, bool na_rm = false){
  if(is_true(any(is_na(x)))) {
    if(na_rm) x = na_omit(x);
    else return(NA_REAL);
  }
  
  int n = x.size();
  double out = R_PosInf;
  
  for (int i = 0; i < n; ++i) {
    out = std::min(out, x[i]);
  }
return(out);
}
```


```r
x <- rnorm(100)
min(x)
```

```
## [1] -2.525875
```

```r
minC(x)
```

```
## [1] -2.525875
```

## 5. which.min() using min_element, or which.max() using max_element.


```cpp
#include <Rcpp.h>
#include <algorithm>

using namespace Rcpp;

// [[Rcpp::export(name=which.minC)]]
int which_min(NumericVector x){
  x = na_omit(x);
  NumericVector::iterator min;
  
  min = std::min_element(x.begin(), x.end());
  
  return(std::distance(x.begin(), min)+1);
}
```


```r
x <- rnorm(100)
which.min(x)
```

```
## [1] 36
```

```r
which.minC(x)
```

```
## [1] 36
```


## 6. setdiff(), union(), and intersect() for integers using sorted ranges and set_union, set_intersection and set_difference.


```cpp
#include <Rcpp.h>
#include <algorithm>

using namespace Rcpp;

// [[Rcpp::export]]
IntegerVector setdiffC(IntegerVector x, IntegerVector y) {
  IntegerVector out(x.size());
  x.sort();
  y.sort();
  
  out.fill(NA_INTEGER);
  
  std::set_difference(x.begin(), x.end(), y.begin(), y.end(), out.begin());
  
  out = na_omit(out);
  
  return(out);
}
```


```r
x <- sample(1:10)
y <- sample(6:15)
x
```

```
##  [1]  1  7  9  8  4  6  2  5  3 10
```

```r
y
```

```
##  [1] 10  6  9 15  7 12  8 14 13 11
```

```r
setdiff(x,y)
```

```
## [1] 1 4 2 5 3
```

```r
setdiffC(x,y)
```

```
## [1] 1 2 3 4 5
```

```cpp
#include <Rcpp.h>
#include <algorithm>

using namespace Rcpp;

// [[Rcpp::export]]
IntegerVector setunionC(IntegerVector x, IntegerVector y) {
  IntegerVector out(x.size()+y.size());
  x.sort();
  y.sort();
  
  out.fill(NA_INTEGER);
  
  std::set_union(x.begin(), x.end(), y.begin(), y.end(), out.begin());
  
  out = na_omit(out);
  
  return(out);
}
```


```r
x <- sample(1:10)
y <- sample(6:15)
x
```

```
##  [1]  1  9  4  6  8  7  5  2  3 10
```

```r
y
```

```
##  [1] 11  7  6 15 12 10  9  8 14 13
```

```r
union(x,y)
```

```
##  [1]  1  9  4  6  8  7  5  2  3 10 11 15 12 14 13
```

```r
setunionC(x,y)
```

```
##  [1]  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15
```

```cpp
#include <Rcpp.h>
#include <algorithm>

using namespace Rcpp;

// [[Rcpp::export]]
IntegerVector setintersectionC(IntegerVector x, IntegerVector y) {
  IntegerVector out(x.size()+y.size());
  x.sort();
  y.sort();
  
  out.fill(NA_INTEGER);
  
  std::set_intersection(x.begin(), x.end(), y.begin(), y.end(), out.begin());
  
  out = na_omit(out);
  
  return(out);
}
```


```r
x <- sample(1:10)
y <- sample(6:15)
x
```

```
##  [1]  5  7  9  8  3  4  6 10  1  2
```

```r
y
```

```
##  [1] 12 15  7 13 10  6  8 14 11  9
```

```r
intersect(x,y)
```

```
## [1]  7  9  8  6 10
```

```r
setintersectionC(x,y)
```

```
## [1]  6  7  8  9 10
```

