---
title: "Why you should learn purrr"
author: "Alberto Almuiña"
date: '2021-03-11T02:13:14-05:00'
description: An introduction to functional programming with `purrr`
slug: purrr
tags:
- purrr
- functional programming
categories: 
- tidyverse
---

## Why should you learn to use purrr?

According to the description of the package on its official page, "purrr improves R's set of functional programming (FP) tools by providing a complete and consistent set of tools for working with functions and vectors".

But then what is a functional programming language? As Hadley explains in [Advanced R](https://adv-r.hadley.nz/fp.html), there are many definitions that make a language functional, although we can find two characteristics common to all of them:

-   *First-class functions:* This means that the functions behave like any other structure. That is, you can store it in a variable, store them in lists, pass them as arguments to other functions, create them inside other functions, or even return them as a result of a function.

-   Many functional languages also require *pure functions*. A function is considered pure if it satisfies the following two characteristics:

    1.  The output only depends on the input arguments, so if you call the function with the same arguments, you will get the same result. Functions like `run.if ()` or `read.csv ()` are not pure.

    2.  The function has no side effects like changing the value of a global variable or writing to disk. Functions like `print ()` or `<-` are not pure.

Strictly speaking, R is not a functional language because it does not require you to write pure functions. However, in parts of your code you can adopt a functional style and in fact you should. But what is a functional style? A functional style consists of breaking down a large problem into smaller pieces and solving each of those pieces from a function or a set of functions. What is achieved is to decompose the problem into isolated functions that operate independently and that are easier to understand. There are three techniques to break the problem down into smaller pieces:

-   *Functionals* are functions that take a function as an argument that solves the problem for a single input and generalizes it to handle any number of inputs. An example would be the `lapply()` function.

-   Function factories: functions that create other functions.

-   Function operators: functions that take a function as an argument and return another function as output.

In this post we will focus on the functionals through the `purrr` package.

## Functionals with `purrr`

A functional is a function that takes a function as input and returns a vector as output (or at least that was the idea in the beginning, currently the output is modulable according to the type of functional. In this way, we can obtain functionalities that return us a list, others that return a vector or a data frame). The fundamental functional in purrr is `map()`, which receives as arguments a vector, a list or a data frame and a function and applies this function to each element of the vector, list or data frame and returns the result in a list. To have a better intuition, let's see an image:

![Source: Advanced R](/img/purrr_files/map.PNG)

What happens is the following:

-   If the input argument is a vector, `purrr::map()` will apply the function passed as an argument to each element of the vector.

-   If the input argument is a list, then map() will apply the function passed as an argument to each element of the list.

-   On the contrary, if the input argument is a data frame, then map() will apply the function to each column of it.

Let's look at a simple example:


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

We must bear in mind that `purrr::map` always returns a list, but sometimes this is not the most comfortable or practical. If we know in advance the output we want, we can use any of the map variants (there are more than those shown in the following example, I recommend that you explore the existing options by typing `map_`):


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

Another option is to use anonymous functions as a parameter, as follows:


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

Even if you don't feel like typing so much, purrr offers a shortcut so you can write your anonymous functions as follows:


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

To understand what is happening, we are going to make use of the `purrr::as_mapper()` function:


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

The function arguments seem a bit strange but they allow you to refer to `.` for one argument functions, to` .x .y` for two argument functions and to `..1, ..2, ..3 etc. `for a function with an arbitrary number of arguments. `.` remains for compatibility but is not recommended for use.

To use additional arguments of a function, there are two ways to do it: the first is through an anonymous function:


```r
purrr::map_dbl(list(1:5, c(1, 4, NA)), 
               ~ mean(.x, na.rm = TRUE))
```

```
## [1] 3.0 2.5
```

The second is to use the arguments after the function itself. This happens because the arguments are passed to the function through the argument `...`. As an image is worth a thousand words, let's better see it graphically:

![Source: Advanced R](/img/purrr_files/map_args.PNG)


```r
purrr::map_dbl(list(1:5, c(1, 4, NA)),
               mean, na.rm = TRUE)
```

```
## [1] 3.0 2.5
```

These are the foundations from which you can start using the purrr functionalities in a safe way. The move to functionals that iterate over multiple arguments as input rather than one (that is, the move to generalization) is trivial, as we will see below.

## map variants

`purrr` offers us a lot of versatility when it comes to iterating and for this it offers us another set of functions whose objective is to take a function as input and apply it to each pair of extracted elements, either from two vectors, or from two lists. The basic function of this set of functions is `purrr::map2()`, which, analogously to map(), will always return a list. To understand it better, let's go with a very simple example. Let's try to add two vectors element by element:


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

A bit cumbersome right? Yes! We are going to apply everything we learned in the previous sections. First, we can return a vector if we know the type we want to return, in this case 'double', simply by using the variants of the basic function with `map2_* desired-type*`:


```r
a <- c(1, 2, 3)
b <- c(1, 2, 3)

add <- function(x, y) {x+y}

purrr::map2_dbl(a, b, add)
```

```
## [1] 2 4 6
```

This is all? No, remember that we can use anonymous functions so as not to write so much code, when we use such short functions:


```r
purrr::map2_dbl(1:3, 1:3, ~{.x +.y})
```

```
## [1] 2 4 6
```

The next step in generalization would be the use of the `purrr::pmap()` function, although possibly by now you already intuit how this function works, right? The idea here is to pass a list as an argument that contains inside X vectors, lists or data frames on which the function passed as an argument will be applied. To do this, let's imagine that I want to add the elements of three vectors, the generalization of the previous example:


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

Now it is only a matter of simplifying applying the same logic as up to now, selecting the *pmap_* function that returns the result with the format we want and using an anonymous function to make it less cumbersome (remember that when there were more than two arguments, the correct syntax within the anonymous function was *.. 1, ..2, ..3, ..4 etc* to identify the corresponding arguments):


```r
a <- c(1, 2, 3)
b <- c(1, 2, 3)
c <- c(1 ,2, 3)

purrr::pmap_dbl(list(a, b, c), ~{..1 + ..2 + ..3})
```

```
## [1] 3 6 9
```

Another very interesting set of functions are `purrr::walk()` and its variants (walk2, pwalk). These functions are born because sometimes we are not interested in the function returning a result, but in the side effects that the function produces. This is the case for functions like print(), message(), plot() or write.csv(). When we use this type of function with `purrr::map()` we not only get the desired side effect, but we also get a list with null values. Let's look at the following example:


```r
saludar <- function(.nombre) {message(paste('Hi ', .nombre))}
nombres <- c('Alberto', 'Diana')

purrr::map(nombres, saludar)
```

```
## Hi  Alberto
```

```
## Hi  Diana
```

```
## [[1]]
## NULL
## 
## [[2]]
## NULL
```

Our interest is in printing the message, but we don't want to get any results back, and this is where `walk()` comes in. This function basically what it does internally is call the *map* function and return the results invisibly:

![Source: Advanced R](/img/purrr_files/walk.PNG)


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
## Hi  Alberto
```

```
## Hi  Diana
```

Another very common use case is when we want to save several files to disk in several paths. For this we will use the `purrr:walk2()` function, which will take as arguments the data frames to be saved, the routes and the function that will be in charge of carrying out this task:


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

Basically what is happening when we call walk2 is the following: write.csv(cyls [[1]], paths [[1]]), write.csv(cyls [[2]], paths [[2]] ), write.csv(cyls [[3]], paths [[3]]).

Another very interesting function is `purrr::modify()`, which works like map() but guarantees that the output of your function will have the same type as your input. This means that if your input is a vector, your output will be too. If your input is a data frame, your output will be too. Let's see an example:


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

## Practical Example: Star Wars Planets

To delve further into the `purrr` universe, let's see how we could use what we learned and other additional features from the package to explore a list containing information on 61 planets from the Star Wars saga. We will first see the information about the first planet to know what information we can find:


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

An advantage of purrr is that it is pipe-friendly, so we can use its functionals with the famous magritrr pipe. For example, let's see what are the names of the 61 planets in Star Wars:


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

As you can see, with purrr we can also extract elements from a list or object on which we will iterate in a simple way! Let's make things a little more complicated. Let's count the number of characters in each name:


```r
planets %>% purrr::map_chr(~.$name) %>% purrr::map_int(~str_length(.x))
```

```
##  [1]  8  8  4  7  6  5  5  9  6  8  6  8  8 11  7  7 14  9  7  6  8  5  9  9 10
## [26] 11  7  9  7  8  9  7  8  9  8  6 11  7  7  4 10  5 11  8  7  7  7  5  8  6
## [51]  7 12  5  4  5 10  5  5  6  8  5
```

And if now I want to keep those that contain more than 10 characters? Of course! purrr offers two functions, `keep` and` discard` that precisely allow us to select or discard elements based on a predicate:


```r
planets %>% purrr::map_chr(~.$name) %>% purrr::map_int(~str_length(.x)) %>% purrr::keep(~ .x > 8)
```

```
##  [1]  9 11 14  9  9  9 10 11  9  9  9 11 10 11 12 10
```

Although you can also directly filter the list and keep the elements that interest you. What does this mean? That I want to keep the planets on the list whose names are longer than 8 characters. Let's do it:


```r
lista_reducida <- planets %>% purrr::keep(~str_length(.$name)>8)

length(lista_reducida)
```

```
## [1] 16
```

Another interesting function is `pluck`, which allows you to flexibly extract elements from a list. In this case we simply extract the first element:


```r
lista_reducida %>% purrr::pluck(1) #same as lista_reducida[[1]]
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

Other very interesting functions are `partial()` and `compose()`. partial() allows you to modify a function to assign values to the parameters that interest you, while compose allows you to concatenate several functions so that they are executed in a certain order. Let's see an example of how to apply these functions. We are going to take the terrain field and we are going to replace the commas with hyphens and then we will put the characters in title format (the first letter of each word in uppercase):


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

## Practical Example: Nested Gapminder

Finally, we are going to see a practical example of how to combine a nested dataset with the `purrr` functionals to exploit all the power of both solutions. But first of all, what is a nested data frame? Basically, it is a data frame that has one or more columns made up of lists of data frames. To make the concept clear, let's look at an example:


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

As can be seen, we have a dataset (tibble) in which for each continent and country, in the 'data' column we have another dataset (tibble) stored. A good place to learn more about these types of datasets is on the [tidyr](https://tidyr.tidyverse.org/articles/nest.html) page itself  (since they are created with the tidyr::nest function). To extract the information in these tibbles in the data column, we can use the *pluck* function learned earlier. For example, let's look at the saved data for Afghanistan:


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

One of the great advantages of this format is that it allows us to apply the purrr functionalities within the data frame itself, for example, we can apply an ARIMA model with the data of each tibble in the 'data' column to model the life expectancy in function of the other variables: population and per capita GDP. What we do is save the information of each model in a new column called 'arima':


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

Let's see the ARIMA model selected for life expectancy in Afghanistan:


```r
gap_nested %>% pluck('arima', 1)
```

```
## parsnip model object
## 
## Fit time:  371ms 
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

Now we can extract the values from the model for the training period and store them in another column:


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

Finally, we are going to calculate the rsq for each model from the fitted values and the observed values of life expectancy. As for each country and continent we will obtain a single value, we will use the function `tidy::unnest()` at the end to be able to directly visualize this value (and that it is not nested in a list in the column 'rsq' that we are going to create) :


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


As you can see, the combination of `purrr` and` tidyr::nest` is very powerful to find elegant and efficient solutions. This is only a sketch of what can be done with the functionals of the package, but purrr offers many more functionalities that I definitely recommend you explore. Of course, once you understand how this package works, you will hardly want to abandon it in the future.        






