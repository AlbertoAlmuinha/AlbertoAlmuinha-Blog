---
title: "Box Package"
author: "Alberto Almuiña"
date: '2021-02-28T21:13:14-05:00'
description: Reusable and Modular R Code with the Box Package
slug: box-package
tags:
- box
- modulos
- librerias
categories: 
- R-tips
---

# Introduction to the `box` package

`box` is one of my new favorite packages in the R ecosystem. Developed by Konrad Rudolph, its main purpose is to allow us to organize the code in a much more modular way, mainly through two mechanisms, as indicated in its page of [pkgdown](https://klmr.me/box/index.html):

- Allows you to write modular code by treating R code files and folders as independent (potentially nested) modules, without requiring the user to package the code in a package (wow!)

- It provides a new syntax for importing reusable code (both packages and modules) that is more powerful and less error prone than classic `library` or` require` by limiting the number of names that are available.


## Import Reusable Code

First of all, we are going to explain the example that we can find in the `box` web page itself (remember not to do library (box), as you will get an error):


```r
box::use(
    purrr,                          # 1
    tbl   = tibble,                 # 2
    dplyr = dplyr[filter, select],  # 3
    stats[st_filter = filter, ...]  # 4
)
```

What are we indicating with this box :: use statement?

1. First, the `purrr` package is imported and its functions are made accessible through the` $ `operator:


```r
purrr$reduce(c(1:10), sum)
```

```
## [1] 55
```

2. Second, the `tibble` package with the alias` tbl` is imported, so we access its functions as follows:


```r
library(dplyr)
set.seed(123)

df <- tbl$tibble(date  = seq.Date(from  = as.Date('2020-12-31'), 
                            length.out = 8, 
                            by = 'quarter'),
           value = rnorm(8)) %>%
  tbl$add_column(ID = 'base', .before = 'date')

df
```

```
## # A tibble: 8 x 3
##   ID    date         value
##   <chr> <date>       <dbl>
## 1 base  2020-12-31 -0.560 
## 2 base  2021-03-31 -0.230 
## 3 base  2021-07-01  1.56  
## 4 base  2021-10-01  0.0705
## 5 base  2021-12-31  0.129 
## 6 base  2022-03-31  1.72  
## 7 base  2022-07-01  0.461 
## 8 base  2022-10-01 -1.27
```

3. Third, the dplyr library is imported and also attach is used on the names dplyr::filter and dplyr::select.


```r
select(df, value)
```

```
## # A tibble: 8 x 1
##     value
##     <dbl>
## 1 -0.560 
## 2 -0.230 
## 3  1.56  
## 4  0.0705
## 5  0.129 
## 6  1.72  
## 7  0.461 
## 8 -1.27
```

```r
filter(df, value > 0.5)
```

```
## # A tibble: 2 x 3
##   ID    date       value
##   <chr> <date>     <dbl>
## 1 base  2021-07-01  1.56
## 2 base  2022-03-31  1.72
```

```r
dplyr$first(df$date)
```

```
## [1] "2020-12-31"
```

4. Atachs all the functions of the stats package (this is what the three ellipsis represents) and also uses the alias st_filter for the filter function. *In this way we can have the filter functions of dplyr and stats coexisting at the same time and without risk of errors.*


```r
st_filter(df$value, filter = rep(1, 3), sides = 1, circular = TRUE)
```

```
## Time Series:
## Start = 1 
## End = 8 
## Frequency = 1 
## [1] -1.3646207 -2.0557144  0.7680552  1.3990392  1.7585044  1.9148611  2.3052689
## [8]  0.9109200
```

## So how do I replace `library (pkg)`?

If what you want is to load the package and attach it in the 'search path', then the exact instruction you are looking for is the following:


```r
box::use(dplyr[...]) #equivalent library(dplyr)
```

What is the difference then with the following?


```r
box::use(dplyr)
```

The difference is that this last statement generates a dplyr object whose functions you can access through the '$' operator, but it does not attach the package.


# Reusable Modules

Perhaps one of the biggest advantages of `box` is the use of reusable modules without the need to create a package for it. Now that we have seen a short introduction on how to work with box to handle packages, let's see how we can use this package to load a module. The first thing we must do is create a script with the functions that we want to import, in our example case it will be these two functions:


```r
#' @export
subscribe = function () {
  
    message('You should subscribe to my NewsletteR!')
}

#' @export
bye = function () {
  
    message('I hope you liked this post :) I hope to see you soon here again!')
}
```

First, we must modify the options('box.path') to the path where we have our script so that we are able to import it correctly.


```r
options(box.path = getwd())

box::use(./box_post)
```

Once imported, we can access the functions found in the script through the `$` operator:


```r
box_post$subscribe()
```

```
## You should subscribe to my NewsletteR!
```

I hope you liked this post, see you in the next one!


```r
box_post$bye()
```

```
## I hope you liked this post :) I hope to see you soon here again!
```















