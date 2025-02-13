---
title: '**Spatial Optimization: Maximum Covering Location Problem**'
author: "Trevor Kapuvari, Nohman Akhtari"
date: "02/19/2024"
output:
  html_document:
    keep_md: yes
    toc: yes
    theme: flatly
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
In this study, we use ArcGIS Pro and CPLEX to solve the Maximum Location Coverage Problem in theory and in real-life use. The Maximum Location Coverage Problem (MLCP) is a NP-hard maximization problem in computational complexity theory that optimizes to cover a maximum of demand subject to constraints. A typical use for MLCP is the siting of public service facilities, as public funds should be deployed to their maximum efficiency. Mathematically speaking, the MLCP has the form


$$
\text{max} \sum_{i \in I} g_iY_i \\
\text{s.t.} \sum_{j \in N_i}x_j \geq Y_i \ \forall i \in I, \ (1)\\
\sum_{j \in J} x_j \leq p, \ (2) \\
x_j, y_i \in \{0,1\}, \\
y_i = \begin{cases} 1 & \text{if } i \text{ is covered by at least one facility} \\
                    0 & \text{otherwise }\end{cases}
$$

# Methodology

In this form, $$g_i$$ is the demand at a location and $$Y_i$$ is a decision variable that is constrained to be either zero or one. Thus, the objective functions aims to capture the most amount of demand possible given that we can place a maximum of $p$ facilities (constraint (2), also called budget constraint in Game Theory)) and that a demand node can only be covered if at least one facility is covering it (constraint (1)).

It should be noted that the MCLP can he highly sensitive to the input variables. This is to say that slight changes in the constraint can lead to utterly distinct optimization results, and that therefore, thorough prior research is required for a proper use of MCLP.

After solving the maximization problem with CPLEX, the MCLP will give us the ideal location of facilities given our optimization problem. These type of MCLP analyses are especially useful in cases where the real world distribution of facilities is compared to the ideal distribution of facilities, since policy makers can clearly draw conclusions on what locations to potentially close, at least, to know where and where not to open new public facilities.

# MCLP in Concept

Consider the following coverage matrix. We have 8 demand nodes (i=1....8)
and 8 potential sites –facilities- (j=1....8). You can interpret this as follows:
Demand node i=1 can be covered by sites j=1,2 and 3, demand node 2 can be
covered by sites j.


```r
knitr::include_graphics("https://github.com/TrevorKap/Classes_MUSA/raw/e3a51a54503019804be0736d3863cf56d10efb12/SpatialOptimization/MatrixHW1.png")
```

![](https://github.com/TrevorKap/Classes_MUSA/raw/e3a51a54503019804be0736d3863cf56d10efb12/SpatialOptimization/MatrixHW1.png)<!-- -->

When solving for which facilities to choose first, the most optimal/most effective place to start is the facility with the most demand. In this case, g2. g2 has the largest demand and when we are given the ability to choose more facilities, the most logical approach is to cover the next largest demand. This flow cascades until it reaches the maximum amount of allowed covered sites.



```r
knitr::include_graphics("https://github.com/TrevorKap/Classes_MUSA/raw/acbff38ff5cb1b18fb2719a1d426b8a908301f10/SpatialOptimization/PartA.png")
```

![](https://github.com/TrevorKap/Classes_MUSA/raw/acbff38ff5cb1b18fb2719a1d426b8a908301f10/SpatialOptimization/PartA.png)<!-- -->

When starting with the outlier, facility 2, roughly 80% of demand is already covered. Following the next largest nodes of demand fractionally increase coverage until there is marginal returns per additional covered site (P value).

# Cast Study | Moscow, Idaho

Our first observation of the cover location problem brings us to the town of Moscow, Idaho. Moscow, like much of the state, has a small-town suburban landscape where cars are the primary source for transportation. Public transportation takes the form of buses but is overshadowed in usage compared to automobiles. We are observing how a person's willingness to travel (i.e walk) can effect the potential coverage of bus stops in the town. 

As alluded to in the methodology part, the constraints of the optimization problem can greatly affect the ideal location of sites. Below, we see how a willingness to walk 250m vs 500m to a bus stop can significantly increase coverage. 


```r
knitr::include_graphics("https://github.com/TrevorKap/Classes_MUSA/raw/0cd9832396cc9eb5e1d218bfea1cdc3129fcf0f2/SpatialOptimization/SpatialOptHW1.png")
```

![](https://github.com/TrevorKap/Classes_MUSA/raw/0cd9832396cc9eb5e1d218bfea1cdc3129fcf0f2/SpatialOptimization/SpatialOptHW1.png)<!-- -->

We first look at the Pareto-Front for different budget constraints and see the typical decreasing margins. This is because in an optimization problem, the "best" locations are picked first, and the "worst" locations last. In this context, best and worse is measured as in demand coverage. Each line, regardless of willingness to travel, eventually 'flatlines' because the problem no longer becomes how many people can be reached per station, but the lack of stations capable of covering new distances. Regardless, having the double in willing distance (250m to 500m) results in a double in accessible population. The greatest increase in change (slope) is between when adding 1 to 6 bus stops, where 500m bus stops are more than double in outreach than 250m ones. 

In the map below, we are comparing a difference between 250 meters and 500 meters for 10 sampled bus stops. Leveraging CPLEX optimization calculations, we will evaluate which bus stops cover the largest amount of people assuming specified willingness to travel and choosing 10 pre-existing stations.  


```r
knitr::include_graphics("https://github.com/TrevorKap/Classes_MUSA/raw/07c1cc0cfadea9492a46b1b9558bdf460171e594/SpatialOptimization/Optimized%20Bus%20Stop.png")
```

![](https://github.com/TrevorKap/Classes_MUSA/raw/07c1cc0cfadea9492a46b1b9558bdf460171e594/SpatialOptimization/Optimized%20Bus%20Stop.png)<!-- -->


We then see how the effect plays out visually and realize that that doubling the willingness to travel makes a big difference. Covering 250 meters at a time leaves critical gaps between stations, where would prefer to drive than consider public transit. When the travel distance is 500m, we notice some bus stations become redundant with minor overlap in certain areas. In this case, many sections of the town are covered by the bus stations and pre-existing ones that were not provided a buffer (not optimally placed) should be relocated to service other areas. 

# Conclusion

There are three main shortcomings which we want to address. Firstly,  Euclidean and Network Distance can severely differ in the sense that the Euclidean metric overestimates the reachable population. Secondly, the model doe not take into account whether the stops offered to the population actually satisfy their travel needs. It could be that accessibility is high, but the routes offered are rarely used. This also includes issues of directionality. Lastly, the true geography of the network is not taken into account. It could be that in some areas, there are severe elevation differences making routes that are officially walkable extremely difficult for elderly or disabled population. One prime example for a city where this would apply is Lisbon that is built on several hills. Generally, while the model suffices in identifying optimal locations that automates planning and maximizes coverage, it requires additional adjustment to account for externalities that are not depicted through a 2-D plane. 

