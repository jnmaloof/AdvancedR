---
title: "Chapter 7"
author: "Julin Maloof"
date: "2022-10-22"
output: 
  html_document: 
    keep_md: yes
---




```r
library(rlang)
```

What is the difference between `rlang::env()` and `new.env()`?


```r
e1 <- env(a = "John",
          b = 23,
          c = FALSE)
env_print(e1)
```

```
## <environment: 0x12447a4a8>
## Parent: <environment: global>
## Bindings:
## • a: <chr>
## • b: <dbl>
## • c: <lgl>
```


```r
e2 <- new.env()
e2$a <- "Paulo"
e2$b <- 42
e2$c <- TRUE

env_print(e2)
```

```
## <environment: 0x1107d0cf8>
## Parent: <environment: global>
## Bindings:
## • a: <chr>
## • b: <dbl>
## • c: <lgl>
```


```r
env_parent() #why rlang?
```

```
## <environment: package:rlang>
## attr(,"name")
## [1] "package:rlang"
## attr(,"path")
## [1] "/Library/Frameworks/R.framework/Versions/4.2-arm64/Resources/library/rlang"
```


```r
env_parents()
```

```
##  [[1]] $ <env: package:rlang>
##  [[2]] $ <env: package:stats>
##  [[3]] $ <env: package:graphics>
##  [[4]] $ <env: package:grDevices>
##  [[5]] $ <env: package:utils>
##  [[6]] $ <env: package:datasets>
##  [[7]] $ <env: package:methods>
##  [[8]] $ <env: Autoloads>
##  [[9]] $ <env: package:base>
## [[10]] $ <env: empty>
```

## 7.2.7 Exercises

### 1. List three ways in which an environment differs from a list.

_1. not ordered_
_2. names must be unique_
_3. has a parent_

### 2. Create an environment as illustrated by this picture.


```r
e2 <- env()
e2$loop <- e2
env_print(e2)
```

```
## <environment: 0x1128cc1f8>
## Parent: <environment: global>
## Bindings:
## • loop: <env>
```


### 3. Create a pair of environments as illustrated by this picture.


```r
e3a <- env()
e3b <- env()
e3a$loop <- e3b
e3b$dedoop <- e3a

env_print(e3a)
```

```
## <environment: 0x1147f3e08>
## Parent: <environment: global>
## Bindings:
## • loop: <env>
```

```r
env_print(e3b)
```

```
## <environment: 0x11478cfc0>
## Parent: <environment: global>
## Bindings:
## • dedoop: <env>
```


### 4. Explain why e[[1]] and e[c("a", "b")] don’t make sense when e is an environment.

_`e[[1]]` does not make sense because objects in an environment are not ordered so there is no "first" element_

_I am not as sure about `e[c("a", "b")]` but I guess because you can't subset an environment (but why not?)_

### 5. Create a version of env_poke() that will only bind new names, never re-bind old names. Some programming languages only do this, and are known as single assignment languages.


```r
env_poke2 <- function(env = caller_env(), nm, value) {
  if(env_has(env, nm)) {
    stop(paste0("environment ", env_label(env), " already has a binding called '", nm, "'\n"))
  } else {
    env_poke(env, nm, value)
  }
}
```


```r
e5 <- env(a=3)
env_poke(e5, "a", 5)
env_print(e5)
```

```
## <environment: 0x111e6e7d0>
## Parent: <environment: global>
## Bindings:
## • a: <dbl>
```

```r
env_poke2(e5, "b", 5)
env_print(e5)
```

```
## <environment: 0x111e6e7d0>
## Parent: <environment: global>
## Bindings:
## • a: <dbl>
## • b: <dbl>
```

```r
env_poke2(e5, "a", 12) #error, as expected
```

```
## Error in env_poke2(e5, "a", 12): environment 0x111e6e7d0 already has a binding called 'a'
```


### 6. What does this function do? How does it differ from <<- and why might you prefer it?


```r
rebind <- function(name, value, env = caller_env()) {
  if (identical(env, empty_env())) {
    stop("Can't find `", name, "`", call. = FALSE)
  } else if (env_has(env, name)) {
    env_poke(env, name, value)
  } else {
    rebind(name, value, env_parent(env))
  }
}
rebind("a", 10)
```

```
## Error: Can't find `a`
```

```r
#> Error: Can't find `a`
a <- 5
rebind("a", 10)
a
```

```
## [1] 10
```

```r
#> [1] 10
```

_`<<-` will set the value in the parent environment.  This function will attempt to set it in the local environment (or specified environment) and if there is not a binding there it will go up a level (and keep going if necessary)_

## 7.3.1 Exercises

### 1. Modify where() to return all environments that contain a binding for name. Carefully think through what type of object the function will need to return.


```r
where <- function(name, env = caller_env()) {
  if (identical(env, empty_env())) {
    # Base case
    stop("Can't find ", name, call. = FALSE)
  } else if (env_has(env, name)) {
    # Success case
    env
  } else {
    # Recursive case
    where(name, env_parent(env))
  }
}

where_all <- function(name, env = caller_env()) {
  if(!exists("wa_result")) wa_result <<- list() # hold results
  if (identical(env, empty_env())) {
    if(length(wa_result)>0) {
      on.exit(rm("wa_result", envir = globalenv()))
      return(wa_result) # we are at the empty environment and have results to return
    } else {
    # Base case
    stop("Can't find ", name, call. = FALSE)
    } # if else length 
  } else {
    if (env_has(env, name)) wa_result <<- c(wa_result, env)
    # found one
    # Recurse to keep searching
    where_all(name, env_parent(env))
  }
}
```



```r
mean <- function(x) mean(x) # just to get something in multiple environments
where("mean")
```

```
## <environment: R_GlobalEnv>
```

```r
where_all("mean")
```

```
## [[1]]
## <environment: R_GlobalEnv>
## 
## [[2]]
## <environment: base>
```

It would be nicer to have the results held somewhere other than the global environment.
Best would be to use the environment of the first function call but not sure how to access that.

### 2. Write a function called fget() that finds only function objects. It should have two arguments, name and env, and should obey the regular scoping rules for functions: if there’s an object with a matching name that’s not a function, look in the parent. For an added challenge, also add an inherits argument which controls whether the function recurses up the parents or only looks in one environment.


