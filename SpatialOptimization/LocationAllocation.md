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
Human nature, when travelling for any goal/purpose is to measure, is to take the shortest route possible. In terms of economics, amenities are situated within a general radius of one another that accommodate a person's frequency of travel to said locations. Pharmacies are often frequented by the general population and therefore do not require a large amount of people to sustain them. Hence, these places are often small but several throughout a region. In our study, we measure the difference in distance traveled (as a  collective) increases when there are less pharmacies in a given area. 

# Methodology
Blah blah blah g

$$
\text{max} \sum_{i \in I} g_iY_i \\
\text{s.t.} \sum_{j \in N_i}x_j \geq Y_i \ \forall i \in I, \ (1)\\
\sum_{j \in J} x_j \leq p, \ (2) \\
x_j, Y_i \in \{0,1\}, \\
Y_i = \begin{cases} 1 & \text{if } i \text{ is covered by at least one facility} \\
                    0 & \text{otherwise }\end{cases}.
$$



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

The map shows all pharmacies when comparing the p-value of 35 (white) and 15 (black). In the model, all pharmacies chosen when p = 15 were also chosen when p = 35. The P35 pharmacies are scattered slightly less than the P15 pharmacies due to the abundance of additions the model needed to make in order to meet the quoted amount of facilities required. When only choosing 15 pharmacies the model took a more spread-out approach covering generally equal distances throughout the county. What the model does not consider is the demand capacity each pharmacy accommodates. When considered distance alone, the distance allocation is the optimal strategy yet when factoring demand, the additional pharmacies after P=15 started to focus on the inner-city core. In essence, the fractional coverage created when increasing the number of pharmacies from 15 to 35 focus greater attention on the inner city and, because of demand capacity not considered, marginally improve coverage. 



```r
knitr::include_graphics("https://github.com/TrevorKap/Classes_MUSA/raw/604516eac37357921ca422d10afb507ff430280a/SpatialOptimization/MapAllhw3.png")
```

![](https://github.com/TrevorKap/Classes_MUSA/raw/604516eac37357921ca422d10afb507ff430280a/SpatialOptimization/MapAllhw3.png)<!-- -->

This map shows all pharmacies that currently exist. We notice that even if this amount has the lowest weighted total miles as depicted from the graph, some pharmacies have little contribution to supporting the general network. We examine these 'lesser contributions' on the edges of the country where demand nodes can redirect to other pharmacies and only fractionally increase total miles. The only true improvement that can be made, when not considered demand capacity, is to relocate inner-city pharmacies to ones further out where some point must travel large distances. 

# Conclusion

Our study finds that additional pharmacies after 15 marginally improve distance traveled, and there can be a 'sweetspot' between 15 and 35. The model does not factor capacity and demand influx implied by the demand nodes, and can most likely be shown when accounted for total weighted distance per pharmacies chosen. Each P-Median value assists in finding the optimal number of pharmacies for the county and allows us to evaluate where in the county is the most distance traveled to each point's nearest pharmacy. 

