---
title: 'rstudioapi:: jobRunScript()'
author: "Alberto Almuiña"
date: '2020-02-01T21:13:14-05:00'
description: Inicie procesos en otra instancia de Rstudio mientras continúa trabajando
  en su instancia actual.
slug: jobrunscript_es
categories: 
- R-tips
tags: 
- jobrunscript
---

****
## Función `rstudioapi::jobRunScript()` 
****

¿Cuántas veces ha ejecutado su código en R y ha tenido que esperar sin hacer nada mientras termina? ¿Pierde horas mientras entrena a su modelo? Ahora esto es cosa del pasado, gracias a los 'jobs' de Rstudio. Esta funcionalidad le permite iniciar varias instancias de Rstudio donde puede iniciar sus procesos, lo que le permite continuar utilizando su instancia principal sin perder productividad.
Además, Rstudio ha lanzado un paquete llamado rstudioapi que le permite conectarse a Rstudio y administrarlo. Esto le permite comunicar sus scripts con la terminal o con la funcionalidad de los 'jobs', entre muchas otras cosas.

Además de las mejoras discutidas anteriormente, creemos que hay un caso de uso particular que puede ser realmente interesante. Imagine que está lanzando una aplicación o gadget de Shiny (o cualquier otra aplicación que congela su instancia) y desea utilizar otra aplicación al mismo tiempo o mejor aún, desea llamar a otra aplicación desde dentro de la otra.

Hasta hace poco, no era posible lograr esto (dos comandos 'runApp' no se pueden iniciar). Afortunadamente, gracias a la función `jobRunScript()` puedes llamar a otra aplicación de Shiny desde un script. Esta función también le permite importar el entorno global al entorno de su aplicación para que pueda utilizar variables almacenadas fuera de la aplicación.

Veamos un ejemplo:


```r
library(rstudioapi)

data("airquality")

a<-rstudioapi::jobRunScript('script.R',
                            name = 'example_job',
                            importEnv = TRUE)
```

Una vez que se inicia la función, devuelve un ID (almacenado en la variable 'a' en el ejemplo) que le permite interactuar más tarde con ese 'job'. De esta manera, puede modificar el estado de su 'job' o eliminarlo, entre otras cosas.


```r
rstudioapi::jobRemove(a)
#or
rstudioapi::jobSetState(a, state = 'succeeded')
```


En las siguientes publicaciones, también hablaremos sobre cómo usar la línea de comandos con este paquete RStudio.

¡Espero que este consejo te ayude a mejorar tu productividad y tus aplicaciones!
