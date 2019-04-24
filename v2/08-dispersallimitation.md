---
title: '8. Dispersal limitation'
output:
  html_document:
    fig_caption: yes
    keep_md: yes
    mathjax: "http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"
    theme: united
    toc: true
    toc_float: true
---



One of the most disputed assumptions of species distribution models is that species are at equilibrium with their environment. This assumptions means that species are supposed to occupy their full range of suitable environmental conditions. In reality, it is unlikely, because of dispersal limitations, biotic interactions, etc., which precludes species to occupy areas which are theoretically suitable. As a consequence, it is worth testing how well modelling techniques perform when this assumption is violated.

This is why we introduced the possibility of biasing the distribution of species, to simulate species which are not at equilibrium. This possibility is implemented in the function `limitDistribution`.

## 8.1. An introduction example

Let's use the same virtual species we generated above:

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
worldclim <- getData("worldclim", var = "bio", res = 10)

# Formatting of the response functions
my.parameters <- formatFunctions(bio1 = c(fun = 'dnorm', mean = 250, sd = 50),
                                 bio12 = c(fun = 'dnorm', mean = 4000, sd = 2000))

# Generation of the virtual species
my.first.species <- generateSpFromFun(raster.stack = worldclim[[c("bio1", "bio12")]],
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
my.first.species <- convertToPA(my.first.species,
                                beta = 0.7, plot = FALSE)
```

```
##    --- Determing species.prevalence automatically according to alpha and beta
```

```
##    Logistic conversion finished:
##               
## - beta = 0.7
## - alpha = -0.05
## - species prevalence =0.0337387365039066
```



Now, let's assume our species originates from South America, and has not been able to disperse through the oceans (result in figure 8.2).


```r
my.first.species <- limitDistribution(my.first.species, 
                                      geographical.limit = "continent",
                                      area = "South America",
                                      plot = FALSE)

par(mfrow = c(2, 1))
plot(my.first.species$pa.raster, main = "Theoretical distribution")
plot(my.first.species$occupied.area, main = "Realised distribution")
```

![Fig. 8.2 Distribution of a species limited to South America](08-dispersallimitation_files/figure-html/dist2-1.png)

In the following sections, we see how to customise this function, but [the usage is basically the same as when applying a sampling bias.](07-sampleoccurrences.html#uneven-sampling-intensity)  
  


## 8.2. Customisation of the parameters

There are six main possibilities to limit the distribution to a particular area.

### 8.2.1. Using countries, regions or continents

As illustrated in the example above, use `geographical.limit = "country"`, `geographical.limit = continent` or `geographical.limit = "region"` and provide the correct name(s) of the area to `area`


```r
my.sp1 <- limitDistribution(my.first.species, 
                            geographical.limit = "country",
                            area = c("Brazil", "Venezuela"))
```

![Fig. 8.3 Distribution of a species arbitrary limited to Brazil and Venezuela](08-dispersallimitation_files/figure-html/dist3-1.png)


### 8.2.2. Using a polygon

Set `geographical.limit = "polygon"`, and provide a polygon (of type `SpatialPolygons` or `SpatialPolygonsDataFrame` from package `sp`) to the argument `area`.


```r
philippines <- getData("GADM", country = "PHL", level = 0)
my.sp2 <- limitDistribution(my.first.species, 
                            geographical.limit = "polygon",
                            area = philippines)
```

```
## Warning in limitDistribution(my.first.species, geographical.limit =
## "polygon", : Polygon projection is not checked. Please make sure you have
## the same projections between your polygon and your presence-absence raster
```

![Fig. 8.4 Distribution of a species limited to the Philippines](08-dispersallimitation_files/figure-html/dist4-1.png)

### 8.2.3. Using an extent object  
Set `geographical.limit = "extent"`, and provide an extent to the argument `area` ([see section 7.2.3. if you are not familiar with extents](07-sampleoccurrences.html#providing-an-extent-object)). You can also simply set `geographical.limit = "extent"`, and click twice on the map when asked to:


```r
my.extent <- extent(-80, -20, -35, -5)
my.sp2 <- limitDistribution(my.first.species, 
                            geographical.limit = "extent",
                            area = my.extent)
plot(my.extent, add = TRUE)
```

![Fig. 8.5 Distribution of a species limited to a particular extent](08-dispersallimitation_files/figure-html/dist5-1.png)

### 8.2.4. Using a raster of suitable habitat
Let's first generate an example raster of habitat patches. [For this, I will use this function
that you can find here.](https://github.com/Farewe/CooccurrenceIssue/blob/master/scripts/functions/patch_generation.R)



```r
suitable.habitat <- generate.patches(worldclim[[1]], n.patches = 100, patch.size = 300)
plot(suitable.habitat)
```

![Fig. 8.6 Raster of suitable habitat patches](08-dispersallimitation_files/figure-html/dist5c-1.png)
Then, we will restrict the distribution of our species to these suitable habitat patches:


```r
my.sp3 <- limitDistribution(my.first.species, 
                            geographical.limit = "raster",
                            area = suitable.habitat)
```

![Fig. 8.7 Distribution of our species restricted to suitable habitat patches](08-dispersallimitation_files/figure-html/dist5d-1.png)

                            
## 8.3. Sampling occurrence points in the dispersal-limited distribution

Once the distribution of a species has been limited with `limitDistribution()`, you just have to apply `sampleOccurrences` on this species: it will automatically sample from the realised distribution of the species.


```r
my.first.species <- limitDistribution(my.first.species, 
                                      geographical.limit = "continent",
                                      area = "South America",
                                      plot = FALSE)

sampleOccurrences(my.first.species, n = 30)
```

![Fig. 8.8 Occurrence sampled in the distribution of a species limited to South America](08-dispersallimitation_files/figure-html/dist6-1.png)

```
## Occurrence points sampled from a virtual species
## 
## - Type: presence only
## - Number of points: 30
## - No sampling bias
## - Detection probability: 
##    .Probability: 1
##    .Corrected by suitability: FALSE
## - Probability of identification error (false positive): 0
## - Multiple samples can occur in a single cell: No
## 
## First 10 lines: 
##            x          y Real Observed
## 1  -66.08333  3.9166667    1        1
## 2  -71.08333 -0.9166667    1        1
## 3  -77.75000 -4.4166667    1        1
## 4  -68.25000 -1.5833333    1        1
## 5  -67.25000  2.9166667    1        1
## 6  -71.41667  2.2500000    1        1
## 7  -59.91667  6.9166667    1        1
## 8  -54.75000 -0.4166667    1        1
## 9  -72.08333  5.5833333    1        1
## 10 -84.58333 15.7500000    1        1
## ... 20 more lines.
```
