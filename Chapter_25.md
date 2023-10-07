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
## .Call(<pointer: 0x1053b5580>, x, y, z)
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
## .Call(<pointer: 0x1053d4740>)
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
## 1 sumR(x)    136.96ms 137.38ms      7.27    3.98MB        0
## 2 sum(x)       14.1ms  14.17ms     70.3         0B        0
## 3 sumC(x)      8.44ms   8.51ms    117.      2.49KB        0
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
## 1 pdistR(0.5, y)   1.92ms   1.99ms      496.    7.63MB     125.
## 2 pdistC(0.5, y) 360.19µs 558.79µs     1723.    7.63MB     233.
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
##  [1]  2 12 11 20  9 10 14  7  3 19 16 18  6  5 15  4 13 17  8  1
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
## [1] 1.074365
```

```r
varC(x)
```

```
## [1] 1.074365
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
## [1] -2.517992
```

```r
minC(x)
```

```
## [1] -2.517992
```

```r
minC(x, TRUE)
```

```
## [1] -2.517992
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
## [1] -2.517992
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
## [1] -2.517992
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
## [1] -1.842712  3.291668
```

```r
rangeC(x)
```

```
## [1] -1.842712  3.291668
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
## [1] -1.842712  3.291668
```

```r
rangeC(x, na_rm = TRUE)
```

```
## [1] -1.842712  3.291668
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
## [1] -0.0636123
```

```r
meanC(x)
```

```
## [1] -0.0636123
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
## [1] -0.04955349
```

```r
meanC(x, na_rm = TRUE)
```

```
## [1] -0.04955349
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
## [1] 0.01121501
```

```r
meanC2(x)
```

```
## [1] 0.01121501
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
## [1] -0.0001017236
```

```r
meanC2(x, na_rm = TRUE)
```

```
## [1] -0.0001017236
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
##  [1]   9.032548  19.145673  29.902298  39.101266  49.692950  58.811062
##  [7]  69.155170  80.905577  89.463995 100.019770 111.590377 122.712648
## [13] 132.591415 141.528664         NA         NA         NA         NA
## [19]         NA         NA
```

```r
cumsumC(x)
```

```
##  [1]   9.032548  19.145673  29.902298  39.101266  49.692950  58.811062
##  [7]  69.155170  80.905577  89.463995 100.019770 111.590377 122.712648
## [13] 132.591415 141.528664         NA         NA         NA         NA
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
##  [1]  0.7330773 -0.7557204 -1.7216703  0.2949422  0.5100427  0.3153265
##  [7] -1.4066663  0.7686268 -0.8292127  3.3512770 -1.7563712 -1.0250924
## [13]  1.7486029         NA         NA -0.3221163  1.5150344 -0.9226099
## [19]  3.2607937
```

```r
diffC(x)
```

```
##  [1]  0.7330773 -0.7557204 -1.7216703  0.2949422  0.5100427  0.3153265
##  [7] -1.4066663  0.7686268 -0.8292127  3.3512770 -1.7563712 -1.0250924
## [13]  1.7486029         NA         NA -0.3221163  1.5150344 -0.9226099
## [19]  3.2607937
```

