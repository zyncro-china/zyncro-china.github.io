---
title: Deploy R application to shinyapps.io
date: 2014-03-18 01:53 UTC
tags: R, statics, web
---

## An R hosting Service

Today I got an invitation for ShinyApps.io testing. This seems to be very
useful because other PaaS like Heroku not allowing install R.

## Installation

Just follow the guide from
https://github.com/rstudio/shinyapps/blob/master/guide/guide.mdhttps://github.com/rstudio/shinyapps/blob/master/guide/guide.md


## run the App

```R

library(shiny)
runApp()
```
## Trouble Shooting

When the first time I deploy my apps, it always returns: 

```shell

> deployApp('.')
Preparing to deploy application...Error: /v1/applications/ 400 - Bad Request
```

Finally I found that it is because my project directory is used as the app
name, so change it to something else make it works.

## Multiple pages solution

Shiny App only support Single Page Application workflow, but there is some
discussion for multiple pages:
https://groups.google.com/forum/#!searchin/shiny-discuss/multiple/shiny-discuss/GAiZ4m13EHY/yKpkqmv9QuMJ
