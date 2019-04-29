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
## - species prevalence =0.0338242766299243
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
## 1   73.41667 19.7500000    1        1
## 2  101.75000  3.7500000    1        1
## 3  -11.25000  7.0833333    1        1
## 4  -78.41667 -4.5833333    1        1
## 5  122.25000 11.2500000    1        1
## 6  137.91667 -4.9166667    1        1
## 7  111.08333 -0.4166667    1        1
## 8  -66.25000  0.4166667    1        1
## 9  -74.58333 -5.2500000    1        1
## 10 -73.75000 -3.5833333    1        1
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
##   ..- attr(*, "pid")= int 11756
##   ..- attr(*, "Rversion")=Classes 'R_system_version', 'package_version', 'numeric_version'  hidden list of 1
##   ..- attr(*, "load")= chr(0) 
##   ..- attr(*, "attach")= chr(0) 
##   ..- attr(*, "class")= chr "recordedplot"
##  $ sample.points               :'data.frame':	30 obs. of  4 variables:
##   ..$ x       : num [1:30] 73.4 101.8 -11.2 -78.4 122.2 ...
##   ..$ y       : num [1:30] 19.75 3.75 7.08 -4.58 11.25 ...
##   ..$ Real    : num [1:30] 1 1 1 1 1 1 1 1 1 1 ...
##   ..$ Observed: num [1:30] 1 1 1 1 1 1 1 1 1 1 ...
##  - attr(*, "RNGkind")= chr [1:2] "Mersenne-Twister" "Inversion"
##  - attr(*, "seed")= int [1:626] 403 240 -783267279 -1072830632 -1244324702 1938186215 1315597702 -550798737 -802745507 -407459021 ...
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
##            x            y Real Observed
## 1   73.41667  19.75000000    1        1
## 2  101.75000   3.75000000    1        1
## 3  -11.25000   7.08333333    1        1
## 4  -78.41667  -4.58333333    1        1
## 5  122.25000  11.25000000    1        1
## 6  137.91667  -4.91666667    1        1
## 7  111.08333  -0.41666667    1        1
## 8  -66.25000   0.41666667    1        1
## 9  -74.58333  -5.25000000    1        1
## 10 -73.75000  -3.58333333    1        1
## 11 101.75000   3.58333333    1        1
## 12 131.25000  -0.25000000    1        1
## 13 -72.91667  -6.58333333    1        1
## 14 101.08333   6.25000000    1        1
## 15 134.58333  -5.58333333    1        1
## 16 -73.58333  -4.08333333    1        1
## 17 106.91667  -7.41666667    1        1
## 18 -77.58333   1.75000000    1        1
## 19 -57.25000  -4.91666667    1        1
## 20 -55.08333  -1.25000000    1        1
## 21 143.08333  -8.41666667    1        1
## 22 103.08333   1.08333333    1        1
## 23 115.08333  -1.91666667    1        1
## 24 119.25000  -2.25000000    1        1
## 25 -66.08333  -3.08333333    1        1
## 26 179.25000 -18.08333333    1        1
## 27 -51.58333  -2.91666667    1        1
## 28 -72.58333   0.08333333    1        1
## 29 -92.91667  15.25000000    1        1
## 30 -70.58333  -1.25000000    1        1
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
##    .True:0
##    .Observed:0
## - Multiple samples can occur in a single cell: No
## 
## First 10 lines: 
##            x          y Real Observed
## 1  -91.25000  41.083333    0        0
## 2  124.75000 -25.250000    0        0
## 3   83.75000  47.750000    0        0
## 4  142.75000 -33.750000    0        0
## 5   44.25000  48.416667    0        0
## 6   71.58333  41.750000    0        0
## 7  152.08333  -4.583333    0        0
## 8   35.91667  28.416667    0        0
## 9   96.91667  42.583333    0        0
## 10 -37.08333  -8.250000    0        0
## ... 20 more lines.
```

You can see that the points are sampled randomly throughout the whole raster. However, we may have only a few or even no presence points at all because the samples are purely random at the moment. We can define the number of presences and the number of absences by using the parameter [`sample.prevalence`](#defining-the-sample-prevalence).

You may be interested in extracting the true probability of occurrence at each sampled point, to compare with your SDM results. To do this, use argument `extract.probability`:

```r
# Sampling of 'presence-absence' occurrences
PA.points <- sampleOccurrences(my.first.species,
                               n = 30,
                               type = "presence-absence",
                               extract.probability = TRUE,
                               plot = FALSE)
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
##             x         y Real Observed true.probability
## 1   150.08333  71.25000    0        0     8.315280e-07
## 2   -58.75000 -35.41667    0        0     2.601265e-06
## 3    77.91667  20.41667    0        0     1.694370e-04
## 4  -107.75000  30.91667    0        0     1.428352e-06
## 5   -71.08333 -14.25000    0        0     8.325917e-07
## 6   107.75000  58.08333    0        0     8.315281e-07
## 7  -115.08333  60.08333    0        0     8.315284e-07
## 8   -61.91667  52.41667    0        0     8.315293e-07
## 9     7.25000  15.25000    0        0     1.337543e-05
## 10  101.58333  28.25000    0        0     8.903669e-07
## ... 20 more lines.
```

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

![Fig 7.4 A polygon of the brazil](07-sampleoccurrences_files/figure-html/samp7-1.png)

Now, to limit the sampling area to our polygon, we simply provide it to the argument `sampling.area` of the function `sampleOccurrences`:


```r
PA.points <- sampleOccurrences(my.first.species,
                               n = 30,
                               type = "presence-absence",
                               sampling.area = brasil)
```

![Fig 7.5 Points sampled within the Brazil polygon](07-sampleoccurrences_files/figure-html/samp8-1.png)

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
##    .True:0.0666666666666667
##    .Observed:0.0666666666666667
## - Multiple samples can occur in a single cell: No
## 
## First 10 lines: 
##            x          y Real Observed
## 1  -55.75000 -20.583333    0        0
## 2  -35.25000  -7.750000    0        0
## 3  -57.58333 -16.750000    0        0
## 4  -43.91667 -19.416667    0        0
## 5  -50.25000  -2.083333    1        1
## 6  -39.75000 -16.083333    0        0
## 7  -37.41667 -10.250000    0        0
## 8  -52.58333 -23.250000    0        0
## 9  -60.58333   1.916667    0        0
## 10 -55.41667  -1.583333    0        0
## ... 20 more lines.
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

![Fig 7.6 Points sampled within the Brazil polygon, with a suitable zoom](07-sampleoccurrences_files/figure-html/samp9-1.png)

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
##    .True:0.18
##    .Observed:0.02
## - Multiple samples can occur in a single cell: No
## 
## First 10 lines: 
##            x           y Real Observed
## 1  -43.08333 -18.4166667    0        0
## 2  -58.25000 -16.0833333    0        0
## 3  -65.41667  -0.5833333    1        0
## 4  -54.08333 -28.4166667    0        0
## 5  -45.25000 -20.9166667    0        0
## 6  -61.08333  -8.5833333    0        0
## 7  -48.75000  -9.2500000    0        0
## 8  -47.25000  -5.2500000    0        0
## 9  -60.58333   0.5833333    0        0
## 10 -63.41667  -3.9166667    0        0
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
##            x         y Real Observed
## 1  -62.25000 -3.083333    1        1
## 2  -67.75000 -4.083333    1       NA
## 3  -55.25000 -9.916667    1       NA
## 4  -67.41667  0.750000    1       NA
## 5  -56.41667 -9.583333    1       NA
## 6  -65.58333 -3.083333    1       NA
## 7  -61.25000 -7.916667    1       NA
## 8  -67.58333  0.750000    1       NA
## 9  -65.08333 -3.750000    1        1
## 10 -61.58333 -6.250000    1       NA
## ... 40 more lines.
```

In this case, we see that out of all the possible sampling points, only a fraction are sampled as presence points.

### 7.3.2. Detection probability as a function of environmental suitability

You can further complexify the detection probability by making it dependent on the environmental suitability. In this case, cells will be weighted by the environmental suitability: less suitable cells will have a lesser chance of detecting the species. This can be seen as an impact on the population size: a higher environmental suitability increases the population size, and thus the detection probability. How does this work in practice?

The initial probability of detection ($P_{di}$) is multiplied by the environmental suitability ($S$) to obtain the final probability of detection ($P_d$):

$$P_d = P_{di} \times S$$

Hence, if our species has a probability of detection of 0.5, the final probability of detection will be:

* 0.5 if the cell has an environmental suitability of 1
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
##    .True:0.1
##    .Observed:0.5
## - Multiple samples can occur in a single cell: No
## 
## First 10 lines: 
##            x          y Real Observed
## 1  -62.58333   1.583333    0        0
## 2  -72.25000  -6.083333    1        1
## 3  -55.91667 -21.916667    0        1
## 4  -36.08333  -8.416667    0        1
## 5  -51.08333  -6.916667    0        0
## 6  -66.91667  -4.916667    0        1
## 7  -66.25000  -5.250000    1        1
## 8  -56.08333 -17.583333    0        0
## 9  -52.75000 -32.416667    0        1
## 10 -49.91667 -26.583333    0        0
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
##    .True:0.15
##    .Observed:0.95
## - Multiple samples can occur in a single cell: No
## 
## First 10 lines: 
##            x           y Real Observed
## 1  -51.58333 -31.2500000    0        1
## 2  -66.75000  -6.5833333    0        1
## 3  -43.58333  -8.5833333    0        1
## 4  -62.91667   2.4166667    0        1
## 5  -70.25000  -8.5833333    1        1
## 6  -53.41667  -8.4166667    0        1
## 7  -53.91667 -21.0833333    0        1
## 8  -63.41667  -2.9166667    0        1
## 9  -50.91667  -0.5833333    0        1
## 10 -42.41667  -7.4166667    0        1
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

```
## Warning in sampleOccurrences(my.first.species, n = 100, bias = "polygon", : Polygon projection is not checked. Please make sure you have the 
##             same projections between your polygon and your presence-absence
##             raster
```

![Fig 7.15 Presence only points sampled with strong bias for the Philippines, using a polygon](07-sampleoccurrences_files/figure-html/samp18-1.png)

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
## [1] 25
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
## 1  122.25000  16.250000    1        1
## 2  -52.58333  -1.083333    1        1
## 3  -58.41667  -6.750000    1        1
## 4  109.25000   1.583333    1        1
## 5  -63.75000  -5.416667    1        1
## 6   91.75000  25.916667    1        1
## 7  112.58333  -0.750000    1        1
## 8  149.91667 -10.250000    1        1
## 9  121.41667  15.916667    1        1
## 10  90.91667  22.750000    1        1
## ... 20 more lines.
```
