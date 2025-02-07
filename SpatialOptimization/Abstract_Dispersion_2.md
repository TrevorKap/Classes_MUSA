---
title: '**Dispersion Optimization in Practice**'
author: "Trevor Kapuvari, Nohman Akhtari"
date: "03/20/2024"
output:
  html_document:
    keep_md: yes
    toc: yes
    theme: lumen
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
The p-dispersion problem is a technique to maximize the distance between the two closest pair of points in a network. The function of this optimization technique is to disseminate a value, or object, as far as possible from others of the same type. 

In this practice study, we leverage the p-dispersion problem to a political analysis regarding homeowner eligibility for registered sex-offenders. Our objective is to locate available homes within a municipality that both enables the social rehabilitation process while mitigating recidivism risk. For our scenario, a city council agreed to consider rental properties as potential locations for registered sex-offenders seeking residency. We are examining potential locations for these individuals to reside, while considering the distance between each location to the nearest school, park, and other sex-offender occupied rentals. 

# Methodology 
The p-dispersion maximization problem that maximizes the distance between chosen points is depicted below.


$$
\text{max } Z\\
\text{s.t. } Z - m (2-X_i-X_j) \leq d_{ij}\\
\sum_{i} X_i = p \\ 
X_id'_{ik} \leq b\\
X_i \in \{0,1\}\\
Z \geq 0
$$
$Z$ is the minimum distance between two points,$d_{ij}$ is the distance between locations $i$ and $j,$ $p$ is the number of points to be sited, and $m$ is a number that is used to establish what is called an upper effective bound on $Z$. Note that if points at both site $i$ and $j$ are chosen, we are left with $Z \leq d_{ij}$ implying that we must chose $d_{ij}$ as distance. The upper bound on $Z$ only comes into action if none or either locations are chosen, giving us $Z \leq d_{ij}+2m$ or $Z \leq d_{ij}+m,$ respectively.

Note that for each placed point, the constraints also include a minimum distance $b$ that the sited point must be away from. The index $k$ is the location index for points for which we want to build up the distance.

Further, it has to be clarified that the notion of distance has to be thought of in an abstract way in the sense that it could be pure walking distance, but also aerial distance or time to get to a specified location. The choice of the distance variable is eventually up to the person who builds the model, but should be adjusted specifically to each use case to make the results more robust.

# System Evaluation

The graph below displays two key metrics when understanding the computation and results of the p-dispersion problem. The top graph measures the distance between chosen rentals. As expected, as the number of eligible rentals to house the offenders increases, the distance between each rental decreases. Each increment of two rentals results in roughly half the distance between one another. 

Meanwhile, the computation processing (run time) to calculate and identify this solution takes exponentially longer as the number of rentals increase. For reference, 576 seconds (as shown with 6 rentals) is equivalent to 9.6 minutes. Meanwhile, 19,521 seconds (as shown with 8 rentals) is equivalent to ~5.4 hours. This correlation and results are explained by the necessary diligence and cross-validation process required by the computer to calculate the optimal solution.


```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/Homework5Graph1.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/Homework5Graph1.png)<!-- -->


# Application

Without considering any further restrictions, the map below shows where rental can be contingent on the number of rentals allowed. Each number (p value) is represented by a colored-ring due to repeated solutions for certain locations. 


```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/Homework5Map1.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/Homework5Map1.png)<!-- -->

The city council has additional restrictions that, depending on the number of rentals allowed, impact the location of where rentals can be. The restriction scenario imposes a 500m or 750m ban from schools and 500m or 250m ban from parks. The map below shows each restriction and the potential locations for rentals if only two locations were allowed. For this specific case, the ban with either distance does not change the eligible locations. 


```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/Homework5Map2.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/Homework5Map2.png)<!-- -->


# Conclusion
Through the p-dispersion formula and program, we are able to identify optimal locations while maximizing distance apart from each chosen location. This is a powerful tool for any municipality or organization looking to optimize the dispersion of a specific object or value. In our case, we were able to identify the optimal locations for rental properties to house registered sex-offenders. As excellent of a tool this practice is, computational power and socio-political implications are important to consider when applying this to real-world scenarios. 

