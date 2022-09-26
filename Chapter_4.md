---
title: "Chapter 4"
author: "Julin Maloof"
date: "2022-09-24"
output: 
  html_document: 
    keep_md: yes
---

## Quiz

1. What is the result of subsetting a vector with positive integers, negative integers, a logical vector, or a character vector?

_Positive integers select elements by positions; negative integers exclude elements by positions; a logical vector selects by position and needs to be the same length as the vector (or maybe it can be recycled); a character vector selects by name.

2. What’s the difference between [, [[, and $ when applied to a list? 

_[ returns a list with those elements, [[ extracts that element and doesn't return a list (unless the element itself is a list), $ will pull out an element by name_

3. When should you use drop = FALSE?

When you do not want a data frame to be converted to a vector (or maybe when you don't want to lose dimensions)

4. If x is a matrix, what does x[] <- 0 do? How is it different from x <- 0?

_`x[] <- 0` will set all elements to 0.  `x <- 0` will change x to a vector of length 1 with a value of 0._ 

5. How can you use a named vector to relabel categorical variables?

_I am not sure what this is asking_

## 4.2 Selecting multiple elements

skipped what I knew... this is the code for selecting from matrices with matrices:


```r
vals <- outer(1:5, 1:5, FUN = "paste", sep = ",")
vals
```

```
##      [,1]  [,2]  [,3]  [,4]  [,5] 
## [1,] "1,1" "1,2" "1,3" "1,4" "1,5"
## [2,] "2,1" "2,2" "2,3" "2,4" "2,5"
## [3,] "3,1" "3,2" "3,3" "3,4" "3,5"
## [4,] "4,1" "4,2" "4,3" "4,4" "4,5"
## [5,] "5,1" "5,2" "5,3" "5,4" "5,5"
```

```r
#>      [,1]  [,2]  [,3]  [,4]  [,5] 
#> [1,] "1,1" "1,2" "1,3" "1,4" "1,5"
#> [2,] "2,1" "2,2" "2,3" "2,4" "2,5"
#> [3,] "3,1" "3,2" "3,3" "3,4" "3,5"
#> [4,] "4,1" "4,2" "4,3" "4,4" "4,5"
#> [5,] "5,1" "5,2" "5,3" "5,4" "5,5"

vals[c(4, 15)] #Selects the 4th and 15th elements, going down columns
```

```
## [1] "4,1" "5,3"
```


```r
select <- matrix(ncol = 2, byrow = TRUE, c(
  1, 1,
  3, 1,
  2, 4
))

select
```

```
##      [,1] [,2]
## [1,]    1    1
## [2,]    3    1
## [3,]    2    4
```

```r
vals[select]
```

```
## [1] "1,1" "3,1" "2,4"
```
Oh, cool, so these are selecting by "address"

### 4.2.6 Exercises

#### 1. Fix each of the following common data frame subsetting errors:

__Original__

```r
mtcars[mtcars$cyl = 4, ]
mtcars[-1:4, ]
mtcars[mtcars$cyl <= 5]
mtcars[mtcars$cyl == 4 | 6, ]
```

__Fixed__

```r
mtcars[mtcars$cyl == 4, ]
```

```
##                 mpg cyl  disp  hp drat    wt  qsec vs am gear carb
## Datsun 710     22.8   4 108.0  93 3.85 2.320 18.61  1  1    4    1
## Merc 240D      24.4   4 146.7  62 3.69 3.190 20.00  1  0    4    2
## Merc 230       22.8   4 140.8  95 3.92 3.150 22.90  1  0    4    2
## Fiat 128       32.4   4  78.7  66 4.08 2.200 19.47  1  1    4    1
## Honda Civic    30.4   4  75.7  52 4.93 1.615 18.52  1  1    4    2
## Toyota Corolla 33.9   4  71.1  65 4.22 1.835 19.90  1  1    4    1
## Toyota Corona  21.5   4 120.1  97 3.70 2.465 20.01  1  0    3    1
## Fiat X1-9      27.3   4  79.0  66 4.08 1.935 18.90  1  1    4    1
## Porsche 914-2  26.0   4 120.3  91 4.43 2.140 16.70  0  1    5    2
## Lotus Europa   30.4   4  95.1 113 3.77 1.513 16.90  1  1    5    2
## Volvo 142E     21.4   4 121.0 109 4.11 2.780 18.60  1  1    4    2
```

```r
mtcars[-1:-4, ]
```

```
##                      mpg cyl  disp  hp drat    wt  qsec vs am gear carb
## Hornet Sportabout   18.7   8 360.0 175 3.15 3.440 17.02  0  0    3    2
## Valiant             18.1   6 225.0 105 2.76 3.460 20.22  1  0    3    1
## Duster 360          14.3   8 360.0 245 3.21 3.570 15.84  0  0    3    4
## Merc 240D           24.4   4 146.7  62 3.69 3.190 20.00  1  0    4    2
## Merc 230            22.8   4 140.8  95 3.92 3.150 22.90  1  0    4    2
## Merc 280            19.2   6 167.6 123 3.92 3.440 18.30  1  0    4    4
## Merc 280C           17.8   6 167.6 123 3.92 3.440 18.90  1  0    4    4
## Merc 450SE          16.4   8 275.8 180 3.07 4.070 17.40  0  0    3    3
## Merc 450SL          17.3   8 275.8 180 3.07 3.730 17.60  0  0    3    3
## Merc 450SLC         15.2   8 275.8 180 3.07 3.780 18.00  0  0    3    3
## Cadillac Fleetwood  10.4   8 472.0 205 2.93 5.250 17.98  0  0    3    4
## Lincoln Continental 10.4   8 460.0 215 3.00 5.424 17.82  0  0    3    4
## Chrysler Imperial   14.7   8 440.0 230 3.23 5.345 17.42  0  0    3    4
## Fiat 128            32.4   4  78.7  66 4.08 2.200 19.47  1  1    4    1
## Honda Civic         30.4   4  75.7  52 4.93 1.615 18.52  1  1    4    2
## Toyota Corolla      33.9   4  71.1  65 4.22 1.835 19.90  1  1    4    1
## Toyota Corona       21.5   4 120.1  97 3.70 2.465 20.01  1  0    3    1
## Dodge Challenger    15.5   8 318.0 150 2.76 3.520 16.87  0  0    3    2
## AMC Javelin         15.2   8 304.0 150 3.15 3.435 17.30  0  0    3    2
## Camaro Z28          13.3   8 350.0 245 3.73 3.840 15.41  0  0    3    4
## Pontiac Firebird    19.2   8 400.0 175 3.08 3.845 17.05  0  0    3    2
## Fiat X1-9           27.3   4  79.0  66 4.08 1.935 18.90  1  1    4    1
## Porsche 914-2       26.0   4 120.3  91 4.43 2.140 16.70  0  1    5    2
## Lotus Europa        30.4   4  95.1 113 3.77 1.513 16.90  1  1    5    2
## Ford Pantera L      15.8   8 351.0 264 4.22 3.170 14.50  0  1    5    4
## Ferrari Dino        19.7   6 145.0 175 3.62 2.770 15.50  0  1    5    6
## Maserati Bora       15.0   8 301.0 335 3.54 3.570 14.60  0  1    5    8
## Volvo 142E          21.4   4 121.0 109 4.11 2.780 18.60  1  1    4    2
```

```r
mtcars[mtcars$cyl <= 5,]
```

```
##                 mpg cyl  disp  hp drat    wt  qsec vs am gear carb
## Datsun 710     22.8   4 108.0  93 3.85 2.320 18.61  1  1    4    1
## Merc 240D      24.4   4 146.7  62 3.69 3.190 20.00  1  0    4    2
## Merc 230       22.8   4 140.8  95 3.92 3.150 22.90  1  0    4    2
## Fiat 128       32.4   4  78.7  66 4.08 2.200 19.47  1  1    4    1
## Honda Civic    30.4   4  75.7  52 4.93 1.615 18.52  1  1    4    2
## Toyota Corolla 33.9   4  71.1  65 4.22 1.835 19.90  1  1    4    1
## Toyota Corona  21.5   4 120.1  97 3.70 2.465 20.01  1  0    3    1
## Fiat X1-9      27.3   4  79.0  66 4.08 1.935 18.90  1  1    4    1
## Porsche 914-2  26.0   4 120.3  91 4.43 2.140 16.70  0  1    5    2
## Lotus Europa   30.4   4  95.1 113 3.77 1.513 16.90  1  1    5    2
## Volvo 142E     21.4   4 121.0 109 4.11 2.780 18.60  1  1    4    2
```

```r
mtcars[mtcars$cyl == 4 | mtcars$cyl == 6, ]
```

```
##                 mpg cyl  disp  hp drat    wt  qsec vs am gear carb
## Mazda RX4      21.0   6 160.0 110 3.90 2.620 16.46  0  1    4    4
## Mazda RX4 Wag  21.0   6 160.0 110 3.90 2.875 17.02  0  1    4    4
## Datsun 710     22.8   4 108.0  93 3.85 2.320 18.61  1  1    4    1
## Hornet 4 Drive 21.4   6 258.0 110 3.08 3.215 19.44  1  0    3    1
## Valiant        18.1   6 225.0 105 2.76 3.460 20.22  1  0    3    1
## Merc 240D      24.4   4 146.7  62 3.69 3.190 20.00  1  0    4    2
## Merc 230       22.8   4 140.8  95 3.92 3.150 22.90  1  0    4    2
## Merc 280       19.2   6 167.6 123 3.92 3.440 18.30  1  0    4    4
## Merc 280C      17.8   6 167.6 123 3.92 3.440 18.90  1  0    4    4
## Fiat 128       32.4   4  78.7  66 4.08 2.200 19.47  1  1    4    1
## Honda Civic    30.4   4  75.7  52 4.93 1.615 18.52  1  1    4    2
## Toyota Corolla 33.9   4  71.1  65 4.22 1.835 19.90  1  1    4    1
## Toyota Corona  21.5   4 120.1  97 3.70 2.465 20.01  1  0    3    1
## Fiat X1-9      27.3   4  79.0  66 4.08 1.935 18.90  1  1    4    1
## Porsche 914-2  26.0   4 120.3  91 4.43 2.140 16.70  0  1    5    2
## Lotus Europa   30.4   4  95.1 113 3.77 1.513 16.90  1  1    5    2
## Ferrari Dino   19.7   6 145.0 175 3.62 2.770 15.50  0  1    5    6
## Volvo 142E     21.4   4 121.0 109 4.11 2.780 18.60  1  1    4    2
```

#### 2. Why does the following code yield five missing values? (Hint: why is it different from x[NA_real_]?)



```r
x <- 1:5
x[NA]
```

```
## [1] NA NA NA NA NA
```

```r
x[NA_real_]
```

```
## [1] NA
```

_I guess because [NA] is recycled and when you have an NA value it always returns NA in extraction.  But why is [NA_real_] different? Maybe because it is testing for NA_Real?

#### 3. What does upper.tri() return? How does subsetting a matrix with it work? Do we need any additional subsetting rules to describe its behaviour?


```r
x <- outer(1:5, 1:5, FUN = "*")
x
```

```
##      [,1] [,2] [,3] [,4] [,5]
## [1,]    1    2    3    4    5
## [2,]    2    4    6    8   10
## [3,]    3    6    9   12   15
## [4,]    4    8   12   16   20
## [5,]    5   10   15   20   25
```

```r
upper.tri(x)
```

```
##       [,1]  [,2]  [,3]  [,4]  [,5]
## [1,] FALSE  TRUE  TRUE  TRUE  TRUE
## [2,] FALSE FALSE  TRUE  TRUE  TRUE
## [3,] FALSE FALSE FALSE  TRUE  TRUE
## [4,] FALSE FALSE FALSE FALSE  TRUE
## [5,] FALSE FALSE FALSE FALSE FALSE
```

```r
x[upper.tri(x)]
```

```
##  [1]  2  3  6  4  8 12  5 10 15 20
```

_`upper.tri()` returns a matrix of TRUES and FALSES where TRUES correspond to the upper triangle.  We do not need any additional rules_

#### 4. Why does mtcars[1:20] return an error? How does it differ from the similar mtcars[1:20, ]?


```r
dim(mtcars)
```

```
## [1] 32 11
```

_When you are subsetting a data frame and do not include a comma, it subsets columns.  `mtcars` has less than 20 columns._

#### 5. Implement your own function that extracts the diagonal entries from a matrix (it should behave like diag(x) where x is a matrix).


```r
m <- matrix(1:32, ncol=8)
m
```

```
##      [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8]
## [1,]    1    5    9   13   17   21   25   29
## [2,]    2    6   10   14   18   22   26   30
## [3,]    3    7   11   15   19   23   27   31
## [4,]    4    8   12   16   20   24   28   32
```

```r
diag(m)
```

```
## [1]  1  6 11 16
```

```r
diag2 <- function(x) {
  end <- ifelse(nrow(x) > ncol(x), length(x), nrow(x)^2) #anyway more clever to do this?
  d <- seq(1, end, by=nrow(x)+1)
  x[d]
}

diag2(m)
```

```
## [1]  1  6 11 16
```

alternate way

```r
diag3 <- function(x) {
  select <- 1:min(dim(x))
  select <- cbind(select,select)
  x[select]
}

diag(matrix(1:32,ncol=4))
```

```
## [1]  1 10 19 28
```

```r
diag3(matrix(1:32,ncol=4))
```

```
## [1]  1 10 19 28
```

```r
diag(matrix(1:32,ncol=8))
```

```
## [1]  1  6 11 16
```

```r
diag3(matrix(1:32,ncol=8))
```

```
## [1]  1  6 11 16
```


#### 6. What does df[is.na(df)] <- 0 do? How does it work?


```r
df <- as.data.frame(m)
df
```

```
##   V1 V2 V3 V4 V5 V6 V7 V8
## 1  1  5  9 13 17 21 25 29
## 2  2  6 10 14 18 22 26 30
## 3  3  7 11 15 19 23 27 31
## 4  4  8 12 16 20 24 28 32
```

```r
df[3,2] <- NA
df
```

```
##   V1 V2 V3 V4 V5 V6 V7 V8
## 1  1  5  9 13 17 21 25 29
## 2  2  6 10 14 18 22 26 30
## 3  3 NA 11 15 19 23 27 31
## 4  4  8 12 16 20 24 28 32
```

```r
df[is.na(df)] <- 0
df
```

```
##   V1 V2 V3 V4 V5 V6 V7 V8
## 1  1  5  9 13 17 21 25 29
## 2  2  6 10 14 18 22 26 30
## 3  3  0 11 15 19 23 27 31
## 4  4  8 12 16 20 24 28 32
```
_subsets the df to positions with NA and replaces them with 0_

## 4.3

Huh?

```r
for (i in 2:length(x)) {
  out[[i]] <- fun(x[[i]], out[[i - 1]])
}
```


### Exercises 4.3.5

#### 1. Brainstorm as many ways as possible to extract the third value from the cyl variable in the mtcars dataset.


```r
mtcars[3,"cyl"]
```

```
## [1] 4
```

```r
mtcars[3,2]
```

```
## [1] 4
```

```r
mtcars$cyl[3]
```

```
## [1] 4
```

```r
mtcars[["cyl"]][3]
```

```
## [1] 4
```

```r
purrr::pluck(mtcars, "cyl", 3)
```

```
## [1] 4
```


#### 2. Given a linear model, e.g., mod <- lm(mpg ~ wt, data = mtcars), extract the residual degrees of freedom. Then extract the R squared from the model summary (summary(mod))


```r
mod <- lm(mpg ~ wt, data = mtcars)
str(mod)
```

```
## List of 12
##  $ coefficients : Named num [1:2] 37.29 -5.34
##   ..- attr(*, "names")= chr [1:2] "(Intercept)" "wt"
##  $ residuals    : Named num [1:32] -2.28 -0.92 -2.09 1.3 -0.2 ...
##   ..- attr(*, "names")= chr [1:32] "Mazda RX4" "Mazda RX4 Wag" "Datsun 710" "Hornet 4 Drive" ...
##  $ effects      : Named num [1:32] -113.65 -29.116 -1.661 1.631 0.111 ...
##   ..- attr(*, "names")= chr [1:32] "(Intercept)" "wt" "" "" ...
##  $ rank         : int 2
##  $ fitted.values: Named num [1:32] 23.3 21.9 24.9 20.1 18.9 ...
##   ..- attr(*, "names")= chr [1:32] "Mazda RX4" "Mazda RX4 Wag" "Datsun 710" "Hornet 4 Drive" ...
##  $ assign       : int [1:2] 0 1
##  $ qr           :List of 5
##   ..$ qr   : num [1:32, 1:2] -5.657 0.177 0.177 0.177 0.177 ...
##   .. ..- attr(*, "dimnames")=List of 2
##   .. .. ..$ : chr [1:32] "Mazda RX4" "Mazda RX4 Wag" "Datsun 710" "Hornet 4 Drive" ...
##   .. .. ..$ : chr [1:2] "(Intercept)" "wt"
##   .. ..- attr(*, "assign")= int [1:2] 0 1
##   ..$ qraux: num [1:2] 1.18 1.05
##   ..$ pivot: int [1:2] 1 2
##   ..$ tol  : num 1e-07
##   ..$ rank : int 2
##   ..- attr(*, "class")= chr "qr"
##  $ df.residual  : int 30
##  $ xlevels      : Named list()
##  $ call         : language lm(formula = mpg ~ wt, data = mtcars)
##  $ terms        :Classes 'terms', 'formula'  language mpg ~ wt
##   .. ..- attr(*, "variables")= language list(mpg, wt)
##   .. ..- attr(*, "factors")= int [1:2, 1] 0 1
##   .. .. ..- attr(*, "dimnames")=List of 2
##   .. .. .. ..$ : chr [1:2] "mpg" "wt"
##   .. .. .. ..$ : chr "wt"
##   .. ..- attr(*, "term.labels")= chr "wt"
##   .. ..- attr(*, "order")= int 1
##   .. ..- attr(*, "intercept")= int 1
##   .. ..- attr(*, "response")= int 1
##   .. ..- attr(*, ".Environment")=<environment: R_GlobalEnv> 
##   .. ..- attr(*, "predvars")= language list(mpg, wt)
##   .. ..- attr(*, "dataClasses")= Named chr [1:2] "numeric" "numeric"
##   .. .. ..- attr(*, "names")= chr [1:2] "mpg" "wt"
##  $ model        :'data.frame':	32 obs. of  2 variables:
##   ..$ mpg: num [1:32] 21 21 22.8 21.4 18.7 18.1 14.3 24.4 22.8 19.2 ...
##   ..$ wt : num [1:32] 2.62 2.88 2.32 3.21 3.44 ...
##   ..- attr(*, "terms")=Classes 'terms', 'formula'  language mpg ~ wt
##   .. .. ..- attr(*, "variables")= language list(mpg, wt)
##   .. .. ..- attr(*, "factors")= int [1:2, 1] 0 1
##   .. .. .. ..- attr(*, "dimnames")=List of 2
##   .. .. .. .. ..$ : chr [1:2] "mpg" "wt"
##   .. .. .. .. ..$ : chr "wt"
##   .. .. ..- attr(*, "term.labels")= chr "wt"
##   .. .. ..- attr(*, "order")= int 1
##   .. .. ..- attr(*, "intercept")= int 1
##   .. .. ..- attr(*, "response")= int 1
##   .. .. ..- attr(*, ".Environment")=<environment: R_GlobalEnv> 
##   .. .. ..- attr(*, "predvars")= language list(mpg, wt)
##   .. .. ..- attr(*, "dataClasses")= Named chr [1:2] "numeric" "numeric"
##   .. .. .. ..- attr(*, "names")= chr [1:2] "mpg" "wt"
##  - attr(*, "class")= chr "lm"
```


```r
mod$df.residual
```

```
## [1] 30
```

```r
mod.sum <- summary(mod)
str(mod.sum)
```

```
## List of 11
##  $ call         : language lm(formula = mpg ~ wt, data = mtcars)
##  $ terms        :Classes 'terms', 'formula'  language mpg ~ wt
##   .. ..- attr(*, "variables")= language list(mpg, wt)
##   .. ..- attr(*, "factors")= int [1:2, 1] 0 1
##   .. .. ..- attr(*, "dimnames")=List of 2
##   .. .. .. ..$ : chr [1:2] "mpg" "wt"
##   .. .. .. ..$ : chr "wt"
##   .. ..- attr(*, "term.labels")= chr "wt"
##   .. ..- attr(*, "order")= int 1
##   .. ..- attr(*, "intercept")= int 1
##   .. ..- attr(*, "response")= int 1
##   .. ..- attr(*, ".Environment")=<environment: R_GlobalEnv> 
##   .. ..- attr(*, "predvars")= language list(mpg, wt)
##   .. ..- attr(*, "dataClasses")= Named chr [1:2] "numeric" "numeric"
##   .. .. ..- attr(*, "names")= chr [1:2] "mpg" "wt"
##  $ residuals    : Named num [1:32] -2.28 -0.92 -2.09 1.3 -0.2 ...
##   ..- attr(*, "names")= chr [1:32] "Mazda RX4" "Mazda RX4 Wag" "Datsun 710" "Hornet 4 Drive" ...
##  $ coefficients : num [1:2, 1:4] 37.285 -5.344 1.878 0.559 19.858 ...
##   ..- attr(*, "dimnames")=List of 2
##   .. ..$ : chr [1:2] "(Intercept)" "wt"
##   .. ..$ : chr [1:4] "Estimate" "Std. Error" "t value" "Pr(>|t|)"
##  $ aliased      : Named logi [1:2] FALSE FALSE
##   ..- attr(*, "names")= chr [1:2] "(Intercept)" "wt"
##  $ sigma        : num 3.05
##  $ df           : int [1:3] 2 30 2
##  $ r.squared    : num 0.753
##  $ adj.r.squared: num 0.745
##  $ fstatistic   : Named num [1:3] 91.4 1 30
##   ..- attr(*, "names")= chr [1:3] "value" "numdf" "dendf"
##  $ cov.unscaled : num [1:2, 1:2] 0.38 -0.1084 -0.1084 0.0337
##   ..- attr(*, "dimnames")=List of 2
##   .. ..$ : chr [1:2] "(Intercept)" "wt"
##   .. ..$ : chr [1:2] "(Intercept)" "wt"
##  - attr(*, "class")= chr "summary.lm"
```


```r
mod.sum$r.squared
```

```
## [1] 0.7528328
```

### Exercises 4.5.9

#### 1. How would you randomly permute the columns of a data frame? (This is an important technique in random forests.) Can you simultaneously permute the rows and columns in one step?


```r
df <- data.frame(x=LETTERS[1:5], y=1:5, z=letters[26:22])
df
```

```
##   x y z
## 1 A 1 z
## 2 B 2 y
## 3 C 3 x
## 4 D 4 w
## 5 E 5 v
```

```r
#columns
df[,sample(ncol(df))]
```

```
##   x z y
## 1 A z 1
## 2 B y 2
## 3 C x 3
## 4 D w 4
## 5 E v 5
```

```r
#columns and rows

df[sample(nrow(df)), sample(ncol(df))]
```

```
##   y x z
## 1 1 A z
## 3 3 C x
## 2 2 B y
## 5 5 E v
## 4 4 D w
```


#### 2. How would you select a random sample of m rows from a data frame? What if the sample had to be contiguous (i.e., with an initial row, a final row, and every row in between)?

_I think the question means a random subset or random subsample_

```r
# 5 rows at random
mtcars[sample(nrow(mtcars), size = 5),]
```

```
##                      mpg cyl  disp  hp drat    wt  qsec vs am gear carb
## Porsche 914-2       26.0   4 120.3  91 4.43 2.140 16.70  0  1    5    2
## Cadillac Fleetwood  10.4   8 472.0 205 2.93 5.250 17.98  0  0    3    4
## Lincoln Continental 10.4   8 460.0 215 3.00 5.424 17.82  0  0    3    4
## Merc 230            22.8   4 140.8  95 3.92 3.150 22.90  1  0    4    2
## Toyota Corona       21.5   4 120.1  97 3.70 2.465 20.01  1  0    3    1
```

```r
# contiguous 5 rows
start <- sample(nrow(mtcars)-4)
mtcars[sample(start:(start+5), size = 5),]
```

```
## Warning in start:(start + 5): numerical expression has 28 elements: only the
## first used

## Warning in start:(start + 5): numerical expression has 28 elements: only the
## first used
```

```
##                    mpg cyl disp  hp drat    wt  qsec vs am gear carb
## Datsun 710        22.8   4  108  93 3.85 2.320 18.61  1  1    4    1
## Mazda RX4 Wag     21.0   6  160 110 3.90 2.875 17.02  0  1    4    4
## Valiant           18.1   6  225 105 2.76 3.460 20.22  1  0    3    1
## Hornet Sportabout 18.7   8  360 175 3.15 3.440 17.02  0  0    3    2
## Hornet 4 Drive    21.4   6  258 110 3.08 3.215 19.44  1  0    3    1
```


#### 3. How could you put the columns in a data frame in alphabetical order?


```r
mtcars[, order(colnames(mtcars))]
```

```
##                     am carb cyl  disp drat gear  hp  mpg  qsec vs    wt
## Mazda RX4            1    4   6 160.0 3.90    4 110 21.0 16.46  0 2.620
## Mazda RX4 Wag        1    4   6 160.0 3.90    4 110 21.0 17.02  0 2.875
## Datsun 710           1    1   4 108.0 3.85    4  93 22.8 18.61  1 2.320
## Hornet 4 Drive       0    1   6 258.0 3.08    3 110 21.4 19.44  1 3.215
## Hornet Sportabout    0    2   8 360.0 3.15    3 175 18.7 17.02  0 3.440
## Valiant              0    1   6 225.0 2.76    3 105 18.1 20.22  1 3.460
## Duster 360           0    4   8 360.0 3.21    3 245 14.3 15.84  0 3.570
## Merc 240D            0    2   4 146.7 3.69    4  62 24.4 20.00  1 3.190
## Merc 230             0    2   4 140.8 3.92    4  95 22.8 22.90  1 3.150
## Merc 280             0    4   6 167.6 3.92    4 123 19.2 18.30  1 3.440
## Merc 280C            0    4   6 167.6 3.92    4 123 17.8 18.90  1 3.440
## Merc 450SE           0    3   8 275.8 3.07    3 180 16.4 17.40  0 4.070
## Merc 450SL           0    3   8 275.8 3.07    3 180 17.3 17.60  0 3.730
## Merc 450SLC          0    3   8 275.8 3.07    3 180 15.2 18.00  0 3.780
## Cadillac Fleetwood   0    4   8 472.0 2.93    3 205 10.4 17.98  0 5.250
## Lincoln Continental  0    4   8 460.0 3.00    3 215 10.4 17.82  0 5.424
## Chrysler Imperial    0    4   8 440.0 3.23    3 230 14.7 17.42  0 5.345
## Fiat 128             1    1   4  78.7 4.08    4  66 32.4 19.47  1 2.200
## Honda Civic          1    2   4  75.7 4.93    4  52 30.4 18.52  1 1.615
## Toyota Corolla       1    1   4  71.1 4.22    4  65 33.9 19.90  1 1.835
## Toyota Corona        0    1   4 120.1 3.70    3  97 21.5 20.01  1 2.465
## Dodge Challenger     0    2   8 318.0 2.76    3 150 15.5 16.87  0 3.520
## AMC Javelin          0    2   8 304.0 3.15    3 150 15.2 17.30  0 3.435
## Camaro Z28           0    4   8 350.0 3.73    3 245 13.3 15.41  0 3.840
## Pontiac Firebird     0    2   8 400.0 3.08    3 175 19.2 17.05  0 3.845
## Fiat X1-9            1    1   4  79.0 4.08    4  66 27.3 18.90  1 1.935
## Porsche 914-2        1    2   4 120.3 4.43    5  91 26.0 16.70  0 2.140
## Lotus Europa         1    2   4  95.1 3.77    5 113 30.4 16.90  1 1.513
## Ford Pantera L       1    4   8 351.0 4.22    5 264 15.8 14.50  0 3.170
## Ferrari Dino         1    6   6 145.0 3.62    5 175 19.7 15.50  0 2.770
## Maserati Bora        1    8   8 301.0 3.54    5 335 15.0 14.60  0 3.570
## Volvo 142E           1    2   4 121.0 4.11    4 109 21.4 18.60  1 2.780
```

