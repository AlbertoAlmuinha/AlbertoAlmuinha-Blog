---
title: "Gmailr R package"
author: "Alberto Almuiña"
date: '2020-03-22T21:13:14-05:00'
description: Library to use Gmail from R
slug: gmailr
categories: 
- R-tips
tags: 
- gmailr
---

Hi!

Have you heard about the gmailr package? It allows you to access the Gmail RESTful API from R so you can send or read emails.

It is very useful in batch processes to notify you if there has been an error or if everything has finished ok.

You can also use it inside your Shiny apps to send some information to your users (like documentation or recommendations).

You can find more information in the following link: https://lnkd.in/gCwfNav

Here is a little example:


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

