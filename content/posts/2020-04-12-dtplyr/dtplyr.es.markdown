---
title: "Librería dtplyr en R"
author: "Alberto Almuiña"
date: '2020-04-12T21:13:14-05:00'
description: Combina el rendimiento de data.table con la sintaxis legible de dplyr.
slug: dtplyr_es
tags:
- dplyr
- data.table
- dtplyr
categories: 
- Rendimiento
---

## Introducción al paquete dtplyr

Siempre ha habido un intenso debate en la comunidad R sobre cuál es la mejor biblioteca para el manejo de datos: `dplyr` con su sintaxis simple y alta legibilidad o` data.table` con su increíble velocidad. La mayoría de los usuarios usan dplyr porque tiene una curva de aprendizaje más rápida y es más intuitiva.

** Entonces ... ¿qué es dtplyr? **

`dtplyr` es una librería que permite usar la sintaxis dplyr en objetos de clase data.table. Esto se logra mediante una evaluación diferida, lo que significa que escribe su código, pero no se evaluará hasta que devuelva los resultados (con las funciones as.data.frame, as.data.table o as_tibble). Este enfoque permite mejorar la velocidad hasta que sea casi similar a la que ofrece data.table.

## ¿Son comparables los rendimientos de data.table y dtplyr?

Sin duda sí. Pero también debemos tener en cuenta que dtplyr siempre será algo más lento que data.table. ¿Por qué? Como se indica en el repositorio oficial del paquete, por tres razones principales:

* Cada verbo dplyr debe hacer un trabajo para convertir la sintaxis dplyr en sintaxis data.table. **Esto lleva un tiempo proporcional a la complejidad del código de entrada, no a los datos de entrada, por lo que debe ser una sobrecarga insignificante para grandes conjuntos de datos.**

* Algunas expresiones data.table no tienen un equivalente directo de dplyr.

* Para coincidir con la semántica dplyr, `mutate ()` no se modifica en su lugar de forma predeterminada. Esto significa que la mayoría de las expresiones que involucran `mutate ()` deben hacer una copia que no sería necesaria si estuviera usando data.table directamente. (Puede optar por este comportamiento en `lazy_dt ()` con immutable = FALSE).

## ¿Cómo usar dtplyr?


```r
library(data.table)
library(dtplyr)
library(dplyr, warn.conflicts = FALSE)

data('mtcars')
```

El primer paso es crear una tabla diferida con la función `lazy_dt ()`:


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

Puede ver en 'Call' el código traducido a la sintaxis data.table. Probemos la traducción de algún verbo típico de dplyr:


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


Como se mencionó al principio de la publicación, para extraer los resultados debemos usar una de las siguientes funciones: `as.data.frame ()`, `as.data.table ()` o `as_tibble`. Vamos a ver:


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


## Conclusiones

Tanto dplyr como data.table son opciones excelentes y maduras para el manejo de datos. Si es un usuario habitual de data.table, quizás dtplyr sea una opción interesante si necesita una mejor legibilidad. Si, por otro lado, usted es un usuario habitual de dplyr, este paquete será de gran ayuda para aumentar el rendimiento al analizar grandes conjuntos de datos manteniendo la sintaxis a la que está acostumbrado.









