---
title: '1. Input data'
output:
  html_document:
    fig_caption: yes
    keep_md: yes
    mathjax: "http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"
    theme: united
    toc: true
    toc_float: true
---
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
## coord. ref. : +proj=longlat +datum=WGS84 +ellps=WGS84 +towgs84=0,0,0 
## names       :  bio1,  bio2,  bio3,  bio4,  bio5,  bio6,  bio7,  bio8,  bio9, bio10, bio11, bio12, bio13, bio14, bio15, ... 
## min values  :  -269,     9,     8,    72,   -59,  -547,    53,  -251,  -450,   -97,  -488,     0,     0,     0,     0, ... 
## max values  :   314,   211,    95, 22673,   489,   258,   725,   375,   364,   380,   289,  9916,  2088,   652,   261, ...
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
## coord. ref. : +proj=longlat +datum=WGS84 +ellps=WGS84 +towgs84=0,0,0 
## names       :  bio1,  bio2,  bio3,  bio4 
## min values  :  -269,     9,     8,    72 
## max values  :   314,   211,    95, 22673
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
## coord. ref. : +proj=longlat +datum=WGS84 +ellps=WGS84 +towgs84=0,0,0 
## names       : bio1, bio12 
## min values  : -269,     0 
## max values  :  314,  9916
```

And we can plot our variables:



```r
# Plotting bio1 and bio12
par(oma = c(0.1, 0.1, 0.1, 2.1))
plot(worldclim[[c("bio1", "bio12")]])
```

![Fig. 1.1 Variables bio1 (annual mean temperature in °C * 10) and bio2 (annual precipitation)](01-inputdata_files/figure-html/wcex2-1.png)
