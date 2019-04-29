---
title: "6. Exploring virtual species"
output:
  html_document:
    fig_caption: yes
    keep_md: yes
    mathjax: "http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"
    theme: united
    toc: true
    toc_float: true
---

This section is mainly intended to users not very familiar with R. For example, if you are not sure how to obtain the maps of generated virtual species, read this section. If you simply want to extract (sample) occurrence points for your virtual species, then you should jump to the next section.


## 6.1. Consult the details of a generated virtual species

Let's create a simple virtual species:

```r
library(virtualspecies)
```

```
## Loading required package: raster
```

```
## Loading required package: sp
```

```r
# Worldclim data
worldclim <- getData("worldclim", var = "bio", res = 10)

# Formatting of the response functions
my.parameters <- formatFunctions(bio1 = c(fun = 'dnorm', mean = 250, sd = 50),
                                 bio12 = c(fun = 'dnorm', mean = 4000, sd = 2000))

# Generation of the virtual species
my.species <- generateSpFromFun(raster.stack = worldclim[[c("bio1", "bio12")]],
                                parameters = my.parameters)
```

```
## Generating virtual species environmental suitability...
```

```
##  - The response to each variable was rescaled between 0 and 1. To
##             disable, set argument rescale.each.response = FALSE
```

```
##  - The final environmental suitability was rescaled between 0 and 1.
##             To disable, set argument rescale = FALSE
```

```r
# Conversion to presence-absence
my.species <- convertToPA(my.species,
                          beta = 0.7)
```

```
##    --- Determing species.prevalence automatically according to alpha and beta
```

```
##    Logistic conversion finished:
##               
## - beta = 0.7
## - alpha = -0.05
## - species prevalence =0.0338978411382996
```

![Fig. 6.1 Automatic illustration of the randomly generated species](06-virtualspeciesobjects_files/figure-html/output1-1.png)

If we want to know how it was generated, we simply type the object name in the R console:


```r
my.species
```

```
## Virtual species generated from 2 variables:
##  bio1, bio12
## 
## - Approach used: Responses to each variable
## - Response functions:
##    .bio1  [min=-269; max=314] : dnorm   (mean=250; sd=50)
##    .bio12  [min=0; max=9916] : dnorm   (mean=4000; sd=2000)
## - Each response function was rescaled between 0 and 1
## - Environmental suitability formula = bio1 * bio12
## - Environmental suitability was rescaled between 0 and 1
## 
## - Converted into presence-absence:
##    .Method = probability
##    .probabilistic method    = logistic
##    .alpha (slope)           = -0.05
##    .beta  (inflexion point) = 0.7
##    .species prevalence      = 0.0338978411382996
```

And a summary of how the virtual species was generated appears:

* It shows us the variables used.
* It shows us the approach used and all the details of the approach, so we can use it to reconstruct another virtual species with the exact same parameters later on. It also provides us the range of values of our environmental variables (bio1 (mean annual temperature) ranged from -269 (-26.9Â°C) to 314 (31.4Â°C)). This is helpful to quickly get an idea of the preferences of our species; for example here we see that we have a species living in hot environments, with a peak at 250 (25Â°C).
* If a conversion to presence-absence was performed, it shows us the parameters of the conversion, and provides the species prevalence (the species prevalence is always calculated and provided).
* If you have introduced a distribution bias (will be seen in a later section), it will provide information about this particular bias.


## 6.2. Plot the virtual species map

Plotting the distribution maps of a virtual species is straightforward:

```r
plot(my.species)
```

![Fig. 6.2 Maps obtained when using the function `plot()` on virtual species objects](06-virtualspeciesobjects_files/figure-html/output3-1.png)

If the environmental sutiability has been converted into presence-absence, then the plot will conveniently display both the environmental suitability and the presence-absence map.

## 6.3. Plot the species-environment relationship

As illustrated several times in this tutorial, there is a function to automatically generate an appropriate plot for your virtual species: `plotResponse`


```r
plotResponse(my.species)
```

![Fig. 6.3 Example of figure obtained when running `plotResponse()` on a virtual species object](06-virtualspeciesobjects_files/figure-html/output3bis-1.png)

## 6.4. Plot the relationship between suitability and probability of occurrence

If you converted your environmental suitability into presence-absence with a probabilistic approach, chances are that you modified the environmental suitability function, e.g. if you used a logistic method or if you wanted to reach a specific prevalence. You may be interested in the relationship between environmental suitability and probability of occurrence, which can be plotted with `plotSuitabilityToProba`


```r
plotSuitabilityToProba(my.species)
```

![Fig. 6.4 Example of figure obtained when running `plotSuitabilityToProba()` on a virtual species object](06-virtualspeciesobjects_files/figure-html/output3ter-1.png)


## 6.5. Extracting elements of the virtual species, such as the rasters of environmental suitability

The virtual species object is structured as a `list` in R, which roughly means that it is an object containing many "sub-objects". When you run functions on your virtual species object, such as the conversion into presence-absence, then new sub-objects are added or replaced in the list.

There is a function allowing you to see the content of the list: `str()`


```r
str(my.species)
```

```
## List of 6
##  $ approach                 : chr "response"
##  $ details                  :List of 5
##   ..$ variables            : chr [1:2] "bio1" "bio12"
##   ..$ formula              : chr "bio1 * bio12"
##   ..$ rescale.each.response: logi TRUE
##   ..$ rescale              : logi TRUE
##   ..$ parameters           :List of 2
##  $ suitab.raster            :Formal class 'RasterLayer' [package "raster"] with 12 slots
##  $ PA.conversion            : Named chr [1:5] "probability" "logistic" "-0.05" "0.7" ...
##   ..- attr(*, "names")= chr [1:5] "conversion.method" "probabilistic.method" "alpha" "beta" ...
##  $ probability.of.occurrence:Formal class 'RasterLayer' [package "raster"] with 12 slots
##  $ pa.raster                :Formal class 'RasterLayer' [package "raster"] with 12 slots
##  - attr(*, "class")= chr [1:2] "virtualspecies" "list"
```

We are informed that the object is a `list` containing 5 elements (sub-objects), that you can read on the lines starting with a `$`: `approach`, `details`, `suitab.raster`, `PA.conversion` and `pa.raster`.

You can extract each element using the `$`: for example, to extract the suitability raster, type


```r
my.species$suitab.raster
```

```
## class       : RasterLayer 
## dimensions  : 900, 2160, 1944000  (nrow, ncol, ncell)
## resolution  : 0.1666667, 0.1666667  (x, y)
## extent      : -180, 180, -60, 90  (xmin, xmax, ymin, ymax)
## coord. ref. : +proj=longlat +datum=WGS84 +ellps=WGS84 +towgs84=0,0,0 
## data source : in memory
## names       : layer 
## values      : 0, 1  (min, max)
```

If you are interested in the probability of occurrence raster, type


```r
my.species$probability.of.occurrence
```

```
## class       : RasterLayer 
## dimensions  : 900, 2160, 1944000  (nrow, ncol, ncell)
## resolution  : 0.1666667, 0.1666667  (x, y)
## extent      : -180, 180, -60, 90  (xmin, xmax, ymin, ymax)
## coord. ref. : +proj=longlat +datum=WGS84 +ellps=WGS84 +towgs84=0,0,0 
## data source : in memory
## names       : layer 
## values      : 8.31528e-07, 0.9975274  (min, max)
```

If you are interested in the presence-absence raster, type


```r
my.species$pa.raster
```

```
## class       : RasterLayer 
## dimensions  : 900, 2160, 1944000  (nrow, ncol, ncell)
## resolution  : 0.1666667, 0.1666667  (x, y)
## extent      : -180, 180, -60, 90  (xmin, xmax, ymin, ymax)
## coord. ref. : +proj=longlat +datum=WGS84 +ellps=WGS84 +towgs84=0,0,0 
## data source : in memory
## names       : layer 
## values      : 0, 1  (min, max)
```


You can also see that we have "sub-sub-objects", in the lines starting with `..$`: these are objects contained within the sub-object `details`. You can also extract them easily:


```r
my.species$details$variables
```

```
## [1] "bio1"  "bio12"
```

However, the sub-sub-sub-objects (level 3 of depth and beyond) are not listed when you use `str()` on the entire virtual species object. For example, if we extract the `parameters` object from the details, we can see that it contains all the function names and their parameters:


```r
my.species$details$parameters
```

```
## $bio1
## $bio1$fun
##     fun 
## "dnorm" 
## 
## $bio1$args
## mean   sd 
##  250   50 
## 
## $bio1$min
## [1] -269
## 
## $bio1$max
## [1] 314
## 
## 
## $bio12
## $bio12$fun
##     fun 
## "dnorm" 
## 
## $bio12$args
## mean   sd 
## 4000 2000 
## 
## $bio12$min
## [1] 0
## 
## $bio12$max
## [1] 9916
```

```r
# Looking at how it is structured:
str(my.species$details$parameters)
```

```
## List of 2
##  $ bio1 :List of 4
##   ..$ fun : Named chr "dnorm"
##   .. ..- attr(*, "names")= chr "fun"
##   ..$ args: Named num [1:2] 250 50
##   .. ..- attr(*, "names")= chr [1:2] "mean" "sd"
##   ..$ min : num -269
##   ..$ max : num 314
##  $ bio12:List of 4
##   ..$ fun : Named chr "dnorm"
##   .. ..- attr(*, "names")= chr "fun"
##   ..$ args: Named num [1:2] 4000 2000
##   .. ..- attr(*, "names")= chr [1:2] "mean" "sd"
##   ..$ min : num 0
##   ..$ max : num 9916
```


Hence, the main message here is if you want to explore the content of the virtual species object, use the function `str()`, look at which sub-objects you are interested in, and extract them with `$`. 



## 6.6. Saving the virtual species objects for later use



If you want to save a virtual species object, you can save it on your hard drive, using the R function `saveRDS()`:


```r
saveRDS(my.species, file = "MyVirtualSpecies.RDS")
```

You can load it in a later session of R, using `readRDS()`:


```r
my.species <- readRDS("MyVirtualSpecies.RDS")
```
