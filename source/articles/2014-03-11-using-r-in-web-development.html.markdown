---
title: Using R in web development
date: 2014-03-11 05:47 UTC
tags: R,web,bootstrap
---


## Statistic Tool: R

I have been want to use R in my work for a long time. But seem it is not so
easy to combine it into the normal work flow very easily.

## Now we have a web development using R and Bootstrap: Shiny

Shiny seems would be very useful for rapid development for some web project.

I can pass all the statistic job to R, which is more suitable for it, then
I can develop other business logic in javascript. And maybe I can learn some
statistic knowledges by using R. That is great!

## Shiny setup

Shiny is a package in R. So to install shiny:

```shell

install.packages("shiny")
```

Then in a directory, prepare 2 R files:

1. ui.R
1. server.R

### Bootstrap2.3.2 components and other libs
According to the source, shiny integrate following javascript libs:

```

▾ inst/
  ▾ www/
    ▾ shared/
      ▸ bootstrap/
      ▸ datatables/
      ▸ datepicker/
      ▸ font-awesome/
      ▸ highlight/
      ▸ jqueryui/
      ▸ selectize/
      ▸ showdown/
      ▸ slider/
        jquery.js
```

Shiny use Bootstrap2.3.2. And it is very easy to use the components available.

In ui.R:

```R

shinyUI(pageWithSidebar(
  headerPanel("Minimal Example"),
  sidebarPanel(
               textInput(inputId = 'comment',
                         label = "Say",
                         value = "")),
  mainPanel(
            h3("This is you saying it"),
            textOutput('textDisplay')
            )

))
```

Notice that we use headerPanel, sidebarPanel and mainPanel, they are defined in R/bootstrap.R . And there are more UI componts defined in that file.

In server.R, we will update the output:

```R

library(shiny)
shinyServer(function(input, output) {
  output$textDisplay <-renderText({
    paste0("You said ", input$comment, " there are ", nchar(input$comment), " characters in this.")
  })
})
```
It does 2 things:

1. gives the output name: testDisplay
1. it tells shiny that the content contained whthin is reactive
1. paste0 links strings without spaces

### run gist

```

> runGist(1233456)
```

### show all widgets

```

> runGist(6571951)
```

### more about reactivity

To understand the reactive programming used in Shiny, I need to read the [Shiny
tutorial page](http://rstudio.github.io/shiny/tutorial/#reactivity-overview)

There are 3 kinds of objects in reactive programming:

1. Reactive source
1. Reactive conductor
1. Reactive endpoint

In the simple case, we only need Reactive source and the Reactive endpoint. But
in the case that we don't want to calculate the same middle result several
times, we can use Rective conductor. The following is an example for fib():

```R

fib <- function(n) ifelse(n<3, 1, fib(n-1)+fib(n-2))

shinyServer(function(input, output) {
  currentFib         <- reactive({ fib(as.numeric(input$n)) })

  output$nthValue    <- renderText({ currentFib() })
  output$nthValueInv <- renderText({ 1 / currentFib() })
})

```

### More notes for reactive object
We can't access the reactive value or expressions from outside a reactive
context.

The following code can't work:

```R

shinyServer(function(input, output) {
  # Will give error
  currentFib      <- fib(as.numeric(input$n))
  output$nthValue <- renderText({ currentFib })
})
```

But we can define a function, then call it in the reactive world:

```R

shinyServer(function(input, output) {
  # OK, as long as this is called from the reactive world:
  currentFib <- function() {
    fib(as.numeric(input$n))
  }

  output$nthValue <- renderText({ currentFib() })
})
```

### Reactive object implementation

1. Reactive values are an implementation of Reactive sources; that is, 
 they are an implementation of that role.
1. Reactive expressions are an implementation of Reactive conductors. They can
 access reactive values or other reactive expressions, and they return a value.

    A reactive expressions can be useful for caching the results of any procedure that happens in response to user input, including:
    
    1. accessing a database
    1. reading data from a file
    1. downloading data over the network
    1. performing an expensive computation
1. Observers are an implementation of Reactive endpoints. They can access
 reactive sources and reactive expressions, and they don’t return a value;
 they are used for their side effects.
## manipulate CSV

### import CSV into data frame

```R

data <- read.table("simulation1.csv", header=TRUE, sep=",")
```

### query

```R
league_table$team[league_table$home_wins > 8]
```
### merge

```R

> league <- merge(league_table, points, by='team')
```

### sorting

```R

> with(league, league[order(-pts),])
```

### insert more rows

```R

rbind(with(league, league[order(-pts),]), another_table)
```

### format date string

```R

alldates <- format(as.Date(sent_data$date), '%Y-%m')
```
## user login

We should be able to use nginx or apach to do the http basic authentication,
then foward the request to local Shiny server
