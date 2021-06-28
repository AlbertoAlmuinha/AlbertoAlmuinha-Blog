---
title: "Librería Box"
author: "Alberto Almuiña"
date: '2021-02-28T21:13:14-05:00'
description: Código R reutilizable y modular con el paquete Box
slug: box-package
tags:
- box
- modulos
- librerias
categories: 
- R-tips
---

# Introducción a la librería `box`

`box` es una de mis nuevas librerías favoritas en el ecosistema R. Desarrollada por Konrad Rudolph, su propósito principal es permitirnos organizar el código de una forma mucho más modular, principalmente a través de dos mecanismos, como bien se indica en su página de [pkgdown](https://klmr.me/box/index.html):

- Permite escribir código modular al tratar archivos y carpetas de código R como módulos independientes (potencialmente anidados), sin requerir que el usuario empaquete el código en una librería (wow!)

- Proporciona una nueva sintaxis para importar código reutilizable (tanto de paquetes como de módulos) que es más potente y menos propenso a errores que los clásicos `library` o `require` al limitar el número de nombres que están disponibles.


## Importar Código Reutilizable

En primer lugar, vamos a explicar el ejemplo que podemos encontrar en la propia página web de `box` (recuerda no hacer library(box), pues obtendrás un error):


```r
box::use(
    purrr,                          # 1
    tbl   = tibble,                 # 2
    dplyr = dplyr[filter, select],  # 3
    stats[st_filter = filter, ...]  # 4
)
```

Qué estamos indicando con esta instrucción de box::use?

1. En primer lugar, se importa el paquete `purrr` y sus funciones se hacen accesibles a través del operador `$`:


```r
purrr$reduce(c(1:10), sum)
```

```
## [1] 55
```

2. En segundo lugar, se importa el paquete `tibble` con el alias `tbl`, por lo que accedemos a sus funciones del siguiente modo:


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

3. En tercer lugar, se importa la librería dplyr y además se usa `attach` sobre los nombres dplyr::filter y dplyr::select.


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

4. 'Atachs' todas las funciones de la librería stats (esto es lo que representan los tres puntos suspensivos) y además utiliza para la función filter el alias st_filter. *De esta forma podemos tener conviviendo las funciones filter de dplyr y stats al mismo tiempo y sin riesgo de errores.*


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

## Entonces, cómo reemplazo `library(pkg)`?

Si lo que deseas es cargar el paquete y attach el mismo en el 'search path', entonces la instrucción exacta que buscas es la siguiente:


```r
box::use(dplyr[...]) #equivalent library(dplyr)
```

Cuál es la diferencia entonces con lo siguiente?


```r
box::use(dplyr)
```

La diferencia consiste en que esta última instrucción genera un objeto dplyr cuyas funciones puedes accedes a través del operador '$', pero no usa 'attach' (por lo tanto, tu paquete no se encontrará en el search path).


# Módulos Reutilizables

Quizá una de las mayores ventajas de `box` sea la utilización de módulos reutilizables sin la necesidad de crear un paquete para ello. Ahora que ya hemos visto una pequeña introducción sobre como trabajar con box para manejar paquetes, vamos a ver como podemos usar este paquete para cargar algún modulo. Lo primero que debemos hacer es crear un script con las funciones que queramos importar, en nuestro caso de ejemplo serán estas dos funciones:


```r
#' @export
subscribe = function () {
  
  message('Deberias subscribirte a mi NewsletteR!')
}

#' @export
bye = function () {
  
  message('Espero que te haya gustado este post :) Espero verte por aqui pronto!')
}
```

En primer lugar, debemos modificar el options('box.path') a la ruta donde tengamos nuestro script para que seamos capaces de importarlo de forma correcta.


```r
options(box.path = getwd())

box::use(./box_post_es)
```

Una vez importado, ya podemos acceder a las funciones que se encuentran en el script a través del operador `$`:


```r
box_post_es$subscribe()
```

```
## Deberias subscribirte a mi NewsletteR!
```

Espero que os haya gusto este post, nos vemos en el próximo!


```r
box_post_es$bye()
```

```
## Espero que te haya gustado este post :) Espero verte por aqui pronto!
```















