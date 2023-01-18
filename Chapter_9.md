---
title: "Chapter 9"
author: "Julin Maloof"
date: "2023-01-16"
output: 
  html_document: 
    keep_md: yes
---



# Chapter 9


```r
library(tidyverse)
```

```
## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
## ✔ ggplot2 3.3.6      ✔ purrr   0.3.4 
## ✔ tibble  3.1.8      ✔ dplyr   1.0.10
## ✔ tidyr   1.2.0      ✔ stringr 1.4.1 
## ✔ readr   2.1.2      ✔ forcats 0.5.2 
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
```


## 9.2.6 Exercises

### 1. 
Use `as_mapper()` to explore how purrr generates anonymous functions for the integer, character, and list helpers. What helper allows you to extract attributes? Read the documentation to find out.


```r
f <- as_mapper(~ . + 1)
f
```

```
## <lambda>
## function (..., .x = ..1, .y = ..2, . = ..1) 
## . + 1
## attr(,"class")
## [1] "rlang_lambda_function" "function"
```

```r
f(1:3)
```

```
## [1] 2 3 4
```

```r
f <- as_mapper(2)
f
```

```
## function (x, ...) 
## pluck(x, 2, .default = NULL)
## <environment: 0x123bca7e0>
```

```r
f(3:1)
```

```
## [1] 2
```



```r
f <- as_mapper(c("a", "b", "c"))
f
```

```
## function (x, ...) 
## pluck(x, "a", "b", "c", .default = NULL)
## <environment: 0x123ecf790>
```

```r
f(1:3)
```

```
## NULL
```

```r
f(c(a=4,b=3, c=1))
```

```
## NULL
```

```r
f(list(a=4,b=3,c=1))
```

```
## NULL
```

```r
f(list(a=list(b=list(c=23))))
```

```
## [1] 23
```
Ahhh, each argument takes you one level deeper into the list

Can use attr_getter


```r
f <- attr_getter("class")
f
```

```
## function (x) 
## attr(x, attr, exact = TRUE)
## <bytecode: 0x1242aeb10>
## <environment: 0x1242ae7c8>
```

```r
f(mpg)
```

```
## [1] "tbl_df"     "tbl"        "data.frame"
```


### 2.
map(1:3, ~ runif(2)) is a useful pattern for generating random numbers, but map(1:3, runif(2)) is not. Why not? Can you explain why it returns the result that it does?


```r
map(1:3, ~ runif(2))
```

```
## [[1]]
## [1] 0.9941078 0.9550497
## 
## [[2]]
## [1] 0.2377857 0.7361001
## 
## [[3]]
## [1] 0.04844734 0.12811884
```

```r
map(1:3, runif(2))
```

```
## [[1]]
## NULL
## 
## [[2]]
## NULL
## 
## [[3]]
## NULL
```

```r
as_mapper(runif(2))
```

```
## function (x, ...) 
## pluck(x, 0.803714730776846, 0.556438132189214, .default = NULL)
## <environment: 0x124819040>
```

_In the second form, the output from `runif(2)` is taken as input to `as_mapper` and interpreted as positions? names? to extract 


### 3.
Use the appropriate map() function to:

a. Compute the standard deviation of every column in a numeric data frame.


```r
df <- as_data_frame(matrix(rnorm(100),ncol=5))
```

```
## Warning: `as_data_frame()` was deprecated in tibble 2.0.0.
## Please use `as_tibble()` instead.
## The signature and semantics have changed, see `?as_tibble`.
## This warning is displayed once every 8 hours.
## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was generated.
```

```
## Warning: The `x` argument of `as_tibble.matrix()` must have unique column names if `.name_repair` is omitted as of tibble 2.0.0.
## Using compatibility `.name_repair`.
## This warning is displayed once every 8 hours.
## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was generated.
```

```r
df
```

```
## # A tibble: 20 × 5
##         V1       V2     V3     V4      V5
##      <dbl>    <dbl>  <dbl>  <dbl>   <dbl>
##  1  0.638  -0.00110  0.287 -2.71   0.852 
##  2 -1.31   -0.856   -0.628 -1.38  -1.76  
##  3  1.45    0.847    0.154  0.666  1.95  
##  4 -0.329   0.0672  -0.515 -1.09  -0.0322
##  5  1.41   -2.18    -0.121 -1.15  -0.285 
##  6  1.34   -0.650   -0.628  1.27  -0.892 
##  7 -1.00   -0.879   -1.27  -0.292  1.30  
##  8  0.259   0.591   -0.878  0.212 -2.15  
##  9  0.487  -0.307    0.137  0.743  0.200 
## 10  0.934  -0.728    1.38  -1.79   1.30  
## 11 -0.514   0.421   -2.10  -1.29  -1.71  
## 12 -0.959  -0.412    0.625 -0.526  1.21  
## 13 -0.0189 -0.623    0.953 -0.641  0.227 
## 14 -0.0320  0.226   -1.12  -0.226  0.0501
## 15  0.138  -1.06    -1.17   0.585 -0.0646
## 16  0.387  -2.22     0.234  0.389  0.701 
## 17  0.0692 -0.323    1.33  -1.15   0.736 
## 18 -1.30   -0.648    1.21   0.735  1.21  
## 19  0.272  -1.11    -1.69   0.423 -1.77  
## 20 -0.508   1.45     0.971 -0.539 -0.578
```

```r
map_dbl(df, sd)
```

```
##        V1        V2        V3        V4        V5 
## 0.8455983 0.9037792 1.0384383 1.0240635 1.1909832
```


b. Compute the standard deviation of every numeric column in a mixed data frame. (Hint: you’ll need to do it in two steps.)


```r
columns <- map_lgl(mpg, is.numeric)
map_dbl(mpg[columns], sd)
```

```
##    displ     year      cyl      cty      hwy 
## 1.291959 4.509646 1.611534 4.255946 5.954643
```
Or


```r
mpg %>%
  summarize(across(.cols = where(is.numeric), sd ))
```

```
## # A tibble: 1 × 5
##   displ  year   cyl   cty   hwy
##   <dbl> <dbl> <dbl> <dbl> <dbl>
## 1  1.29  4.51  1.61  4.26  5.95
```

c. Compute the number of levels for every factor in a data frame.



```r
columns <- map_lgl(iris, is.factor)
map_dbl(iris[columns], nlevels)
```

```
## Species 
##       3
```

### 4.
The following code simulates the performance of a t-test for non-normal data. Extract the p-value from each test, then visualise.

```r
trials <- map(1:100, ~ t.test(rpois(10, 10), rpois(7, 10)))

map_dbl(trials, "p.value") %>% 
  hist()
```

![](Chapter_9_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

### 5.
The following code uses a map nested inside another map to apply a function to every element of a nested list. Why does it fail, and what do you need to do to make it work?


```r
x <- list(
  list(1, c(3, 9)),
  list(c(3, 6), 7, c(4, 7, 6))
)

triple <- function(x) x * 3

map(x, ~ map(.x, .f= triple))
```

```
## [[1]]
## [[1]][[1]]
## [1] 3
## 
## [[1]][[2]]
## [1]  9 27
## 
## 
## [[2]]
## [[2]][[1]]
## [1]  9 18
## 
## [[2]][[2]]
## [1] 21
## 
## [[2]][[3]]
## [1] 12 21 18
```

```r
#map(x, map, .f = triple)
#> Error in .f(.x[[i]], ...): unused argument (function (.x, .f, ...)
#> {
#> .f <- as_mapper(.f, ...)
#> .Call(map_impl, environment(), ".x", ".f", "list")
#> })
```
_The problem was that the ".f" argument was going to the outer map._

### 6.

Use map() to fit linear models to the mtcars dataset using the formulas stored in this list:


```r
formulas <- list(
  mpg ~ disp,
  mpg ~ I(1 / disp),
  mpg ~ disp + wt,
  mpg ~ I(1 / disp) + wt
)

map(formulas, lm, data=mtcars)
```

```
## [[1]]
## 
## Call:
## .f(formula = .x[[i]], data = ..1)
## 
## Coefficients:
## (Intercept)         disp  
##    29.59985     -0.04122  
## 
## 
## [[2]]
## 
## Call:
## .f(formula = .x[[i]], data = ..1)
## 
## Coefficients:
## (Intercept)    I(1/disp)  
##       10.75      1557.67  
## 
## 
## [[3]]
## 
## Call:
## .f(formula = .x[[i]], data = ..1)
## 
## Coefficients:
## (Intercept)         disp           wt  
##    34.96055     -0.01772     -3.35083  
## 
## 
## [[4]]
## 
## Call:
## .f(formula = .x[[i]], data = ..1)
## 
## Coefficients:
## (Intercept)    I(1/disp)           wt  
##      19.024     1142.560       -1.798
```

Better...


```r
tib <- tibble(formulas = list(
  mpg ~ disp,
  mpg ~ I(1 / disp),
  mpg ~ disp + wt,
  mpg ~ I(1 / disp) + wt
))

tib <- tib %>% 
  mutate(lm = map(formulas, lm, mtcars))

tib %>% mutate(glance = map(lm, broom::glance)) %>% unnest(glance)
```

```
## # A tibble: 4 × 14
##   formulas  lm     r.squared adj.r.s…¹ sigma stati…²  p.value    df logLik   AIC
##   <list>    <list>     <dbl>     <dbl> <dbl>   <dbl>    <dbl> <dbl>  <dbl> <dbl>
## 1 <formula> <lm>       0.718     0.709  3.25    76.5 9.38e-10     1  -82.1  170.
## 2 <formula> <lm>       0.860     0.855  2.29   184.  2.49e-14     1  -71.0  148.
## 3 <formula> <lm>       0.781     0.766  2.92    51.7 2.74e-10     2  -78.1  164.
## 4 <formula> <lm>       0.884     0.876  2.12   110.  2.79e-14     2  -67.9  144.
## # … with 4 more variables: BIC <dbl>, deviance <dbl>, df.residual <int>,
## #   nobs <int>, and abbreviated variable names ¹​adj.r.squared, ²​statistic
```

```r
tib %>% mutate(tidy = map(lm, broom::tidy)) %>% unnest(tidy)
```

```
## # A tibble: 10 × 7
##    formulas  lm     term         estimate std.error statistic  p.value
##    <list>    <list> <chr>           <dbl>     <dbl>     <dbl>    <dbl>
##  1 <formula> <lm>   (Intercept)   29.6      1.23        24.1  3.58e-21
##  2 <formula> <lm>   disp          -0.0412   0.00471     -8.75 9.38e-10
##  3 <formula> <lm>   (Intercept)   10.8      0.799       13.5  3.06e-14
##  4 <formula> <lm>   I(1/disp)   1558.     115.          13.6  2.49e-14
##  5 <formula> <lm>   (Intercept)   35.0      2.16        16.2  4.91e-16
##  6 <formula> <lm>   disp          -0.0177   0.00919     -1.93 6.36e- 2
##  7 <formula> <lm>   wt            -3.35     1.16        -2.88 7.43e- 3
##  8 <formula> <lm>   (Intercept)   19.0      3.45         5.51 6.13e- 6
##  9 <formula> <lm>   I(1/disp)   1143.     200.           5.72 3.47e- 6
## 10 <formula> <lm>   wt            -1.80     0.733       -2.45 2.04e- 2
```

### 7.

Fit the model mpg ~ disp to each of the bootstrap replicates of mtcars in the list below, then extract the $R^2$ of the model fit (Hint: you can compute the $R^2$ with summary().)


```r
bootstrap <- function(df) {
  df[sample(nrow(df), replace = TRUE), , drop = FALSE]
}

bootstraps <- map(1:10, ~ bootstrap(mtcars))

map(bootstraps, ~ lm(mpg ~ disp, .x)) %>%
  map(summary) %>%
  map_dbl("r.squared")
```

```
##  [1] 0.7283290 0.7093970 0.8260510 0.7229403 0.7655010 0.7077993 0.7991398
##  [8] 0.7447954 0.7183999 0.7278988
```

## 9.4.6 Exercises

### 1.
Explain the results of modify(mtcars, 1).


```r
mtcars
```

```
##                      mpg cyl  disp  hp drat    wt  qsec vs am gear carb
## Mazda RX4           21.0   6 160.0 110 3.90 2.620 16.46  0  1    4    4
## Mazda RX4 Wag       21.0   6 160.0 110 3.90 2.875 17.02  0  1    4    4
## Datsun 710          22.8   4 108.0  93 3.85 2.320 18.61  1  1    4    1
## Hornet 4 Drive      21.4   6 258.0 110 3.08 3.215 19.44  1  0    3    1
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
modify(mtcars, 1)
```

```
##                     mpg cyl disp  hp drat   wt  qsec vs am gear carb
## Mazda RX4            21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Mazda RX4 Wag        21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Datsun 710           21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Hornet 4 Drive       21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Hornet Sportabout    21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Valiant              21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Duster 360           21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Merc 240D            21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Merc 230             21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Merc 280             21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Merc 280C            21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Merc 450SE           21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Merc 450SL           21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Merc 450SLC          21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Cadillac Fleetwood   21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Lincoln Continental  21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Chrysler Imperial    21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Fiat 128             21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Honda Civic          21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Toyota Corolla       21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Toyota Corona        21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Dodge Challenger     21   6  160 110  3.9 2.62 16.46  0  1    4    4
## AMC Javelin          21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Camaro Z28           21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Pontiac Firebird     21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Fiat X1-9            21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Porsche 914-2        21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Lotus Europa         21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Ford Pantera L       21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Ferrari Dino         21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Maserati Bora        21   6  160 110  3.9 2.62 16.46  0  1    4    4
## Volvo 142E           21   6  160 110  3.9 2.62 16.46  0  1    4    4
```
_clearly it is taking the first row, but keeps the rownames.  I would have thought it would keep the first column._

_Oh, I see it is going to go through each item in the last (each column) and then take the first item there_


### 2.
Rewrite the following code to use iwalk() instead of walk2(). What are the advantages and disadvantages?


```r
temp <- "./temp"
cyls <- split(mtcars, mtcars$cyl)
paths <- file.path(temp, paste0("cyl-", names(cyls), ".csv"))
walk2(cyls, paths, write.csv)
```


```r
cyls <- split(mtcars, mtcars$cyl)
iwalk(cyls, ~ write.csv(.x, file.path(temp, paste0("cyl-", .y, ".csv"))))
```

_fewer lines and variable, but maybe harder to read_

### 3. Explain how the following code transforms a data frame using functions stored in a list.


```r
trans <- list(
  disp = function(x) x * 0.0163871,
  am = function(x) factor(x, labels = c("auto", "manual"))
)

nm <- names(trans)
mtcars[nm] <- map2(trans, mtcars[nm], function(f, var) f(var))
```

_trans is a list of 2 functions; mtcars[nm] is a list of 2 columns.  map applies the first function to the first column, etc_.

Compare and contrast the map2() approach to this map() approach:


```r
mtcars[nm] <- map(nm, ~ trans[[.x]](mtcars[[.x]]))
```

_ugly!_

### 4.
What does write.csv() return, i.e. what happens if you use it with map2() instead of walk2()?


```r
temp <- "./temp"
cyls <- split(mtcars, mtcars$cyl)
paths <- file.path(temp, paste0("cyl-", names(cyls), ".csv"))
map2(cyls, paths, write.csv)
```

```
## $`4`
## NULL
## 
## $`6`
## NULL
## 
## $`8`
## NULL
```

_NULL_
