---
title: 'R Tip: reformulate() function'
author: "Alberto Almuiña"
date: '2020-01-17T21:13:14-05:00'
description: How to store a formula inside a variable?
slug: reformulate
categories: 
- R-tips
tags: 
- reformulate
---

**************
## `reformulate()` function
**************

Imagine you want to create an app that imputes the missing values of a certain column. The user will select this column in a slider and you can collect the value on your server.R. Let's see a case:


```r
library(recipes)
library(VIM)

data("airquality")

aggr(airquality, prop = F, combined = T, numbers = T, sortVars = T, sortCombs = T)
```

<img src="/posts/2020-01-17-reformulate/reformulate_files/figure-html/unnamed-chunk-1-1.png" width="672" />

```
## 
##  Variables sorted by number of missings: 
##  Variable Count
##     Ozone    37
##   Solar.R     7
##      Wind     0
##      Temp     0
##     Month     0
##       Day     0
```

As you can see, we have two columns with missing values. Suppose the user selects the 'Ozone' column. We are going to use the `recipes` package to impute the values with the knn algorithm:


```r
.f<-'Ozone ~ .'

airquality<-recipe(.f, airquality) %>% 
            step_knnimpute(all_outcomes(), neighbors = 3) %>% 
            prep() %>% 
            juice()
```

```
## Warning: `step_knnimpute()` was deprecated in recipes 0.1.16.
## Please use `step_impute_knn()` instead.
```

```
## Error: `x` should be a data frame, matrix, or tibble
```


We get an error because the function is not able to interpret the string as a formula. Let's use the reformulate function to create a formula


```r
.f<-reformulate(termlabels = '.', response = 'Ozone')

airquality<-recipe(.f, airquality) %>% 
            step_knnimpute(all_outcomes(), neighbors = 3) %>% 
            prep() %>% 
            juice()
```

```
## Warning: `step_knnimpute()` was deprecated in recipes 0.1.16.
## Please use `step_impute_knn()` instead.
```

```r
aggr(airquality, prop = F, combined = T, numbers = T, sortVars = T, sortCombs = T)
```

<img src="/posts/2020-01-17-reformulate/reformulate_files/figure-html/unnamed-chunk-3-1.png" width="672" />

```
## 
##  Variables sorted by number of missings: 
##  Variable Count
##   Solar.R     7
##      Wind     0
##      Temp     0
##     Month     0
##       Day     0
##     Ozone     0
```

It can be seen that now the function has worked correctly, as we can see in the graph, since we only have one variable with missing values.

****
### Another Option: `as.formula()` function
****

Another option to achieve the same result is to use the function as.formula().


```r
data('airquality')

.f<-as.formula('Ozone ~ .')

airquality<-recipe(.f, airquality) %>% 
            step_knnimpute(all_outcomes(), neighbors = 3) %>% 
            prep() %>% 
            juice()
```

```
## Warning: `step_knnimpute()` was deprecated in recipes 0.1.16.
## Please use `step_impute_knn()` instead.
```

```r
aggr(airquality, prop = F, combined = T, numbers = T, sortVars = T, sortCombs = T)
```

<img src="/posts/2020-01-17-reformulate/reformulate_files/figure-html/unnamed-chunk-4-1.png" width="672" />

```
## 
##  Variables sorted by number of missings: 
##  Variable Count
##   Solar.R     7
##      Wind     0
##      Temp     0
##     Month     0
##       Day     0
##     Ozone     0
```













