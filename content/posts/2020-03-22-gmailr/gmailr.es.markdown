---
title: "Librería Gmailr"
author: "Alberto Almuiña"
date: '2020-03-22T21:13:14-05:00'
description: Librería para usar Gmail desde R
slug: gmailr_es
categories: 
- R-tips
tags: 
- gmailr
---

¡Hola!

¿Has oído hablar del paquete gmailr? Te permite acceder a la API RESTful de Gmail desde R para que puedas enviar o leer correos electrónicos.

Es muy útil en los procesos por lotes para notificarte si ha habido algún error o si todo ha terminado de la manera esperada.

También se puede usar dentro de aplicaciones Shiny para enviar información a los usuarios (como documentación o recomendaciones).

Puedes encontrar más información en el siguiente enlace: https://lnkd.in/gCwfNav

Aquí hay un pequeño ejemplo:


```r
library(gmailr)

options(gargle_oob_default = TRUE)
options(gargle_oauth_cache = "httr-oauth")
gmailr::gm_auth_configure(key = Sys.setenv("cli"), secret = Sys.setenv("secret"), appname = 'DSpotyApp')
gm_auth(email = "albertogonzalezalmuinha@gmail.com")

email<-gm_mime() %>%
    gm_from('albertogonzalezalmuinha@gmail.com') %>%
    gm_to('albertogonzalezalmuinha@gmail.com') %>%
    gm_subject('Hello World') %>%
    gm_text_body('Hello World')

gm_send_message(email)
```

