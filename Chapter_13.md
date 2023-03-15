---
title: "Chapter 13"
author: "Julin Maloof"
date: "2023-03-04"
output: 
  html_document: 
    keep_md: yes
---




```r
library(sloop)
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

## 13.2 Basics

S3 objects have at least a class attribute


```r
f <- factor(c("a", "b", "c"))

typeof(f)
```

```
## [1] "integer"
```

```r
attributes(f)
```

```
## $levels
## [1] "a" "b" "c"
## 
## $class
## [1] "factor"
```
get the basetype by `unclassing`

```r
unclass(f)
```

```
## [1] 1 2 3
## attr(,"levels")
## [1] "a" "b" "c"
```

S3 objects behave differently from their base types when passed to a generic funciton aka __genereic__.

Can check if a function is a generic using `sloop::ftype()`


```r
ftype(print)
```

```
## [1] "S3"      "generic"
```

```r
ftype(summary)
```

```
## [1] "S3"      "generic"
```

```r
ftype(unclass)
```

```
## [1] "primitive"
```

generic is a middleman, matching a class to the correct implementation (method) for the function.  This is called __method dispatch__


```r
s3_dispatch(print(f))
```

```
## => print.factor
##  * print.default
```

To see a method, use `sloop::s3_get_method()`


```r
s3_get_method(weighted.mean.Date)
```

```
## function (x, w, ...) 
## .Date(weighted.mean(unclass(x), w, ...))
## <bytecode: 0x12b083e40>
## <environment: namespace:stats>
```



```r
s3_methods_generic("print")
```

```
## # A tibble: 306 × 4
##    generic class    visible source             
##    <chr>   <chr>    <lgl>   <chr>              
##  1 print   acf      FALSE   registered S3method
##  2 print   AES      FALSE   registered S3method
##  3 print   all_vars FALSE   registered S3method
##  4 print   anova    FALSE   registered S3method
##  5 print   any_vars FALSE   registered S3method
##  6 print   aov      FALSE   registered S3method
##  7 print   aovlist  FALSE   registered S3method
##  8 print   ar       FALSE   registered S3method
##  9 print   Arima    FALSE   registered S3method
## 10 print   arima0   FALSE   registered S3method
## # … with 296 more rows
```

### 13.2.1 Exercises

#### 1. Describe the difference between t.test() and t.data.frame(). When is each function called?

_`t.test()` is a generic, so it is called when the user calls`t.test()` and then it will dispatch the correct method.  `t.dta.frame()` is a method invoked when the user calls `t()` on a dataframe object._


```r
ftype(t.test)
```

```
## [1] "S3"      "generic"
```

```r
ftype(t.data.frame)
```

```
## [1] "S3"     "method"
```

```r
s3_methods_generic("t.test")
```

```
## # A tibble: 2 × 4
##   generic class   visible source             
##   <chr>   <chr>   <lgl>   <chr>              
## 1 t.test  default FALSE   registered S3method
## 2 t.test  formula FALSE   registered S3method
```

```r
s3_methods_generic("t")
```

```
## # A tibble: 6 × 4
##   generic class      visible source             
##   <chr>   <chr>      <lgl>   <chr>              
## 1 t       data.frame TRUE    base               
## 2 t       default    TRUE    base               
## 3 t       gtable     FALSE   registered S3method
## 4 t       ts         FALSE   registered S3method
## 5 t       vctrs_sclr FALSE   registered S3method
## 6 t       vctrs_vctr FALSE   registered S3method
```

#### 2. Make a list of commonly used base R functions that contain . in their name but are not S3 methods.


```r
fxns <- tibble(fn_name={ls("package:base") %>%
  str_subset(pattern=fixed("."))}) %>%
  filter(map_lgl(fn_name, ~ {get(.) %>% is_function()})) %>%
  filter(!map_lgl(fn_name, is_s3_method))

fxns$fn_name
```

```
##   [1] "!.hexmode"                "!.octmode"               
##   [3] "all.equal"                "all.names"               
##   [5] "all.vars"                 "as.array"                
##   [7] "as.call"                  "as.character"            
##   [9] "as.complex"               "as.data.frame"           
##  [11] "as.Date"                  "as.difftime"             
##  [13] "as.double"                "as.environment"          
##  [15] "as.expression"            "as.factor"               
##  [17] "as.function"              "as.hexmode"              
##  [19] "as.integer"               "as.list"                 
##  [21] "as.logical"               "as.matrix"               
##  [23] "as.name"                  "as.null"                 
##  [25] "as.numeric"               "as.numeric_version"      
##  [27] "as.octmode"               "as.ordered"              
##  [29] "as.package_version"       "as.pairlist"             
##  [31] "as.POSIXct"               "as.POSIXlt"              
##  [33] "as.qr"                    "as.raw"                  
##  [35] "as.single"                "as.symbol"               
##  [37] "as.table"                 "as.vector"               
##  [39] "attr.all.equal"           "char.expand"             
##  [41] "data.class"               "data.frame"              
##  [43] "data.matrix"              "default.stringsAsFactors"
##  [45] "dir.create"               "dir.exists"              
##  [47] "dyn.load"                 "dyn.unload"              
##  [49] "env.profile"              "eval.parent"             
##  [51] "expand.grid"              "file.access"             
##  [53] "file.append"              "file.choose"             
##  [55] "file.copy"                "file.create"             
##  [57] "file.exists"              "file.info"               
##  [59] "file.link"                "file.mode"               
##  [61] "file.mtime"               "file.path"               
##  [63] "file.remove"              "file.rename"             
##  [65] "file.show"                "file.size"               
##  [67] "file.symlink"             "find.package"            
##  [69] "format.info"              "format.pval"             
##  [71] "gc.time"                  "inverse.rle"             
##  [73] "is.array"                 "is.atomic"               
##  [75] "is.call"                  "is.character"            
##  [77] "is.complex"               "is.data.frame"           
##  [79] "is.double"                "is.element"              
##  [81] "is.environment"           "is.expression"           
##  [83] "is.factor"                "is.finite"               
##  [85] "is.function"              "is.infinite"             
##  [87] "is.integer"               "is.language"             
##  [89] "is.list"                  "is.loaded"               
##  [91] "is.logical"               "is.matrix"               
##  [93] "is.na"                    "is.na<-"                 
##  [95] "is.name"                  "is.nan"                  
##  [97] "is.null"                  "is.numeric"              
##  [99] "is.numeric_version"       "is.object"               
## [101] "is.ordered"               "is.package_version"      
## [103] "is.pairlist"              "is.primitive"            
## [105] "is.qr"                    "is.R"                    
## [107] "is.raw"                   "is.recursive"            
## [109] "is.single"                "is.symbol"               
## [111] "is.table"                 "is.unsorted"             
## [113] "is.vector"                "La.svd"                  
## [115] "library.dynam"            "library.dynam.unload"    
## [117] "list.dirs"                "list.files"              
## [119] "lower.tri"                "make.names"              
## [121] "make.unique"              "margin.table"            
## [123] "mat.or.vec"               "match.arg"               
## [125] "match.call"               "match.fun"               
## [127] "Math.data.frame"          "Math.Date"               
## [129] "Math.difftime"            "Math.factor"             
## [131] "Math.POSIXt"              "max.col"                 
## [133] "mem.maxNSize"             "mem.maxVSize"            
## [135] "memory.profile"           "new.env"                 
## [137] "on.exit"                  "Ops.data.frame"          
## [139] "Ops.Date"                 "Ops.difftime"            
## [141] "Ops.factor"               "Ops.numeric_version"     
## [143] "Ops.ordered"              "Ops.POSIXt"              
## [145] "parent.env"               "parent.env<-"            
## [147] "parent.frame"             "path.expand"             
## [149] "path.package"             "pmax.int"                
## [151] "pmin.int"                 "pos.to.env"              
## [153] "proc.time"                "prop.table"              
## [155] "qr.coef"                  "qr.fitted"               
## [157] "qr.Q"                     "qr.qty"                  
## [159] "qr.qy"                    "qr.R"                    
## [161] "qr.resid"                 "qr.solve"                
## [163] "qr.X"                     "R.home"                  
## [165] "R.Version"                "read.dcf"                
## [167] "reg.finalizer"            "rep.int"                 
## [169] "row.names"                "row.names<-"             
## [171] "sample.int"               "save.image"              
## [173] "seq.int"                  "set.seed"                
## [175] "sink.number"              "sort.int"                
## [177] "sort.list"                "storage.mode"            
## [179] "storage.mode<-"           "Summary.data.frame"      
## [181] "Summary.Date"             "Summary.difftime"        
## [183] "Summary.factor"           "Summary.numeric_version" 
## [185] "Summary.ordered"          "Summary.POSIXct"         
## [187] "Summary.POSIXlt"          "sys.call"                
## [189] "sys.calls"                "Sys.chmod"               
## [191] "Sys.Date"                 "sys.frame"               
## [193] "sys.frames"               "sys.function"            
## [195] "Sys.getenv"               "Sys.getlocale"           
## [197] "Sys.getpid"               "Sys.glob"                
## [199] "Sys.info"                 "sys.load.image"          
## [201] "Sys.localeconv"           "sys.nframe"              
## [203] "sys.on.exit"              "sys.parent"              
## [205] "sys.parents"              "Sys.readlink"            
## [207] "sys.save.image"           "Sys.setenv"              
## [209] "Sys.setFileTime"          "Sys.setLanguage"         
## [211] "Sys.setlocale"            "Sys.sleep"               
## [213] "sys.source"               "sys.status"              
## [215] "Sys.time"                 "Sys.timezone"            
## [217] "Sys.umask"                "Sys.unsetenv"            
## [219] "Sys.which"                "system.file"             
## [221] "system.time"              "unix.time"               
## [223] "upper.tri"                "which.max"               
## [225] "which.min"                "write.dcf"               
## [227] "xpdrows.data.frame"
```


#### 3. What does the as.data.frame.data.frame() method do? Why is it confusing? How could you avoid this confusion in your own code?


```r
as.data.frame.data.frame
```

```
## function (x, row.names = NULL, ...) 
## {
##     cl <- oldClass(x)
##     i <- match("data.frame", cl)
##     if (i > 1L) 
##         class(x) <- cl[-(1L:(i - 1L))]
##     if (!is.null(row.names)) {
##         nr <- .row_names_info(x, 2L)
##         if (length(row.names) == nr) 
##             attr(x, "row.names") <- row.names
##         else stop(sprintf(ngettext(nr, "invalid 'row.names', length %d for a data frame with %d row", 
##             "invalid 'row.names', length %d for a data frame with %d rows"), 
##             length(row.names), nr), domain = NA)
##     }
##     x
## }
## <bytecode: 0x12b41bfc8>
## <environment: namespace:base>
```
`as.data.frame.data.frame` removes any class info that preceeds the data.frame class.  It also does some checking to make sure that the number of rownames matches the number of rows and throws an error if it is incorrect.  

Not sure how to make this less confusing.

#### 4. Describe the difference in behaviour in these two calls.

```r
set.seed(1014)
some_days <- as.Date("2017-01-31") + sample(10, 5)

mean(some_days)
```

```
## [1] "2017-02-06"
```

```r
#> [1] "2017-02-06"
mean(unclass(some_days))
```

```
## [1] 17203.4
```

```r
#> [1] 17203


attributes(some_days)
```

```
## $class
## [1] "Date"
```

```r
s3_dispatch(mean(some_days))
```

```
## => mean.Date
##  * mean.default
```

```r
s3_dispatch(mean(unclass(some_days)))
```

```
##    mean.double
##    mean.numeric
## => mean.default
```
The first call used the `mean.Date` method, the second `mean.default`

#### 5. What class of object does the following code return? What base type is it built on? What attributes does it use?


```r
x <- ecdf(rpois(100, 10))
x
```

```
## Empirical CDF 
## Call: ecdf(rpois(100, 10))
##  x[1:18] =      2,      3,      4,  ...,     18,     19
```

```r
#> Empirical CDF 
#> Call: ecdf(rpois(100, 10))
#>  x[1:18] =  2,  3,  4,  ..., 2e+01, 2e+01
#>  

attributes(x)
```

```
## $class
## [1] "ecdf"     "stepfun"  "function"
## 
## $call
## ecdf(rpois(100, 10))
```

```r
is_function(x)
```

```
## [1] TRUE
```

```r
x(5)
```

```
## [1] 0.06
```
this returns a function based on a stepfun.


#### 6. What class of object does the following code return? What base type is it built on? What attributes does it use?


```r
x <- table(rpois(100, 5))
x
```

```
## 
##  1  2  3  4  5  6  7  8  9 10 
##  7  5 18 14 15 15 14  4  5  3
```

```r
#> 
#>  1  2  3  4  5  6  7  8  9 10 
#>  7  5 18 14 15 15 14  4  5  3

attributes(x)
```

```
## $dim
## [1] 10
## 
## $dimnames
## $dimnames[[1]]
##  [1] "1"  "2"  "3"  "4"  "5"  "6"  "7"  "8"  "9"  "10"
## 
## 
## $class
## [1] "table"
```

## 13.3 Classes

To make an obkect of an instance of a class, set the class attribute.  Either

* use `structure` to create it
* use `class` to set it.


```r
# Create and assign class in one step
x <- structure(list(), class = "my_class")

# Create, then set class
x <- list()
class(x) <- "my_class"
```

To check on a class of an object:

* use `class` or `inherits`


```r
class(x)
```

```
## [1] "my_class"
```

```r
#> [1] "my_class"
inherits(x, "my_class")
```

```
## [1] TRUE
```

```r
#> [1] TRUE
inherits(x, "your_class")
```

```
## [1] FALSE
```

```r
#> [1] FALSE
```

### 13.3.1 Constructors

Because S3 doesn't provide formal class definitions, we need to handle that ourselves.  That is, we have to make sure that each instance has the same strucure (same base type and same attributes with the same types).  To do that we build our own constructor.

The constructor should follow three principles:

* Be called `new_myclass` (where myclass changes to reflect the class name)
* Have one argument for the base object and one for each attribute.
* Check the types of the base object and attributes.

Example:


```r
new_Date <- function(x = double()) {
  stopifnot(is.double(x))
  structure(x, class = "Date")
}

new_Date(c(-1, 0, 1))
```

```
## [1] "1969-12-31" "1970-01-01" "1970-01-02"
```

```r
#> [1] "1969-12-31" "1970-01-01" "1970-01-02"
```

Constructors are meant for internal use.  If you are developing a package and users will be making new objects of the given class, you need a helper function.

### 13.3.2 Validators


```r
new_factor <- function(x = integer(), levels = character()) {
  stopifnot(is.integer(x))
  stopifnot(is.character(levels))

  structure(
    x,
    levels = levels,
    class = "factor"
  )
}

new_factor(1:5, "a")
```

```
## Error in as.character.factor(x): malformed factor
```

```r
#> Error in as.character.factor(x): malformed factor
new_factor(0:1, "a")
```

```
## Error in as.character.factor(x): malformed factor
```

```r
#> Error in as.character.factor(x): malformed factor
```



```r
validate_factor <- function(x) {
  values <- unclass(x)
  levels <- attr(x, "levels")

  if (!all(!is.na(values) & values > 0)) {
    stop(
      "All `x` values must be non-missing and greater than zero",
      call. = FALSE
    )
  }

  if (length(levels) < max(values)) {
    stop(
      "There must be at least as many `levels` as possible values in `x`",
      call. = FALSE
    )
  }

  x
}

validate_factor(new_factor(1:5, "a"))
```

```
## Error: There must be at least as many `levels` as possible values in `x`
```

```r
#> Error: There must be at least as many `levels` as possible values in `x`
validate_factor(new_factor(0:1, "a"))
```

```
## Error: All `x` values must be non-missing and greater than zero
```

```r
#> Error: All `x` values must be non-missing and greater than zero
```

Why does the outer function have the opportunity to check the output of the inner function since its should already throw an error?

### 13.3.3 Helpers

Helpers help users create objects of a given class

A helper should:

* Have the same name as the class
* finish by calling the constructor, and the validator if it exists,
* Create carefully crafter error messages
* Have a thoughtfully crafter user interface with good defaults

### 13.3.4 Exercises
#### 1. Write a constructor for data.frame objects. What base type is a data frame built on? What attributes does it use? What are the restrictions placed on the individual elements? What about the names?

attributes are names, row.names, and class


```r
new_data.frame <- function(..., row.names=NULL, check.names=TRUE) {
  df <- list(...)
 # df <- list(a=1:10, b=5)
  lengths <- sapply(df, length)
  nrow <- max(lengths)
  if (!all(lengths==nrow)) {
    df[lengths!=nrow] <- lapply(df[lengths!=nrow], rep, length.out=nrow)
  }
  if(check.names) names(df) <- make.names(names(df), unique=TRUE)
  if(is.null(row.names)) row.names<-1:nrow
  class(df) <- "data.frame"
  attr(df, "row.names") <- row.names
  df
}

mydf <- new_data.frame(a=1:10, b=5, c("a","b"), d=LETTERS[1:10], "JM")

mydf
```

```
##     a b X d X.1
## 1   1 5 a A  JM
## 2   2 5 b B  JM
## 3   3 5 a C  JM
## 4   4 5 b D  JM
## 5   5 5 a E  JM
## 6   6 5 b F  JM
## 7   7 5 a G  JM
## 8   8 5 b H  JM
## 9   9 5 a I  JM
## 10 10 5 b J  JM
```

```r
attributes(mydf)
```

```
## $names
## [1] "a"   "b"   "X"   "d"   "X.1"
## 
## $class
## [1] "data.frame"
## 
## $row.names
##  [1]  1  2  3  4  5  6  7  8  9 10
```
There are many things that I am not dealing with here, including, what if the user passes matrices, lists, data.frames, etc, to the constuctor.  Plus I am handling ragged lengths differently from default.

#### 2. Enhance my factor() helper to have better behaviour when one or more values is not found in levels. What does base::factor() do in this situation?



```r
base::factor(1:10, levels=c(1:9) )
```

```
##  [1] 1    2    3    4    5    6    7    8    9    <NA>
## Levels: 1 2 3 4 5 6 7 8 9
```

```r
base::factor(1:10, levels=1:9)  %>% as.integer()
```

```
##  [1]  1  2  3  4  5  6  7  8  9 NA
```

The base function will convert that value to an NA


```r
new_factor <- function(x = integer(), levels = character()) {
  stopifnot(is.integer(x))
  stopifnot(is.character(levels))

  structure(
    x,
    levels = levels,
    class = "factor"
  )
}

validate_factor <- function(x) {
  values <- unclass(x)
  levels <- attr(x, "levels")

  if (!all(!is.na(values) & values > 0)) {
    stop(
      "All `x` values must be non-missing and greater than zero",
      call. = FALSE
    )
  }

  if (length(levels) < max(values)) {
    stop(
      "There must be at least as many `levels` as possible values in `x`",
      call. = FALSE
    )
  }

  x
}

validate_factor2 <- function(x) {
  values <- unclass(x)
  levels <- attr(x, "levels")

  if (!all(na.omit(values) > 0)) {
    stop(
      "All `x` values must be greater than zero",
      call. = FALSE
    )
  }

  if (length(levels) < max(values, na.rm = TRUE)) {
    stop(
      "There must be at least as many `levels` as possible values in `x`",
      call. = FALSE
    )
  }

  x
}

factor <- function(x = character(), levels = unique(x)) {
  ind <- match(x, levels)
  validate_factor(new_factor(ind, levels))
}


factor2 <- function(x = character(), levels = unique(x)) {
  ind <- match(x, levels)
  validate_factor2(new_factor(ind, levels))
}
```


```r
base::factor(1:10, as.character(9:1))
```

```
##  [1] 1    2    3    4    5    6    7    8    9    <NA>
## Levels: 9 8 7 6 5 4 3 2 1
```

```r
factor2(1:10, as.character(9:1))
```

```
##  [1] 1    2    3    4    5    6    7    8    9    <NA>
## Levels: 9 8 7 6 5 4 3 2 1
```
Hmmm I ended up changing the validtaion function, not the helper function.  probably I did something wrong.

#### 3. Carefully read the source code of factor(). What does it do that my constructor does not?

Deals with ordeered, deals with exclude, but probably he is getting at something else.

#### 4. Factors have an optional “contrasts” attribute. Read the help for C(), and briefly describe the purpose of the attribute. What type should it have? Rewrite the new_factor() constructor to include this attribute.

The attribute determiens the type of statistical contrasts applied in aov and lm


```r
new_factor <- function(x = integer(), levels = character(), contrasts=NULL, how.many=NULL) {
  stopifnot(is.integer(x))
  stopifnot(is.character(levels))

  f <- structure(
    x,
    levels = levels,
    class = "factor"
  )
  
  if(!is.null(contrasts)) {
    if(!is.null(how.many)) how.many <- nlevels(f)
    f <-C(f, contrasts, how.many)
  }
  
  f
}

new_factor(1:10, levels=LETTERS[1:10])
```

```
##  [1] A B C D E F G H I J
## Levels: A B C D E F G H I J
```

```r
new_factor(1:10, levels=LETTERS[1:10], contrasts = "contr.helmert")
```

```
##  [1] A B C D E F G H I J
## attr(,"contrasts")
##   [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9]
## A   -1   -1   -1   -1   -1   -1   -1   -1   -1
## B    1   -1   -1   -1   -1   -1   -1   -1   -1
## C    0    2   -1   -1   -1   -1   -1   -1   -1
## D    0    0    3   -1   -1   -1   -1   -1   -1
## E    0    0    0    4   -1   -1   -1   -1   -1
## F    0    0    0    0    5   -1   -1   -1   -1
## G    0    0    0    0    0    6   -1   -1   -1
## H    0    0    0    0    0    0    7   -1   -1
## I    0    0    0    0    0    0    0    8   -1
## J    0    0    0    0    0    0    0    0    9
## Levels: A B C D E F G H I J
```


#### 5. Read the documentation for utils::as.roman(). How would you write a constructor for this class? Does it need a validator? What might a helper do?

constructor could take roman or latin numbers.  If latin, would need to do a series of integer division and keep track of the remainder from each (to be the numerator in the next division).  If roman, would need to get the latin representation

validator should make sure that numbers are integers between 1 and 3899

Not sure about a helper


```r
three <- as.roman(3)

attributes(three)
```

```
## $class
## [1] "roman"
```

```r
str(three)
```

```
##  'roman' int III
```

## 13.4 Generics and Methods

### 13.4.1 Method Dispatch

Generics use `UseMethod()` to dispatch to the correct method.

We can see what is being chosen using `sloop::s3_dispatch()`


```r
x <- Sys.Date()
s3_dispatch(print(x))
```

```
## => print.Date
##  * print.default
```



### 13.4.2 Finding methods

Use `sloop::s3_methods_generic` (for a method) or `sloop::s3_methods_class()` for a class


```r
s3_methods_generic("mean")
```

```
## # A tibble: 7 × 4
##   generic class      visible source             
##   <chr>   <chr>      <lgl>   <chr>              
## 1 mean    Date       TRUE    base               
## 2 mean    default    TRUE    base               
## 3 mean    difftime   TRUE    base               
## 4 mean    POSIXct    TRUE    base               
## 5 mean    POSIXlt    TRUE    base               
## 6 mean    quosure    FALSE   registered S3method
## 7 mean    vctrs_vctr FALSE   registered S3method
```

```r
s3_methods_class("ordered")
```

```
## # A tibble: 6 × 4
##   generic       class   visible source             
##   <chr>         <chr>   <lgl>   <chr>              
## 1 as.data.frame ordered TRUE    base               
## 2 Ops           ordered TRUE    base               
## 3 relevel       ordered FALSE   registered S3method
## 4 scale_type    ordered FALSE   registered S3method
## 5 Summary       ordered TRUE    base               
## 6 type_sum      ordered FALSE   registered S3method
```

### 13.4.3 Creating Methods

Only write a method if you own the generic or the class

A method must have the same arguments as its generic.

### 13.4.4 Exercises

#### 1. Read the source code for t() and t.test() and confirm that t.test() is an S3 generic and not an S3 method. What happens if you create an object with class test and call t() with it? Why?


```r
?t
?t.test

s3_dispatch(t.test(rnorm(10)))
```

```
##    t.test.double
##    t.test.numeric
## => t.test.default
```

```r
result <- t.test(rnorm(10))

print("t.test result")
```

```
## [1] "t.test result"
```

```r
result
```

```
## 
## 	One Sample t-test
## 
## data:  rnorm(10)
## t = 2.0548, df = 9, p-value = 0.07007
## alternative hypothesis: true mean is not equal to 0
## 95 percent confidence interval:
##  -0.07766228  1.61697685
## sample estimates:
## mean of x 
## 0.7696573
```

```r
print("str of t.test result")
```

```
## [1] "str of t.test result"
```

```r
str(result)
```

```
## List of 10
##  $ statistic  : Named num 2.05
##   ..- attr(*, "names")= chr "t"
##  $ parameter  : Named num 9
##   ..- attr(*, "names")= chr "df"
##  $ p.value    : num 0.0701
##  $ conf.int   : num [1:2] -0.0777 1.617
##   ..- attr(*, "conf.level")= num 0.95
##  $ estimate   : Named num 0.77
##   ..- attr(*, "names")= chr "mean of x"
##  $ null.value : Named num 0
##   ..- attr(*, "names")= chr "mean"
##  $ stderr     : num 0.375
##  $ alternative: chr "two.sided"
##  $ method     : chr "One Sample t-test"
##  $ data.name  : chr "rnorm(10)"
##  - attr(*, "class")= chr "htest"
```

```r
print("t(result)")
```

```
## [1] "t(result)"
```

```r
t(result)
```

```
## 
## 
## 
## data:
```

```r
print("str(t(result))")
```

```
## [1] "str(t(result))"
```

```r
str(t(result))
```

```
## List of 10
##  $ : Named num 2.05
##   ..- attr(*, "names")= chr "t"
##  $ : Named num 9
##   ..- attr(*, "names")= chr "df"
##  $ : num 0.0701
##  $ : num [1:2] -0.0777 1.617
##   ..- attr(*, "conf.level")= num 0.95
##  $ : Named num 0.77
##   ..- attr(*, "names")= chr "mean of x"
##  $ : Named num 0
##   ..- attr(*, "names")= chr "mean"
##  $ : num 0.375
##  $ : chr "two.sided"
##  $ : chr "One Sample t-test"
##  $ : chr "rnorm(10)"
##  - attr(*, "dim")= int [1:2] 1 10
##  - attr(*, "dimnames")=List of 2
##   ..$ : NULL
##   ..$ : chr [1:10] "statistic" "parameter" "p.value" "conf.int" ...
##  - attr(*, "class")= chr "htest"
```

```r
print("s3_dispatch(t(result))")
```

```
## [1] "s3_dispatch(t(result))"
```

```r
s3_dispatch(t(result))
```

```
##    t.htest
## => t.default
```


```r
t_result <- t(result)
print(t_result)
```

```
## 
## 
## 
## data:
```

```r
s3_dispatch(print(t_result))
```

```
## => print.htest
##  * print.default
```

```r
unclass(t_result)
```

```
##      statistic parameter p.value    conf.int  estimate  null.value stderr   
## [1,] 2.054816  9         0.07006562 numeric,2 0.7696573 0          0.3745626
##      alternative method              data.name  
## [1,] "two.sided" "One Sample t-test" "rnorm(10)"
```

So `t()` does its thing but keeps the class as "htest" so `print` uses the htest method and messes it up.


```r
x <- structure(1:10, class = "test")
x
```

```
##  [1]  1  2  3  4  5  6  7  8  9 10
## attr(,"class")
## [1] "test"
```

```r
t(x)
```

```
##      [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10]
## [1,]    1    2    3    4    5    6    7    8    9    10
## attr(,"class")
## [1] "test"
```

#### 2. What generics does the table class have methods for?


```r
s3_methods_class("table")
```

```
## # A tibble: 11 × 4
##    generic       class visible source             
##    <chr>         <chr> <lgl>   <chr>              
##  1 [             table TRUE    base               
##  2 aperm         table TRUE    base               
##  3 as_tibble     table FALSE   registered S3method
##  4 as.data.frame table TRUE    base               
##  5 Axis          table FALSE   registered S3method
##  6 lines         table FALSE   registered S3method
##  7 plot          table FALSE   registered S3method
##  8 points        table FALSE   registered S3method
##  9 print         table TRUE    base               
## 10 summary       table TRUE    base               
## 11 tail          table FALSE   registered S3method
```


#### 3. What generics does the ecdf class have methods for?


```r
s3_methods_class("ecdf")
```

```
## # A tibble: 4 × 4
##   generic  class visible source             
##   <chr>    <chr> <lgl>   <chr>              
## 1 plot     ecdf  TRUE    stats              
## 2 print    ecdf  FALSE   registered S3method
## 3 quantile ecdf  FALSE   registered S3method
## 4 summary  ecdf  FALSE   registered S3method
```


#### 4. Which base generic has the greatest number of defined methods?


```r
fxns <- tibble(fn_name=ls("package:base")) %>%
    filter(map_lgl(fn_name, ~ {get(.) %>% is_function()})) %>%
  filter(map_lgl(fn_name, is_s3_generic)) %>%
  mutate(methods=map(fn_name, s3_methods_generic)) %>%
  mutate(n.methods=map_int(methods, nrow)) %>%
  arrange(desc(n.methods))

fxns
```

```
## # A tibble: 172 × 3
##    fn_name       methods            n.methods
##    <chr>         <list>                 <int>
##  1 print         <tibble [307 × 4]>       307
##  2 format        <tibble [137 × 4]>       137
##  3 [             <tibble [58 × 4]>         58
##  4 summary       <tibble [44 × 4]>         44
##  5 as.character  <tibble [41 × 4]>         41
##  6 plot          <tibble [34 × 4]>         34
##  7 as.data.frame <tibble [33 × 4]>         33
##  8 [[            <tibble [25 × 4]>         25
##  9 [<-           <tibble [21 × 4]>         21
## 10 c             <tibble [19 × 4]>         19
## # … with 162 more rows
```

#### 5. Carefully read the documentation for UseMethod() and explain why the following code returns the results that it does. What two usual rules of function evaluation does UseMethod() violate?


```r
g <- function(x) {
  x <- 10
  y <- 10
  UseMethod("g")
}
g.default <- function(x) c(x = x, y = y)

x <- 1
y <- 1
g(x)
```

```
##  x  y 
##  1 10
```

```r
#>  x  y 
#>  1 10
```

From help on `UseMethod` "Any local variables defined before the call to UseMethod are retained "

#### 6. What are the arguments to [? Why is this a hard question to answer?


```r
`[`
```

```
## .Primitive("[")
```


## 13.5 Object Styles


