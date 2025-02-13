---
title: '**Location Allocation, Measuring Shortest Possible Distance**'
subtitle: "Trevor Kapuvari, Nohman Akhtari"
author: "University of Pennsylvania"  
date: "02/23/2024"
output:
  html_document:
    keep_md: yes
    toc: yes
    theme: readable
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
Human nature, when travelling for any goal/purpose, is to take the shortest route possible. Pharmacies are frequented by the general population and therefore do not require a large amount of people to sustain them. Hence, pharmacies are often small and several throughout a region. In our study, we measure the difference in distance traveled (as a  collective) when there are a specified number of pharmacies in a given area. We do so by using the Location-Allocation problem to find the ideal distribution of pharmacies, and then will start playing with the parameters of our model. Our model evaluates the pre-existing pharmacies in a given region with an input on a specified number of these pharmacies to select. The model's objective is to connect each demand node (population point) to the closest selected pharmacy. Ultimately, this result aims to reduce the amount of miles driven by all populations with respect to the specified number of pharmacies. 

# Methodology
Location-Allocation optimization is a method used to ideally situate facilities with the goal to satisfy all demand with the shortest path possible to each site. To set up our equation, we first introduce some notation. Let $n \in \mathbb{N}$ denote the number of demand areas, and let $m \in \mathbb{N}$ denote the number of potential facility sites. Further, let

$$
i = \text{index of demand area}\\
j = \text{index of potential facility site}\\
d_{ij} = \text{shortest distance from demand area } i \text{ to potential facility site }j\\
a_i = \text{demand at area } i\\
p = \text{number of facilities to be located}\\
Y_j = \begin{cases} 1, & \text{if facility at site } j \text{ is located}\\0, & \text{otherwise} \end{cases}\\
X_{ij} = \begin{cases} 1, & \text{if demand at area } i \text{ is satisfied by facility at }j\\0, & \text{otherwise} \end{cases}.
$$
Note that we have some degree of flexibility when defining $d_{ij}.$ For example, we could use network travel time or some other indirect metric of distance depending on the goal of the analysis.

With this notation, the Location-Allocation optimization problem can be given by

$$
min \sum_{i=1}^n\sum_{j=1}^ma_id_{ij}X_{ij}\\
\text{subject to} \\
\sum_{j=1}^m X_{ij} = 1 \ \forall i\\
X_{ij} \leq Y_j \ \forall i,j \\
\sum_{j=1}^mY_j=p \\
Y_j \in \{0,1\}\ \forall j \\
X_{ij} \in \{0,1\} \  \forall i,j.
$$
The idea behind the given objective function is to look at the product of demand at area $i$ and the defined distance to the facility at site $j.$ In an ideal scenario, we satisfy our goal of covering all demand by locating each facility such that the total walking distance is minimized globally. The term $X_{ij}$ is a binary that serves as selector. 

Our first constraint requires that each demand area $i$ will have exactly 1 facility at location $j$ that covers it. The second constraint ensures that we can only satisfy demand at area $i$ if there actually exists a facility at location $j,$ and the third conditions specifies the number of total locations that can be placed. The last two conditions require that both $Y_j$ and $X_{ij}$ are binary.

The intuition behind the objective function and the first two constraints is that each demand node will is assumed to walk to that demand node that, in combination with all other demand nodes, minimizes global distance. This implies that our function will find the facility closest to our demand node satisfying all the conditions above.


# Case Study | Mecklenburg  County, North Carolina

In Mecklenburg County we counted pharmacies throughout the county to evaluate which parcels were closest proximity to each pharmacy. Due to Esri policy on certain network analysis tools and computational capabilities, mild altering of data was required. The limits per 'Location Allocation' tool process was 10,000 data rows per run. The dataset provided featured over 12,000 rows. However, over 3,000 rows featured a population of 0, as in no one lived there. These rows would not contribute to the total weighted miles or any quantified measurement. Therefore, they were filtered and removed from the dataset. 

Below features the graph of the total weighted miles to each pharmacy in the county. 'Total Weighted Miles' is defined as the cumulative sum of the miles driven by all members of a population to their nearest pharmacy. 



```r
knitr::include_graphics("https://github.com/TrevorKap/Classes_MUSA/raw/604516eac37357921ca422d10afb507ff430280a/SpatialOptimization/PharmacyMilesHW3.png")
```

![](https://github.com/TrevorKap/Classes_MUSA/raw/604516eac37357921ca422d10afb507ff430280a/SpatialOptimization/PharmacyMilesHW3.png)<!-- -->

The values of 15 and 35 are highlighted to show the mild difference between the P values (P-Median) of each. Meanwhile, we notice there is a vast difference starting from 1 to 41, yet marginally decreases thereafter and increasingly smaller margins as more pharmacies as added. 


```r
knitr::include_graphics("https://github.com/TrevorKap/Classes_MUSA/raw/604516eac37357921ca422d10afb507ff430280a/SpatialOptimization/Map35.png")
```

![](https://github.com/TrevorKap/Classes_MUSA/raw/604516eac37357921ca422d10afb507ff430280a/SpatialOptimization/Map35.png)<!-- -->

The map shows all pharmacies when comparing the p-value of 35 (white) and 15 (black). In the model, all pharmacies chosen when p = 15 were also chosen when p = 35. The P35 pharmacies are scattered slightly less than the P15 pharmacies due to the abundance of additions the model needed to make in order to meet the quoted amount of facilities required. When only choosing 15 pharmacies the model took a more spread-out approach covering generally equal distances throughout the county. What the model does not consider is the demand capacity each pharmacy accommodates. When considering distance alone, the distance allocation is the optimal strategy yet when factoring demand, the additional pharmacies after P=15 started to focus on the inner-city core. In essence, the fractional coverage created when increasing the number of pharmacies from 15 to 35 focus greater attention on the inner city and, because of demand capacity not considered, marginally improve coverage. 



```r
knitr::include_graphics("https://github.com/TrevorKap/Classes_MUSA/raw/604516eac37357921ca422d10afb507ff430280a/SpatialOptimization/MapAllhw3.png")
```

![](https://github.com/TrevorKap/Classes_MUSA/raw/604516eac37357921ca422d10afb507ff430280a/SpatialOptimization/MapAllhw3.png)<!-- -->

This map shows all pharmacies that currently exist. We notice that even if this amount has the lowest weighted total miles as depicted from the graph, some pharmacies have little contribution to supporting the general network. We examine these 'lesser contributions' on the edges of the country where demand nodes can redirect to other pharmacies and only fractionally increase total miles. The only true improvement that can be made, when not considering demand capacity, is to relocate inner-city pharmacies to ones further out where some point must travel large distances. 

# Conclusion

Our study finds that additional pharmacies after 15 marginally improve distance traveled, and there can be a 'sweetspot' between 15 and 35. The model does not factor capacity and demand influx implied by the demand nodes, and can most likely be shown when accounted for total weighted distance per pharmacies chosen. Each P-Median value assists in finding the optimal number of pharmacies for the county and allows us to evaluate where in the county is the most distance traveled to each point's nearest pharmacy. 

