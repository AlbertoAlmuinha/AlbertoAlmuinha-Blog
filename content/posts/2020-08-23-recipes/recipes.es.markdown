---
title: 'Tidymodels: el paquete `recipes` '
author: "Alberto Almuiña"
date: '2020-08-23T21:13:14-05:00'
description: Aprenda a preprocesar sus datos de una manera simple y consistente.
slug: recipes_es
categories: 
- tidymodels
tags:
- preprocesamiento
- recipes
---
<script src="/rmarkdown-libs/kePrint/kePrint.js"></script>
<link href="/rmarkdown-libs/lightable/lightable.css" rel="stylesheet" />
<script src="/rmarkdown-libs/kePrint/kePrint.js"></script>
<link href="/rmarkdown-libs/lightable/lightable.css" rel="stylesheet" />
<script src="/rmarkdown-libs/kePrint/kePrint.js"></script>
<link href="/rmarkdown-libs/lightable/lightable.css" rel="stylesheet" />

****
## ¿Qué es tidymodels?
****

`tidymodels` es un nuevo ecosistema que consiste en una serie de paquetes que facilitan el proceso de modelado en proyectos de ciencia de datos. Permite de manera unificada realizar remuestreo, preprocesamiento de datos, aporta una interfaz de modelado unificado y validación de resultados. El ciclo completo sería el siguiente:

![](/img/recipes.es_files/cycle.jpg)


En esta publicación nos centraremos en el paso del preprocesamiento de datos con el paquete recipes.

****
## Introducción a `recipes`
****

Este paquete nace del esfuerzo de reunir todos los pasos de preparación de datos antes de aplicar un modelo de manera simple, eficiente y consistente. `recipes` nace de la analogía entre preparar una receta de cocina y preprocesar sus datos ... ¿cuál es la similitud? Ambos siguen algunos pasos antes de cocinar (modelar).

Cada receta consta de cuatro pasos fundamentales:

* `recipe()`: Se especifica la fórmula (variables predictoras y variables de respuesta).

* `step_xxx()`: Se definen los pasos a seguir: imputación de missing values, variables dummy, centrado, escalado, etc.

* `prep()`: Preparación de la receta. Esto significa que se utiliza un conjunto de datos para analizar cada paso en él.

* `bake()`: Aplique los pasos de preprocesamiento a sus conjuntos de datos.


En esta publicación cubriremos estas cuatro partes y veremos ejemplos de buenas prácticas. Espero poder convencerte de que en adelante, tidymodels en general y recipes en particular, son una buena opción.

## Un breve ejemplo: Airquality dataset


```r
library(rsample)
library(recipes)

data("airquality")

head(airquality) %>% 
  knitr::kable('html') %>% 
  kableExtra::kable_styling(position = 'left', 
                            font_size = 10, 
                            fixed_thead = list(enabled = T, background = "blue")
                            ) 
```

<table class="table" style="font-size: 10px; ">
 <thead>
  <tr>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Ozone </th>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Solar.R </th>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Wind </th>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Temp </th>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Month </th>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Day </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;"> 41 </td>
   <td style="text-align:right;"> 190 </td>
   <td style="text-align:right;"> 7.4 </td>
   <td style="text-align:right;"> 67 </td>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 36 </td>
   <td style="text-align:right;"> 118 </td>
   <td style="text-align:right;"> 8.0 </td>
   <td style="text-align:right;"> 72 </td>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 12 </td>
   <td style="text-align:right;"> 149 </td>
   <td style="text-align:right;"> 12.6 </td>
   <td style="text-align:right;"> 74 </td>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:right;"> 3 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 18 </td>
   <td style="text-align:right;"> 313 </td>
   <td style="text-align:right;"> 11.5 </td>
   <td style="text-align:right;"> 62 </td>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:right;"> 4 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> 14.3 </td>
   <td style="text-align:right;"> 56 </td>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:right;"> 5 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 28 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> 14.9 </td>
   <td style="text-align:right;"> 66 </td>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:right;"> 6 </td>
  </tr>
</tbody>
</table>

Primero, vamos a separar nuestro conjunto de datos en un conjunto de datos de entrenamiento y uno de test. Utilizaremos el 80% de nuestros datos para entrenar y el 20% restante para testear.


```r
(.df<-airquality %>% initial_split(prop = 0.8))
```

```
## <Analysis/Assess/Total>
## <122/31/153>
```

```r
.df_train<-.df %>% training()

.df_test<-.df %>% testing()
```


Se ver cómo `rsample` nos arroja un objeto dividido, donde se nos dice cuántos registros se utilizan en cada conjunto de datos y el total. A partir de las funciones `training ()` y `testing ()` podemos extraer los datos correspondientes.

**Creando un recipiente**

Ahora es el momento de crear nuestra primera receta.


```r
.recipe<-recipe(Ozone ~ ., data = .df_train)

summary(.recipe)
```

```
## # A tibble: 6 x 4
##   variable type    role      source  
##   <chr>    <chr>   <chr>     <chr>   
## 1 Solar.R  numeric predictor original
## 2 Wind     numeric predictor original
## 3 Temp     numeric predictor original
## 4 Month    numeric predictor original
## 5 Day      numeric predictor original
## 6 Ozone    numeric outcome   original
```

Se puede ver cómo 'recipes' asigna a cada variable un tipo y un rol. Esto nos permitirá seleccionar posteriormente a qué variables aplicar un paso de preprocesamiento en función de su tipología o rol.
Una opción muy interesante que permite recipes es actualizar los roles de las variables. Por ejemplo, en este conjunto de datos tenemos dos columnas que tienen valores faltantes: 'Ozone' y 'Solar.R'. Podemos asignar a estas variables un rol específico que luego nos permitirá identificarlas. Crearemos el nuevo rol 'NA_Variable' con la función `udpate_role()`:


```r
.recipe<-.recipe %>% update_role(Ozone, Solar.R, new_role = 'NA_Variable')

summary(.recipe)
```

```
## # A tibble: 6 x 4
##   variable type    role        source  
##   <chr>    <chr>   <chr>       <chr>   
## 1 Solar.R  numeric NA_Variable original
## 2 Wind     numeric predictor   original
## 3 Temp     numeric predictor   original
## 4 Month    numeric predictor   original
## 5 Day      numeric predictor   original
## 6 Ozone    numeric NA_Variable original
```


**Seleccionando los pasos de preprocesamiento**

Una vez que se crea la receta, es el turno de agregar los pasos necesarios para llevar a cabo el preprocesamiento de los datos. Tenemos muchos pasos para elegir:


```r
ls('package:recipes', pattern = '^step_')
```

```
##  [1] "step_arrange"       "step_bagimpute"     "step_bin2factor"   
##  [4] "step_BoxCox"        "step_bs"            "step_center"       
##  [7] "step_classdist"     "step_corr"          "step_count"        
## [10] "step_cut"           "step_date"          "step_depth"        
## [13] "step_discretize"    "step_downsample"    "step_dummy"        
## [16] "step_factor2string" "step_filter"        "step_geodist"      
## [19] "step_holiday"       "step_hyperbolic"    "step_ica"          
## [22] "step_impute_bag"    "step_impute_knn"    "step_impute_linear"
## [25] "step_impute_lower"  "step_impute_mean"   "step_impute_median"
## [28] "step_impute_mode"   "step_impute_roll"   "step_indicate_na"  
## [31] "step_integer"       "step_interact"      "step_intercept"    
## [34] "step_inverse"       "step_invlogit"      "step_isomap"       
## [37] "step_knnimpute"     "step_kpca"          "step_kpca_poly"    
## [40] "step_kpca_rbf"      "step_lag"           "step_lincomb"      
## [43] "step_log"           "step_logit"         "step_lowerimpute"  
## [46] "step_meanimpute"    "step_medianimpute"  "step_modeimpute"   
## [49] "step_mutate"        "step_mutate_at"     "step_naomit"       
## [52] "step_nnmf"          "step_normalize"     "step_novel"        
## [55] "step_ns"            "step_num2factor"    "step_nzv"          
## [58] "step_ordinalscore"  "step_other"         "step_pca"          
## [61] "step_pls"           "step_poly"          "step_profile"      
## [64] "step_range"         "step_ratio"         "step_regex"        
## [67] "step_relevel"       "step_relu"          "step_rename"       
## [70] "step_rename_at"     "step_rm"            "step_rollimpute"   
## [73] "step_sample"        "step_scale"         "step_select"       
## [76] "step_shuffle"       "step_slice"         "step_spatialsign"  
## [79] "step_sqrt"          "step_string2factor" "step_unknown"      
## [82] "step_unorder"       "step_upsample"      "step_window"       
## [85] "step_YeoJohnson"    "step_zv"
```

Algunos de los más comunes son los siguientes:

* **`step_XXXimpute():`** Métodos para imputar missing values como meanimpute, medianimpute, knnimpute ...

* **`step_range():`** Normaliza los datos numéricos para estar dentro de un rango de valores predefinido.

* **`step_center():`** Normaliza los datos numéricos para tener una media de cero.

* **`step_scale():`** Normaliza los datos numéricos para tener una desviación estándar de uno.

* **`step_dummy():`** Convierte datos nominales (por ejemplo, caracteres o factores) en uno o más términos numéricos de modelo binario para los niveles de los datos originales.

* **`step_other():`** Paso que potencialmente agrupará valores que ocurren con poca frecuencia en una categoría de "otros".

El orden en que se ejecutan los pasos es importante, como se puede leer en la [página oficial del paquete](https://tidymodels.github.io/recipes/articles/Ordering.html):

1. Imputar
2. Transformaciones individuales para la asimetría y otros problemas.
3. Discretice (si es necesario y si no tiene otra opción)
4. Crea variables dummy
5. Crea interacciones
6. Pasos de normalización (centrado, escalado, rango, etc.)
7. Transformación multivariada (por ejemplo, PCA)


Además, en cada paso debemos especificar a qué columnas afecta ese paso. Hay varias formas de hacerlo, mencionaremos las más comunes:

1. Pasando el nombre de la variable en el primer argumento

2. Selección por el papel de las variables con las funciones `all_predictors()` y `all_outcomes()`. Como en nuestro caso hemos cambiado los roles 'predeterminados' a 'NA_Variable', podemos usar la función `has_role()` para seleccionarlos.

3. Seleccionando por el tipo de las variables con las funciones `all_nominal()` y `all_numerical()`.

4. Usando los selectores dplyr como `contains()`, `starts_with()` o `ends_with()` funciones.


Vamos a aplicar algunos de estos pasos a nuestro ejemplo:


```r
.recipe<-.recipe %>% 
          step_meanimpute(has_role('NA_Variable'), -Solar.R) %>%
          step_knnimpute(contains('.R'), neighbors = 3) %>%
          step_center(all_numeric(), -has_role('NA_Variable')) %>%
          step_scale(all_numeric(), -has_role('NA_Variable'))
```

```
## Warning: `step_knnimpute()` was deprecated in recipes 0.1.16.
## Please use `step_impute_knn()` instead.
```

```
## Warning: `step_meanimpute()` was deprecated in recipes 0.1.16.
## Please use `step_impute_mean()` instead.
```

```r
.recipe
```

```
## Data Recipe
## 
## Inputs:
## 
##         role #variables
##  NA_Variable          2
##    predictor          4
## 
## Operations:
## 
## Mean Imputation for has_role("NA_Variable"), -Solar.R
## K-nearest neighbor imputation for contains(".R")
## Centering for all_numeric(), -has_role("NA_Variable")
## Scaling for all_numeric(), -has_role("NA_Variable")
```

Se puede ver cómo combinando todas las técnicas de selección de variables obtenemos una gran versatilidad. También comentar que cuando se usa el signo menos significa que las columnas que cumplen con esta condición están excluidas del paso de preprocesamiento. También se puede ver cómo el objeto de recetas ahora especifica todos los pasos que se llevarán a cabo y en qué variables.

**Preparando la receta**

Ha llegado el momento de preparar la receta en un conjunto de datos. Una vez preparado, podemos aplicar esta receta en múltiples conjuntos de datos:


```r
(.recipe<-.recipe %>% prep(.df_train))
```

```
## Data Recipe
## 
## Inputs:
## 
##         role #variables
##  NA_Variable          2
##    predictor          4
## 
## Training data contained 122 data points and 36 incomplete rows. 
## 
## Operations:
## 
## Mean Imputation for Ozone [trained]
## K-nearest neighbor imputation for Wind, Temp, Month, Day [trained]
## Centering for Wind, Temp, Month, Day [trained]
## Scaling for Wind, Temp, Month, Day [trained]
```

Se observa cómo la receta ahora está 'entrenada'.

**Hornear la receta**

Ahora podemos aplicar esta receta a otro conjunto de datos, por ejemplo a los datos de prueba:


```r
.df_test<-.recipe %>% bake(.df_test)

head(.df_test) %>% 
  knitr::kable('html') %>% 
  kableExtra::kable_styling(position = 'left', 
                            font_size = 10, 
                            fixed_thead = list(enabled = T, background = "blue")
                            ) 
```

<table class="table" style="font-size: 10px; ">
 <thead>
  <tr>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Solar.R </th>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Wind </th>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Temp </th>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Month </th>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Day </th>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Ozone </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;"> 334 </td>
   <td style="text-align:right;"> 0.3277434 </td>
   <td style="text-align:right;"> -1.4315911 </td>
   <td style="text-align:right;"> -1.3028525 </td>
   <td style="text-align:right;"> 0.0018752 </td>
   <td style="text-align:right;"> 14 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 307 </td>
   <td style="text-align:right;"> 0.4670629 </td>
   <td style="text-align:right;"> -1.2120385 </td>
   <td style="text-align:right;"> -1.3028525 </td>
   <td style="text-align:right;"> 0.1162603 </td>
   <td style="text-align:right;"> 34 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 42 </td>
   <td style="text-align:right;"> -0.6474931 </td>
   <td style="text-align:right;"> -2.2000253 </td>
   <td style="text-align:right;"> -1.3028525 </td>
   <td style="text-align:right;"> 1.2601115 </td>
   <td style="text-align:right;"> 38 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 286 </td>
   <td style="text-align:right;"> -0.4803097 </td>
   <td style="text-align:right;"> 0.1052773 </td>
   <td style="text-align:right;"> -0.6206722 </td>
   <td style="text-align:right;"> -1.7139017 </td>
   <td style="text-align:right;"> 38 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 120 </td>
   <td style="text-align:right;"> 0.3277434 </td>
   <td style="text-align:right;"> -0.4436043 </td>
   <td style="text-align:right;"> -0.6206722 </td>
   <td style="text-align:right;"> 0.3450305 </td>
   <td style="text-align:right;"> 12 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 137 </td>
   <td style="text-align:right;"> -0.0066234 </td>
   <td style="text-align:right;"> -0.1142753 </td>
   <td style="text-align:right;"> -0.6206722 </td>
   <td style="text-align:right;"> 0.4594157 </td>
   <td style="text-align:right;"> 13 </td>
  </tr>
</tbody>
</table>


Finalmente lo ponemos todo junto:


```r
.df<-recipe(Ozone ~ ., data = .df_train) %>%
   
       update_role(Ozone, Solar.R, new_role = 'NA_Variable') %>%
   
       step_meanimpute(has_role('NA_Variable'), -Solar.R) %>%
       step_knnimpute(contains('.R'), neighbors = 3) %>%
       step_center(all_numeric(), -has_role('NA_Variable')) %>%
       step_scale(all_numeric(), -has_role('NA_Variable')) %>%
   
       prep(.df_train) %>%
       bake(.df_test)
```

```
## Warning: `step_knnimpute()` was deprecated in recipes 0.1.16.
## Please use `step_impute_knn()` instead.
```

```
## Warning: `step_meanimpute()` was deprecated in recipes 0.1.16.
## Please use `step_impute_mean()` instead.
```

```r
head(.df) %>% 
  knitr::kable('html') %>% 
  kableExtra::kable_styling(position = 'left', 
                            font_size = 10, 
                            fixed_thead = list(enabled = T, background = "blue")
                            ) 
```

<table class="table" style="font-size: 10px; ">
 <thead>
  <tr>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Solar.R </th>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Wind </th>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Temp </th>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Month </th>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Day </th>
   <th style="text-align:right;position: sticky; top:0; background-color: blue;"> Ozone </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;"> 334 </td>
   <td style="text-align:right;"> -2.785283 </td>
   <td style="text-align:right;"> -8.614430 </td>
   <td style="text-align:right;"> -5.602534 </td>
   <td style="text-align:right;"> -1.828072 </td>
   <td style="text-align:right;"> 14 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 307 </td>
   <td style="text-align:right;"> -2.746463 </td>
   <td style="text-align:right;"> -8.590328 </td>
   <td style="text-align:right;"> -5.602534 </td>
   <td style="text-align:right;"> -1.814988 </td>
   <td style="text-align:right;"> 34 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 42 </td>
   <td style="text-align:right;"> -3.057022 </td>
   <td style="text-align:right;"> -8.698786 </td>
   <td style="text-align:right;"> -5.602534 </td>
   <td style="text-align:right;"> -1.684149 </td>
   <td style="text-align:right;"> 38 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 286 </td>
   <td style="text-align:right;"> -3.010438 </td>
   <td style="text-align:right;"> -8.445718 </td>
   <td style="text-align:right;"> -5.137164 </td>
   <td style="text-align:right;"> -2.024332 </td>
   <td style="text-align:right;"> 38 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 120 </td>
   <td style="text-align:right;"> -2.785283 </td>
   <td style="text-align:right;"> -8.505972 </td>
   <td style="text-align:right;"> -5.137164 </td>
   <td style="text-align:right;"> -1.788820 </td>
   <td style="text-align:right;"> 12 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 137 </td>
   <td style="text-align:right;"> -2.878451 </td>
   <td style="text-align:right;"> -8.469820 </td>
   <td style="text-align:right;"> -5.137164 </td>
   <td style="text-align:right;"> -1.775737 </td>
   <td style="text-align:right;"> 13 </td>
  </tr>
</tbody>
</table>


Como se ha visto, este paquete ofrece una amplia gama de posibilidades y facilidades para llevar a cabo la tarea de preprocesamiento. Muchos otros temas interesantes sobre este paquete se han dejado de lado en esta publicación: crear su propio paso de preprocesamiento o combinar este paquete con rsamples para aplicar múltiples recetas a particiones utilizadas por técnicas de bootstrapping o cv-Folds. Tal vez para una próxima publicación.


