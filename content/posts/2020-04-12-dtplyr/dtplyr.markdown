---
title: "dtplyr R package"
author: "Alberto Almuiña"
date: '2020-04-12T21:13:14-05:00'
description: Combine data.table performance with readable dplyr syntax.
slug: dtplyr
tags:
- dtplyr
- data.table
- dplyr
categories: 
- Performance
---

## Introduction to dtplyr package

There has always been an intense debate in the R community about which is the best library to data wrangling: `dplyr` with its simple syntax and high readability or `data.table` with its incredible speed. Most useRs go for dplyr because it has a faster learning curve and is more intuitive.

**So.. what is dtplyr?**

`dtplyr` is a library that allows you to use the dplyr syntax on data.table class objects. This is accomplished through lazy evaluation, which means you write your code but it won't be evaluated until you return the results (with the as.data.frame, as.data.table, or as_tibble functions). This approach allows to improve the speed until it is almost similar to the one offered by data.table.

## Are data.table and dtplyr performances comparable?

Without a doubt, yes. But we also have to keep in mind that dtplyr will always be somewhat slower than data.table. Why? As stated in the package's official repository, for three main reasons:

* Each dplyr verb must do some work to convert dplyr syntax to data.table syntax. **This takes time proportional to the complexity of the input code, not the input data, so should be a negligible overhead for large datasets.**

* Some data.table expressions have no direct dplyr equivalent.

* To match dplyr semantics, `mutate()` does not modify in place by default. This means that most expressions involving `mutate()` must make a copy that would not be necessary if you were using data.table directly. (You can opt out of this behaviour in `lazy_dt()` with immutable = FALSE).

## How to use dtplyr?


```r
library(data.table)
library(dtplyr)
library(dplyr, warn.conflicts = FALSE)

data('mtcars')
```

The first step is to create a lazy table with the `lazy_dt()` function:


```r
(mtcars_lazy<-lazy_dt(mtcars))
```

```
## Source: local data table [32 x 11]
## Call:   `_DT1`
## 
##     mpg   cyl  disp    hp  drat    wt  qsec    vs    am  gear  carb
##   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
## 1  21       6   160   110  3.9   2.62  16.5     0     1     4     4
## 2  21       6   160   110  3.9   2.88  17.0     0     1     4     4
## 3  22.8     4   108    93  3.85  2.32  18.6     1     1     4     1
## 4  21.4     6   258   110  3.08  3.22  19.4     1     0     3     1
## 5  18.7     8   360   175  3.15  3.44  17.0     0     0     3     2
## 6  18.1     6   225   105  2.76  3.46  20.2     1     0     3     1
## # ... with 26 more rows
## 
## # Use as.data.table()/as.data.frame()/as_tibble() to access results
```

You can see in 'Call' the code translated to data.table syntax. Let's try the translation of some typical verb of dplyr: 


```r
mtcars_lazy %>%
  filter(cyl == 6) %>%
  select(c(mpg, cyl))
```

```
## Source: local data table [7 x 2]
## Call:   `_DT1`[cyl == 6, .(mpg, cyl)]
## 
##     mpg   cyl
##   <dbl> <dbl>
## 1  21       6
## 2  21       6
## 3  21.4     6
## 4  18.1     6
## 5  19.2     6
## 6  17.8     6
## # ... with 1 more row
## 
## # Use as.data.table()/as.data.frame()/as_tibble() to access results
```


As mentioned at the beginning of the post, to extract the results we must use one of the following functions: `as.data.frame()`, `as.data.table()` or `as_tibble`. Let's check it out:


```r
mtcars_lazy %>%
  filter(cyl == 6) %>%
  select(c(mpg, cyl)) %>%
  as.data.table()
```

```
##     mpg cyl
## 1: 21.0   6
## 2: 21.0   6
## 3: 21.4   6
## 4: 18.1   6
## 5: 19.2   6
## 6: 17.8   6
## 7: 19.7   6
```


## Conclusions

Both dplyr and data.table are excellent and mature options for data wrangling. If you are a regular user of data.table, maybe dtplyr is an interesting option if you need a better readability. If, on the other hand, you are a regular user of dplyr, this package will be of great help to increase the performance when analyzing large datasets while maintaining the syntax that you are used to.







