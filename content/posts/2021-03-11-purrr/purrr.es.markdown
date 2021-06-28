---
title: "Por qué deberías aprender a usar purrr"
author: "Alberto Almuiña"
date: '2021-03-11T02:13:14-05:00'
description: Introducción a la programación funcional con `purrr`
slug: purrr-es
tags:
- purrr
- functional programming
categories: 
- tidyverse
---

## Por qué deberías aprender a usar purrr?

Según la propia descripción que se hace del paquete en la página oficial del mismo, "purrr mejora el conjunto de herramientas de programación funcional (FP) de R al proporcional un conjunto de herramientas completo y consistente para trabajar con funciones y vectores".

Pero entonces, qué es un lenguaje de programación funcional? Según explica Hadley en [Advanced R](https://adv-r.hadley.nz/fp.html) hay muchas definiciones que hacen funcional un lenguaje, si bien podemos encontrar dos características comunes a todas ellas:

-   *Funciones de primera clase:* Esto significa que las funciones se comportan como cualquier otra estructura. Es decir, puedes almacenarla en una variable, almacenarlas en listas, pasarlas como argumentos a otras funciones, crearlas dentro de otras funciones o incluso devolverlas como resultado de una función.

-   Muchos lenguajes funcionales requieren también *funciones puras*. Una función se considera pura si satisface las siguientes dos características:

    1.  La salida sólo depende de los argumentos de la entrada, por lo que si llamas a la función con los mismos argumentos, obtendrás el mismo resultado. Funciones como `run.if()` o `read.csv()` no son puras.

    2.  La función no tiene efectos secundarios como cambiar el valor de una variable global o escribir a disco. Funciones como `print()` o `<-` no son puras.

Estrictamente hablando, R no es un lenguaje funcional porque no requiere que escribas funciones puras. Sin embargo, en partes de tu código si puedes adoptar un estilo funcional y de hecho deberías. Pero, qué es un estilo funcional? Un estilo funcional consiste en descomponer un problema grande en piezas más pequeñas y resolver cada una de esas piezas a partir de una función o un conjunto de funciones. Lo que se consigue es descomponer el problema en funciones aisladas que operan independientemente y que son más fáciles de entender. Existen tres técnicas para descomponer el problema en piezas más pequeñas:

-   Los *funcionales* son funciones que toman una función como argumento que resuelve el problema para un solo input y lo generaliza para manejar cualquier número de inputs. Un ejemplo sería la función `lapply()`.

-   Factorías de funciones: funciones que crean otras funciones.

-   Operadores de funciones: funciones que toman como argumento una función y devuelven como salida otra función.

En este post nos centraremos en los funcionales a través del paquete `purrr`.

## Funcionales con `purrr`

Un funcional es una función que toma una función como entrada y devuelve un vector como salida (o al menos esa era la idea en un inicio, actualmente la salida es modulable según el tipo de funcional. De esta forma, podemos obtener funcionales que nos devuelvan una lista, otros que nos devuelvan un vector o un data frame). El funcional fundamental en purrr es `map()`, que recibe como argumentos un vector, una lista o un data frame y una función y aplica dicha función a cada elemento del vector, lista o data frame y devuelve el resultado en una lista. Para tener una mejor intuición, veamos una imagen:

![Fuente: Advanced R](/img/purrr.es_files/map.PNG)

Lo que sucede es lo siguiente:

-   Si el argumento de entrada es un vector, `purrr::map()` aplicará la función pasada como argumento a cada elemento del vector.

-   Si el argumento de entrada es una lista, entonces map() aplicará la función pasada como argumento a cada elemento de la lista.

-   Por el contrario, si el argumento de entrada es un data frame, entonces map() aplicará la función a cada columna del mismo.

Veamos un ejemplo sencillo:


```r
library(purrr)
library(tidyverse)

double <- function(x) x*2

purrr::map(c(1:3), double)
```

```
## [[1]]
## [1] 2
## 
## [[2]]
## [1] 4
## 
## [[3]]
## [1] 6
```

```r
purrr::map(list(2, 4, 10), double)
```

```
## [[1]]
## [1] 4
## 
## [[2]]
## [1] 8
## 
## [[3]]
## [1] 20
```

Debemos tener en cuenta que `purrr::map` siempre devuelve una lista, pero en ocasiones esto no es lo más cómodo ni práctico. Si conocemos de antemano la salida que deseamos, podemos utilizar alguna de las variantes de map (existen más de las que se muestran en el siguiente ejemplo, te recomiendo que explores las opciones existentes tecleando `map_`):


```r
purrr::map_dbl(c(1:3), double)
```

```
## [1] 2 4 6
```

```r
purrr::map_chr(c(1:3), double)
```

```
## [1] "2.000000" "4.000000" "6.000000"
```

```r
purrr::map_lgl(c(1:3), is.na)
```

```
## [1] FALSE FALSE FALSE
```

Otra opción es utilizar funciones anónimas como parámetro, de la siguiente forma:


```r
purrr::map(c(1:3), function(x) x*2)
```

```
## [[1]]
## [1] 2
## 
## [[2]]
## [1] 4
## 
## [[3]]
## [1] 6
```

Incluso, si no te apetece teclear tanto, purrr ofrece un shortcut para que puedas escribir tus funciones anónimas de la siguiente manera:


```r
purrr::map(c(1:3), ~{.x*2})
```

```
## [[1]]
## [1] 2
## 
## [[2]]
## [1] 4
## 
## [[3]]
## [1] 6
```

Para entender lo que está sucediendo, vamos a hacer uso de la función `purrr::as_mapper()`:


```r
purrr::as_mapper(~{.x*2})
```

```
## <lambda>
## function (..., .x = ..1, .y = ..2, . = ..1) 
## {
##     .x * 2
## }
## attr(,"class")
## [1] "rlang_lambda_function" "function"
```

Los argumentos de la función parecen un poco extraños pero te permiten hacer referencia a `.` para funciones de un argumento, a `.x e .y` para funciones de dos argumentos y a `..1, ..2, ..3 etc` para una función con un número arbitrario de argumentos. `.` permanece por compatibilidad pero no se recomienda utilizarlo.

Para utilizar argumentos adicionales de una función, existen dos maneras para hacerlo: la primera es a través de una función anónima:


```r
purrr::map_dbl(list(1:5, c(1, 4, NA)), 
               ~ mean(.x, na.rm = TRUE))
```

```
## [1] 3.0 2.5
```

La segunda es utilizar los argumentos después de la propia función. Esto sucede porque los argumentos son pasados a la función a través del argumento `...`. Como una imagen vale más que mil palabras, mejor veámoslo de forma gráfica:

![Fuente: Advanced R](/img/purrr.es_files/map_args.PNG)


```r
purrr::map_dbl(list(1:5, c(1, 4, NA)),
               mean, na.rm = TRUE)
```

```
## [1] 3.0 2.5
```

Estos son los fundamentos a partir de los cuales puedes comenzar a utilizar los funcionales de purrr de una forma segura. El paso a los funcionales que iteran sobre múltiples argumentos como input en vez de sobre uno (es decir, el paso a la generalización) es trivial, como veremos a continuación.

## Los variantes de map

`purrr` nos ofrece mucha versatilidad a la hora de iterar y para ello nos brinda otro set de funciones cuyo objetivo es tomar como input una función y aplicarla a cada par de elementos extraídos o bien de dos vectores, o bien de dos listas. La función básica de este set de funciones es `purrr::map2()`, que análogamente a map(), siempre retornará una lista. Para entenderlo mejor, vamos con un ejemplo muy sencillo. Vamos a intentar sumar dos vectores elemento a elemento:


```r
a <- c(1, 2, 3)
b <- c(1, 2, 3)

add <- function(x, y) {x+y}

purrr::map2(a, b, add)
```

```
## [[1]]
## [1] 2
## 
## [[2]]
## [1] 4
## 
## [[3]]
## [1] 6
```

Un poco engorroso verdad? Pues sí! Vamos a aplicar todo lo aprendido en las secciones anteriores. En primer lugar, podemos devolver un vector si sabemos el tipo que deseamos devolver, en este caso 'double', simplemente usando las variantes de la función básica con `map2_*tipo-deseado*`:


```r
a <- c(1, 2, 3)
b <- c(1, 2, 3)

add <- function(x, y) {x+y}

purrr::map2_dbl(a, b, add)
```

```
## [1] 2 4 6
```

Esto es todo? No, recuerda que podemos usar funciones anónimas para no escribir tanto código, cuando usamos funciones tan cortas:


```r
purrr::map2_dbl(1:3, 1:3, ~{.x +.y})
```

```
## [1] 2 4 6
```

El siguiente paso en generalización sería el uso de la función `purrr::pmap()`, aunque posiblemente a estas alturas ya intuyas como funciona esta función, verdad? La idea aquí es pasar una lista como argumento que contenga en su interior X vectores, listas o data frames sobre los que se aplicará la función pasada como argumento. Para ello, imaginemos que quiero sumar los elementos de tres vectores, la generalización del ejemplo anterior:


```r
a <- c(1, 2, 3)
b <- c(1, 2, 3)
c <- c(1 ,2, 3)

add <- function(x, y, z) {x + y + z}

purrr::pmap(list(a, b, c), add)
```

```
## [[1]]
## [1] 3
## 
## [[2]]
## [1] 6
## 
## [[3]]
## [1] 9
```

Ahora sólo es cuestión de simplificar aplicando la misma lógica que hasta ahora, seleccionar la función *pmap\_* que nos devuelva el resultado con el formato que nosotros queramos y utilizar una función anónima para que sea menos engorroso (recuerda que cuando había más de dos argumentos, la sintaxis correcta dentro de la función anónima era *..1, ..2, ..3, ..4 etc* para identificar los correspondientes argumentos):


```r
a <- c(1, 2, 3)
b <- c(1, 2, 3)
c <- c(1 ,2, 3)

purrr::pmap_dbl(list(a, b, c), ~{..1 + ..2 + ..3})
```

```
## [1] 3 6 9
```

Otro set de funciones muy interesantes son `purrr::walk()` y sus variantes (walk2, pwalk). Estas funciones nacen porque en ocasiones no estamos interesados en que la función nos devuelva un resultado, si no en los efectos secundarios que produce la función. Este es el caso de funciones como print(), message(), plot() o write.csv(). Cuando usamos este tipo de funciones con `purrr::map()` no sólo obtenemos el efecto secundario deseado, si no que ademas obtendremos una lista con valores nulos. Fijémonos en el siguiente ejemplo:


```r
saludar <- function(.nombre) {message(paste('Hola ', .nombre))}
nombres <- c('Alberto', 'Diana')

purrr::map(nombres, saludar)
```

```
## Hola  Alberto
```

```
## Hola  Diana
```

```
## [[1]]
## NULL
## 
## [[2]]
## NULL
```

Nuestro interés radica en imprimir el mensaje, pero no deseamos obtener ningún resultado de vuelta, y aquí es donde entra en juego `walk()`. Esta función básicamente lo que hace internamente es llamar a la función *map* y devolver los resultados de manera invisible:

![Fuente: Advanced R](/img/purrr.es_files/walk.PNG)


```r
body(walk)
```

```
## {
##     map(.x, .f, ...)
##     invisible(.x)
## }
```

```r
purrr::walk(nombres, saludar)
```

```
## Hola  Alberto
```

```
## Hola  Diana
```

Otro caso de uso muy común es cuando queremos guardar en disco varios ficheros en varias rutas. Para ello utilizaremos la función `purrr:walk2()`, que tomará como argumentos los data frames a guardar, las rutas y la función que se encargará de realizar dicha tarea:


```r
temp <- tempfile()
dir.create(temp)

cyls <- split(mtcars, mtcars$cyl)
paths <- file.path(temp, paste0("cyl-", names(cyls), ".csv"))

purrr::walk2(cyls, paths, write.csv)

dir(temp)
```

```
## [1] "cyl-4.csv" "cyl-6.csv" "cyl-8.csv"
```

Básicamente, lo que está sucediendo cuando llamamos a walk2 es lo siguiente: write.csv(cyls[[1]], paths[[1]]), write.csv(cyls[[2]], paths[[2]]), write.csv(cyls[[3]], paths[[3]]).

Otra función muy interesante es `purrr::modify()`, la cuál funciona como map() pero te garantiza que la salida de tu función tendrá el mismo tipo que tu entrada. Esto significa que si tu entrada es un vector, tu salida también lo será. Si tu entrada es un data frame, tu salida también lo será. Veamos un ejemplo:


```r
purrr::modify(c(1, 2, 3), ~ .x * 2)
```

```
## [1] 2 4 6
```


```r
purrr::modify(list(1, 2, 3), ~ .x * 2)
```

```
## [[1]]
## [1] 2
## 
## [[2]]
## [1] 4
## 
## [[3]]
## [1] 6
```


```r
purrr::modify(data.frame(valor = c(1, 2, 3)), ~ .x * 2)
```

```
##   valor
## 1     2
## 2     4
## 3     6
```

## Ejemplo Práctico: Planetas Star Wars

Para ahondar más en el universo `purrr` vamos a ver como podríamos utilizar lo aprendido y otras funciones adicionales del paquete para explorar una lista que contiene información sobre 61 planetas de la saga de Star Wars. Vamos a ver en primer lugar la información sobre el primer planeta para saber qué información nos podemos encontrar:


```r
library(repurrrsive)

planets<-repurrrsive::sw_planets
planets[[1]]
```

```
## $name
## [1] "Alderaan"
## 
## $rotation_period
## [1] "24"
## 
## $orbital_period
## [1] "364"
## 
## $diameter
## [1] "12500"
## 
## $climate
## [1] "temperate"
## 
## $gravity
## [1] "1 standard"
## 
## $terrain
## [1] "grasslands, mountains"
## 
## $surface_water
## [1] "40"
## 
## $population
## [1] "2000000000"
## 
## $residents
## [1] "http://swapi.co/api/people/5/"  "http://swapi.co/api/people/68/"
## [3] "http://swapi.co/api/people/81/"
## 
## $films
## [1] "http://swapi.co/api/films/6/" "http://swapi.co/api/films/1/"
## 
## $created
## [1] "2014-12-10T11:35:48.479000Z"
## 
## $edited
## [1] "2014-12-20T20:58:18.420000Z"
## 
## $url
## [1] "http://swapi.co/api/planets/2/"
```

Una ventaja de purrr es que es pipe-friendly, por lo que podremos utilizar sus funcionales con el famoso pipe de magritrr. Por ejemplo, vamos a ver cuáles son los nombres de los 61 planetas de Star Wars:


```r
planets %>% purrr::map_chr(~.$name)
```

```
##  [1] "Alderaan"       "Yavin IV"       "Hoth"           "Dagobah"       
##  [5] "Bespin"         "Endor"          "Naboo"          "Coruscant"     
##  [9] "Kamino"         "Geonosis"       "Utapau"         "Mustafar"      
## [13] "Kashyyyk"       "Polis Massa"    "Mygeeto"        "Felucia"       
## [17] "Cato Neimoidia" "Saleucami"      "Stewjon"        "Eriadu"        
## [21] "Corellia"       "Rodia"          "Nal Hutta"      "Dantooine"     
## [25] "Bestine IV"     "Ord Mantell"    "unknown"        "Trandosha"     
## [29] "Socorro"        "Mon Cala"       "Chandrila"      "Sullust"       
## [33] "Toydaria"       "Malastare"      "Dathomir"       "Ryloth"        
## [37] "Aleen Minor"    "Vulpter"        "Troiken"        "Tund"          
## [41] "Haruun Kal"     "Cerea"          "Glee Anselm"    "Iridonia"      
## [45] "Tholoth"        "Iktotch"        "Quermia"        "Dorin"         
## [49] "Champala"       "Mirial"         "Serenno"        "Concord Dawn"  
## [53] "Zolan"          "Ojom"           "Skako"          "Muunilinst"    
## [57] "Shili"          "Kalee"          "Umbara"         "Tatooine"      
## [61] "Jakku"
```

Como ves, con purrr también podemos extraer elementos de una lista u objeto sobre el que iteremos de forma sencilla! Compliquemos un poco más las cosas. Vamos a contar el número de caracteres de cada nombre:


```r
planets %>% purrr::map_chr(~.$name) %>% purrr::map_int(~str_length(.x))
```

```
##  [1]  8  8  4  7  6  5  5  9  6  8  6  8  8 11  7  7 14  9  7  6  8  5  9  9 10
## [26] 11  7  9  7  8  9  7  8  9  8  6 11  7  7  4 10  5 11  8  7  7  7  5  8  6
## [51]  7 12  5  4  5 10  5  5  6  8  5
```

Y si ahora me quiero quedar con aquellos que contengan más de 10 caracteres? Claro! purrr ofrece dos funciones, `keep` y `discard` que precisamente nos permiten seleccionar o descartar elementos en base a un predicado:


```r
planets %>% purrr::map_chr(~.$name) %>% purrr::map_int(~str_length(.x)) %>% purrr::keep(~ .x > 8)
```

```
##  [1]  9 11 14  9  9  9 10 11  9  9  9 11 10 11 12 10
```

Aunque también puedes filtrar directamente la lista y quedarte con los elementos que te interesan. Qué significa esto? Que quiero quedarme con los planetas de la lista cuyos nombres tengan más de 8 caracteres. Hagámoslo:


```r
lista_reducida <- planets %>% purrr::keep(~str_length(.$name)>8)

length(lista_reducida)
```

```
## [1] 16
```

Otra función interesante es `pluck`, la cual te permite extraer elementos de una lista de forma flexible. En este caso simplemente extraemos el primer elemento:


```r
lista_reducida %>% purrr::pluck(1) #Equivalente a lista_reducida[[1]]
```

```
## $name
## [1] "Coruscant"
## 
## $rotation_period
## [1] "24"
## 
## $orbital_period
## [1] "368"
## 
## $diameter
## [1] "12240"
## 
## $climate
## [1] "temperate"
## 
## $gravity
## [1] "1 standard"
## 
## $terrain
## [1] "cityscape, mountains"
## 
## $surface_water
## [1] "unknown"
## 
## $population
## [1] "1000000000000"
## 
## $residents
## [1] "http://swapi.co/api/people/34/" "http://swapi.co/api/people/55/"
## [3] "http://swapi.co/api/people/74/"
## 
## $films
## [1] "http://swapi.co/api/films/5/" "http://swapi.co/api/films/4/"
## [3] "http://swapi.co/api/films/6/" "http://swapi.co/api/films/3/"
## 
## $created
## [1] "2014-12-10T11:54:13.921000Z"
## 
## $edited
## [1] "2014-12-20T20:58:18.432000Z"
## 
## $url
## [1] "http://swapi.co/api/planets/9/"
```

Otras funciones muy interesantes son `partial()` y `compose()`. partial() te permite modificar una función para asignar unos valores a los parámetros que te interesen, mientras que compose te permite concatenar varias funciones para que se ejecuten en un determinado orden. Veamos un ejemplo de cómo poder aplicar estas funciones. Vamos a coger el campo terrain y vamos a sustituir las comas por guiones y posteriormente pondremos los caracteres con formato titulo (la primera letra de cada palabra en mayúscula):


```r
string_replace <- purrr::partial(str_replace_all, pattern = ',', replacement = '-')

.f <- purrr::compose(str_to_title, string_replace)

planets %>% purrr::map_chr(~.$terrain) %>% purrr::map_chr(.f)
```

```
##  [1] "Grasslands- Mountains"                   
##  [2] "Jungle- Rainforests"                     
##  [3] "Tundra- Ice Caves- Mountain Ranges"      
##  [4] "Swamp- Jungles"                          
##  [5] "Gas Giant"                               
##  [6] "Forests- Mountains- Lakes"               
##  [7] "Grassy Hills- Swamps- Forests- Mountains"
##  [8] "Cityscape- Mountains"                    
##  [9] "Ocean"                                   
## [10] "Rock- Desert- Mountain- Barren"          
## [11] "Scrublands- Savanna- Canyons- Sinkholes" 
## [12] "Volcanoes- Lava Rivers- Mountains- Caves"
## [13] "Jungle- Forests- Lakes- Rivers"          
## [14] "Airless Asteroid"                        
## [15] "Glaciers- Mountains- Ice Canyons"        
## [16] "Fungus Forests"                          
## [17] "Mountains- Fields- Forests- Rock Arches" 
## [18] "Caves- Desert- Mountains- Volcanoes"     
## [19] "Grass"                                   
## [20] "Cityscape"                               
## [21] "Plains- Urban- Hills- Forests"           
## [22] "Jungles- Oceans- Urban- Swamps"          
## [23] "Urban- Oceans- Swamps- Bogs"             
## [24] "Oceans- Savannas- Mountains- Grasslands" 
## [25] "Rocky Islands- Oceans"                   
## [26] "Plains- Seas- Mesas"                     
## [27] "Unknown"                                 
## [28] "Mountains- Seas- Grasslands- Deserts"    
## [29] "Deserts- Mountains"                      
## [30] "Oceans- Reefs- Islands"                  
## [31] "Plains- Forests"                         
## [32] "Mountains- Volcanoes- Rocky Deserts"     
## [33] "Swamps- Lakes"                           
## [34] "Swamps- Deserts- Jungles- Mountains"     
## [35] "Forests- Deserts- Savannas"              
## [36] "Mountains- Valleys- Deserts- Tundra"     
## [37] "Unknown"                                 
## [38] "Urban- Barren"                           
## [39] "Desert- Tundra- Rainforests- Mountains"  
## [40] "Barren- Ash"                             
## [41] "Toxic Cloudsea- Plateaus- Volcanoes"     
## [42] "Verdant"                                 
## [43] "Lakes- Islands- Swamps- Seas"            
## [44] "Rocky Canyons- Acid Pools"               
## [45] "Unknown"                                 
## [46] "Rocky"                                   
## [47] "Unknown"                                 
## [48] "Unknown"                                 
## [49] "Oceans- Rainforests- Plateaus"           
## [50] "Deserts"                                 
## [51] "Rainforests- Rivers- Mountains"          
## [52] "Jungles- Forests- Deserts"               
## [53] "Unknown"                                 
## [54] "Oceans- Glaciers"                        
## [55] "Urban- Vines"                            
## [56] "Plains- Forests- Hills- Mountains"       
## [57] "Cities- Savannahs- Seas- Plains"         
## [58] "Rainforests- Cliffs- Canyons- Seas"      
## [59] "Unknown"                                 
## [60] "Desert"                                  
## [61] "Deserts"
```

## Ejemplo Práctico: Nested Gapminder

Finalmente, vamos a ver un ejemplo práctico de como combinar un dataset anidado (nested) con los funcionales de `purrr` para explotar toda la potencia de ambas soluciones. Pero en primer lugar, qué es un data frame anidado? Básicamente, es un data frame que tiene una o más columnas formadas por listas de data frames. Para que quede claro el concepto, veamos un ejemplo:


```r
(gap_nested <- repurrrsive::gap_nested)
```

```
## # A tibble: 142 x 3
##    country     continent data             
##    <fct>       <fct>     <list>           
##  1 Afghanistan Asia      <tibble [12 x 4]>
##  2 Albania     Europe    <tibble [12 x 4]>
##  3 Algeria     Africa    <tibble [12 x 4]>
##  4 Angola      Africa    <tibble [12 x 4]>
##  5 Argentina   Americas  <tibble [12 x 4]>
##  6 Australia   Oceania   <tibble [12 x 4]>
##  7 Austria     Europe    <tibble [12 x 4]>
##  8 Bahrain     Asia      <tibble [12 x 4]>
##  9 Bangladesh  Asia      <tibble [12 x 4]>
## 10 Belgium     Europe    <tibble [12 x 4]>
## # ... with 132 more rows
```

Como se puede observar, tenemos un dataset (tibble) en el que por cada continente y país, en la columna 'data' tenemos almacenado otro dataset (tibble). Un buen lugar para aprender más sobre este tipo de datasets es en la propia página de [tidyr](https://tidyr.tidyverse.org/articles/nest.html) (ya que se crean con la función tidyr::nest). Para extraer la información en estos tibbles en la columna data, podemos utilizar la función *pluck* aprendida anteriormente. Por ejemplo, vamos a ver los datos guardados para Afghanistan:


```r
gap_nested %>% pluck('data', 1)
```

```
## # A tibble: 12 x 4
##     year lifeExp      pop gdpPercap
##    <int>   <dbl>    <int>     <dbl>
##  1  1952    28.8  8425333      779.
##  2  1957    30.3  9240934      821.
##  3  1962    32.0 10267083      853.
##  4  1967    34.0 11537966      836.
##  5  1972    36.1 13079460      740.
##  6  1977    38.4 14880372      786.
##  7  1982    39.9 12881816      978.
##  8  1987    40.8 13867957      852.
##  9  1992    41.7 16317921      649.
## 10  1997    41.8 22227415      635.
## 11  2002    42.1 25268405      727.
## 12  2007    43.8 31889923      975.
```

Una de las grandes ventajas de este formato es que nos permite aplicar los funcionales de purrr dentro del propio data frame, por ejemplo, podemos aplicar un modelo ARIMA con los datos de cada tibble en la columna 'data' para modelar la esperanza de vida en función de las otras variables: la población y el pib per cápita. Lo que hacemos es guardar en una nueva columna denominada 'arima' la información de cada modelo:


```r
library(modeltime)
library(tidymodels)
```

```
## -- Attaching packages -------------------------------------- tidymodels 0.1.3 --
```

```
## v broom        0.7.6      v rsample      0.1.0 
## v dials        0.0.9      v tune         0.1.5 
## v infer        0.5.4      v workflows    0.2.2 
## v modeldata    0.1.0      v workflowsets 0.0.2 
## v parsnip      0.1.5      v yardstick    0.0.8 
## v recipes      0.1.16
```

```
## -- Conflicts ----------------------------------------- tidymodels_conflicts() --
## x scales::discard() masks purrr::discard()
## x dplyr::filter()   masks stats::filter()
## x recipes::fixed()  masks stringr::fixed()
## x dplyr::lag()      masks stats::lag()
## x yardstick::spec() masks readr::spec()
## x recipes::step()   masks stats::step()
## * Use tidymodels_prefer() to resolve common conflicts.
```

```r
gap_nested <- gap_nested %>%
              mutate(arima = map(data, ~arima_reg(mode = 'regression') %>%
                                  set_engine('auto_arima') %>%
                                  fit(lifeExp ~ pop + gdpPercap + as.Date(ISOdate(year, 1, 1)), data = .x)))
```

```
## frequency = 1 observations per 5 years
```

```
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
## frequency = 1 observations per 5 years
```

Veamos el modelo ARIMA seleccionado para la esperanza de vida en Afghanistán:


```r
gap_nested %>% pluck('arima', 1)
```

```
## parsnip model object
## 
## Fit time:  361ms 
## Series: outcome 
## Regression with ARIMA(2,0,0) errors 
## 
## Coefficients:
##          ar1      ar2  intercept  pop  gdp_percap
##       1.8426  -0.8951    32.7412    0      0.0014
## s.e.  0.0937   0.0906     1.6874    0      0.0013
## 
## sigma^2 estimated as 0.5101:  log likelihood=-12.82
## AIC=37.65   AICc=54.45   BIC=40.56
```

Ahora podemos extraer los valores del modelo para el periodo de entrenamiento y almacenarlos en otra columna:


```r
gap_nested <- gap_nested %>% 
              mutate(fitted = map(arima, ~.$fit$models$model_1$fitted))

gap_nested %>% pluck('fitted', 1)
```

```
## Time Series:
## Start = 1 
## End = 12 
## Frequency = 1 
##  [1] 29.44926 29.80294 31.99193 33.64891 35.85104 38.16024 40.12213 40.87741
##  [9] 41.51748 42.88908 41.41370 42.97920
```

Finalmente, vamos a calcular el rsq para cada modelo a partir de los fitted values y de los valores observados de la esperanza de vida. Como para cada país y continente obtendremos un único valor, usaremos la función `tidyr::unnest()` al final para poder visualizar directamente este valor (y que no quede anidado en una lista en la columna 'rsq' que vamos a crear):


```r
gap_nested <- gap_nested %>%
              mutate(rsq = map2(fitted, 
                                data, 
                                function(.fitted, .data) {yardstick::rsq_vec(as.vector(.fitted), .data$lifeExp)})) %>%
              tidyr::unnest(rsq)

gap_nested
```

```
## # A tibble: 142 x 6
##    country     continent data              arima    fitted      rsq
##    <fct>       <fct>     <list>            <list>   <list>    <dbl>
##  1 Afghanistan Asia      <tibble [12 x 4]> <fit[+]> <ts [12]> 0.988
##  2 Albania     Europe    <tibble [12 x 4]> <fit[+]> <ts [12]> 0.938
##  3 Algeria     Africa    <tibble [12 x 4]> <fit[+]> <ts [12]> 0.980
##  4 Angola      Africa    <tibble [12 x 4]> <fit[+]> <ts [12]> 0.981
##  5 Argentina   Americas  <tibble [12 x 4]> <fit[+]> <ts [12]> 0.995
##  6 Australia   Oceania   <tibble [12 x 4]> <fit[+]> <ts [12]> 0.984
##  7 Austria     Europe    <tibble [12 x 4]> <fit[+]> <ts [12]> 0.986
##  8 Bahrain     Asia      <tibble [12 x 4]> <fit[+]> <ts [12]> 0.982
##  9 Bangladesh  Asia      <tibble [12 x 4]> <fit[+]> <ts [12]> 0.994
## 10 Belgium     Europe    <tibble [12 x 4]> <fit[+]> <ts [12]> 0.986
## # ... with 132 more rows
```


 Como se puede ver, la combinación de `purrr` y `tidyr::nest` es muy poderosa para encontrar soluciones elegantes y eficientes. Esto es sólo es un esbozo de lo que se puede hacer con los funcionales del paquete, pero purrr ofrece muchas más funcionalidades que sin duda te recomiendo que explores. Desde luego, una vez que comprendes el funcionamiento de este paquete, dificilmente querrás abandonarlo en el futuro.        






