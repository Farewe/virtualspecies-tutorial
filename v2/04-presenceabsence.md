---
title: '4. Converting environmental suitability to presence-absence'
output:
  html_document:
    fig_caption: yes
    keep_md: yes
    mathjax: "http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"
    theme: united
    toc: true
    toc_float: true
---



## 4.1. Introduction: why should we avoid a threshold conversion?
What we did so far was defining the relationship between the species and its environment. It was the most important part, but what it provides us is a spatial distribution of the species' environmental suitability, not a spatial distribution of the species' presences/absences. Hence we now have to convert the environmental suitability into presence-absence.

The simplest approach, which was also the most widely used until 2014, consists in defining a threshold of environmental suitability, below which conditions are unsuitable, so absence is attributed; and above which conditions are suitable, so persence is attributed. However, you should completely avoid this approach which is *pure evil*:

* Most importantly, this creates completely unrealistic species:
    + Real species are often absent from areas of high suitability, because of factors acting at smaller spatial scales, such as biotic pressure (competition, predation), disturbances, stochastic events.
    + Real species often occur in unsuitable areas, because of very particular conditions allowing their occurrence (microclimatic/microhabitat conditions, climatic refugia).
* The threshold almost completely removes the previously defined relationship between the species and its environment. The gradual aspect is lost: the above-threshold part of the environmental gradient is always fully suitable, while the below-threshold part is always fully unsuitable.

So, how can we convert our environmental suitability into presence-absence without a threshold? 

We use a probabilistic approach: the probability of getting a presence of the species in a given pixel is dependent on its suitability in that pixel:

![Fig. 4.1 Logistic curve used for a probabilistic conversion into presence-absence](04-presenceabsence_files/figure-html/conv1-1.png)

With the example above, a pixel with environmental suitability equal to 0.6 has 88% chance of having species presence, and 12% chance of having species absence.

This means that we convert the environmental suitability of each pixel into a probability of occurrence. This probability of occurrence is then used to sample presence or absence in each cell, i.e., we make a random draw of presence or absence weighted by the probability of occurrence. As a consequence, repeated realisation of the presence-absence conversion will produce different occupancy maps, each providing a valid realisation of the true species distribution map.

This conversion is fully customisable, and can range from threshold conversion to logistic to linear conversion, by adjusting parameters (explained in section 4.2):


![Fig. 4.2 Contrasting examples of conversion curves and their result on the distribution range of the same virtual species.](04-presenceabsence_files/figure-html/conv2-1.png)

The logistic conversion looks much more like what a real species distribution is than the linear and threshold-like conversions.

In the next section we will see how to perform the conversion, and how to customise it.

## 4.2. Customisation of the conversion

At first you may be interested to see the function in action, before we try to customise it. Here it is!


```r
# Let's use the same species we generated before:
my.parameters <- formatFunctions(bio1 = c(fun = 'dnorm', mean = 250, sd = 50),
                                 bio12 = c(fun = 'dnorm', mean = 4000, sd = 2000))
my.first.species <- generateSpFromFun(raster.stack = worldclim[[c("bio1", "bio12")]],
                                      parameters = my.parameters,
                                      plot = FALSE)

# Run the conversion to presence-absence
pa1 <- convertToPA(my.first.species, plot = TRUE)
```

![Fig. 4.3 Maps of environmental suitability and presence-absence of the virtual species](04-presenceabsence_files/figure-html/conv3-1.png)

You have probably noticed that the conversion was performed without you choosing any parameter. This is because by default, the function randomly assigns parameters to the conversion. What are these parameters?

They are the parameters $\alpha$ and $\beta$ which determine the shape of the logistic curve:

* $\beta$ controls the inflexion point,

![Fig. 4.4 Impact of beta on the conversion curve](04-presenceabsence_files/figure-html/conv4-1.png)

* and $\alpha$ drives the 'slope' of the curve

![Fig. 4.4 Impact of alpha on the conversion curve](04-presenceabsence_files/figure-html/conv5-1.png)


The parameters are fairly simple to customise:



```r
pa2 <- convertToPA(my.first.species,
                   beta = 0.65, alpha = -0.07,
                   plot = TRUE)
```

![Fig. 4.5 Conversion with beta = 0.65, alpha = -0.07](04-presenceabsence_files/figure-html/conv6-1.png)


```r
# You can modify the conversion of your object if you did not like it:
pa2 <- convertToPA(pa2,
                   beta = 0.3, alpha = -0.02,
                   plot = TRUE)
```

![Fig. 4.5 Conversion with beta = 0.65, alpha = -0.07](04-presenceabsence_files/figure-html/conv7-1.png)

In the next sections I summarise how to choose appropriate values of `alpha` and `beta`, and also I introduce a third parameter which may be very useful to generate virtual species distributions: the species prevalence (i.e., the number of places occupied by the species, out of the total number of available places). 


### 4.2.1. beta

`beta` is very simple to grasp, as it is the inflexion point of the curve. Hence, looking at the three beta curves above, we can see that a lower `beta` will increase the probability of finding suitable conditions for the species (wider distribution range). A higher `beta` will decrease the probability of finding suitable conditions (smaller distribution range).

### 4.2.2. alpha

`alpha` may be more difficult to grasp, as it is dependent on the range of values of the `x` axis (in our case, the environmental suitability, ranging from 0 to 1):

* If `alpha` is approximately equal to the range of`x` or greater (in absolute value), then the conversion will have a linear shape. In our case, it means  values of `alpha` below -.3).
* If `alpha` is about 5-10% of the range of `x`, then the conversion will be logistic. In our case, you can have nice logistic curves for values of `alpha` between -0.1 and -0.01.
* If `alpha` is much smaller compared to `x` (in absolute value), then the conversion will be threshold-like. In our case, if means values of `alpha` in the range [-0.001, 0[.

### 4.2.3. Conversion to presence-absence based on a value of species prevalence

_The species prevalence is the number of places (here, pixels) actually occupied by the species out of the total number of places (pixels) available. Do not confuse the **SPECIES PREVALENCE** with the [**SAMPLE PREVALENCE**](07-sampleoccurrences.html#defining-the-sample-prevalence), which in turn is the proportion of samples in which you have found the species._

Numerous authors have shown the importance of the species prevalence in species distribution modelling, and how it can bias models. As a consequence, when generating virtual species distributions to test particular protocols or modelling techniques, you may be interested in testing different values of species prevalence. However, it is important to know that the species prevalence is dependent on **the species-environment relationship**, **the shape of the probabilistic conversion curve** AND **the spatial distribution of environmental conditions**. As a consequence, the function has to try different shapes of conversion curve to find a conversion according to your chosen value of species prevalence. Sometimes, it is not possible to reach a particular species prevalence, in that case the function will choose the conversion curve providing results closest to your species prevalence.

If you want to generate a species with a particular species prevalence, then you also have to fix either `alpha` or `beta`. I strongly advise to fix a value of `alpha` (this is the default paramter, with `alpha = -0.05`), so that the function will try to find an appropriate conversion by testing different values of `beta`. If you prefer to fix the value of `beta`, then the function will try different values of `alpha`, but it is likely that it will not be able to find a conversion generating a species with the appropriate prevalence.

Let's see it in practice:


```r
# Let's generate a species occupying 20% of the world
# Default settings, alpha = -0.05
sp.0.2 <- convertToPA(my.first.species,
                      species.prevalence = 0.2)
```

```
##    --- Determing beta automatically according to alpha and species.prevalence
```

![Fig. 4.6 Conversion of a species with a prevalence of 0.2, _i.e._ occupying 20% of the world (which is quite large)](04-presenceabsence_files/figure-html/conv8-1.png)

```r
sp.0.2
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
##    .alpha (slope)           = -0.05
##    .beta  (inflexion point) = 0.234375
##    .species prevalence      = 0.199
```


```r
# Now, a species occupying 1.5% of the world
# Change alpha to have a slightly more steep curve
sp.0.015 <- convertToPA(my.first.species,
                        species.prevalence = 0.015,
                        alpha = -0.015)
```

```
##    --- Determing beta automatically according to alpha and species.prevalence
```

![Fig. 4.7 Conversion of a species with a prevalence of 0.015, _i.e._ occupying 1.5% of the world](04-presenceabsence_files/figure-html/conv9-1.png)

```r
sp.0.015
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
##    .alpha (slope)           = -0.015
##    .beta  (inflexion point) = 0.828125
##    .species prevalence      = 0.014
```


```r
# Let's try by fixing beta rather than alpha
# We want a species occupying 10% of the world, with a high value of beta
sp.10 <- convertToPA(my.first.species,
                     species.prevalence = 0.1,
                     alpha = NULL,
                     beta = 0.9)
```

```
##    --- Determing alpha automatically according to beta and species.prevalence
```

![Fig. 4.8 Conversion of a species with a prevalence of 0.1, _i.e._ occupying 10% of the world](04-presenceabsence_files/figure-html/conv10-1.png)

```r
sp.10
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
##    .alpha (slope)           = -0.31346875
##    .beta  (inflexion point) = 0.9
##    .species prevalence      = 0.091
```

It worked, but the resulting species does not look realistic at all: alpha was below -0.3, which means that we had a quasi-linear conversion curve, producing this unrealistic presence-absence map.



```r
# Now an impossible task: a low value of beta for the same requirements:
sp.10bis <- convertToPA(my.first.species,
                        species.prevalence = 0.1,
                        alpha = NULL,
                        beta = 0.3)
```

```
##    --- Determing alpha automatically according to beta and species.prevalence
```

```
## Warning in convertToPA(my.first.species, species.prevalence = 0.1, alpha = NULL, : Warning, the desired species prevalence cannot be obtained, because of the chosen beta and available environmental conditions (see details).
##                         The closest possible estimate of prevalence was 0.15 
## Perhaps you can try a higher beta value.
```

![Fig. 4.9 Conversion of a species whose asked prevalence (0.1) cannot be reached because of a too low value of beta](04-presenceabsence_files/figure-html/conv11-1.png)

```r
sp.10bis
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
##    .alpha (slope)           = -0.001
##    .beta  (inflexion point) = 0.3
##    .species prevalence      = 0.147
```
