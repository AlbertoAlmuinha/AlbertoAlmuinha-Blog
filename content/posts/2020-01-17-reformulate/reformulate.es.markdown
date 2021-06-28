---
title: "La función `reformulate()`"
author: "Alberto Almuiña"
date: '2020-01-17T21:13:14-05:00'
description: Cómo almacenar una fórmula en una variable?
slug: reformulate_es
categories: 
- R-tips
tags: 
- reformulate
---

**************
## Función `reformulate()` 
**************

Imagine que desea crear una aplicación que impute los missing values de una determinada variable. El usuario seleccionará esta columna en un slider y podrá recopilar el valor en su server.R. Veamos un caso:


```r
library(recipes)
library(VIM)

data("airquality")

aggr(airquality, prop = F, combined = T, numbers = T, sortVars = T, sortCombs = T)
```

<img src="/posts/2020-01-17-reformulate/reformulate.es_files/figure-html/unnamed-chunk-1-1.png" width="672" />

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

Como puede ver, tenemos dos columnas con missing values. Supongamos que el usuario selecciona la columna 'Ozone'. Vamos a utilizar el paquete `recipes` para imputar los valores con el algoritmo knn:


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


Obtenemos un error porque la función no puede interpretar la cadena como una fórmula. Usemos la función 'reformulate' para crear una fórmula:


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

<img src="/posts/2020-01-17-reformulate/reformulate.es_files/figure-html/unnamed-chunk-3-1.png" width="672" />

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

Se puede ver que ahora la función ha funcionado correctamente, como podemos ver en el gráfico, ya que solo tenemos una variable con missing values.

****
### Otra opción: la función `as.formula()` 
****

Otra opción para lograr el mismo resultado es usar la función as.formula ().


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

<img src="/posts/2020-01-17-reformulate/reformulate.es_files/figure-html/unnamed-chunk-4-1.png" width="672" />

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













