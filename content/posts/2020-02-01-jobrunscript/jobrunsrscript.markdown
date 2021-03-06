---
title: 'rstudioapi:: jobRunScript()'
author: "Alberto Almuiña"
date: '2020-02-01T21:13:14-05:00'
description: Launch processes in another instance of Rstudio while you continue working
  on your current instance.
slug: jobrunscript
categories: 
- R-tips
tags: 
- jobrunscript
---

****
## `rstudioapi::jobRunScript()` function
****

How many times have you run your code in R and have had to wait idly while it ends? Wasted hours while training your model? Now that is a thing of the past, thanks Rstudio's jobs. 
This functionality allows you to start several instances of Rstudio where you can launch your processes, allowing you to continue using your main instance without losing productivity. 
Also, Rstudio has released a package called rstudioapi that allows you to connect to Rstudio and manage it. This allows you to communicate your scripts with the terminal or with the functionality of the jobs, among many other things.

In addition to the improvements discussed above, we believe that there is a particular use case that can be really interesting. Imagine that you are launching a shiny app or gadget (or any other application that freezes your instance) and want to use another app at the same time or better yet, you want to call another app from within the other.


Until not long ago, it was not possible to achieve this (two 'runApp' commands cannot be launched). Fortunately, thanks to the `jobRunScript()` function you can call another shiny application from a script. This function also allows you to import the global environment to the environment of your app so that in this way you can use variables stored outside the app.

Let's see an example:


```r
library(rstudioapi)

data("airquality")

a<-rstudioapi::jobRunScript('script.R',
                            name = 'example_job',
                            importEnv = TRUE)
```

Once the function is launched, it returns an ID (stored in variable 'a' in example) that allows you to interact later with that job. In this way, you could modify the status of your job or delete it, among other things.


```r
rstudioapi::jobRemove(a)
#or
rstudioapi::jobSetState(a, state = 'succeeded')
```


In the following posts, we will also talk about how to use the command line with this RStudio package.

I hope this tip helps you improve your productivity and your applications!
