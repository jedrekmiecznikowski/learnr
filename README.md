## Overview

The **tutor** package makes it easy to turn any [R Markdown](http://rmarkdown.rstudio.com) document into an interactive tutorial. To create a tutorial, just use `library(tutor)` within your Rmd file to activate tutorial mode, then use the `exercise = TRUE` attribute to turn code chunks into exercises. 

For example, here's a very simple tutorial:

    ----
    title: "Hello, Tutor!"
    output: html_document
    runtime: shiny_prerendered
    ----
    
    ```{r setup, include=FALSE}
    library(tutor)
    ```
    
    The following code computes the answer to 1+1. Change it so it computes 2 + 2:
    
    ```{r, exercise=TRUE}
    1 + 1
    ```
    
This is what the running tutorial document looks like after the user has entered their answer:

<kbd>
<img src="README_files/images/hello.png"  width="650" height="189" style="border: solid 1px #cccccc;"/>
</kbd>    
    
You can run a live version of this tutorial with:

```r
rmarkdown::run(system.file("examples/hello.Rmd", package = "tutor"))
```
    
    
We'll go thorugh this example in more detail below. First though let's cover how to install and get started with the **tutor** package.


## Getting Started

### Installation

You can install the development version of the **tutor** package from GitHub as follows:

```r
devtools::install_github("rstudio/tutor", auth_token = "33cdbf9d899fe6eff5022e67e21f08964f7c7b19")
```

You should also install the current [RStudio Preview Release](https://www.rstudio.com/products/rstudio/download/preview/) (v1.0.44 or higher) as it includes tools for easily running and previewing tutorials.

### Creating a Tutorial

A tutorial is just a standard R Markdown document that has three additional attributes:

1. It uses the `runtime: shiny_prerendered` directive in the YAML header.
2. It loads the **tutor** package.
3. It includes one or more code chunks with the `exercise=TRUE` attribute.

The `runtime: shiny_prerendered` element included in the YAML hints at the underlying implementation of tutorails: they are simply Shiny applications which use an R Markdown document as their user-interface rather than the traditional `ui.R` file.

You can copy and paste the simple "Hello, Tutor!" example from above to get started creating your own tutorials.

Note that you aren't limited to the default `html_document` format when creating tutorials. Here's an example of embedding a tutorial within a `slidy_presentation`:

<kbd>
<img src="README_files/images/slidy.png" width="650" height="474" style="border: solid 1px #cccccc;"/>
</kbd>

You can run a live version of this tutorial with:

```r
rmarkdown::run(system.file("examples/slidy.Rmd", package = "tutor"))
```


## Tutorial Exercises

There are some special considerations for code chunks with `exercise=TRUE` which are covered in more depth below.

### Standalone Code

When a code chunk with `exercise=TRUE` is evaluated it's evaulated in a standalone environment (in other words, it doesn't have access to previous computations from within the document other than those provided in the `setup` chunk). This constraint is imposed so that users can execute exercises in any order (i.e. correct execution of one exercise never depends on completion of a prior exercise).

You can however arrange for per-exercise chunk setup code to be run to ensure that the environment is primed correctly. To do this give your exercise chunk a label (e.g. `exercise1`) then add another chunk with the same label plus a `-setup` suffix (e.g. `exercise1-setup`). For example, here we provide a setup chunk to ensure that the correct dataset is always available within an exercise's evaluation environment:


    ```{r exercise1-setup}
    nycflights <- nycflights13::flights
    ```
    
    ```{r exercise1, exercise=TRUE}
    # Change the filter to select February rather than January
    nycflights <- filter(nycflights, month == 1)
    ```

As mentioned above, you can also have global setup code that all chunks will get the benefit of by including a global `setup` chunk. For example, if there were multiple chunks that needed access to the original version of the flights datset you could do this:


    ```{r setup, include=FALSE}
    nycflights <- nycflights13::flights
    ```
    
    ```{r exercise1, exercise=TRUE}
    # Change the filter to select February rather than January
    filter(nycflights, month == 1)
    ```

    ```{r exercise1, exercise=TRUE}
    # Change the sort order to Ascending
    arrange(nycflights, desc(arr_delay))
    ```

### Evaluation

By default, exercise code chunks are NOT pre-evaluated (i.e there is no initial output for them). However, in some cases you may want to show initial exercise output (especially for exercises like the ones above where the user is asked to modify code rather than write new code from scratch).

You can arrange for an exercise to be pre-evaluated (and it's output shown) using the `exercise.eval` chunk option. For example:

    ```{r, exercise=TRUE, exercise.eval=TRUE}
    # Change the filter to select February rather than January
    filter(nycflights, month == 1)
    ```
    
You can also set a global default for exercise evaluation using `knitr::opts_chunk` within your global setup chunk, for example:

    ```{r setup, include=FALSE}
    knitr::opts_chunk$set(exercise.eval = TRUE)
    ```

## Using Shiny


The **tutor** package uses `runtime: shiny_prerendered` to turn regular R Markdown documents into live tutorials. Since tutorials are Shiny applications at their core, it's also possible to add other forms of interaction and interactivity using Shiny (e.g. for teaching a statistical concept interactively). 

The basic technique is to add a `context="server"` attribute to code chunks that are part of the Shiny server as opposed to UI definition. For example:

    ```{r, echo=FALSE}
    sliderInput("bins", "Number of bins:", min = 1, max = 50, value = 30)
    plotOutput("distPlot")
    ```
    
    ```{r, context="server"}
    output$distPlot <- renderPlot({
      x <- faithful[, 2]  # Old Faithful Geyser data
      bins <- seq(min(x), max(x), length.out = input$bins + 1)
      hist(x, breaks = bins, col = 'darkgray', border = 'white')
    })
    ```

You can learn more by reading the [Prerendered Shiny Documents](http://rmarkdown.rstudio.com/authoring_shiny_prerendered.html) article on the R Markdown website.

## External Resources

You may wish to include external resources (images, videos, CSS, etc.) within your tutorial documents. Since the tutorial will be deployed as a Shiny applications, you need to ensure that these resources are placed within one of several directories which are reachable by the Shiny server:

<table>
<thead>
<tr class="header">
<th>Directory</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>images/</code></td>
<td>Image files (e.g. PNG, JPEG, etc.)</td>
</tr>
<tr class="even">
<td><code>css/</code></td>
<td>CSS stylesheets</td>
</tr>
<tr class="odd">
<td><code>js/</code></td>
<td>JavaScript scripts</td>
</tr>
<tr class="even">
<td><code>www/</code></td>
<td>Any other files (e.g. downloadable datasets)</td>
</tr>
</tbody>
</table>

The reason that all files within the directory of the main Rmd can't be referenced from within the web document is that many of these files are application source code and data, which may not be something you want to be downloadable by end users. By restricting the files which can be referenced to the above directories you can control which files are downloadable and which are not.

## Deploying Tutorials

Tutorials are Shiny applications that are run using the `rmarkdown::run` function rather than the `shiny::runApp` function:

```r
rmarkdown::run("tutorial.Rmd")
```

This means that tutorials can be deployed all of the same ways that Shiny applications can, including running locally on an end-user's machine or running on a Shiny Server or hosting service like shinyapps.io. See the [Deployment](http://rmarkdown.rstudio.com/authoring_shiny_prerendered.html#deployment) section of the `runtime: shiny_prerendered` documentation for additional details.

Note that there is one important difference between tutorials and most other Shiny applications you deploy: with tutorials the end user can directly execute R code on the server. This creates some special considerations around resource usage and security which are discussed below.

### Resource Usage



### Security





