---
title: '5. Generating random species'
output:
  html_document:
    fig_caption: yes
    keep_md: yes
    mathjax: "http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"
    theme: united
    toc: true
    toc_float: true
---

The `virtualspecies` package embeds a function to randomly generate virtual species `generateRandomSp`, with many customisable options to constrain the randomisation process.

Let's take a look:


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

my.stack <- worldclim[[c("bio2", "bio5", "bio6", "bio12", "bio13", "bio14")]]
random.sp <- generateRandomSp(my.stack)
```

```
##  - Perfoming the pca
```

```
##  - Defining the response of the species along PCA axes
```

```
##  - Calculating suitability values
```

```
##  - Converting into Presence - Absence
```

```
##    --- Determing species.prevalence automatically according to alpha and beta
```

![Fig. 5.1 A species randomly generated with `generateRandomSp`](05-randomspecies_files/figure-html/rand1-1.png)

```r
random.sp
```

```
## Virtual species generated from 6 variables:
##  bio2, bio5, bio6, bio12, bio13, bio14
## 
## - Approach used: Response to axes of a PCA
## - Axes:  1, 2 ;  83.24 % explained by these axes
## - Responses to axes:
##    .Axis 1  [min=-19; max=2.6] : dnorm (mean=0.5838015; sd=3.377431)
##    .Axis 2  [min=-3.44; max=10.95] : dnorm (mean=0.2761077; sd=0.1933877)
## - Environmental suitability was rescaled between 0 and 1
## 
## - Converted into presence-absence:
##    .Method = probability
##    .alpha (slope)           = -0.1
##    .beta  (inflexion point) = 0.509509509509509
##    .species prevalence      = 0.114
```

We can see that the species was generated using a PCA approach. Indeed, [as explained in the PCA section](#second-approach-generate-virtual-species-with-a-principal-components-analysis), when you have a lot of variables, it becomes very difficult to generate a species with realistic environmental requirements. Hence, by default the function `generateRandomSp` uses a PCA approach if you have 6 or more variables, and a 'response functions' approach if you have less than 6 variables.

In the next sections I detail the many possible customisations for the function `generateRandomSp`.

## 5.1. General parameters

### 5.1.1. Choosing the approach

You can choose `approach = `:

* `"automatic"`: a 'response' approach is used if you have less than 6 variables and a 'PCA' approach is used if you have 6 or more variables
* `"random"`: a random approach is chosen (response or PCA)
* `"response"`: to use a response approach
* `"pca"`: to use a pca approach

### 5.1.2. Rescaling of the environmental suitability

By default, `rescale = TRUE`, which means that the environmental suitability is rescaled between 0 and 1.

### 5.1.3. Conversion to presence-absence

You can choose to enable the conversion of environmental suitability to presence-absence. To do this, set `convert.to.PA = TRUE`. You can customise the conversion:

* choose the conversion method with `PA.method`. You should leave it to `probability` unless you have a very particular reason to use the `threshold` approach.
* define the parameters `alpha`, `beta` and `species.prevalence` exactly as explained in the convert to presence-absence section, or leave them as they are to create a random conversion into presence-absence

## 5.2. Parameters specific to a 'response' approach

### 5.2.1. Define the possible response functions

At the moment, four relations are possible for a random generation of virtual species: Gaussian (`gaussian`), linear (`linear`), logistic (`logistic`) and quadratic (`quadratic`) relations. By default, all the relation types are used. You can choose to use any combination with the argument `relations`:


```r
# A species with gaussian and logistic response functions
random.sp1 <- generateRandomSp(worldclim[[1:3]], 
                              relations = c("gaussian", "logistic"),
                              convert.to.PA = FALSE)
```

```
##  - Determining species' response to predictor variables
```

```
##  - Calculating species suitability
```

![Fig. 5.2 A species randomly generated with `generateRandomSp`, with gaussian and logistic response functions](05-randomspecies_files/figure-html/rand2-1.png)

```r
random.sp1
```

```
## Virtual species generated from 3 variables:
##  bio3, bio2, bio1
## 
## - Approach used: Responses to each variable
## - Response functions:
##    .bio3  [min=8; max=95] : logisticFun   (alpha=-8.7; beta=73.4596484596485)
##    .bio2  [min=9; max=211] : logisticFun   (alpha=-16.3436363636364; beta=128.132447132447)
##    .bio1  [min=-269; max=314] : logisticFun   (alpha=-36.57; beta=-212.673981673982)
## - Each response function was rescaled between 0 and 1
## - Environmental suitability formula = bio1 * bio2 * bio3
## - Environmental suitability was rescaled between 0 and 1
```


```r
plotResponse(random.sp1)
```

![Response functions of the randomly generated species](05-randomspecies_files/figure-html/rand3-1.png)



```r
# A purely gaussian species
random.sp2 <- generateRandomSp(worldclim[[1:3]], 
                              relations = "gaussian",
                              convert.to.PA = FALSE)
```

```
##  - Determining species' response to predictor variables
```

```
##  - Calculating species suitability
```

![Fig. 5.4 A species randomly generated with `generateRandomSp`, with gaussian response functions only](05-randomspecies_files/figure-html/rand4-1.png)

```r
random.sp2
```

```
## Virtual species generated from 3 variables:
##  bio3, bio1, bio2
## 
## - Approach used: Responses to each variable
## - Response functions:
##    .bio3  [min=8; max=95] : dnorm   (mean=91.7148471484715; sd=68.1242912429124)
##    .bio1  [min=-269; max=314] : dnorm   (mean=-19.7654576545766; sd=316.694596945969)
##    .bio2  [min=9; max=211] : dnorm   (mean=69.1432814328143; sd=108.337723377234)
## - Each response function was rescaled between 0 and 1
## - Environmental suitability formula = bio1 * bio2 * bio3
## - Environmental suitability was rescaled between 0 and 1
```


```r
plotResponse(random.sp2)
```

![Fig. 5.5 Response functions of the randomly generated species](05-randomspecies_files/figure-html/rand5-1.png)


### 5.2.2. Rescale individual response functions

As explained in the section on the 'response' approach, you can choose to rescale or not each individual response function, with the argument `rescale.each.response`. `TRUE` by default.

### 5.2.3. Try to find a realistic species

An important issue with the generation of random responses to the environment, is that you can obtain a species willing to live in summer temperatures of 35�C and winter temperature of -50�C. This may interesting for generating species from another planet, but you are probably more interested in generating species that can actually live on Earth. There is therefore an option to do that, activated by default: `realistic.sp`.

When activating this argument, the function will proceed step-by-step to try defining a realistic species. At step one, one of the variable is chosen, and the program randomly determines a response function for this variable. Then, it will compute the environmental suitability of this species. At step two, the program will pick a second variable, and will constrain its random generation depending on the environmental suitability obtained at step one. For example, if at step 2 a gaussian response is picked, then the mean will be chosen in areas where the species had a high environmental suitability. Then, the environmental suitability is recalculated on the basis of the first two response functions. At step 3, another variable is picked, a response function randomly generated with respect to areas where the species already has a high suitability, and so on until there are no variables left. While this process can help, it does not always work, and can provide completely unrealistic results also. In this case, you should try different runs, or switch to a 'PCA' approach.

Let's see an example :


```r
realistic.sp <- generateRandomSp(worldclim[[c(1, 5)]],
                                 realistic.sp = TRUE,
                                 convert.to.PA = FALSE)
```

```
##  - Determining species' response to predictor variables
```

```
##  - Calculating species suitability
```

![Fig. 5.6 A species randomly generated, constrained to find realistic environmental requirements](05-randomspecies_files/figure-html/rand6-1.png)

```r
unrealistic.sp <- generateRandomSp(worldclim[[c(1, 5)]],
                                   realistic.sp = FALSE,
                                   convert.to.PA = FALSE)
```

```
##  - Determining species' response to predictor variables
## 
##  - Calculating species suitability
```

![Fig. 5.7 A species randomly generated, with no constraints](05-randomspecies_files/figure-html/rand6-2.png)

Note that you can always let the function randomly determine the conversion to presence-absence by changing the argument `convert.to.PA = TRUE` (be careful: since parameters are defined randomly, the results can be bizarre), or you can do it by yourself later:


```r
realistic.sp <- convertToPA(realistic.sp, beta = 0.5)
```

```
##    --- Determing species.prevalence automatically according to alpha and beta
```

![Fig. 5.8 Modification of the conversion threshold of the previously generated species in fig. 5.6](05-randomspecies_files/figure-html/rand7-1.png)

If, in spite of all your attempts, you are struggling to obtain satisfactory species, then perhaps you should try the 'PCA' approach.

### 5.2.4. Define how response functions are combined to compute the environmental suitability

You can define how response functions are combined with the argument `sp.type`, exactly as explained in the 'response' approach section:

* `species.type = "additive"`: the response functions are added.
* `species.type = "multiplicative"`: the response functions are multiplied (default value). 
* `species.type = "mixed"`: there is a mix of additions and products to calculate the environmental suitability, randomly generated.

## 5.3. Parameters specific to a 'PCA' approach

### 5.3.1. Generate a random species with a PCA approach on a low-memory computer

As explained in the PCA section, if you have low-memory computer, or are working with very large raster files (large extent, fine resolution), then you can still perform the PCA by using a sample of the pixels of your raster.

To do this, set `sample.points = TRUE`, and choose the number of pixels to sample, depending on your computer's memory, with `nb.points`.


```r
# A safe run for a low memory computer
safe.run.sp <- generateRandomSp(worldclim[[c(1, 5, 6)]], 
                                sample.points = TRUE,
                                nb.points = 1000)
```

```
##  - Determining species' response to predictor variables
```

```
##  - Calculating species suitability
```

```
##  - Converting into Presence - Absence
```

```
##    --- Determing species.prevalence automatically according to alpha and beta
```

![Fig 5.9 A virtual species randomly generated from a PCA approach, with a memory-safe procedure](05-randomspecies_files/figure-html/rand8-1.png)

### 5.3.2. Generate stenotopic or eurytopic species

You can modify the `niche.breadth` argument to generate stenotopic (`niche.breadth = 'narrow'`), eurytopic (`niche.breadth = 'wide'`), or any type of species (`niche.breadth = 'any'`).