---
title: "Shiny Workshop - RSE conference 2018"
output: github_document
---




## Introduction

Shiny lets us build interactive web apps, using R.   This means we can use (pretty much) all of R's extensive (and extensible) data analysis and visualisation features in our app.  Essentially, we can take almost any analysis we've done in R, and then make it interactive.   We can run our apps locally, within R Studio (this is what we'll do most of today), make them standalone, either by deploying them to a [Shiny Server](https://www.rstudio.com/products/shiny/shiny-server/), or to a hosting service, such as https://shinyapps.io (we'll do this today), or even including them in a Markdown document.

R studio provide a gallery of other Shiny apps: [https://shiny.rstudio.com/gallery/]

Shiny works well with many widely used R packages, such as [ggplot2](https://ggplot2.tidyverse.org/), and [Leaflet for R](https://rstudio.github.io/leaflet/).  

In this workshop, we're going to use data from the [Gapminder project](https://www.gapminder.org) to visualise how GDP per capita and life expectancy have changed over time.   This takes one of Hans Rosling's 
[innovative visualisations](https://youtu.be/jbkSRLYSojo?t=90) and puts it into Shiny. The app we'll produce can be found here: [https://mawds.shinyapps.io/worked_example/]



> ## Aside
>
> I've tried to minimise the amount of non-Shiny material in this workshop, but we will need to so some manipulation of the data. I've adopted a [tidyverse](https://www.tidyverse.org/) based approach to doing this, using pipes and filters.  If you're more comfortable using base R you're very welcome to use that approach.   If you need a quick refresher on using the tidyverse to manipulate tabular data, [this episode](https://uomresearchit.github.io/r-tidyverse-intro/04-dplyr/) of the [Data Analysis using R course](https://uomresearchit.github.io/r-tidyverse-intro/), which we teach at the University of Manchester may be useful.
> 
> To minimise the amount of formatting of graphs etc, I've provided functions that will generate the graphs we'll be using, given an appropriate data set.  Where these functions are used you could use your own call to `ggplot()`.


## How the workshop materials work

The material we'll use for this workshop are in the `~/mawdsley` directory.  This contains the gapminder data we'll be plotting (`gapminder.rds`), the Shiny app we're going to be making (a deployed version of this is at https://mawds.shinyapps.io/worked_example/) (in `worked_example/`) and some example code showing how to produce (static) graphs of the gapminder data (in `codeExample.R`).  The example code uses the functions in `plottingFunctions.R` to produce the graphs; this is to reduce the time we spend dealing with the intricacies of ggplot2.

The directory is a git repository.  Each commit is tagged, and represents the solution to an exercise.  (Not sure how best to do this part - either the user makes a *new* git repo in another directory and puts their exercise under version control _or_ creates the app within `~/mawdsley` and leaves their app out of version control, and checks out the various steps).  You can look at the diff for a commit within R Studio, or you can view it on github by clicking the link within the solution, e.g. [git:01_helloworld](https://github.com/UoMResearchIT/RSE18-shiny-workshop-materials/commit/ffe945ba4943bef378d744e941bea6f46f9970c0).

## Creating a Shiny app

The most straightforward way of creating a new Shiny app is to use R Studio.  From the menu, choose, File, New, Shiny Web App.  

Choose a suitable name for your app (e.g. "gapminder").   Leave the other options as their defaults ("Single File", and Create within directory: `~/mawdsley`).

> ## Single vs multiple files
>
>When we create a new Shiny app we can create a single file (`app.R`), or as two separate files (`ui.R` and `server.R`).   The latter format used to be the only method of definining an Shiny app, but can still be useful when building a more complicated app, as it allows us to separate the user interface (`ui.R`) from the server logic (`server.R`).  As we will be building a relatively small app, we'll use the single file approach.  

When we create a new Shiny app in R Studio, it creates an example app that allows us to alter the number of bins in a histogram.  This uses an example data-set that is provided with R of the waiting times for the eruption of the "Old Faithful" geyser.

We can run the app by pressing the "Run App" button in the toolbar (or by pressing Ctrl+Shift+Enter).  This will launch an browser window within R Studio where we can interact with our app.  



> ## Getting help
> 
> There is a cheat sheet for Shiny included with RStudio.  This can be accessed from the menu: Help, Cheatsheets, Web Applications with shiny.


## User interface design.

We can see in the example app that it is split into two sections; the user interface (`ui`), and a server function.   We'll deal with each of these in turn..

The user interface of the default app uses the `fluidPage()` to create the app's layout. The fluidPage layout will automatically respond to changes in browser size.  Within the `fluidPage()`, we use a `sidebarLayout()` to split the page into two sections; the `sidebarPanel()` which contain (in the example) the app's input (the slider) and the `mainPanel()` which in the example app contains the histogram output.   This is a fairly typical layout for a Shiny app (note that there's nothing stopping us putting more output above the `sidebarLayout()` - for example the `titlePanel()`, or after it - we could, for example use a `p()` tag to include further text, or add further widgets, outputs etc.).   Note that each element of the page is an argument to a function, with `fluidPage()` at the top level.  It's easy to forget this when building an interface, and to forget to place the commas between arguments.  

More flexible, but complex layout options are available, such as `navBarPage()`, to produce a page with a navigation bar.  It's also possible to build a [user intrface from scratch](https://shiny.rstudio.com/articles/html-ui.html), using HTML, CSS etc.

For this tutorial we'll stick with the `fluidPage()` approach.  Most common html tags have a builder function associated with them.  So to add a paragraph of text to the app, we can pass another argument to the `fluidPage()` function:


```r
ui <- fluidPage(
   
   # Application title
   titlePanel("Old Faithful Geyser Data"),
   p("Here is some text"),
   # Sidebar with a slider input for number of bins 
   sidebarLayout(....
```

A very important type of element we can add to the app is a widget.  This provides a control that lets us interact with our app.  The default app has a single widget - the slider that  lets us select the number of bins we want the histogram to have.  The first argument of the function defines the inputId - essentially this is the name of the variable that the widget's output (in this case the selected number of bins) is assigned to.  The second argument defines the label for widget.   

Shiny comes with a number of built in widgets that give different ways of interacting with our app. [The Shiny Widget Gallery](https://shiny.rstudio.com/gallery/widget-gallery.html) lists built in widgets, along with example code to use them.

## Exercise:

We want to be able to show the data for selected continents only.   The `checkboxGroupInput()` widget will let us do this.   Add one to the app.  You will need to provide a vector containing the continent names as the `choices` argument; this can be done using `levels(gapminder$continent)`.  By default none of the check-boxes are selected; this can be changed using the `selected` argument. (for now the check boxes won't do anything; we've not made use of the widget's values anywhere in the app - we'll do this shortly)

## Solution:
 
DO THIS


The `mainPanel()` of the user interface contains a single element in the example app.  The `plotOutput()` displays a plot - in the example app called `distPlot`.  We'll come to how that plot is defined in a moment.  For now, note that there are several `...Output()` functions for displaying various types of output from R, such as `renderTable()` and `renderText


### The server function

The server function contains the code behind the webapp.   Each `...Output()` function in the user interface has a corresponding `render...()` function.   The first argument of the `render...()` function contains the R code to generate the thing we want to render.  For example, in the example app, the `renderPlot()` function contains the code that will generate a histogram.  As this neeeds to b a single expression this will typically be contained in `{}`s 

Let's briefly look at the example code that generates the histogram:


```r
   output$distPlot <- renderPlot({
      # generate bins based on input$bins from ui.R
      x    <- faithful[, 2] 
      bins <- seq(min(x), max(x), length.out = input$bins + 1)
      
      # draw the histogram with the specified number of bins
      hist(x, breaks = bins, col = 'darkgray', border = 'white')
   })
```

`input$bins` will contain the value of the slider which was defined in the user interface (which was called `bins`).   This is how we connect elements of the user interface to the the server logic.

The first agument of `renderPlot()` returns a graph; _how_ we make that graph is up to us.  In the example app, base R graphic are used.  We'll be using ggplot2 in this workshop, as it's easier to make nice looking graphs with it.  To simplify things I've put the ggplot functions within a wrapper function - we'll come to this shortly.  The important thing is that the first (and in the example, only) argument returns a graph object.   We can also pass arguments to manually control the width and height of the plot, etc.

## Reactivity (How Shiny responds to events) - inputs change ==> outputs change

Shiny uses a reactive programming model.  This means that when something changes (such as the slider being moved), everything that depends on that something will be updated automatically.   In the example app the only thing that depends on the slider is the `distPlot` graph, so this will be redrawn if we move the slider.

The graph also depends on some other properties of the app, which we can't see directly, such as the window size.  If we resize the window, Shiny knows that the graph depends on that property of the app, and so will redraw the graph.

Shiny automatically takes care of the dependencies between the various elements, and only updates what is needed. 

## Plotting a "gapminder" graph

In order to avoid getting bogged down on the syntax of ggplot2, a function to produce a "gapminder" plot is provided in the `plottingFunctions.R` file.  This uses ggplot2 to produce a graph, deals with setting fixed axes, consistent colours etc.

Stepping aside from Shiny, we can produce a gapminder plot in R using the following code:



```r
# Load the required libraries
library(ggplot2)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(shiny)
```

```
## Loading required package: methods
```

```r
# Load the gapminder data 
gapminder <- readRDS("gapminder.rds")

# Load the plotting functions
source("plottingFunctions.R")

# Produce a plot for a year of data
# produceGapminderPlot expects a single year of data
gapminder %>% 
  filter(year == 2000) %>% 
  produceGapminderPlot()
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png)


## Exercise:

Add another `filter` to the example to only show countries that are in Europe and Africa

## Solution:


```r
gapminder %>% 
  filter(year == 2000) %>% 
  filter(continent %in% c("Europe", "Africa")) %>% 
  produceGapminderPlot()
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png)


## Back to Shiny

The example app doesn't load any external data or functions (The `faithful` dataset is provided with R).  When we run an app, it sets its location as the working directory.  When we deploy an app it is much easier if all its dependencies are in the same directory.  

* Copy the data (`gapminder.rds`) and plotting functions (`plottingFunctions.R`) into your app directory.  

* Modify the example app to load the `ggplot2` and `dplyr` libraries, the gapminder data from `gapminder.rds` and the plotting functions in `plottingFunctions.R`


## Exercises

* Modify the Shiny app to produce a gapminder plot instead of the histogram.   Note that the `produceGapminderPlot()` requires a single year of data, so you will need to `filter()` the data to a single year before passing it to the function.

* Use the widget you created earlier to only show data for the selected continents

* Create a new widget (or modify the `bins` widget) to let the user choose the year of data to plot. You might want to check out the `sep` option to deal with the thousand separator commas.  

* (optional) check out the options for the `sliderInput()` widget and add an animation button.

* If you created a new widget, you can delete the `bins` widget as we are no longer using this.


We've now created a working app, which lets us explore the data by year and filter by continent.  To summarise:

* Define your server function and user interface 
* Create graphs in the server function  using `renderPlot()`, and other types of output using `render....()` functions
* Display graphs using `plotOutput()` (or `...Output()` for other types of output) in your user interface
* Connect widgets to outputs using `input$inputID` 
* Shiny takes care of updating outputs when inputs change (recative programming)

Now lets look at deploying apps..

## Deploying Shiny apps

If we want to use our app out of R Studio we will need to deploy it to a Shiny Server.  There are a number of options for doing this:

* Host a [Shiny Server instance](https://www.rstudio.com/products/shiny/shiny-server/).   Shiny Server comes in both open source and commercial editions.   
   *See also [Shinyproxy](https://www.shinyproxy.io/) to extend the features of the open source version (authentication, containerisation)
* Deploy the app to shinyapps.io
   * Free and paid for versions

Setting up a Shiny Server is beyond the scope of this workshop.   Instead we'll deploy our apps to shinyapps.io

Create an account on shinyapps.io via [https://www.shinyapps.io/admin/#/signup]  You can do this using a Google or Github account, or using a username/password.

Having created an account, we can deploy the app directly from R Studio. To do this, click the blue publish icon (to the right of the run button).   You will be prompted to enter your account token.  This can be obtained from the dashboard for your shinyapps.io account.  Click the account menu (on the left) then select tokens.  Click "show", "show secret" and then copy the token to the clipboard.  This can be pasted into the box in Studio.

Having connected your account, check that the `app.R` file and all its dependencies (`plottingFunctions.R` and `gapminder.rds`) are selected and then choose publish.  The app will be packaged and uploaded to shinyapps.io


## Going further - reactive data

In this example we only use the filtered gapminder data in a single place - to plot the graph.   For this reason we did the filtering within the `renderPlot()` function.   

Let's add another output to the app; we'll show which country is the richest displayed on the graph, and what its GDP per capita is. This means that we'll be using the gapminder data filtered by year and country in two places; on the graph itself and in the logic where we work out what the richest country is.   

To do this, we'll make the a reactive expression, which contains the data we want to plot.  A rective expression is an expression whose value can change during the running of the app.   As mentioned previously, the reactive programming approach means that Shiny will keep track of when reactive expressions have become invalid (for example, if their inputs have changed - in this example if `year` or `continent` change), and will then recalculate them automatically.  

Let's make the plotted data a reactive expression. We add the following code to the `server()` function:


```r
   # The data we wish to plot
   plotData <- reactive({
     gapminder %>% 
       filter(year == input$year) %>% 
       filter(continent %in% input$continent)
   })
```

Reactive expressions return a function, so we refer to the data we're plotting as `plotData()`.  We can use this in our `renderPlot()` function:


```r
   output$gapminderPlot <- renderPlot({
      plotData() %>% 
         produceGapminderPlot() })
```

The app will behave in the same way as before, but by definining the reactive expression `plotData()` we can use the data that we're plotting on the graph elsewhere in the app.

## Exercise

The `renderTable()` function will render a tibble, and `tableOutput()` will display it.   Use these functions, with the reactive data to display a (long) table containing the data displayed on the graph.
  
The `getRichestCountry()` function in `plottingFunctions.R` will return a tibble containing the name of the richest country in a tibble, and the country's GDP per capita. Modify your `renderTable()` function to only display this information for the displayed graph.












