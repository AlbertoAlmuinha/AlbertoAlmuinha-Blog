---
title: '`crayon` R package'
author: "Alberto Almuiña"
date: '2020-05-04T21:13:14-05:00'
description: Add color to terminal output, create styles for notes, warnings or errors.
slug: crayon
categories: 
- R-tips
tags: 
- crayon
---

****
## `crayon` package
****

How many times were you creating a function that printed certain notes, warnings or errors on the screen? Did you have the feeling that your messages were not pretty enough? Now, thanks to the crayon package you can improve the format in which you print your messages, adding color and style. Let's see some example:


```r
library(crayon)

cat(red('This is a red message'))

cat(yellow('\nThis is a yellow message'))

cat(bold('\nThis is a bold message'))

cat(underline('\nThis is an underline message'))

cat(bgGreen('\nThis is a background message'))
```

![](/img/crayon_files/crayon1.PNG)

This package offers the next options:

**Styles**

* reset
* bold
* blurred
* italic
* underline
* inverse
* hidden

**Text Colors**

* black
* red
* green
* yellow
* blue
* magenta
* cyan
* white
* silver

**Background Colors**

* bgBlack
* bgRed
* bgGreen
* bgYellow
* bgBlue
* bgMagenta
* bgCyan
* bgWhite


You can combine the options using the '$' operator:


```r
cat(bgBlack$white$bold('Black background with white message'))

cat(bgYellow$silver$underline('\nYellor background with silver message'))
```

![](/img/crayon_files/crayon2.png)

Styles can also be nested, and then inner style takes precedence. Try it using the '%+%' operator:


```r
cat(red('This is a red line' %+% blue(' that becames blue ') %+% 'and red again'))
```

![](/img/crayon_files/crayon3.png)

Finally, you can also define your own styles:


```r
error<- red$bold
warning<-yellow$italic
note<-cyan

cat(error('This is an error'))
cat(warning('\nThis is a warning'))
cat(note('\nThis is a note'))
```

![](/img/crayon_files/crayon4.png)

