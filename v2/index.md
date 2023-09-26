---
title: 'The virtualspecies R package: a complete tutorial - version 1.6'
author: "Boris Leroy - leroy.boris@gmail.com"
date: "September 2023"
output:
  html_document:
    fig_caption: yes
    keep_md: yes
    mathjax: "http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"
    theme: yeti
---



# Introduction

This complete tutorial introduces all the possibilities of the `virtualspecies` 
package. It was written with the objective to be helpful for both beginners and
experienced R users. You can read this tutorial in full or jump to the
particular section you are looking for. In each section, you will find simple 
examples introducing each function, followed by detailed examples for almost 
every possible parametrisation in `virtualspecies`.


After a small introduction on the spatial data used as input for the 
`virtualspecies` package (section 1.), you will be introduced to the basics of
generating virtual species distributions: the two possible approaches to create 
species-environment relationships (sections 2. and 3.), followed by the 
conversion of environmental suitability to presence-absence (section 4.).
After that, you will see how to generate random virtual species (section 5.), 
and most importantly, how to explore, extract, and use the outputs of
`virtualspecies` (section 6.). Then, you will see how to sample occurrence
points (section 7.). Finally, you will learn about how to general dispersal 
limitations to your virtual species in section 8. 


I will make extensive use of climate data as an example here, but remember that
you can use other types of data, as long as it is in the format described in 
section 1.



# Update 1.6 

`virtualspecies` has been updated to version 1.6 in September 2023. The major
change in this update is that now this package relies upon the R package 
`terra` rather than `raster`; in addition, internally it uses functions from
the package `sf` rather than `sp`. This update was required since multiple 
spatial packages are going to be retired from R, and future developments and
improvements of spatial packages will rely on `terra` and `sf`. 

*This update may cause some breaking changes*. I maintained the changes as 
minimal as possible so that previous code (version <= 1.5) will still be
compatible with the new code (version >= 1.6). However, objects generated
in versions <= 1.5 will likely not be compatible with the package version 1.6.
They can still be converted manually to become compatible, using functions 
from the package `terra` such as `rast()`. If you find yourself
in trouble, email me about that. In addition, the methods in functions
`limitDistribution()` and `sampleOccurrences()` have changed slightly because
they now use the package `rnaturaleath` (formerly they used `rworldmap`).
