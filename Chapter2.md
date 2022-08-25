---
title: "Chapter 2"
author: "Julin Maloof"
date: "2022-08-25"
output: 
  html_document: 
    keep_md: yes
---



## Quiz

1. What are the three properties of a vector, other than its contents? _length, type, ??_
2. What are the four common types of atomic vectors?  What are the two rare types? _Common: numeric, character, boolean, factor; Rare: integer, some single point precicion thing?_
3. What are attributes? How do you get them and set them?  _attributes define the data structure type.  can use `attr()` or `attributes()`
4. How is a list different from an atomic vector?  How is a matrix different from a data frame? _atomic vectors and matrices can only have a single data type per instance, whereas lists and data frames can mix them_
5. Can you have a list that is a matrix? Can a data frame have a column that is a matrix? _I am not sure what this is asking.  You can have a list that contains a matrix and similarly a dataframe column can contain a mstrix.  But I do not think you can have 2D lists.  Probably could have a matrix of lists_

Note that I did not get all (or any?) of these fully correct, see the [answers](http://adv-r.had.co.nz/Data-structures.html#data-structure-answers)

## Exercises 1

### 1. What are the six types of atomic vector? How does a list differ from an atomic vector?

character, logical, integer, double, raw, complex

list can have different data types

### 2. What makes is.vector() and is.numeric() fundamentally different to is.list() and is.character()?

`is.numeric()` can return TRUE for more than one data type (double and integer)

`is.vector()` only returns TRUE of the object is a vector and has no attributes other than names



### 3. Test your knowledge of vector coercion rules by predicting the output of the following uses of c():

`c(1, FALSE)`: 1 0

`c("a", 1)` "a"  "1"

`c(list(1), "a")` list(1), list(a)

`c(TRUE, 1L)` 1, 1

### 4. Why do you need to use unlist() to convert a list to an atomic vector? Why doesn’t as.vector() work?

becasuse lists are a type of vector (just not atomic)

### 5. 

Why is 1 == "1" true?  _1 gets coerced to "1"_

Why is -1 < FALSE true? _because FALSE gets coerced to 0_

Why is "one" < 2 false? _because 2 gets coerced to "2" and the ordering is alphabetic


```r
"a" < "b"
```

```
## [1] TRUE
```

```r
"b" < "a"
```

```
## [1] FALSE
```


#### 6. Why is the default missing value, NA, a logical vector? What’s special about logical vectors? (Hint: think about c(FALSE, NA_character_).)

_logical vectors can be coerced to any other type_

## Exercises 2

### 1 
An early draft used this code to illustrate structure():


```r
structure(1:5, comment = "my attribute")
```

```
## [1] 1 2 3 4 5
```
But when you print that object you don’t see the comment attribute. Why? Is the attribute missing, or is there something else special about it? (Hint: try using help.)


```r
a <- structure(1:5, comment = "my attribute")
a
```

```
## [1] 1 2 3 4 5
```

```r
attributes(a)
```

```
## $comment
## [1] "my attribute"
```

```r
a <- structure(1:5, my_comment = "my attribute")
a
```

```
## [1] 1 2 3 4 5
## attr(,"my_comment")
## [1] "my attribute"
```

```r
attributes(a)
```

```
## $my_comment
## [1] "my attribute"
```

_"comment" is a predefined attribute and by definition is not printed.  see `?comment` _

### 2 
What happens to a factor when you modify its levels?


```r
f1 <- factor(letters)
f1
```

```
##  [1] a b c d e f g h i j k l m n o p q r s t u v w x y z
## Levels: a b c d e f g h i j k l m n o p q r s t u v w x y z
```

```r
levels(f1) <- rev(levels(f1))
f1
```

```
##  [1] z y x w v u t s r q p o n m l k j i h g f e d c b a
## Levels: z y x w v u t s r q p o n m l k j i h g f e d c b a
```

_the printed labels change_

### 3) What does this code do? How do f2 and f3 differ from f1?


```r
f2 <- rev(factor(letters))
f2
```

```
##  [1] z y x w v u t s r q p o n m l k j i h g f e d c b a
## Levels: a b c d e f g h i j k l m n o p q r s t u v w x y z
```

```r
f3 <- factor(letters, levels = rev(letters))
f3
```

```
##  [1] a b c d e f g h i j k l m n o p q r s t u v w x y z
## Levels: z y x w v u t s r q p o n m l k j i h g f e d c b a
```

_f2 reverse the order of the elements in f1 but the factor level assignments are the same (a is still the first level)_

_f3 reverses the order of the factor levels in f1, but the elements stay the same_

## Exercises 3

### 1. What does dim() return when applied to a vector?

```r
dim(1:3)
```

```
## NULL
```

### 2. If is.matrix(x) is TRUE, what will is.array(x) return?

_TRUE, because a matrix is just a 2D array_

```r
m <- matrix(1:25,ncol=5)
is.matrix(m)
```

```
## [1] TRUE
```

```r
is.array(m)
```

```
## [1] TRUE
```


### 3. How would you describe the following three objects? What makes them different to 1:5?


```r
x1 <- array(1:5, c(1, 1, 5))
x2 <- array(1:5, c(1, 5, 1))
x3 <- array(1:5, c(5, 1, 1))
```

These are all 3D arrays, but with data in only 1 dimension.

## Exercises 4

### 1. What attributes does a data frame possess?


```r
df <- data.frame(x=1:3, y=c("A","B","C"))
attributes(df)
```

```
## $names
## [1] "x" "y"
## 
## $class
## [1] "data.frame"
## 
## $row.names
## [1] 1 2 3
```

_names (colnames), class, and row.names_

### 2. What does as.matrix() do when applied to a data frame with columns of different types?

coerces...


```r
as.matrix(df)
```

```
##      x   y  
## [1,] "1" "A"
## [2,] "2" "B"
## [3,] "3" "C"
```


Can you have a data frame with 0 rows? What about 0 columns?

_yes_


```r
data.frame() # 0 rows and 0 columns
```

```
## data frame with 0 columns and 0 rows
```

```r
data.frame(x=numeric(),y=character()) # 0 rows, 2 columns
```

```
## [1] x y
## <0 rows> (or 0-length row.names)
```

