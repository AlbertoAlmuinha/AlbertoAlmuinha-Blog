---
title: "Time Series Stationarity"
author: "Alberto Almuiña"
date: '2021-02-25T21:13:14-05:00'
description: An introduction to stationarity in time series and its importance.
slug: stationarity
tags:
- stationarity
- dickey-fuller
- stochastic process
categories: 
- Time Series
---

## What is a stochastic process?

To understand the concept of stationarity, we must first introduce the concept of stochastic process. Suppose that every day I go to my laboratory to repeat exactly the same experiment, under the same conditions (same temperature, humidity, etc.), in which I measure a function of a variable, for example energy as a function of time. What would you expect to see? What nature tells me is that while the results are very similar to each other, they are not exactly the same. But then, which one do I listen to? Are my results worth nothing? Precisely the theory in charge of giving us an answer and helping us to extract useful information from our experiment will be the stochastic processes.


For example, suppose we take the output of two of our experiments (for simplicity):


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

<img src="/posts/2021-02-25-stationarity/stationarity_files/figure-html/unnamed-chunk-1-1.png" width="672" />

If we now put our two realizations in a three coordinate system:


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

![](/img/stationarity_files/3d.JPG)

So my stochastic process will be given by <div>$X(S,t)$, where S represents the number of realizations and t represents time. If now I set the time, for that certain <div>$t_{i}$ I will have two energy values, one for each realization. Setting the instant of time, what happens is that a random variable appears (with all the properties of random variables). However, if I set the realization what I am left with is a function of a variable (of time in this case). Therefore, we can define a stochastic process equivalently as:

- As a set of temporary realizations and a random index that selects one of them.

- As a set of random variables indexed by the index t.


## What is stationarity?

In the first place, it is important to highlight that there are multiple definitions of stationarity, among which we will highlight the following two:

- *Strong stationarity:* A stochastic process is strong stationary if it is true that the probability that the process takes on a certain value in each state remains constant over time.

- *Weak stationarity:* This is the definition typically used in time series analysis. A stochastic process is considered weak stationary if it meets the following three properties:

    1. <div>$E[X_t] = \mu \ \ \forall t \in T$ , that is, constant mean.

    2. <div>$E[X_t^2] < \infty \ \ \forall t \in T$, that is, that the second moment is finite at any time, which guarantees that the variance is finite.

    3. <div>$Cov(X_{t1},X_{t2}) = Cov(X_{t1+h},X_{t2+h}) \ \ \forall t \in \mathbb{N}, \forall h \in \mathbb{Z}$

    Property 3 inherits homoscedasticity because:

    <div>$Cov(X_t, X_t) = Cov(X_{t+h}, X_{t+h}) \leftrightarrow Var[X_t] = Var[X_{t+h}]\ \ \forall t \in \mathbb{N}, \forall h \in \mathbb{Z}$ 

    which implies that the variance of each state of the process is the same.

To see it much more clearly, let's look at some examples:

![](/img/stationarity_files/media.JPG)

![](/img/stationarity_files/homocedasticidad.JPG)

![](/img/stationarity_files/autocovarianza.JPG)


### Why is it important to know if a time series is stationary?

Mainly, for a question of predictability. Stationary series are much easier to predict. We should note here the abuse of language, *since the time series is not stationary, if not the stochastic process that generates it*. (This being clarified, we will not make a distinction from here on). But then how do I know if my time series is stationary? Here we can help ourselves in three ways:

- Visual Methods, like those seen in the previous images.

- Local: For example, measuring the mean / variance in different periods of time and observing if it changes.

- Statistical tests: The Augmented Dickey-Fuller test.

### Dickey-Fuller Test

The null hypothesis of the Dickey-Fuller test is that there is a unit root (that is, the series is not stationary), while the alternative hypothesis is that there is no unit root and therefore the series is stationary. To reject the null hypothesis we will have to obtain a t-test statistic that is more negative than the critical value, as shown in the following image:

![](/img/stationarity_files/dick_fuller.jpg)

Let's see an example of implementation in R:


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

<img src="/posts/2021-02-25-stationarity/stationarity_files/figure-html/unnamed-chunk-3-1.png" width="672" />

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

We can observe how the statistic is -0.368, so we cannot reject the null hypothesis and therefore the series is not stationary. But then I can't apply those algorithms that require my series to be stationary? Fortunately, there are methods to transform the series and make it stationary.

### Differentiation

The differentiation consists of taking our time series and subtracting each value from its previous value (note that in this way, we are shortening the time series in an observation). The intuition by which this transformation makes a stationary time series is the assumption that your series follows a behavior like the following: <div>$y_{t} = \beta_{0} + \beta_{1}*t + \epsilon_{t}$.

The differentiation process would consist of the following:

<div>$z_{t} = y_{t} - y_{t-1}$ 

substituting in the previous equation we obtain:

<div>$(\beta_{0} + \beta_{1}*t + \epsilon_{t}) - (\beta_{0} + \beta_{1}*(t-1) + \epsilon_{t-1}) = \beta_{1} + (\epsilon_{t} - \epsilon_{t-1})$

As we can see, the mean of the process will be constant and equal to <div>$\beta_{1}$ since the epsilon terms commonly follow a normal distribution so their mean will be zero. The variance will also be constant since I will have the sum of two independent random variables (since the process that originates them is the same, then the variance will be equal to <div>$2k^{2}$)


```r
df <- df %>% timetk::tk_augment_differences(.value = Fuel_Price)

df %>% timetk::plot_time_series(.date_var = Date,
                                .value = Fuel_Price_lag1_diff1,
                                .smooth = F,
                                .interactive = F) 
```

<img src="/posts/2021-02-25-stationarity/stationarity_files/figure-html/unnamed-chunk-4-1.png" width="672" />

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

As we can see, now we can reject the null hypothesis that the time series has a unit root and is non-stationary! In future posts, we will see how to model our series, if you don't want to miss it, follow me on [LinkedIn](https://www.linkedin.com/in/alberto-gonz%C3%A1lez-almui%C3%B1a-b1176881/).
