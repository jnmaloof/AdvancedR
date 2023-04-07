---
title: "Chapter 14"
author: "Julin Maloof"
date: "2023-04-07"
output: 
  html_document: 
    keep_md: yes
---



R6

__"in most cases R6 will lead you to non-idiomatic R code."__


```r
library(R6)
```

Use the `R6Class()` function to define a class.  Assign it to an object that has the same name as the class.  First argument is class name, second argument is public fields and methods.


```r
Accumulator <- R6Class("Accumulator", list(
  sum = 0,
  add = function(x = 1) {
    self$sum <- self$sum + x 
    invisible(self)
  })
)
```


```r
Accumulator
```

```
## <Accumulator> object generator
##   Public:
##     sum: 0
##     add: function (x = 1) 
##     clone: function (deep = FALSE) 
##   Parent env: <environment: R_GlobalEnv>
##   Locked objects: TRUE
##   Locked class: FALSE
##   Portable: TRUE
```

Create a new object of this clas with $new function


```r
x <- Accumulator$new()
```


```r
x$sum
```

```
## [1] 0
```

```r
x$add(4)
x$sum
```

```
## [1] 4
```

## 14.2.1 Method Chaining
 
side-effect R6 methods (those that update a field and are not called to create an output) should return self invisibly

then you can do things like this

```r
x$
  add(10)$
  add(10)$
  sum
```

```
## [1] 24
```

## 14.2.2 Important methods

Generally should define $initialize and $print methods to override the defaults


```r
Person <- R6Class("Person", list(
  name = NULL,
  age = NA,
  initialize = function(name, age = NA) {
    self$name <- name
    self$age <- age
  },
  print = function(...) {
    cat("Person: \n")
    cat("  Name: ", self$name, "\n", sep = "")
    cat("  Age:  ", self$age, "\n", sep = "")
    invisible(self)
  }
))

hadley2 <- Person$new("Hadley")
hadley2
```

```
## Person: 
##   Name: Hadley
##   Age:  NA
```

## 14.2.3 Adding methods after creation

Use $set


```r
Accumulator <- R6Class("Accumulator")
Accumulator$set("public", "sum", 0)
Accumulator$set("public", "add", function(x = 1) {
  self$sum <- self$sum + x 
  invisible(self)
})
```

Will only impact newly created objects, will not be added to previosly created objects of the same class.

## 14.2.5 Inheritance


```r
AccumulatorChatty <- R6Class("AccumulatorChatty", 
  inherit = Accumulator,
  public = list(
    add = function(x = 1) {
      cat("Adding ", x, "\n", sep = "")
      super$add(x = x)
    }
  )
)

x2 <- AccumulatorChatty$new()
x2$add(10)$add(1)$sum
```

```
## Adding 10
## Adding 1
```

```
## [1] 11
```

## 14.2.5 Introspections

Every R6 object has an S3 class hierarchy

```r
class(hadley2)
```

```
## [1] "Person" "R6"
```

## 14.2.6. Exercises

### 1 Create a bank account R6 class that stores a balance and allows you to deposit and withdraw money. Create a subclass that throws an error if you attempt to go into overdraft. Create another subclass that allows you to go into overdraft, but charges you a fee.


```r
BankAccountBasic <- R6Class("BankAcountSimple", list(
  balance = 0,
  deposit = function(amt) {
    self$balance <- self$balance + amt
    invisible(self)
  },
  withdraw = function(amt) {
    self$balance <- self$balance - amt
    invisible(self)
  },
  print = function() {
    cat("The current balance is: ", self$balance, "\n")
    invisible(self)
  }
))

BA1 <- BankAccountBasic$new()

BA1$deposit(100)
BA1$withdraw(33)
BA1
```

```
## The current balance is:  67
```


```r
BankAccount2 <- R6Class("BankAcount2",
                       inherit = BankAccountBasic,
                       public = list(
                         withdraw=function(amt) {
                           stopifnot(self$balance-amt>=0)
                           super$withdraw(amt)
                         }
                       ))

BA2 <- BankAccount2$new()

BA2$deposit(100)
BA2$withdraw(33)
BA2
```

```
## The current balance is:  67
```

```r
BA2$withdraw(100)
```

```
## Error in BA2$withdraw(100): self$balance - amt >= 0 is not TRUE
```

```r
BA2
```

```
## The current balance is:  67
```

```r
BankAccount3 <- R6Class("BankAcount3",
                       inherit = BankAccountBasic,
                       public = list(
                         withdraw=function(amt) {
                           super$withdraw(amt)
                           if (self$balance < 0) {
                             warning("New balance is less than $0, a $5.00 fee is being assessed\n")
                             super$withdraw(5)
                           }
                         }
                       ))

BA3 <- BankAccount3$new()

BA3$deposit(100)
BA3$withdraw(150)
```

```
## Warning in BA3$withdraw(150): New balance is less than $0, a $5.00 fee is being assessed
```

```r
BA3
```

```
## The current balance is:  -55
```

### 2. Create an R6 class that represents a shuffled deck of cards. You should be able to draw cards from the deck with $draw(n), and return all cards to the deck and reshuffle with $reshuffle(). Use the following code to make a vector of cards.


```r
deck <- R6Class("deck", public = list(
  stack=character(),
  drawn=character(),
  initialize = function() {
    suit <- c("♠", "♥", "♦", "♣")
    value <- c("A", 2:10, "J", "Q", "K")
    self$stack <- sample(paste0(rep(value, 4), suit))
    self$drawn <- character()
  },
  reshuffle = function() {
    self$stack <- c(self$stack, self$drawn)
    self$drawn <- character()
    self$stack <- sample(self$stack)
  },
  draw = function(n) {
    newdraw <- self$stack[1:n]
    self$drawn <- c(self$drawn, newdraw)
    self$stack <- self$stack[-1:-n]
    cat(newdraw)
  },
  print = function() {
    cat("Not Drawn: ",self$stack, "\n")
    cat("Drawn: ", self$drawn, "\n")
  }
))

mydeck <- deck$new()
mydeck
```

```
## Not Drawn:  A♥ 3♠ 10♣ 8♦ 2♣ 2♠ J♥ 6♣ 6♠ 9♦ 8♥ 3♣ 10♦ 7♠ 4♦ 3♥ 2♦ Q♥ K♥ 4♣ 8♠ 9♥ 2♥ A♠ 5♣ 7♥ 7♦ J♣ 10♠ Q♠ K♦ 7♣ 9♣ 8♣ 4♥ Q♦ K♣ J♦ J♠ 5♦ 9♠ 6♥ 6♦ Q♣ K♠ 4♠ A♣ A♦ 10♥ 5♥ 3♦ 5♠ 
## Drawn:
```

```r
mydeck$draw(5)
```

```
## A♥ 3♠ 10♣ 8♦ 2♣
```

```r
mydeck
```

```
## Not Drawn:  2♠ J♥ 6♣ 6♠ 9♦ 8♥ 3♣ 10♦ 7♠ 4♦ 3♥ 2♦ Q♥ K♥ 4♣ 8♠ 9♥ 2♥ A♠ 5♣ 7♥ 7♦ J♣ 10♠ Q♠ K♦ 7♣ 9♣ 8♣ 4♥ Q♦ K♣ J♦ J♠ 5♦ 9♠ 6♥ 6♦ Q♣ K♠ 4♠ A♣ A♦ 10♥ 5♥ 3♦ 5♠ 
## Drawn:  A♥ 3♠ 10♣ 8♦ 2♣
```

```r
mydeck$draw(5)
```

```
## 2♠ J♥ 6♣ 6♠ 9♦
```

```r
mydeck
```

```
## Not Drawn:  8♥ 3♣ 10♦ 7♠ 4♦ 3♥ 2♦ Q♥ K♥ 4♣ 8♠ 9♥ 2♥ A♠ 5♣ 7♥ 7♦ J♣ 10♠ Q♠ K♦ 7♣ 9♣ 8♣ 4♥ Q♦ K♣ J♦ J♠ 5♦ 9♠ 6♥ 6♦ Q♣ K♠ 4♠ A♣ A♦ 10♥ 5♥ 3♦ 5♠ 
## Drawn:  A♥ 3♠ 10♣ 8♦ 2♣ 2♠ J♥ 6♣ 6♠ 9♦
```

```r
mydeck$reshuffle()
mydeck
```

```
## Not Drawn:  4♦ A♥ 8♣ 10♥ 8♥ 10♠ J♦ K♣ 5♥ J♠ 8♦ 7♠ 7♦ J♣ K♦ 9♦ Q♠ 8♠ 9♥ 9♠ K♥ 5♠ A♠ 3♠ 4♥ 5♦ A♣ 10♣ 6♠ K♠ 2♥ Q♦ A♦ 7♣ 2♦ 10♦ 3♦ 7♥ 2♣ Q♥ 3♥ 5♣ 2♠ 3♣ 6♦ J♥ 9♣ 4♠ 4♣ 6♣ 6♥ Q♣ 
## Drawn:
```
### 3. Why can’t you model a bank account or a deck of cards with an S3 class?

Beacuse S3 is not modify in place

### 4. Create an R6 class that allows you to get and set the current time zone. You can access the current time zone with Sys.timezone() and set it with Sys.setenv(TZ = "newtimezone"). When setting the time zone, make sure the new time zone is in the list provided by OlsonNames().


```r
TZ <- R6Class("TZ", public=list(
  tz=Sys.timezone(),
  get=function() self$tz,
  set=function(new_tz) {
    stopifnot(new_tz %in% OlsonNames())
    Sys.setenv(TZ = new_tz)
    self$tz <- Sys.timezone()
  }
))

tz <- TZ$new()
tz$get()
```

```
## [1] "US/Hawaii"
```

```r
tz$set("US/Hawaii") # I wish
tz$get()
```

```
## [1] "US/Hawaii"
```

```r
tz$set("Mars")
```

```
## Error in tz$set("Mars"): new_tz %in% OlsonNames() is not TRUE
```

```r
tz$get()
```

```
## [1] "US/Hawaii"
```

### 5. Create an R6 class that manages the current working directory. It should have $get() and $set() methods.


```r
WD <- R6Class("WD", public=list(
  wd=getwd(),
  get=function() self$wd,
  set=function(new_wd) {
    stopifnot(dir.exists(new_wd))
    setwd(new_wd)
    self$wd <- new_wd
  }
))

wd <- WD$new()
wd$get()
```

```
## [1] "/Users/jmaloof/git/AdvancedR"
```

```r
wd$set("../") 
wd$get()
```

```
## [1] "../"
```

```r
wd$set("Mars")
```

```
## Error in wd$set("Mars"): dir.exists(new_wd) is not TRUE
```

```r
wd$get()
```

```
## [1] "../"
```

```r
wd$set("/Users/jmaloof/git/AdvancedR")
wd$get()
```

```
## [1] "/Users/jmaloof/git/AdvancedR"
```

### 5. Why can’t you model the time zone or current working directory with an S3 class?

no modify in place?

### 6. What base type are R6 objects built on top of? What attributes do they have?
