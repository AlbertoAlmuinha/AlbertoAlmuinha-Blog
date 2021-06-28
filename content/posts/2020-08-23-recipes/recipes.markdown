---
title: 'Tidymodels: the `recipes` package '
author: "Alberto Almuiña"
date: '2020-08-23T21:13:14-05:00'
description: Learn how to preprocess your data in a simple and consistent way.
slug: recipes
categories: 
- tidymodels
tags:
- preprocess
- recipes
---
<script src="/rmarkdown-libs/kePrint/kePrint.js"></script>
<link href="/rmarkdown-libs/lightable/lightable.css" rel="stylesheet" />
<script src="/rmarkdown-libs/kePrint/kePrint.js"></script>
<link href="/rmarkdown-libs/lightable/lightable.css" rel="stylesheet" />
<script src="/rmarkdown-libs/kePrint/kePrint.js"></script>
<link href="/rmarkdown-libs/lightable/lightable.css" rel="stylesheet" />

****
## What is tidymodels?
****

`tidymodels` is a new framework consisting of a series of packages that facilitate the modeling process in data science projects. It allows in a unified way to perform resampling, data preprocessing, unified model interface and results validation. The complete cycle would be as follows:

![](/img/recipes_files/cycle.jpg)


In this post we will focus on the step of data preprocessing with the recipes package.

****
## Introduction to `recipes`
****

This package is born from the effort of bringing together all the steps of data preparation before applying a model in a simple, efficient and consistent way. `recipes` is born from the analogy between preparing a kitchen recipe and preprocessing your data ... what is the similarity? Both follow a few steps before cooking (modeling).

Every recipe consists of four fundamental steps:

* `recipe()`: The formula is specified (predictor variables and response variables).

* `step_xxx()`: Define the steps such missing values imputation, dummy variables, centering, scaling and so on.

* `prep()`: Preparation of the recipe. This means that a dataset is used to analyze each step on it.

* `bake()`: Apply the preprocessing steps to your datasets.


In this post we will cover these four parts and we will see examples of good practices. I hope I can convince you that from now on, tidymodels in general and recipes in particular, are your reference ecosystem.

## A brief example: Airquality dataset


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

First, we are going to separate our dataset into a training dataset and a validation one. We will use 80% of our data to train and the remaining 20% to validate.


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


You can see how `rsample` throws us a split object, where we are told how many records are used in each dataset and the total one. With the `training()` and `testing()` methods we can extract the corresponding data.

**Creating a recipe object**

Now it's time to create our first recipe.


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

It can be seen how behind the scenes `recipes` assigns to each variable a type and a role. This will allow us to subsequently select which variables to apply a preprocessing step based on their typology or role. 
A very interesting option that allows recipes is to update the roles of the variables. For example, in this dataset we have two columns that have missing values: 'Ozone' and 'Solar.R'. We can assign these variables a specific role that will later allow us to identify them. We will create the new role 'NA_Variable' with the `udpate_role()` function:


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


**Selecting the preprocessing steps**

Once the recipe is created, it is the turn to add the necessary steps to carry out the preprocessing of the data. We have many steps to choose from:


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

Some of the most common are:

* **`step_XXXimpute():`** Methods to impute missing values such as meanimpute, medianimpute, knnimpute ...

* **`step_range():`** Normalize numeric data to be within a pre-defined range of values.

* **`step_center():`** Normalize numeric data to have a mean of zero.

* **`step_scale():`** Normalize numeric data to have a standard deviation of one.

* **`step_dummy():`** Convert nominal data (e.g. character or factors) into one or more numeric binary model terms for the levels of the original data.

* **`step_other():`** Step that will potentially pool infrequently occurring values into an "other" category.


The order in which the steps are executed is important, as you can read on the [official page of the package](https://tidymodels.github.io/recipes/articles/Ordering.html):

1. Impute
2. Individual transformations for skewness and other issues
3. Discretize (if needed and if you have no other choice)
4. Create dummy variables
5. Create interactions
6. Normalization steps (center, scale, range, etc)
7. Multivariate transformation (e.g. PCA, spatial sign, etc)


In addition, in each step we must specify which columns that step affects. There are several ways to do it, we will mention the most common:

1. Passing the variable name in the first argument

2. Selecting by the role of the variables with `all_predictors()` and `all_outcomes()` functions. As in our case we have changed the 'default' roles to 'NA_Variable', we can use the `has_role()` function to select them.

3. Selecting by the type of the variables with `all_nominal()` and `all_numerical()` functions.

4. Using dplyr selectors as `contains()`, `starts_with()` or `ends_with()` functions.


We are going to apply some of these steps to our example:


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

It can be seen how combining all the variable selection techniques we obtain great versatility. Also comment that when the minus sign is used it means that the columns that meet this condition are excluded from the preprocessing step. It can also be seen how the recipes object now specifies all the steps that will be carried out and on which variables.

**Preparing the recipe**

The time has come to prepare the recipe on a dataset. Once prepared, we can apply this recipe on multiple datasets:


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
## Training data contained 122 data points and 35 incomplete rows. 
## 
## Operations:
## 
## Mean Imputation for Ozone [trained]
## K-nearest neighbor imputation for Wind, Temp, Month, Day [trained]
## Centering for Wind, Temp, Month, Day [trained]
## Scaling for Wind, Temp, Month, Day [trained]
```

It is observed how the recipe is now 'trained'.

**Baking the recipe**


Now we can apply this recipe to another dataset, for example to the test data:


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
   <td style="text-align:right;"> 194 </td>
   <td style="text-align:right;"> -0.4241755 </td>
   <td style="text-align:right;"> -0.9107674 </td>
   <td style="text-align:right;"> -1.402669 </td>
   <td style="text-align:right;"> -0.6204946 </td>
   <td style="text-align:right;"> 42 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 221 </td>
   <td style="text-align:right;"> -0.9134636 </td>
   <td style="text-align:right;"> -0.3816549 </td>
   <td style="text-align:right;"> -1.402669 </td>
   <td style="text-align:right;"> -0.5075091 </td>
   <td style="text-align:right;"> 7 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 78 </td>
   <td style="text-align:right;"> 2.3964265 </td>
   <td style="text-align:right;"> -2.1806373 </td>
   <td style="text-align:right;"> -1.402669 </td>
   <td style="text-align:right;"> 0.2833901 </td>
   <td style="text-align:right;"> 6 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 44 </td>
   <td style="text-align:right;"> -0.1075773 </td>
   <td style="text-align:right;"> -1.6515248 </td>
   <td style="text-align:right;"> -1.402669 </td>
   <td style="text-align:right;"> 0.5093613 </td>
   <td style="text-align:right;"> 11 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 92 </td>
   <td style="text-align:right;"> 0.5544007 </td>
   <td style="text-align:right;"> -1.7573473 </td>
   <td style="text-align:right;"> -1.402669 </td>
   <td style="text-align:right;"> 0.9613036 </td>
   <td style="text-align:right;"> 32 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 252 </td>
   <td style="text-align:right;"> 1.3890686 </td>
   <td style="text-align:right;"> 0.3591026 </td>
   <td style="text-align:right;"> -1.402669 </td>
   <td style="text-align:right;"> 1.5262316 </td>
   <td style="text-align:right;"> 45 </td>
  </tr>
</tbody>
</table>


Finally we put it all together:


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
   <td style="text-align:right;"> 194 </td>
   <td style="text-align:right;"> -3.021482 </td>
   <td style="text-align:right;"> -8.308899 </td>
   <td style="text-align:right;"> -5.89308 </td>
   <td style="text-align:right;"> -1.820458 </td>
   <td style="text-align:right;"> 42 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 221 </td>
   <td style="text-align:right;"> -3.162308 </td>
   <td style="text-align:right;"> -8.252907 </td>
   <td style="text-align:right;"> -5.89308 </td>
   <td style="text-align:right;"> -1.807692 </td>
   <td style="text-align:right;"> 7 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 78 </td>
   <td style="text-align:right;"> -2.209666 </td>
   <td style="text-align:right;"> -8.443280 </td>
   <td style="text-align:right;"> -5.89308 </td>
   <td style="text-align:right;"> -1.718332 </td>
   <td style="text-align:right;"> 6 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 44 </td>
   <td style="text-align:right;"> -2.930360 </td>
   <td style="text-align:right;"> -8.387288 </td>
   <td style="text-align:right;"> -5.89308 </td>
   <td style="text-align:right;"> -1.692800 </td>
   <td style="text-align:right;"> 11 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 92 </td>
   <td style="text-align:right;"> -2.739832 </td>
   <td style="text-align:right;"> -8.398486 </td>
   <td style="text-align:right;"> -5.89308 </td>
   <td style="text-align:right;"> -1.641737 </td>
   <td style="text-align:right;"> 32 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 252 </td>
   <td style="text-align:right;"> -2.499601 </td>
   <td style="text-align:right;"> -8.174518 </td>
   <td style="text-align:right;"> -5.89308 </td>
   <td style="text-align:right;"> -1.577908 </td>
   <td style="text-align:right;"> 45 </td>
  </tr>
</tbody>
</table>


As you have seen, this package offers a wide range of possibilities and facilities for carrying out the preprocessing task. Many other interesting topics about this package have been left out of this post: creating your own preprocessing step or combining this package with rsamples to apply multiple recipes to partitions used by bootstrapping or cv-Folds techniques. Maybe for a next post.


