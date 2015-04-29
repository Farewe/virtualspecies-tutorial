# The virtualspecies R package: a complete tutorial
Boris Leroy - leroy.boris@gmail.com  
May 2015  
\counterwithin{figure}{section}


*******

_Many thanks to **Céline  Bellard** for commenting and correcting this tutorial_

# Introduction

This complete tutorial introduces all the possibilities of the `virtualspecies` package. It was written with the objective to be helpful for both beginners and experienced R users. You can read this tutorial in full or jump to the particular section you are looking for. In each section, you will find simple examples introducing each function, followed by detailed examples for almost every possible parametrisation in `virtualspecies`.


After a small introduction on the spatial data used as input for the `virtualspecies` package (section 1.), you will be introduced to the basics of generating virtual species distributions: the two possible approaches to create species-environment relationships (sections 2. and 3.), followed by the conversion of environmental suitability to presence-absence (section 4.). After that, you will see how to generate random virtual species (section 5.), and most importantly, how to explore, extract, and use the outputs of `virtualspecies` (section 6.). Then, you will see how to sample occurrence points (section 7.). If you are interested, you may also learn about the introduction of distribution limitations in section 8.

In the following sections, you will see how to combine `virtualspecies` with external packages : the species distribution models package biomod (section 9.), and the dispersal limitation package MIGCLIM (section 10. - not yet written).

I will make extensive use of climate data as an example here, but remember that you can use other types of data, as long as it is in the format described in section 1.




*******

# 1. Input data


*******

\setcounter{section}{1}
\setcounter{figure}{0}

`virtualspecies` uses spatialised environmental data as an input. These data must be gridded spatial data in the `RasterStack` format of the package `raster`.

### 1.1. Small introduction to raster data
The input data for virtual species is gridded spatial data (_i.e._, raster data).  
To summarise, a raster is a map regularly divided into small units, called pixels. Each pixel has its own value. These values can be, for example, temperature values, for a raster of temperature.
Raster data can be imported into R using the package `raster`. The command `stack()` will allow you to import your own rasters (stored on your hard drive) into an object of class `RasterStack`. Below you will find some concrete examples.

### 1.2. Required format for virtualspecies
As stated above, the required format is a `RasterStack` containing all the environmental variables with which you want to generate a virtual species distribution. Note that I may use interchangeably the variables and layers in this tutorial, because they roughly refer to the same aspect: a layer is the spatial representation of an environmental variable.

The most important part here is that every layer of your `RasterStack` must be correctly named, because these names will be used when generating virtual species.  
Hence, I strongly advise using explicit names for your layers. You can use "variable1", "variable2", etc. or "layer1", "layer2", etc. but names like "bio1", "bio2", "bio3", ([bioclimatic variable names](http://worldclim.org/bioclim)) or "temp1", "temp2", etc. will reduce confusion.  
You can access the names of the layers of your stack with `names(my.stack)`, and modify them with `names(my.stack) <- c("name1", "name2", etc.)`.


### 1.3. A quick and easy example using WorldClim data
WorldClim (www.worldclim.org) freely provides gridded climate data (temperature and precipitations) for the entire World. 
These data can be downloaded to your hard drive from the website above, and then imported into R using the stack command:
`stack("my.path/MyWorldClimStack.bil")`.  
  
Otherwise, a much simpler solution is to directly download them into R using the function `getData()` (requires an internet connection). 

You have to provide the type of environmental variables you are looking for:  

* tmean = average monthly mean temperature (°C * 10)
* tmin = average monthly minimum temperature (°C * 10)
* tmax = average monthly maximum temperature (°C * 10)
* prec = average monthly precipitation (mm) 
* bio = bioclimatic variables derived from the tmean, tmin, tmax and prec
* alt = altitude (elevation above sea level) (m) (from SRTM)

And the resolution: 0.5, 2.5, 5, or 10 minutes of a degree. Note that at fine resolutions the files downloaded are very heavy and may take a long time; at the finest resolution (0.5) you cannot download the global file, and you have to provide a longitude and latitude to obtain a tile of the world (see `?getData` for help).

Here is the example in practice:


```r
library(raster, quietly = T)
worldclim <- getData("worldclim", var = "bio", res = 10)
worldclim
```

```
## class       : RasterStack 
## dimensions  : 900, 2160, 1944000, 19  (nrow, ncol, ncell, nlayers)
## resolution  : 0.1666667, 0.1666667  (x, y)
## extent      : -180, 180, -60, 90  (xmin, xmax, ymin, ymax)
## coord. ref. : +proj=longlat +datum=WGS84 
## names       : bio1, bio2, bio3, bio4, bio5, bio6, bio7, bio8, bio9, bio10, bio11, bio12, bio13, bio14, bio15, ...
```

You can see that the object is of class `RasterStack`, ready to use for `virtualspecies`. The names of the layers are also convenient.

We can easily access a subset of the layers with `[[]]`:

```r
# The first four layers
worldclim[[1:4]]
```

```
## class       : RasterStack 
## dimensions  : 900, 2160, 1944000, 4  (nrow, ncol, ncell, nlayers)
## resolution  : 0.1666667, 0.1666667  (x, y)
## extent      : -180, 180, -60, 90  (xmin, xmax, ymin, ymax)
## coord. ref. : +proj=longlat +datum=WGS84 
## names       : bio1, bio2, bio3, bio4
```

```r
# Layers bio1 and bio12 (annual mean temperature and annual precipitation)
worldclim[[c("bio1", "bio12")]]
```

```
## class       : RasterStack 
## dimensions  : 900, 2160, 1944000, 2  (nrow, ncol, ncell, nlayers)
## resolution  : 0.1666667, 0.1666667  (x, y)
## extent      : -180, 180, -60, 90  (xmin, xmax, ymin, ymax)
## coord. ref. : +proj=longlat +datum=WGS84 
## names       : bio1, bio12
```

And we can plot our variables:



```r
# Plotting bio1 and bio12
par(oma = c(0.1, 0.1, 0.1, 2.1))
plot(worldclim[[c("bio1", "bio12")]])
```

![Fig. 1.1 Variables bio1 (annual mean temperature in °C * 10) and bio2 (annual precipitation)](virtualspecies-tutorial_files/figure-html/wcex2-1.png) 


*******


# 2. First approach: generate virtual species distributions by defining response functions


*******

\setcounter{section}{2}
\setcounter{figure}{0}

The first approach to generate a virtual species consists in defining its response functions to each of the environmental variables contained in our `RasterStack`. These responses are then combined to calculcate the environmental suitability of the virtual species.

The function providing this approach is `generateSpFromFun`.

## 2.1. An introduction example

Before we start using the package, let's prepare our first simulation of virtual species.

We want to generate a virtual species with two environmental variables, the annual mean temperature `bio1` and annual mean precipitation `bio2`. We want to generate a tropical species, i.e., living in hot and humid environments. We can define bell-shaped response functions to these two variables, as in the following figure:


```
## Loading required package: raster
## Loading required package: sp
```

![Fig. 2.1 Example of bell-shaped response functions to bio1 and bio2, suitable for a tropical species.](virtualspecies-tutorial_files/figure-html/vspload-1.png) 

_Note that bioclimatic temperature variables (here bio1) are is in °C * 10, [as explained here.](http://worldclim.org/bioclim)_

These bell-shaped functions are in fact gaussian distributions functions of the form:

$$ f(x, \mu, \sigma) = \frac{1}{\sigma\sqrt{2\pi}}\, e^{-\frac{(x - \mu)^2}{2 \sigma^2}}$$

If we take the example of bio1 above, the mean $\mu$ is equal to 250 (i.e., 25°C) and standard deviation $\sigma$ is equal to 50 (5°C). Hence the response function for bio1 is:

$$ f(bio1) = \frac{1}{50\sqrt{2\pi}}\, e^{-\frac{(bio1 - 250)^2}{2 \times 50^2}} $$

In R, it is very simple to obtain the result of the equation above, with the function `dnorm`: 


```r
# Suitability of the environment for bio1 = 15 °C
dnorm(x = 150, mean = 250, sd = 50)
```

```
## [1] 0.001079819
```

The idea with `virtualspecies` is to keep the same simplicity when generating a species: we will use the `dnorm` function to generate our virtual species.  

Let's start with the package now:
The first step is to provide to the helper function `formatFunctions` which responses you want for which variables:


```r
library(virtualspecies)

my.parameters <- formatFunctions(bio1 = c(fun = 'dnorm', mean = 250, sd = 50),
                                 bio12 = c(fun = 'dnorm', mean = 4000, sd = 2000))
```

After that, the generation of the virtual species is fairly easy:

```r
# Generation of the virtual species
my.first.species <- generateSpFromFun(raster.stack = worldclim[[c("bio1", "bio12")]],
                                              parameters = my.parameters,
                                              plot = TRUE)
```

![Fig. 2.2 Environmental suitability of the generated virtual species](virtualspecies-tutorial_files/figure-html/resp4-1.png) 

```r
my.first.species
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
```

Congratulations! You have just generated your first virtual species distribution, with the default settings. In the next section you will learn about the simple, yet important settings of this function.

## 2.2. Customisation of parameters

The function `generateSpFromFun` proceeds into four important steps:

1. The response to each environmental variable is calculated according to the user-chosen functions.
2. Each response is rescaled between 0 and 1. This step can be disabled.
3. The responses are combined together to compute the environmental suitability. The user can choose how to combine the responses.
4. The environmental suitability is rescaled between 0 and 1. This step can be disabled.

### 2.2.1. Response functions
You can use any existing function with `generateSpFromFun`, as long as you provide the necessary parameters. 
For example, you can use the function `dnorm` shown above, if you provide the parameters `mean` and `sd`: `formatFunctions(bio1 = c(fun = 'dnorm', mean = 250, sd = 50))`. Naturally you can change the values of `mean` and `sd` to your needs.

You can also use the basic functions provided with the package:

* Linear function: `formatFunctions(bio1 = c(fun = 'linearFun', a = 1, b = 0))`
$$ f(bio1) = a * bio1 + b $$

* Quadratic function: `formatFunctions(bio1 = c(fun = 'quadraticFun', a = 1, b = 2, c = 0))`
$$ f(bio1) = a \times bio1^2 + b \times bio1 + c$$

* Logistic function: `formatFunctions(bio1 = c(fun = 'logisticFun', beta = 150, alpha = -5))`
$$ f(bio1) = \frac{1}{1 + e^{\frac{bio1 - \beta}{\alpha}}} $$

Or you can create your own functions (see the section *How to create your own response functions* if you need help for this).

### 2.2.2. Rescaling of individual response functions

This rescaling is performed because if you use very different response function for different variables, (_e.g._, a Gaussian distribution function with a linear function), then the responses may be disproportionate among variables. By default this rescaling is enabled (`rescale.each.response = TRUE`), but it can be disabled (`rescale.each.response = FALSE`).

### 2.2.3. Combining response functions

There are three main possibilities to combine response functions to calculate the environmental suitability, as defined by the parameters `species.type` and `formula`:

* `species.type = "additive"`: the response functions are added.
* `species.type = "multiplicative"`: the response functions are multiplied. This is the default behaviour of the function.
* `formula = "bio1 + 2 * bio2 + bio3"`: if you choose a formula, then the response functions are combined according to your formula (parameter `species.type` is then ignored).

For example, if you want to generate a species with the same partial responses as in 2.1, but with a strong importance for temperature, then you can specify the formula : `formula = "2 * bio1 + bio12"`


```r
library(virtualspecies)

my.parameters <- formatFunctions(bio1 = c(fun = 'dnorm', mean = 250, sd = 50),
                                 bio12 = c(fun = 'dnorm', mean = 4000, sd = 2000))

# Generation of the virtual species
new.species <- generateSpFromFun(raster.stack = worldclim[[c("bio1", "bio12")]],
                                 parameters = my.parameters,
                                 formula = "2 * bio1 + bio12",
                                 plot = TRUE)
```

```
## [1] "2 * bio1 + bio12"
```

![Fig. 2.3 Environmental suitability of the generated virtual species](virtualspecies-tutorial_files/figure-html/resp5-1.png) 

```r
new.species
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
## - Environmental suitability formula = 2 * bio1 + bio12
## - Environmental suitability was rescaled between 0 and 1
```

One can even make complex interactions between partial responses:


```r
library(virtualspecies)

my.parameters <- formatFunctions(bio1 = c(fun = 'dnorm', mean = 250, sd = 50),
                                 bio12 = c(fun = 'dnorm', mean = 4000, sd = 2000))

# Generation of the virtual species
new.species <- generateSpFromFun(raster.stack = worldclim[[c("bio1", "bio12")]],
                                 parameters = my.parameters,
                                 formula = "3.1 * bio1^2 - 1.4 * sqrt(bio12) * bio1",
                                 plot = TRUE)
```

```
## [1] "3.1 * bio1^2 - 1.4 * sqrt(bio12) * bio1"
```

![Fig. 2.3 Environmental suitability of the generated virtual species](virtualspecies-tutorial_files/figure-html/resp6-1.png) 

```r
new.species
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
## - Environmental suitability formula = 3.1 * bio1^2 - 1.4 * sqrt(bio12) * bio1
## - Environmental suitability was rescaled between 0 and 1
```

Note that this is an example to show the possibilities of the function; I have no idea of the relevance of such a relationship!

### 2.2.4. Rescaling of the final environmental suitability

This final rescaling is performed because the combination of the different responses can lead to very different range of values. It is therefore necessary to allow environmental suitabilities to be comparable among virtual species, and should not be disabled unless you have very precise reasons to do it. The argument `rescale` controls this rescaling (`TRUE` by default).

## 2.3. How to create and use your own response functions

An important aspect of `generateSpFromFun` is that you can create and use your own response functions. In this section we will see how we can do that in practice.  
We will take the example of a simple linear function:
$$ f(x, a, b) = ax + b$$

Our first step will be to create the function in R:


```r
linear.function <- function(x, a, b)
{
  a*x + b
}
```

That's it! You created a function called `linear.function` in R.

Let's try it now:

```r
linear.function(x = 0.5, a = 2, b = 1)
```

```
## [1] 2
```

```r
linear.function(x = 3, a = 4, b = 1)
```

```
## [1] 13
```

```r
linear.function(x = -20, a = 2, b = 0)
```

```
## [1] -40
```

It seems to work properly. Now we will use `linear.function` to generate a virtual species distribution. We want a species responding linearly to the annual mean temperature, and with a gaussian to the annual precipitations:


```r
my.responses <- formatFunctions(bio1 = c(fun = "linear.function", a = 1, b = 0),
                                bio12 = c(fun = "dnorm", mean = 1000, sd = 1000))

generateSpFromFun(raster.stack = worldclim[[c("bio1", "bio12")]],
                  parameters = my.responses, plot = TRUE)
```

![Fig 2.3 Environmental suitability of the generated virtual species](virtualspecies-tutorial_files/figure-html/custfun3-1.png) 

```
## Virtual species generated from 2 variables:
##  bio1, bio12
## 
## - Approach used: Responses to each variable
## - Response functions:
##    .bio1  [min=-269; max=314] : linear.function   (a=1; b=0)
##    .bio12  [min=0; max=9916] : dnorm   (mean=1000; sd=1000)
## - Each response function was rescaled between 0 and 1
## - Environmental suitability formula = bio1 * bio12
## - Environmental suitability was rescaled between 0 and 1
```

And it worked! Your turn now!



*******


# 3. Second approach: generate virtual species with a Principal Components Analysis


*******
\setcounter{section}{3}
\setcounter{figure}{0}

If you try to use the first approach with a large number of variables, _e.g._ 10, then you  will rapidly hit the unextricable problem of unrealistic environmental requirements. When you have 2 environmental variables, it is easy to choose response functions that are compatible. For example, you know that you should not try to generate a species which requires a temperature of the warmest month at 35°C, and a temperature of the coldest month at -25°C, because such conditions are unlikely to exist on Earth. However, if you have 10 variables, the task is much more complicated: if you want to generate a species which is dependent on 5 different temperature variables, 3 precipitation variables, and 2 land use variables, then it is almost impossible to know if your response functions are realistic regarding environmental conditions. 

This is why we implemented the second approach, which consits in generating a Principal Component Analysis (PCA) of all the environmental variables in your `RasterStack`, and then define the response of the species to two axes (pricipal components). Using this approach will greatly help you to generate virtual species which have plausible environmental requirements, whil being dependent on all of your variables.

The function providing this approach is `generateSpFromPCA`.

## 3.1. An introduction example

We want to generate a species dependent on three temperature variables ([bio2, bio5 and bio6](http://worldclim.org/bioclim)) and three precipitation variables ([bio12, bio13, bio14](http://worldclim.org/bioclim) ).


```r
my.stack <- worldclim[[c("bio2", "bio5", "bio6", "bio12", "bio13", "bio14")]]
```

The generation of a virtual species is relatively straightforward:


```r
my.pca.species <- generateSpFromPCA(raster.stack = my.stack)
```

```
##  - Perfoming the pca
## 
##  - Defining the response of the species along PCA axes
## 
##  - Calculating suitability values
```

```r
my.pca.species
```

```
## Virtual species generated from 6 variables:
##  bio2, bio5, bio6, bio12, bio13, bio14
## 
## - Approach used: Response to axes of a PCA
## - Axes:  1 & 2 ;  83.24 % explained by these axes
## - Responses to axes:
##    .Axis 1  [min=-19; max=2.6] : dnorm (mean=0.9394078; sd=3.589539)
##    .Axis 2  [min=-3.44; max=10.95] : dnorm (mean=0.7440176; sd=5.545703)
## - Environmental suitability was rescaled between 0 and 1
```

Something very important to know here is that the PCA is performed on all the pixels of the raster stack. As a consequence, if you use large stacks (large spatial scales, fine resolutions), your computer may not be able to extract all the values. In this case, you can run the PCA on a random subset of values, by setting `sample.points = TRUE` and defining the number of pixels to sample with `nb.points` (default 10000, try less if your computer is struggling). _Note that there is an automatic safety check if you don't set `sample.points = TRUE`, and the function will not if your computer cannot handle it._


```r
# A safe run of the function
my.pca.species <- generateSpFromPCA(raster.stack = my.stack, 
                                    sample.points = TRUE, nb.points = 5000)
```

```
##  - Perfoming the pca
## 
##  - Defining the response of the species along PCA axes
## 
##  - Calculating suitability values
```

```r
my.pca.species
```

```
## Virtual species generated from 6 variables:
##  bio2, bio5, bio6, bio12, bio13, bio14
## 
## - Approach used: Response to axes of a PCA
## - Axes:  1 & 2 ;  83.2 % explained by these axes
## - Responses to axes:
##    .Axis 1  [min=-11.56; max=2.64] : dnorm (mean=1.445674; sd=4.535394)
##    .Axis 2  [min=-3.43; max=8.02] : dnorm (mean=-0.02714928; sd=1.434669)
## - Environmental suitability was rescaled between 0 and 1
```

Congratulations! You have just generated your first virtual species with a PCA. You will probably have noticed that you have not entered any parameter, but the generation has still succeeded. Indeed, when no parameter is entered, the response of the species to PCA axes is randomly generated. The reason behind this choice is that you can hardly choose by yourself the parameters before you have seen the PCA! The next step is therefore to take a look at the PCA, using the function `plotResponse` (note that you can also use the argument `plot = TRUE` in `generateSpFromPCA`)


```r
plotResponse(my.pca.species)
```

![Fig. 3.1 PCA used to generate the virtual species](virtualspecies-tutorial_files/figure-html/pca4-1.png) 


On Fig. 3.1 you can see the PCA of environmental conditions, where each point is the representation of a pixel of your stack in the factorial space. On one of the corners is shown the projection of the variables on this PCA (the position varies for an easier reading, although it is not always perfect). Along each axis, you can see the response of the species: on my example, a wide tolerance to the axis 1, driven mostly by precipitation variables (bio12 and bio13), and an intermediate tolerance to the axis 2, driven mostly by temperature variables (bio2 and bio5). The resulting environmental suitability, as a product of responses to each axis, is illustrated by a coloration of the points, from red (high suitability), to yellow and grey (low/unsuitable pixels).

Now that you have this information in hand, you will be able (in the next section) to define a narrower niche breadth for the species, or to choose a species leaving in hotter, colder, drier or wetter environments. But before that, you probably would like to see how the species' environmental suitability is distributed in space. Nothing's simpler:


```r
plot(my.pca.species)
```

![Fig. 3.2 Environmental suitability of a species generated with a PCA approach](virtualspecies-tutorial_files/figure-html/pca5-1.png) 

## 3.2. Customisation of the parameters

The function `generateSpFromPCA` proceeds into four important steps:
1. The PCA is computed on the basis of the input environmental conditions. You can also provide your own PCA.
2. The Gaussian responses to axes are randomly chosen (only if you did not provide precise parameters) and then computed.
3. The environmental suitability is calculated as a product of the responses to both axes.
4. The environmental suitability is rescaled between 0 and 1. This step can be disabled.


### 3.2.1. Customisation of the responses to axes

_First of all, note that at the moment, the choice is restricted to axes 1 and 2, and to the usage of gaussian functions. The reason behind that is the automatic random generation of parameters, which would become very difficult to automatise for numerous different functions. However, an upcoming version of this function is under developement, which should allow to use different axes and custom functions in the future. If you are interested to get it very soon please e-mail me._

You can customise the Gaussian response functions in two different ways:

1. You can constrain the random generation of parameters by choosing either a narrow-niche or a broad-niche species. To do this, specify the argument `niche.breadth = 'narrow'` or `niche.breadth = 'wide'`. By default this argument is set to `niche.breadth = 'any'`, meaning that you can obtain species with any niche breadth.   
This argument controls the standard deviation of the gaussian distribution. The full details about this is available in the help of the function (`?generateSpFromPCA`)


```r
narrow.species <- generateSpFromPCA(raster.stack = my.stack, sample.points = TRUE,
                                    nb.points = 5000,
                                    niche.breadth = "narrow")
```

```
##  - Perfoming the pca
## 
##  - Defining the response of the species along PCA axes
## 
##  - Calculating suitability values
```


```r
plotResponse(narrow.species)
```

![Fig. 3.3 PCA of a species generated with rather narrow niche breadth](virtualspecies-tutorial_files/figure-html/pca6b-1.png) 


```r
plot(narrow.species)
```

![Fig. 3.4 Environmental suitability of a species generated with rather narrow niche breadth](virtualspecies-tutorial_files/figure-html/pca6c-1.png) 


```r
wide.species <- generateSpFromPCA(raster.stack = my.stack, sample.points = TRUE,
                                    nb.points = 5000,
                                    niche.breadth = "wide")
```

```
##  - Perfoming the pca
## 
##  - Defining the response of the species along PCA axes
## 
##  - Calculating suitability values
```


```r
plotResponse(wide.species)
```

![Fig. 3.5 PCA of a species generated with rather wide niche breadth](virtualspecies-tutorial_files/figure-html/pca7b-1.png) 


```r
plot(wide.species)
```

![Fig. 3.6 Environmental suitability of a species generated with rather wide niche breadth](virtualspecies-tutorial_files/figure-html/pca7c-1.png) 


2. You can define by yourself the mean and standard deviations of the Gaussian responses. To do this, use arguments `means` and `sds` as in the following example.  
Using the figure above, we know that the first axis is driven by precipitation variables, and the second one by temperature variables. To define a species living in colder and wetter environments, we will move the means of Gaussian responses to the right of the first axis (_e.g._, a mean of 0 or above) and to the top of the second axis (_e.g._, a mean of 1 or above). In addition, if we want our species to be stenotopic (narrow niche breadth), we will also define lower standard deviations (_e.g._, standard deviations of 0.25). The correct input will be : `means = c(0, 1)` (a mean of 0 for the first axis and 1 for the second) and `sds = c(0.25, 0.5)` (standard deviations of 0.25 for axes 1 and 2):


```r
my.custom.species <- generateSpFromPCA(raster.stack = my.stack, sample.points = TRUE,
                                       nb.points = 5000,
                                       means = c(0, 1), sds = c(0.5, 0.5))
```

```
##  - Perfoming the pca
## 
##  - Defining the response of the species along PCA axes
## 
##     - You have provided standard deviations, so argument niche.breadth will be ignored.
## 
##  - Calculating suitability values
```


```r
plotResponse(my.custom.species)
```

![Fig. 3.7 PCA of the species generated with custom responses to axes](virtualspecies-tutorial_files/figure-html/pca8b-1.png) 


```r
plot(my.custom.species)
```

![Fig. 3.8 Environmental suitability of the species generated with custom responses to axes](virtualspecies-tutorial_files/figure-html/pca8c-1.png) 

### 3.2.2. Rescaling of the final environmental suitability

The final rescaling is performed for the same reasons as in `generateSpFromFun`. It should not be disabled unless you have very precise reasons to do it. The argument `rescale` controls this rescaling (`TRUE` by default).

### 3.2.3. Using a custom PCA

It is possible, if you need, to use your own PCA. In that case, make sure that the PCA was performed with the function `dudi.pca` of the package [`ade4`](http://cran.r-project.org/web/packages/ade4/index.html). You also need to perform the PCA on the same variables as in your `RasterStack`, entered in the **exact same order**.

One reason why you would want to use a custom PCA is because you have a struggling computer, which requires to generate a PCA from a very reduced subset of your environmental stack (_e.g._, `generateSpFromPCA(my.stack, sample.points = TRUE, nb.points = 250)`). In such a case, the PCA may vary substantially among runs, precluding any tentative to finely customise the responses to axes. It is easy to extract the PCA from a previous run, and use it in the next run(s):


```r
my.first.run <- generateSpFromPCA(raster.stack = my.stack, 
                                  sample.points = TRUE, nb.points = 250)
```

```
##  - Perfoming the pca
## 
##  - Defining the response of the species along PCA axes
## 
##  - Calculating suitability values
```

```r
# You can access the PCA with the following command
my.pca <- my.first.run$details$pca

# And then provide it to your second run
my.second.run <- generateSpFromPCA(raster.stack = my.stack, 
                                   pca = my.pca)
```

```
##  - Defining the response of the species along PCA axes
## 
##  - Calculating suitability values
```


*******


# 4. Conversion of environmental suitability to presence-absence


*******

\setcounter{section}{4}
\setcounter{figure}{0}


## 4.1. Introduction: why should we avoid a threshold-conversion?
What we did so far was defining the relationship between the species and its environment. It was the most important part, but what it provides us is a spatial distribution of the species' environmental suitability, not a spatial distribution of the species' presences/absences. Hence we now have to convert the environmental suitability into presence-absence.

The simplest approach, which was also the most widely used until 2014, consists in defining a threshold of environmental suitability, below which conditions are unsuitable, so absence is attributed; and above which conditions are suitable, so persence is attributed. However, you should completely avoid this approach which is *pure evil*:

* Most importantly, this creates completely unrealistic species:
    + Real species are often absent from areas of high suitability, because of factors acting at smaller spatial scales, such as biotic pressure (competition, predation), disturbances, stochastic events.
    + Real species often occur in unsuitable areas, because of very particular conditions allowing their occurrence (microclimatic/microhabitat conditions, climatic refugia).
* The threshold almost completely removes the previously defined relationship between the species and its environment. The gradual aspect is lost: the above-threshold part of the environmental gradient is always fully suitable, while the below-threshold part is always fully unsuitable.

So, how can we convert our environmental suitability into presence-absence without a threshold? 

We use a probabilistic approach: the probability of getting a presence of the species in a given pixel is dependent on its suitability in that pixel:

![Fig. 4.1 Logistic curve used for a probabilistic conversion into presence-absence](virtualspecies-tutorial_files/figure-html/conv1-1.png) 

With the example above, a pixel with environmental suitability equal to 0.6 has 88% chance of having species presence, and 12% chance of having species absence. 

Of course, this conversion is fully customisable, to simulate different cases:


![Fig. 4.2 Contrasting examples of conversion curves and their result on the distribution range of the same virtual species.](virtualspecies-tutorial_files/figure-html/conv2-1.png) 

The logistic conversion looks much more like what a real species distribution is than the linear and threshold-like conversions.

In the next section we will see how to perform the conversion, and how to customise it.

## 4.2. Customisation of the conversion

At first you may be interested to see the function in action, before we try to customise it. Here it is!


```r
pa1 <- convertToPA(my.first.species, plot = TRUE)
```

![Fig. 4.3 Maps of environmental suitability and presence-absence of the virtual species](virtualspecies-tutorial_files/figure-html/conv3-1.png) 

You have probably noticed that the conversion was performed without you choosing any parameter. This is because by default, the function randomly assigns parameters to the conversion. What are these parameters?

They are the parameters $\alpha$ and $\beta$ which determine the shape of the logistic curve:

* $\beta$ controls the inflexion point,

![Fig. 4.4 Impact of beta on the conversion curve](virtualspecies-tutorial_files/figure-html/conv4-1.png) 

* and $\alpha$ drives the 'slope' of the curve

![Fig. 4.4 Impact of alpha on the conversion curve](virtualspecies-tutorial_files/figure-html/conv5-1.png) 


The parameters are fairly simple to customise:



```r
pa2 <- convertToPA(my.first.species,
                   beta = 0.65, alpha = -0.07,
                   plot = TRUE)
```

![Fig. 4.5 Conversion with beta = 0.65, alpha = -0.07](virtualspecies-tutorial_files/figure-html/conv6-1.png) 


```r
# You can modify the conversion of your object if you did not like it:
pa2 <- convertToPA(pa2,
                   beta = 0.3, alpha = -0.02,
                   plot = TRUE)
```

![Fig. 4.5 Conversion with beta = 0.65, alpha = -0.07](virtualspecies-tutorial_files/figure-html/conv7-1.png) 

In the next sections I summarise how to choose appropriate values of `alpha` and `beta`, and also I introduce a third parameter which may be very useful to generate virtual species distributions: the species prevalance (i.e., the number of places occupied by the species, out of the total number of available places). 


### 4.2.1. beta

`beta` is very simple to grasp, as it is the inflexion point of the curve. Hence, looking at the three beta curves above, we can see that a lower `beta` will increase the probability of finding suitable conditions for the species (wider distribution range). A higher `beta` will decrease the probability of finding suitable conditions (smaller distribution range).

### 4.2.2. alpha

`alpha` may be more difficult to grasp, as it is dependent on the range of values of the `x` axis (in our case, the environmental suitability, ranging from 0 to 1):

* If `alpha` is approximately equal to the range of`x` pr greater (in absolute value), then the conversion will have a linear shape. In our case, it means  values of `alpha` close to -1 or below).
* If `alpha` is about 5-10% of the range of `x`, then the conversion will be logistic. In our case, you can have nice logistic curves for values of `alpha` between -0.1 and -0.01.
* If `alpha` is much smaller compared to `x` (in absolute value), then the conversion will be threshold-like. In our case, if means values of `alpha` in the range [-0.001, 0[.

### 4.2.3. Conversion to presence-absence based on a value of species prevalence

_The species prevalence is the number of places (here, pixels) actually occupied by the species out of the total number of places (pixels) available. Do not confuse the **SPECIES PREVALENCE** with the [**SAMPLE PREVALENCE**](#sample-prevalence), which in turn is the proportion of samples in which you have found the species._

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

![Fig. 4.6 Conversion of a species with a prevalence of 0.2, _i.e._ occupying 20% of the world](virtualspecies-tutorial_files/figure-html/conv8-1.png) 

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

![Fig. 4.7 Conversion of a species with a prevalence of 0.015, _i.e._ occupying 1.5% of the world](virtualspecies-tutorial_files/figure-html/conv9-1.png) 

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
## - Converted into presence-absence:
##    .Method = probability
##    .alpha (slope)           = -0.015
##    .beta  (inflexion point) = 0.75
##    .species prevalence      = 0.024
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

![Fig. 4.8 Conversion of a species with a prevalence of 0.1, _i.e._ occupying 10% of the world](virtualspecies-tutorial_files/figure-html/conv10-1.png) 

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
## - Converted into presence-absence:
##    .Method = probability
##    .alpha (slope)           = -0.31346875
##    .beta  (inflexion point) = 0.9
##    .species prevalence      = 0.092
```
It worked, but the resulting species does not look realistic


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

![Fig. 4.9 Conversion of a species whose asked prevalence (0.1) cannot be reached because of a too low value of beta](virtualspecies-tutorial_files/figure-html/conv11-1.png) 

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
## - Converted into presence-absence:
##    .Method = probability
##    .alpha (slope)           = -0.001
##    .beta  (inflexion point) = 0.3
##    .species prevalence      = 0.147
```


********


# 5. Generating random virtual species



********

\setcounter{section}{5}
\setcounter{figure}{0}


The `virtualspecies` package embeds a function to randomly generate virtual species `generateRandomSp`, with many customisable options to constrain the randomisation process.

Let's take a look:


```r
my.stack <- worldclim[[c("bio2", "bio5", "bio6", "bio12", "bio13", "bio14")]]
random.sp <- generateRandomSp(my.stack)
```

```
##  - Perfoming the pca
## 
##  - Defining the response of the species along PCA axes
## 
##  - Calculating suitability values
## 
##  - Converting into Presence - Absence
## 
##    --- Determing species.prevalence automatically according to alpha and beta
```

![Fig. 5.1 A species randomly generated with `generateRandomSp`](virtualspecies-tutorial_files/figure-html/rand1-1.png) 

```r
random.sp
```

```
## Virtual species generated from 6 variables:
##  bio2, bio5, bio6, bio12, bio13, bio14
## 
## - Approach used: Response to axes of a PCA
## - Axes:  1 & 2 ;  83.24 % explained by these axes
## - Responses to axes:
##    .Axis 1  [min=-19; max=2.6] : dnorm (mean=1.63459; sd=5.21778)
##    .Axis 2  [min=-3.44; max=10.95] : dnorm (mean=0.6151591; sd=0.4546485)
## - Environmental suitability was rescaled between 0 and 1
## - Converted into presence-absence:
##    .Method = probability
##    .alpha (slope)           = -0.1
##    .beta  (inflexion point) = 0.363363363363363
##    .species prevalence      = 0.354
```

We can see that the species was generated using a PCA approach. Indeed, [as explained in the PCA section](#PCA.approach), when you have a lot of variables, it becomes very difficult to generate a species with realistic environmental requirements. Hence, by default the function `generateRandomSp` uses a PCA approach if you have 6 or more variables, and a 'response functions' approach if you have less than 6 variables.

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
                              relations = c("gaussian", "logistic"))
```

```
##  - Determining species' response to predictor variables
## 
##  - Calculating species suitability
## 
##  - Converting into Presence - Absence
## 
##    --- Determing species.prevalence automatically according to alpha and beta
```

![Fig. 5.2 A species randomly generated with `generateRandomSp`, with gaussian and logistic response functions](virtualspecies-tutorial_files/figure-html/rand2-1.png) 

```r
random.sp1
```

```
## Virtual species generated from 3 variables:
##  bio1, bio2, bio3
## 
## - Approach used: Responses to each variable
## - Response functions:
##    .bio1  [min=-269; max=314] : logisticFun   (alpha=7.42; beta=272.035035035035)
##    .bio2  [min=9; max=211] : dnorm   (mean=29.1372813728137; sd=90.6383063830638)
##    .bio3  [min=8; max=95] : dnorm   (mean=61.8418184181842; sd=74.9286292862929)
## - Each response function was rescaled between 0 and 1
## - Environmental suitability formula = bio1 * bio2 * bio3
## - Environmental suitability was rescaled between 0 and 1
## - Converted into presence-absence:
##    .Method = probability
##    .alpha (slope)           = -0.1
##    .beta  (inflexion point) = 0.977977977977978
##    .species prevalence      = 0.009
```


```r
plotResponse(random.sp1)
```

![Response functions of the randomly generated species](virtualspecies-tutorial_files/figure-html/rand3-1.png) 



```r
# A purely gaussian species
random.sp2 <- generateRandomSp(worldclim[[1:3]], 
                              relations = "gaussian")
```

```
##  - Determining species' response to predictor variables
## 
##  - Calculating species suitability
## 
##  - Converting into Presence - Absence
## 
##    --- Determing species.prevalence automatically according to alpha and beta
```

![Fig. 5.4 A species randomly generated with `generateRandomSp`, with gaussian response functions only](virtualspecies-tutorial_files/figure-html/rand4-1.png) 

```r
random.sp2
```

```
## Virtual species generated from 3 variables:
##  bio2, bio3, bio1
## 
## - Approach used: Responses to each variable
## - Response functions:
##    .bio2  [min=9; max=211] : dnorm   (mean=48.1560715607156; sd=86.9457094570946)
##    .bio3  [min=8; max=95] : dnorm   (mean=57.8707787077871; sd=75.520235202352)
##    .bio1  [min=-269; max=314] : dnorm   (mean=104.237902379024; sd=5.51523515235152)
## - Each response function was rescaled between 0 and 1
## - Environmental suitability formula = bio1 * bio2 * bio3
## - Environmental suitability was rescaled between 0 and 1
## - Converted into presence-absence:
##    .Method = probability
##    .alpha (slope)           = -0.1
##    .beta  (inflexion point) = 0.664664664664665
##    .species prevalence      = 0.005
```


```r
plotResponse(random.sp2)
```

![Fig. 5.5 Response functions of the randomly generated species](virtualspecies-tutorial_files/figure-html/rand5-1.png) 


### 5.2.2. Rescale individual response functions

As explained in the section on the 'response' approach, you can choose to rescale or not each individual response function, with the argument `rescale.each.response`. `TRUE` by default.

### 5.2.3. Try to find a realistic species

An important issue with the generation of random responses to the environment, is that you can obtain a species willing to live in summer temperatures of 35°C and winter temperature of -50°C. This may interesting for generating species from another planet, but you are probably more interested in generating species that can actually live on Earth. There is therefore an option to do that, activated by default: `realistic.sp`.

When activating this argument, the function will proceed step-by-step to try defining a realistic species. At step one, one of the variable is chosen, and the program randomly determines a response function for this variable. Then, it will compute the environmental suitability of this species. At step two, the program will pick a second variable, and will constrain its random generation depending on the environmental suitability obtained at step one. For example, if at step 2 a gaussian response is picked, then the mean will be chosen in areas where the species had a high environmental suitability. Then, the environmental suitability is recalculated on the basis of the first two response functions. At step 3, another variable is picked, a response function randomly generated with respect to areas where the species already has a high suitability, and so on until there are no variables left. While this process helps, it does not always work, especially when you have a larger number of variables.

Let's see an example :


```r
realistic.sp <- generateRandomSp(worldclim[[c(1, 5)]],
                                 realistic.sp = TRUE)
```

```
##  - Determining species' response to predictor variables
## 
##  - Calculating species suitability
## 
##  - Converting into Presence - Absence
## 
##    --- Determing species.prevalence automatically according to alpha and beta
```

![Fig. 5.6 A species randomly generated, constrained to find realistic environmental requirements](virtualspecies-tutorial_files/figure-html/rand6-1.png) 

```r
unrealistic.sp <- generateRandomSp(worldclim[[c(1, 5)]],
                                   realistic.sp = FALSE)
```

```
##  - Determining species' response to predictor variables
## 
##  - Calculating species suitability
## 
##  - Converting into Presence - Absence
## 
##    --- Determing species.prevalence automatically according to alpha and beta
```

![Fig. 5.7 A species randomly generated, with no constraints](virtualspecies-tutorial_files/figure-html/rand6-2.png) 

Note that you can always change the conversion to presence-absence while keeping the same species-environment relationships:


```r
realistic.sp <- convertToPA(realistic.sp, beta = 0.5)
```

```
##    --- Determing species.prevalence automatically according to alpha and beta
```

![Fig. 5.8 Modification of the conversion threshold of the previously generated species in fig. 5.6](virtualspecies-tutorial_files/figure-html/rand7-1.png) 

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
## 
##  - Calculating species suitability
## 
##  - Converting into Presence - Absence
## 
##    --- Determing species.prevalence automatically according to alpha and beta
```

![Fig 5.9 A virtual species randomly generated from a PCA approach, with a memory-safe procedure](virtualspecies-tutorial_files/figure-html/rand8-1.png) 

### 5.3.2. Generate stenotopic or eurytopic species

You can modify the `niche.breadth` argument to generate stenotopic (`niche.breadth = 'narrow'`), eurytopic (`niche.breadth = 'wide'`), or any type of species (`niche.breadth = 'any'`).


********


# 6. Exploring and using the outputs of virtualspecies 



********

\setcounter{section}{6}
\setcounter{figure}{0}


This section is mainly intended to users not very familiar with R. For example, if you are not sure how to obtain the maps of generated virtual species, read this section. If you simply want to extract (sample) occurrence points for your virtual species, then you should jump to the next section.


## 6.1. Consult the details of a generated virtual species

Let's create a simple virtual species:

```r
random.sp1 <- generateRandomSp(raster.stack = worldclim[[c("bio2", "bio5")]],
                               relations = "gaussian")
```

```
##  - Determining species' response to predictor variables
## 
##  - Calculating species suitability
## 
##  - Converting into Presence - Absence
## 
##    --- Determing species.prevalence automatically according to alpha and beta
```

![6.1 Automatic illustration of the randomly generated species](virtualspecies-tutorial_files/figure-html/output1-1.png) 

If we want to know how it was generated, we simply type the object name in the R console:


```r
random.sp1
```

```
## Virtual species generated from 2 variables:
##  bio5, bio2
## 
## - Approach used: Responses to each variable
## - Response functions:
##    .bio5  [min=-59; max=489] : dnorm   (mean=192.967439674397; sd=48.7341273412734)
##    .bio2  [min=9; max=211] : dnorm   (mean=5.61687616876169; sd=15.1117711177112)
## - Each response function was rescaled between 0 and 1
## - Environmental suitability formula = bio2 * bio5
## - Environmental suitability was rescaled between 0 and 1
## - Converted into presence-absence:
##    .Method = probability
##    .alpha (slope)           = -0.1
##    .beta  (inflexion point) = 0.652652652652653
##    .species prevalence      = 0.001
```

And a summary of how the virtual species was generated appears:

* It shows us the variables used.
* It shows us the approach used and all the details of the approach, so we can use it to reconstruct another virtual species with the exact same parameters later on. It also provides us the range of values of our environmental variables (bio1 (mean annual temperature) ranged from -269 (-26.9°C) to 314 (31.4°C)). This is helpful to quickly get an idea of the preferences of our species; for example here we see that we have a species living in hot environments, with a peak at 250 (25°C).
* If a conversion to presence-absence was performed, it shows us the parameters of the conversion, and provides the species prevalence (the species prevalence is always calculated and provided).
* If you have introduced a distribution bias (will be seen in a later section), it will provide information about this particular bias.


## 6.2. Plot the virtual species map

Plotting the distribution maps of a virtual species is straightforward:

```r
plot(random.sp1)
```

![6.2 Maps obtained when using the function `plot()` on virtual species objects](virtualspecies-tutorial_files/figure-html/output3-1.png) 

If the environmental sutiability has been converted into presence-absence, then the plot will conveniently display both the environmental suitability and the presence-absence map.

## 6.3. Plot the species-environment relationship

As illustrated several times in this tutorial, there is a function to automatically generate an appropriate plot for your virtual species: `plotResponse`


```r
plotResponse(random.sp1)
```

![6.3 Example of figure obtained when running `plotResponse()` on a virtual species object](virtualspecies-tutorial_files/figure-html/output3bis-1.png) 

## 6.4. Extracting elements of the virtual species, such as the rasters of environmental suitability

The virtual species object is structured as a `list` in R, which roughly means that it is an object containing many "sub-objects". When you run functions on your virtual species object, such as the conversion into presence-absence, then new sub-objects are added or replaced in the list.

There is a function allowing you to see the content of the list: `str()`


```r
str(random.sp1)
```

```
## List of 5
##  $ approach     : chr "response"
##  $ details      :List of 5
##   ..$ variables            : chr [1:2] "bio5" "bio2"
##   ..$ formula              : chr "bio2 * bio5"
##   ..$ rescale.each.response: logi TRUE
##   ..$ rescale              : logi TRUE
##   ..$ parameters           :List of 2
##  $ suitab.raster:Formal class 'RasterLayer' [package "raster"] with 12 slots
##  $ PA.conversion: Named chr [1:4] "probability" "-0.1" "0.652652652652653" "0.001"
##   ..- attr(*, "names")= chr [1:4] "conversion.method" "alpha" "beta" "species.prevalence"
##  $ pa.raster    :Formal class 'RasterLayer' [package "raster"] with 12 slots
##  - attr(*, "class")= chr [1:2] "list" "virtualspecies"
```

We are informed that the object is a `list` containing 5 elements (sub-objects), that you can read on the lines starting with a `$`: `approach`, `details`, `suitab.raster`, `PA.conversion` and `pa.raster`.

You can extract each element using the `$`: for example, to extract the suitability raster, type


```r
random.sp1$suitab.raster
```

```
## class       : RasterLayer 
## dimensions  : 900, 2160, 1944000  (nrow, ncol, ncell)
## resolution  : 0.1666667, 0.1666667  (x, y)
## extent      : -180, 180, -60, 90  (xmin, xmax, ymin, ymax)
## coord. ref. : +proj=longlat +datum=WGS84 
## data source : in memory
## names       : layer 
## values      : 0, 1  (min, max)
```

If you are interested in the presence-absence raster, type


```r
random.sp1$pa.raster
```

```
## class       : RasterLayer 
## dimensions  : 900, 2160, 1944000  (nrow, ncol, ncell)
## resolution  : 0.1666667, 0.1666667  (x, y)
## extent      : -180, 180, -60, 90  (xmin, xmax, ymin, ymax)
## coord. ref. : +proj=longlat +datum=WGS84 
## data source : in memory
## names       : layer 
## values      : 0, 1  (min, max)
```


You can also see that we have "sub-sub-objects", in the lines starting with `..$`: these are objects contained within the sub-object `details`. You can also extract them easily:


```r
random.sp1$details$variables
```

```
## [1] "bio5" "bio2"
```

However, the sub-sub-sub-objects (level 3 of depth and beyond) are not listed when you use `str()` on your virtual species object. For example, if we extract the `parameters` object from the details, we can see that it contains all the function names and their parameters:


```r
random.sp1$details$parameters
```

```
## $bio5
## $bio5$fun
## [1] "dnorm"
## 
## $bio5$args
## $bio5$args$mean
## [1] 192.9674
## 
## $bio5$args$sd
## [1] 48.73413
## 
## 
## $bio5$min
## [1] -59
## 
## $bio5$max
## [1] 489
## 
## 
## $bio2
## $bio2$fun
## [1] "dnorm"
## 
## $bio2$args
## $bio2$args$mean
## [1] 5.616876
## 
## $bio2$args$sd
## [1] 15.11177
## 
## 
## $bio2$min
## [1] 9
## 
## $bio2$max
## [1] 211
```

```r
# Looking at how it is structured:
str(random.sp1$details$parameters)
```

```
## List of 2
##  $ bio5:List of 4
##   ..$ fun : chr "dnorm"
##   ..$ args:List of 2
##   .. ..$ mean: num 193
##   .. ..$ sd  : num 48.7
##   ..$ min : num -59
##   ..$ max : num 489
##  $ bio2:List of 4
##   ..$ fun : chr "dnorm"
##   ..$ args:List of 2
##   .. ..$ mean: num 5.62
##   .. ..$ sd  : num 15.1
##   ..$ min : num 9
##   ..$ max : num 211
```


Hence, the main message here is if you want to explore the content of the virtual species object, use the function `str()`, look at which sub-objects you are interested in, and extract them with `$`. 



## 6.5. Saving the virtual species objects for later use


If you want to save a virtual species object, you can save it on your hard drive, using the R function `save()`:


```r
save(random.sp1, file = "MyVirtualSpecies")
```

You can load it in a later session of R, using `load()`:


```r
load(file = "MyVirtualSpecies")
```

The object will be restored in the memory of R, with its original name.


*******


# 7. Sampling occurrence points


*******

\setcounter{section}{7}
\setcounter{figure}{0}


## 7.1. Basic usage

If you generated virtual species distributions with this package, there is a good chance that your objective is to test a particular modelling protocol or technique. Hence, there is one last step for you to perform: the sampling of species occurrences. This can be done with the function `sampleOccurrences`, with which you can sample either "presence-absence" or "presence only" occurrence data. The function `sampleOccurrences` also provides the possibility to introduce a number of sampling biases, such as uneven spatial sampling intensity, probability of detection, and probability of error.

Let's see an example in practice, using the same tropical species we generated in the beginning of this tutorial:

```r
# Formatting of the response functions
my.parameters <- formatFunctions(bio1 = c(fun = 'dnorm', mean = 250, sd = 50),
                                 bio12 = c(fun = 'dnorm', mean = 4000, sd = 2000))

# Generation of the virtual species
my.first.species <- generateSpFromFun(raster.stack = worldclim[[c("bio1", "bio12")]],
                                      parameters = my.parameters)

# Conversion to presence-absence
my.first.species <- convertToPA(my.first.species,
                                beta = 0.7, plot = FALSE)
```

```
##    --- Determing species.prevalence automatically according to alpha and beta
```


```r
# Sampling of 'presence only' occurrences
presence.points <- sampleOccurrences(my.first.species,
                                     n = 30, # The number of points to sample
                                     type = "presence only")
```

![Fig 7.1 Illustration of the points sampled for our virtual species](virtualspecies-tutorial_files/figure-html/samp1b-1.png) 

We can see that a map was plotted, with the sampled presence points illustrated with black dots. 

Now let's see how our object `presence.points` looks like:


```r
str(presence.points)
```

```
## List of 3
##  $ sample.points        :'data.frame':	30 obs. of  4 variables:
##   ..$ x       : num [1:30] 9.92 123.25 138.75 111.25 -8.42 ...
##   ..$ y       : num [1:30] 1.08 13.75 -5.75 2.75 4.92 ...
##   ..$ Real    : num [1:30] 1 1 1 1 1 1 1 1 1 1 ...
##   ..$ Observed: num [1:30] 1 1 1 1 1 1 1 1 1 1 ...
##  $ detection.probability: num 1
##  $ error.probability    : num 0
##  - attr(*, "class")= chr [1:2] "list" "VSSampledPoints"
```

This is a list containing three elements: 

1. `sampled.points`: a data frame containing the sampled points
2. `detection.probability` : the detection probability of our virtual species
3. `error.probability`: the probability of an erroneous detection of our virtual species

We can specifically access the data frame of sampled points:


```r
presence.points$sampled.points
```

```
## NULL
```

The data frame has four colums: the pixel coordinates `x` and `y`, the real presence/absence (1/0) of the species in the sampled pixel `Real`, and the result of the sampling `Observed` (1 = presence, 0 = absence, NA = no information). The columns `Real` and `Observed` differ only if you have specified a detection probability and/or an error probability.

Now, let's sample the other type of occurrence data: presence-absence.



```r
# Sampling of 'presence-absence' occurrences
PA.points <- sampleOccurrences(my.first.species,
                               n = 30,
                               type = "presence-absence")
```

![Fig 7.2 Illustration of the presence-absence points sampled for our virtual species. Black dots are presences, open dots are absences.](virtualspecies-tutorial_files/figure-html/samp4-1.png) 

```r
PA.points
```

```
## $sample.points
##              x          y Real Observed
## 1   134.750000 -21.083333    0        0
## 2   171.416667 -44.250000    0        0
## 3   102.083333  40.083333    0        0
## 4    22.250000 -14.750000    0        0
## 5  -121.750000  66.916667    0        0
## 6    15.250000  23.916667    0        0
## 7    44.583333   7.583333    0        0
## 8   -55.916667 -22.083333    0        0
## 9   -52.750000  -2.583333    0        0
## 10   56.083333  73.083333    0        0
## 11   35.083333  49.583333    0        0
## 12   28.416667  -6.916667    0        0
## 13 -106.916667  69.416667    0        0
## 14   81.416667  63.583333    0        0
## 15  159.416667  67.416667    0        0
## 16  -86.083333  41.250000    0        0
## 17   36.916667  54.250000    0        0
## 18  -92.250000  69.583333    0        0
## 19    3.083333  51.083333    0        0
## 20   36.083333   8.916667    0        0
## 21  -54.916667 -15.583333    0        0
## 22  116.583333  46.083333    0        0
## 23  106.750000  23.416667    0        0
## 24    7.250000  35.250000    0        0
## 25   33.916667   4.750000    0        0
## 26  132.083333 -11.083333    0        0
## 27   18.250000  -6.750000    0        0
## 28  -83.916667  32.583333    0        0
## 29   93.750000  19.583333    1        1
## 30  100.583333  35.250000    0        0
## 
## $detection.probability
## [1] 1
## 
## $error.probability
## [1] 0
## 
## $sample.prevalence
##     true.sample.prevalence observed.sample.prevalence 
##                 0.03333333                 0.03333333 
## 
## attr(,"class")
## [1] "list"            "VSSampledPoints"
```

You can see that the points are sampled randomly throughout the whole raster. In the next section you will see how to limit the sampling to a region in particular, and after that, how to introduce a bias in the sampling procedure.

## 7.2. Delimiting a sampling area

There are three main possibilities to delimit the sampling areas :

1. Specifying the region(s) of the world (country, continent, region
2. Provide a polygon (of type `SpatialPolygons` or `SpatialPolygonsDataFrame` of package `sp`)
3. Provide an `extent` object (a rectangular area, from the package `raster`)

### 7.2.1. Specifying the region(s) of the world

You can specify any combination of countries, continents and regions of the world to restrict your sampling area, using the argument `sampling.area`. For example:


```r
# Sampling of 'presence-absence' occurrences
PA.points <- sampleOccurrences(my.first.species,
                               n = 30,
                               type = "presence-absence",
                               sampling.area = c("South America", "Mexico"))
```

![Fig 7.3 Illustration of points sampled in a restricted area (here, South America + Mexico)](virtualspecies-tutorial_files/figure-html/samp5-1.png) 



The only restriction is to provide the correct names.

The correct region names are: "Africa", "Antarctica", "Asia", "Australia", "Europe", "North America", "South America"

The correct continent names are: "Africa", "Antarctica", "Australia", "Eurasia", "North America", "South America"

And you can consult the correct names with the following commands:


```r
library(rworldmap, quiet = TRUE)
```

```
## ### Welcome to rworldmap ###
## For a short introduction type : 	 vignette('rworldmap')
```

```r
# Country names
levels(getMap()@data$SOVEREIGNT)
```

```
##   [1] "Afghanistan"                      "Albania"                         
##   [3] "Algeria"                          "Andorra"                         
##   [5] "Angola"                           "Antarctica"                      
##   [7] "Antigua and Barbuda"              "Argentina"                       
##   [9] "Armenia"                          "Australia"                       
##  [11] "Austria"                          "Azerbaijan"                      
##  [13] "Bahrain"                          "Bangladesh"                      
##  [15] "Barbados"                         "Belarus"                         
##  [17] "Belgium"                          "Belize"                          
##  [19] "Benin"                            "Bhutan"                          
##  [21] "Bolivia"                          "Bosnia and Herzegovina"          
##  [23] "Botswana"                         "Brazil"                          
##  [25] "Brunei"                           "Bulgaria"                        
##  [27] "Burkina Faso"                     "Burundi"                         
##  [29] "Cambodia"                         "Cameroon"                        
##  [31] "Canada"                           "Cape Verde"                      
##  [33] "Central African Republic"         "Chad"                            
##  [35] "Chile"                            "China"                           
##  [37] "Colombia"                         "Comoros"                         
##  [39] "Costa Rica"                       "Croatia"                         
##  [41] "Cuba"                             "Cyprus"                          
##  [43] "Czech Republic"                   "Democratic Republic of the Congo"
##  [45] "Denmark"                          "Djibouti"                        
##  [47] "Dominica"                         "Dominican Republic"              
##  [49] "East Timor"                       "Ecuador"                         
##  [51] "Egypt"                            "El Salvador"                     
##  [53] "Equatorial Guinea"                "Eritrea"                         
##  [55] "Estonia"                          "Ethiopia"                        
##  [57] "Federated States of Micronesia"   "Fiji"                            
##  [59] "Finland"                          "France"                          
##  [61] "Gabon"                            "Gambia"                          
##  [63] "Georgia"                          "Germany"                         
##  [65] "Ghana"                            "Greece"                          
##  [67] "Grenada"                          "Guatemala"                       
##  [69] "Guinea"                           "Guinea Bissau"                   
##  [71] "Guyana"                           "Haiti"                           
##  [73] "Honduras"                         "Hungary"                         
##  [75] "Iceland"                          "India"                           
##  [77] "Indonesia"                        "Iran"                            
##  [79] "Iraq"                             "Ireland"                         
##  [81] "Israel"                           "Italy"                           
##  [83] "Ivory Coast"                      "Jamaica"                         
##  [85] "Japan"                            "Jordan"                          
##  [87] "Kashmir"                          "Kazakhstan"                      
##  [89] "Kenya"                            "Kiribati"                        
##  [91] "Kosovo"                           "Kuwait"                          
##  [93] "Kyrgyzstan"                       "Laos"                            
##  [95] "Latvia"                           "Lebanon"                         
##  [97] "Lesotho"                          "Liberia"                         
##  [99] "Libya"                            "Liechtenstein"                   
## [101] "Lithuania"                        "Luxembourg"                      
## [103] "Macedonia"                        "Madagascar"                      
## [105] "Malawi"                           "Malaysia"                        
## [107] "Maldives"                         "Mali"                            
## [109] "Malta"                            "Marshall Islands"                
## [111] "Mauritania"                       "Mauritius"                       
## [113] "Mexico"                           "Moldova"                         
## [115] "Monaco"                           "Mongolia"                        
## [117] "Montenegro"                       "Morocco"                         
## [119] "Mozambique"                       "Myanmar"                         
## [121] "Namibia"                          "Nauru"                           
## [123] "Nepal"                            "Netherlands"                     
## [125] "New Zealand"                      "Nicaragua"                       
## [127] "Niger"                            "Nigeria"                         
## [129] "North Korea"                      "Northern Cyprus"                 
## [131] "Norway"                           "Oman"                            
## [133] "Pakistan"                         "Palau"                           
## [135] "Palestine"                        "Panama"                          
## [137] "Papua New Guinea"                 "Paraguay"                        
## [139] "Peru"                             "Philippines"                     
## [141] "Poland"                           "Portugal"                        
## [143] "Qatar"                            "Republic of Serbia"              
## [145] "Republic of the Congo"            "Romania"                         
## [147] "Russia"                           "Rwanda"                          
## [149] "Saint Kitts and Nevis"            "Saint Lucia"                     
## [151] "Saint Vincent and the Grenadines" "Samoa"                           
## [153] "San Marino"                       "Sao Tome and Principe"           
## [155] "Saudi Arabia"                     "Senegal"                         
## [157] "Seychelles"                       "Sierra Leone"                    
## [159] "Singapore"                        "Slovakia"                        
## [161] "Slovenia"                         "Solomon Islands"                 
## [163] "Somalia"                          "Somaliland"                      
## [165] "South Africa"                     "South Korea"                     
## [167] "South Sudan"                      "Spain"                           
## [169] "Sri Lanka"                        "Sudan"                           
## [171] "Suriname"                         "Swaziland"                       
## [173] "Sweden"                           "Switzerland"                     
## [175] "Syria"                            "Taiwan"                          
## [177] "Tajikistan"                       "Thailand"                        
## [179] "The Bahamas"                      "Togo"                            
## [181] "Tonga"                            "Trinidad and Tobago"             
## [183] "Tunisia"                          "Turkey"                          
## [185] "Turkmenistan"                     "Tuvalu"                          
## [187] "Uganda"                           "Ukraine"                         
## [189] "United Arab Emirates"             "United Kingdom"                  
## [191] "United Republic of Tanzania"      "United States of America"        
## [193] "Uruguay"                          "Uzbekistan"                      
## [195] "Vanuatu"                          "Vatican"                         
## [197] "Venezuela"                        "Vietnam"                         
## [199] "Western Sahara"                   "Yemen"                           
## [201] "Zambia"                           "Zimbabwe"
```

### 7.2.2. Providing a polygon

In this case you have to import in R your own polygon, using commands of the package `sp`. The polygon has to be of type `SpatialPolygons` or `SpatialPolygonsDataFrame`. 

We will use here the example of a `SpatialPolygonsDataFrame` which we can download straight into R using the function `getData()` of the package `raster`. Since our species is a tropical species, let's assume we are working in Brasil only. Remember that this is an example, and that you can import your own polygons into R.

First, let's download our polygon:


```r
brasil <- getData("GADM", country = "BRA", level = 0)
```

![Fig 7.4 A polygon of the brazil](virtualspecies-tutorial_files/figure-html/samp7-1.png) 

Now, to limit the sampling area to our polygon, we simply provide it to the argument `sampling.area` of the function `sampleOccurrences`:


```r
PA.points <- sampleOccurrences(my.first.species,
                               n = 30,
                               type = "presence-absence",
                               sampling.area = brasil)
```

![Fig 7.5 Points sampled within the Brazil polygon](virtualspecies-tutorial_files/figure-html/samp8-1.png) 

```r
PA.points
```

```
## $sample.points
##            x           y Real Observed
## 1  -55.08333 -17.2500000    0        0
## 2  -49.08333 -17.5833333    0        0
## 3  -57.08333 -13.2500000    0        0
## 4  -41.58333 -19.9166667    0        0
## 5  -45.25000  -5.0833333    0        0
## 6  -48.58333 -17.4166667    0        0
## 7  -58.25000  -8.5833333    0        0
## 8  -47.75000  -1.2500000    1        1
## 9  -69.41667  -8.4166667    0        0
## 10 -43.25000 -19.5833333    0        0
## 11 -46.75000 -20.4166667    0        0
## 12 -53.75000  -8.7500000    1        1
## 13 -62.25000   0.7500000    0        0
## 14 -38.25000  -9.4166667    0        0
## 15 -66.75000  -0.7500000    1        1
## 16 -72.41667  -9.4166667    0        0
## 17 -54.58333  -4.0833333    0        0
## 18 -39.08333 -10.9166667    0        0
## 19 -64.08333   0.9166667    0        0
## 20 -53.08333 -11.2500000    0        0
## 21 -54.91667 -15.0833333    0        0
## 22 -42.08333  -7.0833333    0        0
## 23 -57.08333  -4.0833333    0        0
## 24 -53.41667 -15.0833333    0        0
## 25 -60.08333 -14.0833333    0        0
## 26 -62.25000 -11.9166667    0        0
## 27 -71.41667  -8.7500000    0        0
## 28 -43.41667  -6.4166667    0        0
## 29 -55.41667   1.9166667    0        0
## 30 -51.75000 -24.7500000    0        0
## 
## $detection.probability
## [1] 1
## 
## $error.probability
## [1] 0
## 
## $sample.prevalence
##     true.sample.prevalence observed.sample.prevalence 
##                        0.1                        0.1 
## 
## attr(,"class")
## [1] "list"            "VSSampledPoints"
```

It worked!

However, the plot is not very convenient because it kept the scale of our input raster, so we may be interested in manually restricting the plot region to Brasil:


```r
# First we get our data frame of occurrence points
occ <- PA.points$sample.points

plot(my.first.species$pa.raster, 
     xlim = c(-80, -20),
     ylim = c(-35, 5))
plot(brasil, add = TRUE)
points(occ[occ$Observed == 1, c("x", "y")], pch = 16, cex = .8)
points(occ[occ$Observed == 0, c("x", "y")], pch = 1, cex = .8)
```

![Fig 7.6 Points sampled within the Brazil polygon, with a suitable zoom](virtualspecies-tutorial_files/figure-html/samp9-1.png) 

### 7.2.3. Providing an extent object

An extent object is a rectangular area defined by four coordinates xmin/xmax/ymin/ymax. You can easily create an extent object using the command `extent`, of the package `raster`:


```r
my.extent <- extent(-80, -20, -35, -5)
PA.points <- sampleOccurrences(my.first.species,
                               n = 30,
                               type = "presence-absence",
                               sampling.area = my.extent)
plot(my.extent, add = TRUE)
```

![Fig 7.7 Points sampled within our extent object](virtualspecies-tutorial_files/figure-html/samp10-1.png) 

## 7.3. Introducing a sampling bias

### 7.3.1.Detection probability

Detection probability has been shown to be influencial on SDM performance. Hence, you might be interested in generating species with varying degrees of detection probability. This can be done easily, using the argument `detection.probability`, ranging from 0 (species cannot be detected) to 1 (species is always detected):


```r
# Let's try to find samples of a very cryptic species
PA.points <- sampleOccurrences(my.first.species,
                               sampling.area = "Brazil",
                               n = 50,
                               type = "presence-absence",
                               detection.probability = 0.3)
```

![Fig 7.8 Presence-absence points sampled with a low detection probability](virtualspecies-tutorial_files/figure-html/samp11-1.png) 

```r
PA.points
```

```
## $sample.points
##            x           y Real Observed
## 1  -71.25000  -9.7500000    0        0
## 2  -40.08333  -8.5833333    0        0
## 3  -52.41667 -20.7500000    0        0
## 4  -44.75000 -20.7500000    0        0
## 5  -39.41667 -10.7500000    0        0
## 6  -58.75000 -11.0833333    0        0
## 7  -41.41667 -10.0833333    0        0
## 8  -56.25000 -16.9166667    0        0
## 9  -50.41667 -15.4166667    0        0
## 10 -42.41667  -8.9166667    0        0
## 11 -48.41667 -11.0833333    0        0
## 12 -48.91667 -15.0833333    0        0
## 13 -54.25000  -7.9166667    1        0
## 14 -52.41667 -16.4166667    0        0
## 15 -56.75000 -29.9166667    0        0
## 16 -46.41667  -4.9166667    0        0
## 17 -67.08333  -9.4166667    0        0
## 18 -71.08333  -4.9166667    0        0
## 19 -58.75000  -0.4166667    0        0
## 20 -51.08333  -1.0833333    0        0
## 21 -51.58333 -19.0833333    0        0
## 22 -60.91667  -6.2500000    1        1
## 23 -73.41667  -7.4166667    1        0
## 24 -52.91667 -31.9166667    0        0
## 25 -49.25000  -7.2500000    1        0
## 26 -44.75000  -9.0833333    0        0
## 27 -60.08333   0.7500000    0        0
## 28 -63.25000  -1.7500000    0        0
## 29 -61.25000  -0.5833333    0        0
## 30 -53.91667 -16.5833333    0        0
## 31 -42.25000  -7.0833333    0        0
## 32 -42.08333  -9.2500000    0        0
## 33 -59.08333  -6.0833333    0        0
## 34 -51.91667 -29.4166667    0        0
## 35 -63.41667  -7.2500000    0        0
## 36 -43.08333 -18.7500000    0        0
## 37 -47.91667 -18.2500000    0        0
## 38 -53.41667 -10.7500000    0        0
## 39 -67.91667  -0.7500000    1        0
## 40 -52.41667  -2.7500000    0        0
## 41 -64.08333  -6.5833333    0        0
## 42 -60.58333  -9.7500000    0        0
## 43 -45.25000 -16.2500000    0        0
## 44 -61.91667  -4.5833333    1        1
## 45 -51.58333  -5.9166667    0        0
## 46 -57.58333  -5.7500000    0        0
## 47 -53.25000  -3.5833333    1        1
## 48 -64.41667  -1.4166667    1        0
## 49 -49.91667 -25.2500000    0        0
## 50 -54.75000 -11.9166667    0        0
## 
## $detection.probability
## [1] 0.3
## 
## $error.probability
## [1] 0
## 
## $sample.prevalence
##     true.sample.prevalence observed.sample.prevalence 
##                       0.16                       0.06 
## 
## attr(,"class")
## [1] "list"            "VSSampledPoints"
```

You can see that some points sampled in its distribution range are classified as "absences".

If you use this argument when sampling 'presence only' occurrences, then this will result in a lower number of sampled points than asked:


```r
PO.points <- sampleOccurrences(my.first.species,
                               sampling.area = "Brazil",
                               n = 50,
                               type = "presence only",
                               detection.probability = 0.3)
```

![Fig 7.9 Presence only points sampled with a low detection probability](virtualspecies-tutorial_files/figure-html/samp12-1.png) 

```r
PO.points
```

```
## $sample.points
##            x            y Real Observed
## 1  -63.91667  -1.58333333    1       NA
## 2  -60.25000  -5.25000000    1       NA
## 3  -53.91667  -8.91666667    1       NA
## 4  -62.75000  -3.75000000    1       NA
## 5  -47.91667  -2.08333333    1       NA
## 6  -57.25000  -3.91666667    1       NA
## 7  -62.58333  -3.08333333    1        1
## 8  -53.25000  -0.08333333    1       NA
## 9  -67.91667  -2.75000000    1       NA
## 10 -64.08333   1.58333333    1        1
## 11 -70.25000  -7.41666667    1       NA
## 12 -64.75000  -6.58333333    1       NA
## 13 -59.08333  -1.08333333    1       NA
## 14 -72.75000  -6.91666667    1       NA
## 15 -51.58333  -1.08333333    1       NA
## 16 -49.91667  -4.25000000    1       NA
## 17 -67.41667  -2.41666667    1       NA
## 18 -57.08333  -4.25000000    1       NA
## 19 -51.75000   3.58333333    1        1
## 20 -50.58333   1.91666667    1       NA
## 21 -51.75000  -8.08333333    1       NA
## 22 -64.41667  -6.08333333    1       NA
## 23 -65.25000  -2.58333333    1        1
## 24 -66.75000  -7.41666667    1        1
## 25 -51.58333   2.41666667    1       NA
## 26 -64.25000   3.75000000    1        1
## 27 -63.91667  -1.25000000    1       NA
## 28 -51.75000   2.41666667    1       NA
## 29 -52.91667 -12.58333333    1       NA
## 30 -55.58333  -9.41666667    1       NA
## 31 -72.25000  -8.91666667    1       NA
## 32 -72.91667  -7.58333333    1        1
## 33 -58.75000  -8.25000000    1       NA
## 34 -65.75000  -3.08333333    1        1
## 35 -69.25000  -4.75000000    1        1
## 36 -59.08333  -5.25000000    1        1
## 37 -63.75000   1.91666667    1        1
## 38 -65.75000  -1.25000000    1       NA
## 39 -59.08333  -2.75000000    1        1
## 40 -69.41667  -2.75000000    1       NA
## 41 -68.08333  -6.25000000    1       NA
## 42 -48.25000  -2.25000000    1        1
## 43 -58.25000  -1.08333333    1       NA
## 44 -63.58333  -9.75000000    1        1
## 45 -49.58333 -15.75000000    1       NA
## 46 -64.91667  -0.91666667    1       NA
## 47 -66.58333  -5.08333333    1        1
## 48 -67.41667   0.08333333    1        1
## 49 -67.91667  -6.25000000    1        1
## 50 -47.41667  -0.91666667    1       NA
## 
## $detection.probability
## [1] 0.3
## 
## $error.probability
## [1] 0
## 
## attr(,"class")
## [1] "list"            "VSSampledPoints"
```

In this case, we see that out of all the possible sampling points, only a fraction are sampled as presence points.

### 7.3.2. Detection probability as a function of environmental suitability

You can further complexify the detection probability by making it dependent on the environmental suitability. In this case, cells will be weighted by the environmental suitability: less suitable cells will have a lesser chance of detecting the species. This can be seen as an impact on the population size: a higher environmental suitability increases the population size, and thus the detection probability. How does this work in practice?

The initial probability of detection ($P_{di}$) is multiplied by the environmental suitability ($S$) to obtain the final probability of detection ($P_d$):

$$P_d = P_{di} \times S$$

Hence, if our species has a probability of detection of 0.5, the final probability of detection will be:

* 1 if the cell has an environmental suitability of 1
* 0.25 if the cell has an environmental suitability of 0.5

Of course, no occurrence points will be detected outside the distribution range, regardless of their environmental suitability.

You can also keep the detection probability to 1, and set `correct.by.suitabilaty = TRUE`. In that case, the detection probability will be equal to the environmental suitability. This can be seen as a species whose detection probability is strictly dependent on its population size, which in turn is strictly dependent on the environmental suitability.

An example in practice:


```r
PA.points <- sampleOccurrences(my.first.species,
                               sampling.area = "Brazil",
                               n = 100,
                               type = "presence-absence",
                               detection.probability = 1,
                               correct.by.suitability = TRUE,
                               plot = FALSE)

# Below is a custom plot to show the sampled point above the map of environmental suitability
occ <- PA.points$sample.points

plot(my.first.species$suitab.raster, 
     xlim = c(-80, -20),
     ylim = c(-35, 5))
plot(brasil, add = TRUE)
points(occ[occ$Observed == 1, c("x", "y")], pch = 16, cex = .8)
points(occ[occ$Observed == 0, c("x", "y")], pch = 1, cex = .8)
```

![Fig 7.10 Presence-absence points sampled with a detection probability equal to the environmental suitability](virtualspecies-tutorial_files/figure-html/samp13-1.png) 


### 7.3.3. Error probability

You can also introduce an error probability, which is a probability of finding the species where it is absent. This is straightforward with the argument `error.probability`:


```r
PA.points <- sampleOccurrences(my.first.species,
                               sampling.area = "Brazil",
                               n = 20,
                               type = "presence-absence",
                               error.probability = 0.3)
```

![Fig 7.11 Presence-absence points sampled with an error probability of 0.3](virtualspecies-tutorial_files/figure-html/samp14-1.png) 

```r
PA.points
```

```
## $sample.points
##            x           y Real Observed
## 1  -65.75000  -4.4166667    1        1
## 2  -49.75000  -5.7500000    0        0
## 3  -70.25000  -7.5833333    1        1
## 4  -41.08333 -12.5833333    0        1
## 5  -51.58333   1.5833333    1        1
## 6  -56.58333 -12.2500000    1        1
## 7  -44.08333 -13.9166667    0        1
## 8  -50.25000 -23.7500000    0        0
## 9  -54.75000 -17.4166667    0        1
## 10 -60.08333  -9.4166667    0        1
## 11 -55.75000  -1.7500000    1        1
## 12 -71.75000  -9.5833333    0        0
## 13 -46.91667 -22.4166667    0        0
## 14 -63.41667 -10.9166667    1        1
## 15 -54.75000  -4.7500000    0        0
## 16 -45.08333  -6.2500000    0        0
## 17 -47.25000  -0.9166667    1        1
## 18 -56.58333 -13.4166667    0        0
## 19 -42.41667 -21.4166667    0        0
## 20 -56.58333  -0.5833333    1        1
## 
## $detection.probability
## [1] 1
## 
## $error.probability
## [1] 0.3
## 
## $sample.prevalence
##     true.sample.prevalence observed.sample.prevalence 
##                        0.4                        0.6 
## 
## attr(,"class")
## [1] "list"            "VSSampledPoints"
```

Two important remarks with the error probability:

1. The error probability is useless in a 'presence only' sampling scheme, because the sampling strictly occurs within the boundaries of the species distribution range. Nevertheless, **if you still want to introduce errors**, then make a sampling scheme 'presence-absence', and use only the 'presence' points obtained.

2. **There is an interaction between the detection probability and the error probability**: in a cell where the species is present, but not detected, a presence can still be attributed because of an error. See the following (extreme) example, for a species with a low detection probability and high error probability:


```r
PA.points <- sampleOccurrences(my.first.species,
                               sampling.area = "Brazil",
                               n = 20,
                               type = "presence-absence",
                               detection.probability = 0.2,
                               error.probability = 0.8)
```

![Fig 7.12 Presence-absence points sampled with a detection probability of 0.2, and an error probability of 0.8. In numerous pixels the species was not detected, but erroneous presences were nevertheless attributed.](virtualspecies-tutorial_files/figure-html/samp15-1.png) 

```r
PA.points
```

```
## $sample.points
##            x          y Real Observed
## 1  -43.91667  -7.583333    0        1
## 2  -48.25000  -6.583333    0        1
## 3  -45.75000 -22.916667    0        1
## 4  -60.41667   4.250000    0        0
## 5  -50.91667 -11.750000    0        1
## 6  -70.41667  -8.416667    0        1
## 7  -41.08333 -17.583333    0        1
## 8  -52.91667  -6.416667    0        0
## 9  -49.58333 -27.416667    0        1
## 10 -54.41667 -16.583333    0        1
## 11 -46.25000  -5.916667    0        1
## 12 -50.08333 -14.750000    0        1
## 13 -42.41667 -15.250000    0        1
## 14 -50.75000 -29.916667    0        0
## 15 -38.75000 -11.083333    0        1
## 16 -69.25000  -3.916667    0        0
## 17 -53.41667 -30.250000    0        1
## 18 -46.58333 -10.583333    0        0
## 19 -48.41667  -7.583333    0        1
## 20 -41.91667  -9.083333    0        0
## 
## $detection.probability
## [1] 0.2
## 
## $error.probability
## [1] 0.8
## 
## $sample.prevalence
##     true.sample.prevalence observed.sample.prevalence 
##                        0.0                        0.7 
## 
## attr(,"class")
## [1] "list"            "VSSampledPoints"
```

### 7.3.4. Uneven sampling intensity

The `sampleOccurrences` also allows you to introduce a sampling bias such as uneven sampling intensity in space. How does it works?

You will have to define a region in which the sampling is biased, with arguments `bias` and `area`. In this region, the sampling will be biased by a strength equal to the argument `bias.strength`. If `bias.strength` is equal to 50 for example, then the sampling will be 50 times more intense in the chosen area. If `bias.strength`is below 1, then the sampling will be less intense in the biased area than elsewhere.

Let's see an example in practice before we go into the details:


```r
PO.points <- sampleOccurrences(my.first.species,
                               n = 100, 
                               bias = "region",
                               bias.strength = 20,
                               bias.area = "South America")
```

![Fig 7.13 Presence only points sampled with a strong bias in South America (20 times more sample)](virtualspecies-tutorial_files/figure-html/samp16-1.png) 

Now that you have grasped how the bias works, let's see the different possibilities:

* Introducing a bias using country, region or continent names  
Set either `bias = "country"`, `bias = "region"` or `bias = "continent"`, and provide to `bias.area` the name(s) of the country(-ies), region(s) or continent(s).


```r
PO.points <- sampleOccurrences(my.first.species,
                               n = 100, 
                               bias = "country",
                               bias.strength = 20,
                               bias.area = c("Mexico", "Colombia"))
```

![Fig 7.14 Presence only points sampled with a strong bias in Mexico and Colombia](virtualspecies-tutorial_files/figure-html/samp17-1.png) 

* Using a polygon in which the sampling will be biased  
Set `bias = "polygon"`, and provide a polygon (of type `SpatialPolygons` or `SpatialPolygonsDataFrame` from package `sp`) to the argument `bias.area`.


```r
philippines <- getData("GADM", country = "PHL", level = 0)
PO.points <- sampleOccurrences(my.first.species,
                               n = 100, 
                               bias = "polygon",
                               bias.strength = 50,
                               bias.area = philippines)
```

```
## Warning in sampleOccurrences(my.first.species, n = 100, bias = "polygon",
## : Polygon projection is not checked. Please make sure you have the same
## projections between your polygon and your presence-absence raster
```

![Fig 7.15 Presence only points sampled with strong bias for the Philippines, using a polygon](virtualspecies-tutorial_files/figure-html/samp18-1.png) 

* Using an extent object  
Set `bias = "extent"`, and provide an extent to the argument `bias.area` ([see section 7.2.3. if you are not familiar with extents](#providing-an-extent-object)). You can also simply set `bias.area = polygon`, and click twice on the map when asked to:


```r
PO.points <- sampleOccurrences(my.first.species,
                               n = 100, 
                               bias = "extent",
                               bias.strength = 50)
```

* Manually defining weights for all the cells  
A last option is to manually define the weights with which the sampling will be biased. This may be especially useful when you want to precisely create a sampling bias. To do this, set `bias = "manual"`, and provide a raster of weights to the argument `weights`.  
To clear things up, we will see an example together:


```r
library(rworldmap)
# First, we create a raster of weights
# As an arbitrary example, we will use the area of each cell as a weight:
# larger cells have more chance of being sampled
weight.raster <- area(worldclim)

plot(weight.raster)
plot(getMap(), add = TRUE)
```

![Fig 7.16 Raster used to weight the sampling bias: a raster of cell areas](virtualspecies-tutorial_files/figure-html/samp20-1.png) 

Now, let's weight our samplings using this raster:


```r
PO.points <- sampleOccurrences(my.first.species,
                               n = 100, 
                               bias = "manual",
                               weights = weight.raster)
```

![Fig 7.16 Presence only points sampled with a sampling biased according to the area of each cell](virtualspecies-tutorial_files/figure-html/samp21-1.png) 

Of course, the changes are not  visible, because our species occurs mainly in pixels of relatively similar areas. Nevertheless, this example should be useful if you want to create a manual sampling bias later on.



**Important note** Please note that when using rasters based on longitude-latitude, larger cells have _automatically_ more chance of being sampled than smaller cells, so you do not need to do it by yourself. See documentation of the `randomPoints` function of package `dismo` for more details.


### 7.4. Defining the sample prevalence

_The sample prevalence is the proportion of samples in which the species has been found. It is therefore different from the species prevalence which [was mentionned earlier](#Conversion-to-presence-absence-based-on-a-value-of-species-prevalence)._

You can define the desired sample prevalence when sampling presences and absences, with the parameter `sample.prevalence`:


```r
PA.points <- sampleOccurrences(my.first.species,
                               n = 30,
                               type = "presence-absence",
                               sample.prevalence = 0.5)
```

![Fig 7.17 Presence-absence points sampled with a prevalence of 0.5 (i.e., 50% of presence points, 50% of absence points)](virtualspecies-tutorial_files/figure-html/samp22-1.png) 

```r
PA.points
```

```
## $sample.points
##              x           y Real Observed
## 1   109.916667  -7.5833333    1        1
## 2   -53.083333   0.9166667    1        1
## 3   -68.583333  -3.5833333    1        1
## 4   145.416667  -5.0833333    1        1
## 5   -72.583333   5.2500000    1        1
## 6   -60.750000   7.7500000    1        1
## 7   102.250000   5.5833333    1        1
## 8   -70.083333  -4.0833333    1        1
## 9   153.750000   5.2500000    1        1
## 10    8.916667   3.7500000    1        1
## 11  -64.583333  -4.5833333    1        1
## 12  -51.750000  -6.0833333    1        1
## 13  -66.583333  -7.5833333    1        1
## 14  150.583333  -8.4166667    1        1
## 15  133.250000  -1.5833333    1        1
## 16  -67.250000 -33.4166667    0        0
## 17  127.583333  39.0833333    0        0
## 18 -112.916667  34.9166667    0        0
## 19 -117.750000  34.2500000    0        0
## 20   44.583333  19.7500000    0        0
## 21   22.916667 -15.0833333    0        0
## 22   28.750000   9.4166667    0        0
## 23   34.250000  68.2500000    0        0
## 24   75.583333  41.4166667    0        0
## 25  133.416667  69.0833333    0        0
## 26  127.583333  69.9166667    0        0
## 27  -40.750000  -3.7500000    0        0
## 28  -36.250000  80.0833333    0        0
## 29   37.416667  26.9166667    0        0
## 30 -105.416667  59.2500000    0        0
## 
## $detection.probability
## [1] 1
## 
## $error.probability
## [1] 0
## 
## $sample.prevalence
##     true.sample.prevalence observed.sample.prevalence 
##                        0.5                        0.5 
## 
## attr(,"class")
## [1] "list"            "VSSampledPoints"
```

*******

# 8. Introduction a distribution bias

*******

\setcounter{section}{8}
\setcounter{figure}{0}

One of the most disputed assumptions of species distribution models is that species are at equilibrium with their environment. This assumptions means that species are supposed to occupy their full range of suitable environmental conditions. In reality, it is unlikely, because of dispersal limitations, biotic interactions, etc., which precludes species to occupy areas which are theoretically suitable. As a consequence, it is worth testing how well modelling techniques perform when this assumption is violated.

This is why we introduced the possibility of biasing the distribution of species, to simulate species which are not at equilibrium. This possibility is implemented in the function `limitDistribution`.

## 8.1. An introduction example

Let's use the same virtual species we generated above:

```r
# Formatting of the response functions
my.parameters <- formatFunctions(bio1 = c(fun = 'dnorm', mean = 250, sd = 50),
                                 bio12 = c(fun = 'dnorm', mean = 4000, sd = 2000))

# Generation of the virtual species
my.first.species <- generateSpFromFun(raster.stack = worldclim[[c("bio1", "bio12")]],
                                      parameters = my.parameters)

# Conversion to presence-absence
my.first.species <- convertToPA(my.first.species,
                                beta = 0.7, plot = FALSE)
```

```
##    --- Determing species.prevalence automatically according to alpha and beta
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

![Fig. 8.2 Distribution of a species limited to South America](virtualspecies-tutorial_files/figure-html/dist2-1.png) 

In the following sections, we see how to customise this function, but [the usage is basically the same as when applying a sampling bias.](#uneven-sampling-intensity)  
  
.  
.  
.           
.               
.           
.               
.           
.               
.           
.               
.           
.               


## 8.2. Customisation of the parameters

There are 5 main possibilities to limit the distribution to a particular area.

### 8.2.1. Using countries, regions or continents

As illustrated in the example above, use `geographical.limit = "country"`, `geographical.limit = continent` or `geographical.limit = "region"` and provide the correct name(s) of the area to `area`


```r
my.sp1 <- limitDistribution(my.first.species, 
                            geographical.limit = "country",
                            area = c("Brazil", "Venezuela"))
```

![Fig. 8.3 Distribution of a species arbitrary limited to Brazil and Venezuela](virtualspecies-tutorial_files/figure-html/dist3-1.png) 


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

![Fig. 8.4 Distribution of a species limited to the Philippines](virtualspecies-tutorial_files/figure-html/dist4-1.png) 

* Using an extent object  
Set `geographical.limit = "extent"`, and provide an extent to the argument `area` ([see section 7.2.3. if you are not familiar with extents](#providing-an-extent-object)). You can also simply set `geographical.limit = "extent"`, and click twice on the map when asked to:


```r
my.extent <- extent(-80, -20, -35, -5)
my.sp2 <- limitDistribution(my.first.species, 
                            geographical.limit = "extent",
                            area = my.extent)
plot(my.extent, add = TRUE)
```

![Fig. 8.5 Distribution of a species limited to a particular extent](virtualspecies-tutorial_files/figure-html/dist5-1.png) 


*******

# 9. Using virtual species with the SDM package biomod (not yet written)


This section is not yet written; it should be soon.

*******

# 10. Creating dispersal limitations with the MIGCLIM R package (not yet written)

This section is not yet written; it should be soon.
