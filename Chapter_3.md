---
title: "Chapter 3"
author: "Julin Maloof"
date: "2022-09-07"
output: 
  html_document: 
    keep_md: yes
---



# Advanced R Chapter 3: Vectors

## Quizz

1.  What are the four common types of atomic vectors? What are the two rare types? *common: character, numeric, logical, integer; rare: complex, ??*

2.  What are attributes? How do you get them and set them? *well, they are attributes. Metadata such as names, or characteristics that define how the object behaves*

*set with `attr() <-` get with `attr()`*

3.  How is a list different from an atomic vector? How is a matrix different from a data frame?

*a list can have different data types per element whereas an atomic vector cannot. A data frame can have different column types whereas a matrix cannot*

4.  Can you have a list that is a matrix? Can a data frame have a column that is a matrix?

*not sure what is meant by a list that is a matrix. A matrix can be an element of a list. Can you have a matrix of lists?*


```r
# a matrix of lists
matrix(c(list(1:10), list(LETTERS)))
```

```
##      [,1]        
## [1,] integer,10  
## [2,] character,26
```

3.  How do tibbles behave differently from data frames?

    *No row names. No automatic conversion to factors. Special printing methods.*

### 3.2.5 Exercises

#### 1. How do you create raw and complex scalars? (See ?raw and ?complex.)


```r
as.raw(c(1,33, 50, 200))
```

```
## [1] 01 21 32 c8
```


```r
complex(real=1:3, imaginary = -1:1)
```

```
## [1] 1-1i 2+0i 3+1i
```


#### 2. Test your knowledge of the vector coercion rules by predicting the output of the following uses of c():


```r
c(1, FALSE) # 1, 0
```

```
## [1] 1 0
```

```r
c("a", 1) # "a", "1"
```

```
## [1] "a" "1"
```

```r
c(TRUE, 1L) # 1L, 1L
```

```
## [1] 1 1
```

```r
is.integer(c(TRUE, 1L))
```

```
## [1] TRUE
```


#### 3. Why is 1 == "1" true? Why is -1 < FALSE true? Why is "one" < 2 false?

_Coercion_

_In the first example, 1 gets coerced to "1"_

_In the second example, FALSE gets coerced to 0_

_In the third example, 2 gets coerced to "2" and according to character sorting, digits come before letters_

#### 4. Why is the default missing value, NA, a logical vector? What’s special about logical vectors? (Hint: think about c(FALSE, NA_character_).)

_Logicals can be coereced to all other types_

#### 5. Precisely what do is.atomic(), is.numeric(), and is.vector() test for?

_I don't understand the implication that `is.atomic()` isn't behaving as expected; seems to test for atomic types (although not necessarily vectors, maybe that is the point_

_`is.numeric()` tests whether or not a vector can be interpreted as numeric, e.g._


```r
x <- c(1L, 2L)
x
```

```
## [1] 1 2
```

```r
is.numeric(x)
```

```
## [1] TRUE
```

_`is.vector` tests for whether or not the object is a vector with no attributes other than names_


### 3.3.4 Exercises

#### 1. How is setNames() implemented? How is unname() implemented? Read the source code.


```r
setNames
```

```
## function (object = nm, nm) 
## {
##     names(object) <- nm
##     object
## }
## <bytecode: 0x111fbf7e0>
## <environment: namespace:stats>
```


```r
unname
```

```
## function (obj, force = FALSE) 
## {
##     if (!is.null(names(obj))) 
##         names(obj) <- NULL
##     if (!is.null(dimnames(obj)) && (force || !is.data.frame(obj))) 
##         dimnames(obj) <- NULL
##     obj
## }
## <bytecode: 0x1129a44b0>
## <environment: namespace:base>
```

_So these are just shorthand for what you would do yourself_

#### 2. What does dim() return when applied to a 1-dimensional vector? When might you use NROW() or NCOL()?


```r
dim(1:3)
```

```
## NULL
```

#NROW and NCOL are more generic

```r
x <- 1:3
nrow(x)
```

```
## NULL
```

```r
NROW(x)
```

```
## [1] 3
```

```r
ncol(x)
```

```
## NULL
```

```r
NCOL(x)
```

```
## [1] 1
```


#### 3. How would you describe the following three objects? What makes them different from 1:5?


```r
x1 <- array(1:5, c(1, 1, 5))
x2 <- array(1:5, c(1, 5, 1))
x3 <- array(1:5, c(5, 1, 1))
```

_These are 3d arrays_

#### 4. An early draft used this code to illustrate structure():


```r
structure(1:5, comment = "my attribute")
```

```
## [1] 1 2 3 4 5
```

```r
#> [1] 1 2 3 4 5
```

But when you print that object you don’t see the comment attribute. Why? Is the attribute missing, or is there something else special about it? (Hint: try using help.)

_The comment attribute is special and does not get printed_

### 3.4.5 Exercises

#### 1. What sort of object does table() return? What is its type? What attributes does it have? How does the dimensionality change as you tabulate more variables?


```r
x <- table(rep(c("dogs", "cats"), times=c(5,3)))
x
```

```
## 
## cats dogs 
##    3    5
```

```r
cat("\ntype:\n", typeof(x))
```

```
## 
## type:
##  integer
```

```r
cat("\n\nattributes:\n"); attributes(x)
```

```
## 
## 
## attributes:
```

```
## $dim
## [1] 2
## 
## $dimnames
## $dimnames[[1]]
## [1] "cats" "dogs"
## 
## 
## $class
## [1] "table"
```

```r
cat("\n\nclass\n", class(x))
```

```
## 
## 
## class
##  table
```
_Type is integer; class is "table"; dimensions = number of classes_

#### 2. What happens to a factor when you modify its levels?


```r
f1 <- factor(letters)
as.integer(f1)
```

```
##  [1]  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25
## [26] 26
```

```r
f1
```

```
##  [1] a b c d e f g h i j k l m n o p q r s t u v w x y z
## Levels: a b c d e f g h i j k l m n o p q r s t u v w x y z
```



```r
levels(f1) <- rev(levels(f1))
as.integer(f1)
```

```
##  [1]  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25
## [26] 26
```

```r
f1
```

```
##  [1] z y x w v u t s r q p o n m l k j i h g f e d c b a
## Levels: z y x w v u t s r q p o n m l k j i h g f e d c b a
```

The integer represetnation does not change; the mapping of levels to integers does change

#### 3. What does this code do? How do f2 and f3 differ from f1?


```r
f2 <- rev(factor(letters))

f3 <- factor(letters, levels = rev(letters))
```


```r
f2
```

```
##  [1] z y x w v u t s r q p o n m l k j i h g f e d c b a
## Levels: a b c d e f g h i j k l m n o p q r s t u v w x y z
```

```r
as.integer(f2)
```

```
##  [1] 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10  9  8  7  6  5  4  3  2
## [26]  1
```


```r
f3
```

```
##  [1] a b c d e f g h i j k l m n o p q r s t u v w x y z
## Levels: z y x w v u t s r q p o n m l k j i h g f e d c b a
```

```r
as.integer(f3)
```

```
##  [1] 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10  9  8  7  6  5  4  3  2
## [26]  1
```

_f2 reverses the order of the categories (and associated integers) in the vector.  f3 reverses the mapping (which factor levels is "first"), but not the order of categories in the vector._

### 3.5.4 Exercises

#### 1. List all the ways that a list differs from an atomic vector.

_1. can have different data types_

_2. can be hierarchical/ recursive_

_Seems like there should be more than this, given the wording of the question..._

#### 2. Why do you need to use unlist() to convert a list to an atomic vector? Why doesn’t as.vector() work?

_because lists are vectors, just not atomic ones_

#### 3. Compare and contrast c() and unlist() when combining a date and date-time into a single vector.


```r
mydates <- as.Date(c("1967-05-02", "1965-01-20", "1999-10-10"))
mydates
```

```
## [1] "1967-05-02" "1965-01-20" "1999-10-10"
```

```r
mydatetimes <- as.POSIXct(c("2020-03-20 18:30", "2018-11-01 05:40"))
mydatetimes
```

```
## [1] "2020-03-20 18:30:00 EDT" "2018-11-01 05:40:00 EDT"
```


```r
dates.c <- c(mydates, mydatetimes) #coerce to date only
dates.c
```

```
## [1] "1967-05-02" "1965-01-20" "1999-10-10" "2020-03-20" "2018-11-01"
```

```r
class(dates.c)
```

```
## [1] "Date"
```


```r
dates.l <- list(mydates, mydatetimes) #list of two differnt types
dates.l
```

```
## [[1]]
## [1] "1967-05-02" "1965-01-20" "1999-10-10"
## 
## [[2]]
## [1] "2020-03-20 18:30:00 EDT" "2018-11-01 05:40:00 EDT"
```

```r
sapply(dates.l, class)
```

```
## [[1]]
## [1] "Date"
## 
## [[2]]
## [1] "POSIXct" "POSIXt"
```

### 3.6.8 Exercises

#### 1. Can you have a data frame with zero rows? What about zero columns?


```r
df1 <- data.frame(x=numeric(), y=character()) #0 rows, 2 columns
str(df1)
```

```
## 'data.frame':	0 obs. of  2 variables:
##  $ x: num 
##  $ y: chr
```

```r
dim(df1)
```

```
## [1] 0 2
```


```r
df2 <- data.frame()
str(df2)
```

```
## 'data.frame':	0 obs. of  0 variables
```

```r
dim(df2)
```

```
## [1] 0 0
```


#### 2. What happens if you attempt to set rownames that are not unique?


```r
data.frame(x=1:3, y=LETTERS[1:3], row.names = c(1,1,1))
```

```
## Error in data.frame(x = 1:3, y = LETTERS[1:3], row.names = c(1, 1, 1)): duplicate row.names: 1
```


#### 3. If df is a data frame, what can you say about t(df), and t(t(df))? Perform some experiments, making sure to try different column types.


```r
df <- data.frame(x=1:3, y= LETTERS[1:3])
t(df)
```

```
##   [,1] [,2] [,3]
## x "1"  "2"  "3" 
## y "A"  "B"  "C"
```

```r
str(t(df))
```

```
##  chr [1:2, 1:3] "1" "A" "2" "B" "3" "C"
##  - attr(*, "dimnames")=List of 2
##   ..$ : chr [1:2] "x" "y"
##   ..$ : NULL
```

```r
t(t(df))
```

```
##      x   y  
## [1,] "1" "A"
## [2,] "2" "B"
## [3,] "3" "C"
```

_I think the point here is that you don't get back what you started with (because `t()` is going to create a matrix, not a df, and it is going to coerce the data to the same type)_

#### 4. What does as.matrix() do when applied to a data frame with columns of different types? How does it differ from data.matrix()?


```r
as.matrix(df) 
```

```
##      x   y  
## [1,] "1" "A"
## [2,] "2" "B"
## [3,] "3" "C"
```

_`as.matrix()` creates a matrix and coerces the types to the least restrictive_


```r
data.matrix(df)
```

```
##      x y
## [1,] 1 1
## [2,] 2 2
## [3,] 3 3
```
_`data.matrix()` converts everthing to numeric and then creates a numeric matrix_
