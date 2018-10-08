---
title: '7. Sampling occurrences'
output:
  html_document:
    fig_caption: yes
    keep_md: yes
    mathjax: "http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"
    theme: united
    toc: true
    toc_float: true
---

## 7.1. Basic usage

If you generated virtual species distributions with this package, there is a good chance that your objective is to test a particular modelling protocol or technique. Hence, there is one last step for you to perform: the sampling of species occurrences. This can be done with the function `sampleOccurrences`, with which you can sample either "presence-absence" or "presence only" occurrence data. The function `sampleOccurrences` also provides the possibility to introduce a number of sampling biases, such as uneven spatial sampling intensity, probability of detection, and probability of error.

Let's see an example in practice, using the same tropical species we generated in the beginning of this tutorial:

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

![Fig 7.1 Illustration of the points sampled for our virtual species](07-sampleoccurrences_files/figure-html/samp1b-1.png)

We can see that a map was plotted, with the sampled presence points illustrated with black dots. 

Now let's see how our object `presence.points` looks like:


```r
presence.points
```

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
## 1  -15.25000  10.916667    1        1
## 2  121.91667  19.416667    1        1
## 3  -64.58333   1.083333    1        1
## 4  118.08333   6.250000    1        1
## 5  105.91667  -2.583333    1        1
## 6  -48.41667  -0.750000    1        1
## 7  -70.08333  -5.083333    1        1
## 8  179.41667 -17.916667    1        1
## 9  -70.25000   4.083333    1        1
## 10 110.41667  -0.750000    1        1
## ... 20 more lines.
```
This message summarizes us information about the 30 occurrence points we just sampled.
Now let's look at the structure of this object:


```r
str(presence.points)
```

```
## List of 8
##  $ type                        : chr "presence only"
##  $ detection.probability       :List of 2
##   ..$ detection.probability : num 1
##   ..$ correct.by.suitability: logi FALSE
##  $ error.probability           : num 0
##  $ bias                        : NULL
##  $ replacement                 : logi FALSE
##  $ original.distribution.raster:Formal class 'RasterLayer' [package "raster"] with 12 slots
##  $ sample.plot                 :List of 3
##   ..$ :Dotted pair list of 23
##   ..$ : raw [1:35992] 00 00 00 00 ...
##   .. ..- attr(*, "pkgName")= chr "graphics"
##   ..$ :List of 2
##   .. ..- attr(*, "pkgName")= chr "grid"
##   ..- attr(*, "engineVersion")= int 12
##   ..- attr(*, "pid")= int 10032
##   ..- attr(*, "Rversion")=Classes 'R_system_version', 'package_version', 'numeric_version'  hidden list of 1
##   ..- attr(*, "load")= chr(0) 
##   ..- attr(*, "attach")= chr(0) 
##   ..- attr(*, "class")= chr "recordedplot"
##  $ sample.points               :'data.frame':	30 obs. of  4 variables:
##   ..$ x       : num [1:30] -15.2 121.9 -64.6 118.1 105.9 ...
##   ..$ y       : num [1:30] 10.92 19.42 1.08 6.25 -2.58 ...
##   ..$ Real    : num [1:30] 1 1 1 1 1 1 1 1 1 1 ...
##   ..$ Observed: num [1:30] 1 1 1 1 1 1 1 1 1 1 ...
##  - attr(*, "RNGkind")= chr [1:2] "Mersenne-Twister" "Inversion"
##  - attr(*, "seed")= int [1:626] 403 457 1875301826 895817535 -1098775620 320110277 -45531433 813313575 466806938 726079098 ...
##  - attr(*, "class")= chr [1:2] "VSSampledPoints" "list"
```



This is a list containing eight elements: 

1. `type`: the type of occurrence sampled (presence absence or presnece-only)
2. `detection.probability`: the detection probability of our virtual species
3. `error.probability`: the probability of an erroneous detection of our virtual species
4. `bias`: if you introduced a sampling bias, this part will contain the appropriate info
5. `replacement`: a logical indicating if you allowed multiple samples in the same cells
6. `original.distribution.raster`: the raster from which occurrences were sampled
7. `sample.plot`: contains the graphical representation of your samplings, so it can be called later 
8. `sampled.points`: a data frame containing the sampled points

It also says to us that there are several attributes stored, such as the information about 
the randomisation process (`RNGkind` and `seed`). You can use these attributes later to 
reproduce exactly a previous sampling.

Now, we can specifically access the data frame of sampled points:


```r
presence.points$sample.points
```

```
##            x          y Real Observed
## 1  -15.25000  10.916667    1        1
## 2  121.91667  19.416667    1        1
## 3  -64.58333   1.083333    1        1
## 4  118.08333   6.250000    1        1
## 5  105.91667  -2.583333    1        1
## 6  -48.41667  -0.750000    1        1
## 7  -70.08333  -5.083333    1        1
## 8  179.41667 -17.916667    1        1
## 9  -70.25000   4.083333    1        1
## 10 110.41667  -0.750000    1        1
## 11 142.08333  -6.083333    1        1
## 12 152.08333  -5.416667    1        1
## 13  98.58333  13.583333    1        1
## 14  73.25000  17.583333    1        1
## 15 -66.41667  -2.750000    1        1
## 16 147.41667  -9.083333    1        1
## 17 -73.08333  -1.916667    1        1
## 18 165.58333   9.250000    1        1
## 19 139.58333  -6.916667    1        1
## 20 131.91667  -2.916667    1        1
## 21 -64.08333  -9.583333    1        1
## 22 -74.41667   7.916667    1        1
## 23 -83.75000  10.250000    1        1
## 24 108.25000   3.750000    1        1
## 25 114.91667  -1.750000    1        1
## 26 -65.41667  -2.250000    1        1
## 27 -71.58333  -2.416667    1        1
## 28 -65.58333  -2.916667    1        1
## 29 -70.58333 -11.416667    1        1
## 30  98.91667  11.416667    1        1
```

The data frame has four colums: the pixel coordinates `x` and `y`, the real presence/absence (1/0) of the species in the sampled pixel `Real`, and the result of the sampling `Observed` (1 = presence, 0 = absence, NA = no information). The columns `Real` and `Observed` differ only if you have specified a detection probability and/or an error probability.

Now, let's sample the other type of occurrence data: presence-absence.



```r
# Sampling of 'presence-absence' occurrences
PA.points <- sampleOccurrences(my.first.species,
                               n = 30,
                               type = "presence-absence")
```

![Fig 7.2 Illustration of the presence-absence points sampled for our virtual species. Black dots are presences, open dots are absences.](07-sampleoccurrences_files/figure-html/samp4-1.png)

```r
PA.points
```

```
## Occurrence points sampled from a virtual species
## 
## - Type: presence-absence
## - Number of points: 30
## - No sampling bias
## - Detection probability: 
##    .Probability: 1
##    .Corrected by suitability: FALSE
## - Probability of identification error (false positive): 0
## - Sample prevalence: 
##    .True:0.0333333333333333
##    .Observed:0.0333333333333333
## - Multiple samples can occur in a single cell: No
## 
## First 10 lines: 
##              x         y Real Observed
## 1    46.583333 47.583333    0        0
## 2  -126.083333 58.250000    0        0
## 3     9.083333  4.416667    1        1
## 4    65.750000 50.750000    0        0
## 5    11.416667 16.916667    0        0
## 6  -106.416667 55.583333    0        0
## 7     8.250000 35.250000    0        0
## 8    93.583333 49.250000    0        0
## 9    77.416667 65.750000    0        0
## 10   59.750000 74.750000    0        0
## ... 20 more lines.
```

You can see that the points are sampled randomly throughout the whole raster. However, we may have only a few or even no presence points at all because the samples are purely random at the moment. We can define the number of presences and the number of absences by using the parameter [`sample.prevalence`](#defining-the-sample-prevalence).

In the next section you will see how to limit the sampling to a region in particular, and after that, how to introduce a bias in the sampling procedure.

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

![Fig 7.3 Illustration of points sampled in a restricted area (here, South America + Mexico)](07-sampleoccurrences_files/figure-html/samp5-1.png)



The only restriction is to provide the correct names.

The correct region names are: "Africa", "Antarctica", "Asia", "Australia", "Europe", "North America", "South America"

The correct continent names are: "Africa", "Antarctica", "Australia", "Eurasia", "North America", "South America"

And you can consult the correct names with the following commands:


```r
library(rworldmap, quiet = TRUE)
```

```
## ### Welcome to rworldmap ###
```

```
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
_Examples in this section are currently not working because of a bug in raster's `getData()`. Will be updated when it's resolved._
In this case you have to import in R your own polygon, using commands of the package `sp`. The polygon has to be of type `SpatialPolygons` or `SpatialPolygonsDataFrame`. 

We will use here the example of a `SpatialPolygonsDataFrame` which we can download straight into R using the function `getData()` of the package `raster`. Since our species is a tropical species, let's assume we are working in Brasil only. Remember that this is an example, and that you can import your own polygons into R.

First, let's download our polygon:


```r
brasil <- getData("GADM", country = "BRA", level = 0)
```

Now, to limit the sampling area to our polygon, we simply provide it to the argument `sampling.area` of the function `sampleOccurrences`:


```r
PA.points <- sampleOccurrences(my.first.species,
                               n = 30,
                               type = "presence-absence",
                               sampling.area = brasil)

PA.points
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

![Fig 7.7 Points sampled within our extent object](07-sampleoccurrences_files/figure-html/samp10-1.png)

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

![Fig 7.8 Presence-absence points sampled with a low detection probability](07-sampleoccurrences_files/figure-html/samp11-1.png)

```r
PA.points
```

```
## Occurrence points sampled from a virtual species
## 
## - Type: presence-absence
## - Number of points: 50
## - No sampling bias
## - Detection probability: 
##    .Probability: 0.3
##    .Corrected by suitability: FALSE
## - Probability of identification error (false positive): 0
## - Sample prevalence: 
##    .True:0.22
##    .Observed:0.1
## - Multiple samples can occur in a single cell: No
## 
## First 10 lines: 
##            x           y Real Observed
## 1  -55.25000  -4.4166667    0        0
## 2  -54.25000 -12.5833333    0        0
## 3  -40.25000  -8.0833333    0        0
## 4  -38.58333  -6.7500000    0        0
## 5  -54.75000 -29.9166667    0        0
## 6  -44.58333 -17.7500000    0        0
## 7  -51.25000 -10.0833333    1        1
## 8  -60.91667  -0.4166667    0        0
## 9  -48.75000 -26.7500000    0        0
## 10 -52.75000 -26.5833333    0        0
## ... 40 more lines.
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

![Fig 7.9 Presence only points sampled with a low detection probability](07-sampleoccurrences_files/figure-html/samp12-1.png)

```r
PO.points
```

```
## Occurrence points sampled from a virtual species
## 
## - Type: presence only
## - Number of points: 50
## - No sampling bias
## - Detection probability: 
##    .Probability: 0.3
##    .Corrected by suitability: FALSE
## - Probability of identification error (false positive): 0
## - Multiple samples can occur in a single cell: No
## 
## First 10 lines: 
##            x          y Real Observed
## 1  -56.25000 -9.5833333    1       NA
## 2  -51.41667 -4.4166667    1       NA
## 3  -69.41667 -7.5833333    1        1
## 4  -52.41667  1.7500000    1       NA
## 5  -66.75000 -3.0833333    1        1
## 6  -60.75000 -6.0833333    1        1
## 7  -67.75000 -6.5833333    1       NA
## 8  -69.41667 -1.9166667    1       NA
## 9  -65.91667 -3.5833333    1       NA
## 10 -68.08333 -0.9166667    1       NA
## ... 40 more lines.
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
# plot(brasil, add = TRUE)
points(occ[occ$Observed == 1, c("x", "y")], pch = 16, cex = .8)
points(occ[occ$Observed == 0, c("x", "y")], pch = 1, cex = .8)
```

![Fig 7.10 Presence-absence points sampled with a detection probability equal to the environmental suitability](07-sampleoccurrences_files/figure-html/samp13-1.png)


### 7.3.3. Error probability

You can also introduce an error probability, which is a probability of finding the species where it is absent. This is straightforward with the argument `error.probability`:


```r
PA.points <- sampleOccurrences(my.first.species,
                               sampling.area = "Brazil",
                               n = 20,
                               type = "presence-absence",
                               error.probability = 0.3)
```

![Fig 7.11 Presence-absence points sampled with an error probability of 0.3](07-sampleoccurrences_files/figure-html/samp14-1.png)

```r
PA.points
```

```
## Occurrence points sampled from a virtual species
## 
## - Type: presence-absence
## - Number of points: 20
## - No sampling bias
## - Detection probability: 
##    .Probability: 1
##    .Corrected by suitability: FALSE
## - Probability of identification error (false positive): 0.3
## - Sample prevalence: 
##    .True:0.25
##    .Observed:0.35
## - Multiple samples can occur in a single cell: No
## 
## First 10 lines: 
##            x          y Real Observed
## 1  -59.41667  -1.916667    0        0
## 2  -51.58333   1.916667    1        1
## 3  -46.58333 -22.250000    0        0
## 4  -55.75000 -17.916667    0        0
## 5  -51.58333   2.916667    1        1
## 6  -61.25000 -13.250000    0        1
## 7  -37.08333  -9.583333    0        0
## 8  -61.08333   4.250000    0        1
## 9  -70.41667  -7.750000    1        1
## 10 -59.91667  -2.250000    1        1
## ... 10 more lines.
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

![Fig 7.12 Presence-absence points sampled with a detection probability of 0.2, and an error probability of 0.8. In numerous pixels the species was not detected, but erroneous presences were nevertheless attributed.](07-sampleoccurrences_files/figure-html/samp15-1.png)

```r
PA.points
```

```
## Occurrence points sampled from a virtual species
## 
## - Type: presence-absence
## - Number of points: 20
## - No sampling bias
## - Detection probability: 
##    .Probability: 0.2
##    .Corrected by suitability: FALSE
## - Probability of identification error (false positive): 0.8
## - Sample prevalence: 
##    .True:0.25
##    .Observed:0.9
## - Multiple samples can occur in a single cell: No
## 
## First 10 lines: 
##            x           y Real Observed
## 1  -67.08333  -5.9166667    1        1
## 2  -51.58333  -9.0833333    0        1
## 3  -68.25000   0.4166667    1        1
## 4  -52.41667   2.7500000    1        1
## 5  -70.41667  -5.9166667    1        1
## 6  -47.91667 -19.2500000    0        0
## 7  -58.75000 -10.7500000    0        1
## 8  -52.25000  -1.4166667    0        1
## 9  -51.75000 -14.9166667    0        1
## 10 -45.08333 -10.0833333    0        1
## ... 10 more lines.
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

![Fig 7.13 Presence only points sampled with a strong bias in South America (20 times more sample)](07-sampleoccurrences_files/figure-html/samp16-1.png)

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

![Fig 7.14 Presence only points sampled with a strong bias in Mexico and Colombia](07-sampleoccurrences_files/figure-html/samp17-1.png)

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

![Fig 7.16 Raster used to weight the sampling bias: a raster of cell areas](07-sampleoccurrences_files/figure-html/samp20-1.png)

Now, let's weight our samplings using this raster:


```r
PO.points <- sampleOccurrences(my.first.species,
                               n = 100, 
                               bias = "manual",
                               weights = weight.raster)
```

![Fig 7.17 Presence only points sampled with a sampling biased according to the area of each cell](07-sampleoccurrences_files/figure-html/samp21-1.png)

Of course, the changes are not  visible, because our species occurs mainly in pixels of relatively similar areas. Nevertheless, this example should be useful if you want to create a manual sampling bias later on.



**Important note** Please note that when using rasters based on longitude-latitude, larger cells have _automatically_ more chance of being sampled than smaller cells, so you do not need to do it by yourself. See documentation of the `randomPoints` function of package `dismo` for more details.

### 7.3.5. Sampling multiple records in the same cells

In real datasets such as Museum datasets, it is frequent to have multiple records in the same cells. To mimic such datasets, it is possible to allow the function to sample repeatedly in the same cells:


```r
PO.points <- sampleOccurrences(my.first.species,
                               n = 1000, 
                               replacement = TRUE,
                               plot = FALSE)

# Number of duplicated records: 
length(which(duplicated(PO.points$sample.points[, c("x", "y")])))
```

```
## [1] 20
```



## 7.4. Defining the sample prevalence

_The sample prevalence is the proportion of samples in which the species has been found. It is therefore different from the species prevalence which [was mentionned earlier](04-presenceabsence.html#conversion-to-presence-absence-based-on-a-value-of-species-prevalence)._

You can define the desired sample prevalence when sampling presences and absences, with the parameter `sample.prevalence`:


```r
PA.points <- sampleOccurrences(my.first.species,
                               n = 30,
                               type = "presence-absence",
                               sample.prevalence = 0.5)
```

![Fig 7.17 Presence-absence points sampled with a prevalence of 0.5 (i.e., 50% of presence points, 50% of absence points)](07-sampleoccurrences_files/figure-html/samp23-1.png)

```r
PA.points
```

```
## Occurrence points sampled from a virtual species
## 
## - Type: presence-absence
## - Number of points: 30
## - No sampling bias
## - Detection probability: 
##    .Probability: 1
##    .Corrected by suitability: FALSE
## - Probability of identification error (false positive): 0
## - Sample prevalence: 
##    .True:0.5
##    .Observed:0.5
## - Multiple samples can occur in a single cell: No
## 
## First 10 lines: 
##            x          y Real Observed
## 1  -10.75000  7.5833333    1        1
## 2  -85.91667 16.4166667    1        1
## 3  103.75000  1.7500000    1        1
## 4  -70.58333 -2.0833333    1        1
## 5  -67.25000 -4.5833333    1        1
## 6  103.25000 11.4166667    1        1
## 7  159.41667 -7.9166667    1        1
## 8  103.41667  1.9166667    1        1
## 9  -84.75000 15.9166667    1        1
## 10 116.25000 -0.9166667    1        1
## ... 20 more lines.
```
