---
title: '**TITLE HERE**'
author: "Trevor Kapuvari, Nohman Akhtari"
date: "02/19/2024"
output:
  html_document:
    keep_md: yes
    toc: yes
    theme: sandstone
    toc_float: yes
    code_folding: hide
    number_sections: no
    css: style.css
  pdf_document:
    toc: yes
---


```r
library(tidyverse)
library(tidycensus)
library(sf)
library(knitr)
library(viridis)
library(bookdown)
library(MASS)
library(rmarkdown)
options(scipen = 999)
knitr::opts_chunk$set(echo = TRUE)
theme_update(plot.title = element_text(hjust = 0.5))
```


# Introduction

Just state the problem/MLCP whatever The Maximum Location Coverage Problem (MLCP)

# Methodology

readd the formula n nerdy shit, let me know if you need a screenshot added in 


# Part A

Measured at vgvarious distances, intro them to the graphs 


```r
knitr::include_graphics("https://github.com/TrevorKap/Classes_MUSA/raw/c27c6a73146d91b8650bb14d71c8ba311181cab1/SpatialOptimization/SpatialOptHW2chart1New.png")
```

![](https://github.com/TrevorKap/Classes_MUSA/raw/c27c6a73146d91b8650bb14d71c8ba311181cab1/SpatialOptimization/SpatialOptHW2chart1New.png)<!-- -->


Measure the amount of pop that can be accomed depending on how far someone is to travel. 
Explain why it meets  at 12, or almost does
the early flatline for 2.5 and 3k




```r
knitr::include_graphics("https://github.com/TrevorKap/Classes_MUSA/raw/1a547eb4a4aa282760be961b8f770f284639c07c/SpatialOptimization/p10.png")
```

![](https://github.com/TrevorKap/Classes_MUSA/raw/1a547eb4a4aa282760be961b8f770f284639c07c/SpatialOptimization/p10.png)<!-- -->

We see this when we allow a max of 10 bus stops. when its 3k we only need  7. Yet here is the overlap that occurs when its 3k. 
Explain the demand nodes  and how cplex recognizes each need to be covered.

Why did i undissolve the 3k layer but not the 2.5k 





```r
knitr::include_graphics("https://github.com/TrevorKap/Classes_MUSA/raw/1a547eb4a4aa282760be961b8f770f284639c07c/SpatialOptimization/p15.png")
```

![](https://github.com/TrevorKap/Classes_MUSA/raw/1a547eb4a4aa282760be961b8f770f284639c07c/SpatialOptimization/p15.png)<!-- -->

Still plenty of overlap with 2,500m, people dont actually walk this far. P = 7 even when allowing CPLEX to choose 10 stations. We see the inefficient overlap when we allow 15 for 2.5k yet only needs 7 again fro 3k. 

explain the undissolve of 2.5k and why it shows inefficiency 
P = 7 again because it literally never needed it. 


```r
knitr::include_graphics("https://github.com/TrevorKap/Classes_MUSA/raw/c27c6a73146d91b8650bb14d71c8ba311181cab1/SpatialOptimization/SpatialOptHW2chart2New.png")
```

![](https://github.com/TrevorKap/Classes_MUSA/raw/c27c6a73146d91b8650bb14d71c8ba311181cab1/SpatialOptimization/SpatialOptHW2chart2New.png)<!-- -->

When population adjusted how much do we really need? 8
Which distance should be used to understand willingness to travel? 


# Conclusion

how do we measure willingness to travel, how do we change it? 
What do these maps say about the current charlotte transit? Mind you that both the 2.5k and 3k stops are all currently just ONE LINE. 
