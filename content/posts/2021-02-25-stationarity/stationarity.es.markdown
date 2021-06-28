---
title: "Estacionariedad en Series de Tiempo"
author: "Alberto Almuiña"
date: '2021-02-25T21:13:14-05:00'
description: Una introducción a la estacionariedad en series de tiempo y su importancia.
slug: stationarity_es
tags: 
- estacionariedad
- dickey-fuller
- procesos estocásticos
categories: 
- Time Series
---

## Qué es un proceso estocástico?

Para comprender el concepto de estacionariedad, debemos introducir en primer lugar el concepto de proceso estocástico. Supongamos que todos los días voy a mi laboratorio a repetir exactamente el mismo experimento, en las mismas condiciones (misma temperatura, humedad, etc), en el que mido una función de una variable, por ejemplo la energía en función del tiempo. ¿Qué esperaría ver? Lo que la naturaleza me dice es que si bien los resultados son muy similares entre sí, no son exactamente iguales. Pero entonces, a cuál le hago caso? No valen para nada mis resultados? Precisamente la teoría encargada de darnos una respuesta y ayudarnos a extraer información útil de nuestro experimento serán los procesos estocásticos.

Por ejemplo, supongamos que cogemos la salida de dos de nuestros experimentos (por simplificar):


```r
library(zeallot)

set.seed(129)

Z <- stats::rnorm(255, 0, 1) 

c(u, u2, sd, sd2, s, a) %<-% c(0.3, 0.28, 0.2, 0.5, 100, 2)

t <- 1:256

energy <- c(s)        
energy2 <- c(s)

for(i in Z) {
  
    S = s + s*(u/255 + sd/sqrt(255)*i)
    S1 = s + s*(u2/255 + sd2/sqrt(255)*i)
    energy[a] <- S 
    energy2[a] <- S1
    s = S                                 
    a = a + 1
}

plot(t, energy, main="Energía - Tiempo", xlab="Tiempo", ylab="Energia (t)", type="l", col="blue")
lines(t, energy2)
```

<img src="/posts/2021-02-25-stationarity/stationarity.es_files/figure-html/unnamed-chunk-1-1.png" width="672" />

Si ahora ponemos nuestras dos realizaciones en un sistema de tres coordenadas:


```r
library(rgl)

rgl::plot3d(c(t,t), 
            c(energy, energy2), 
            c(rep(1, length(t)), rep(2, length(t))), 
            xlab = 't', 
            ylab = 'E(t)', 
            zlab = 'S', 
            col = 'red')
```

![](/img/stationarity.es_files/3d.JPG)

Entonces mi proceso estocástico vendrá dado por <div>$X(S,t)$, donde S representa el número de realizaciones y t representa el tiempo. Si ahora fijo el tiempo, para ese determinado <div>$t_{i}$ tendré dos valores de energia, uno por cada realización. Fijando el instante de tiempo lo que sucede es que aparece una variable aleatoria (con todas las propiedades de las variables aleatorias). Sin embargo, si fijo la realización lo que me queda es una función de una variable (del tiempo en este caso). Por lo tanto, podemos definir un proceso estocástico de forma equivalente como:

- Como un conjunto de realizaciones temporales y un índice aleatorio que selecciona una de ellas.

- Como un conjunto de variables aleatorias indexadas por el índice t.


## Qué es la estacionariedad?

En primer lugar, es importante resaltar que existen múltiples definiciones de estacionariedad, entre las que destacaremos las siguientes dos:

- *Estacionariedad Fuerte:* Un proceso estocástico es estacionario fuerte si se cumple que la probabilidad de que el proceso tome cierto valor en cada estado permanece constante con el transcurso del tiempo.

- *Estacionariedad Débil:* Esta es la definición típicamente utilizada en el análisis de series temporales. Un proceso estocástico se considera estacionariamente débil si cumple las tres siguientes propiedades:

    1. <div>$E[X_t] = \mu \ \ \forall t \in T$ , es decir, media constante.

    2. <div>$E[X_t^2] < \infty \ \ \forall t \in T$, es decir, que el segundo momento sea finito en cualquier tiempo, lo que garantiza que la varianza sea finita.

    3. <div>$Cov(X_{t1},X_{t2}) = Cov(X_{t1+h},X_{t2+h}) \ \ \forall t \in \mathbb{N}, \forall h \in \mathbb{Z}$

    La propiedad 3 hereda la homocedasticidad porque:

    <div>$Cov(X_t, X_t) = Cov(X_{t+h}, X_{t+h}) \leftrightarrow Var[X_t] = Var[X_{t+h}]\ \ \forall t \in \mathbb{N}, \forall h \in \mathbb{Z}$ 

    lo cual implica que la varianza de cada estado del proceso es la misma.

Para verlo de manera mucho más clara, veamos unos ejemplos:

![](/img/stationarity.es_files/media.JPG)

![](/img/stationarity.es_files/homocedasticidad.JPG)

![](/img/stationarity.es_files/autocovarianza.JPG)


### Por qué es importante saber si una serie de tiempo es estacionaria?

Principalmente, por una cuestión de predictibilidad. Las series estacionarias son mucho más fáciles de predecir. Debemos notar aquí el abuso del lenguaje, pues *la serie temporal no es estacionaria, si no el proceso estocástico que la genera*. (Aclarado esto, no haremos distinción de aquí en adelante). Pero entonces, como sé si mi serie temporal es estacionaria? Aquí podemos ayudarnos de tres métodos:

- Visuales, como los observados en las imagenes anteriores.

- Locales: Por ejemplo, midiendo la media/varianza en periodos distintos de tiempo y observando si esta cambia.

- Tests estadísticos: El test aumentado de Dickey-Fuller.

### Test de Dickey-Fuller

La hipótesis nula del test de Dickey-Fuller es que existe raíz unitaria (es decir, la serie no es estacionaria), mientras que la hipótesis alternativa es que no hay raíz unitaria y por lo tanto la serie es estacionaria. Para rechazar la hipótesis nula tendremos que obtener un estadístico t de la prueba más negativo que el valor crítico, como se muestra en la siguiente imagen:

![](/img/stationarity.es_files/dick_fuller.jpg)

Veamos un ejemplo de implementación en R:


```r
library(urca)
library(timetk)
library(magrittr, include.only = '%>%')

df <- timetk::walmart_sales_weekly

df %>% timetk::plot_time_series(.date_var = Date,
                                .value = Fuel_Price,
                                .smooth = F,
                                .interactive = F)
```

<img src="/posts/2021-02-25-stationarity/stationarity.es_files/figure-html/unnamed-chunk-3-1.png" width="672" />

```r
adf <-urca::ur.df(df$Fuel_Price, selectlags = 'AIC')

urca::summary(adf)
```

```
## 
## ############################################### 
## # Augmented Dickey-Fuller Test Unit Root Test # 
## ############################################### 
## 
## Test regression none 
## 
## 
## Call:
## lm(formula = z.diff ~ z.lag.1 - 1 + z.diff.lag)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.90849 -0.03126 -0.00127  0.03665  0.23581 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## z.lag.1    -0.0003160  0.0008588  -0.368    0.713    
## z.diff.lag  0.2773029  0.0304521   9.106   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.08814 on 997 degrees of freedom
## Multiple R-squared:  0.0768,	Adjusted R-squared:  0.07495 
## F-statistic: 41.47 on 2 and 997 DF,  p-value: < 2.2e-16
## 
## 
## Value of test-statistic is: -0.368 
## 
## Critical values for test statistics: 
##       1pct  5pct 10pct
## tau1 -2.58 -1.95 -1.62
```

Podemos observar como el estadístico es -0.368, por lo que no podemos rechazar la hipótesis nula y por lo tanto la serie no es estacionaria. Pero entonces, no puedo aplicar esos algoritmos que requieren que mi serie sea estacionaria? Afortunadamente, existen métodos para transformar la serie y hacer que esta sea estacionaria.

### Diferenciación

La diferenciación consiste en coger nuestra serie de tiempo y a cada valor restarle su valor anterior (notese que de esta forma, estamos acortando la serie temporal en una observación). La intuición por la que esta transformación hace una serie temporal estacionaria es la suposición de que tu serie sigue un comportamiento como el siguiente: <div>$y_{t} = \beta_{0} + \beta_{1}*t + \epsilon_{t}$.

El proceso de diferenciación consistiría en lo siguiente:

<div>$z_{t} = y_{t} - y_{t-1}$ 

sustituyendo en la ecuación anterior obtenemos:

<div>$(\beta_{0} + \beta_{1}*t + \epsilon_{t}) - (\beta_{0} + \beta_{1}*(t-1) + \epsilon_{t-1}) = \beta_{1} + (\epsilon_{t} - \epsilon_{t-1})$

Como vemos, la media del proceso será constante e igual a <div>$\beta_{1}$ puesto que los términos de epsilon comunmente siguen una distribución normal por lo que su media será cero. La varianza también será constante puesto que tendré la suma de dos variables aleatorias independientes entre sí (como el proceso que las origina es el mismo, entonces la varianza será igual a <div>$2k^{2}$)


```r
df <- df %>% timetk::tk_augment_differences(.value = Fuel_Price)

df %>% timetk::plot_time_series(.date_var = Date,
                                .value = Fuel_Price_lag1_diff1,
                                .smooth = F,
                                .interactive = F) 
```

<img src="/posts/2021-02-25-stationarity/stationarity.es_files/figure-html/unnamed-chunk-4-1.png" width="672" />

```r
adf <- urca::ur.df(diff(df$Fuel_Price), selectlags = 'AIC')

urca::summary(adf)
```

```
## 
## ############################################### 
## # Augmented Dickey-Fuller Test Unit Root Test # 
## ############################################### 
## 
## Test regression none 
## 
## 
## Call:
## lm(formula = z.diff ~ z.lag.1 - 1 + z.diff.lag)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.91057 -0.03206 -0.00014  0.03426  0.22538 
## 
## Coefficients:
##            Estimate Std. Error t value Pr(>|t|)    
## z.lag.1    -0.68369    0.03806  -17.96   <2e-16 ***
## z.diff.lag -0.05445    0.03165   -1.72   0.0857 .  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.08805 on 996 degrees of freedom
## Multiple R-squared:  0.3632,	Adjusted R-squared:  0.3619 
## F-statistic: 284.1 on 2 and 996 DF,  p-value: < 2.2e-16
## 
## 
## Value of test-statistic is: -17.962 
## 
## Critical values for test statistics: 
##       1pct  5pct 10pct
## tau1 -2.58 -1.95 -1.62
```

Como podemos ver, ahora sí podemos rechazar la hipótesis nula de que la serie temporal tiene una raíz unitaria y es no estacionaria! En próximos posts, veremos cómo modelar nuestras series, si no te lo quieres perder, suscríbete ;)
