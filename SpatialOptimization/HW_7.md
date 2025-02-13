---
title: '**Maximum Species Coverage Optimization Problem**'
author: "Trevor Kapuvari, Nohman Akhtari"
date: "03/25/2024"
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
Ecology is a complex system that, despite centuries of research, has not yet be fully conceptualized by machine learning. Modelling ecological systems has been complicated in factoring all externalities, including predictions on how changing any variable may or may not affect the coexistence and stability of a system. Nevertheless, spatial techniques can assist in understanding species preservation, especially in cases in which space has been divided up among stakeholders of a system. This is also partly due to the fact that policy makers can immediately see how introducing new parks or spaces might completely alter the optimization problem for preservation.

In the first part of this study, we will look into endangered species protection in concept, and how the algorithm works to find parcels that are critical for protection. In the second part, we will identify how this applies to Mecklenburg County, North Carolina, and the parallel between the abstract function's behavior and the real-world application.

# Methodology
For this study, we will use the Maximum Species Coverage Optimization Problem (MSCP). We first give the mathematical formulation, and then delve into each term and an explanation:

$$
\text{max } \sum_{i \in I} y_i \\
\text{s.t. } y_i - \sum_{j \in N_{i}} x_j \leq 0 \ \forall i\\
\sum_{j \in J} x_j \leq p \\
x_j, y_i \in \{0,1\} \\
\text{where }\\
x_j = \begin{cases}1 & \text{, if we select parcel }j\\ 0 & \text{, otherwise} \end{cases}\\
y_i = \begin{cases}1 & \text{, if species } i \text{ is represented} \\ 0 & \text{, otherwise} \end{cases} \\
N_i \text{ is the set of parcels where species } i \text{has been observed.}
$$
First, note that O, our objective is to maximize the sum of distinct species represented However, we are constrained by the fact that for each species, we require a parcel $j$ in $N_i$ where we actually observed the species $i.$ This leads us to the first constraint. The second constraint puts a cap to the number of parcels we can select, and the last constraint is the typical binary integer constraint.

The MSCP is quite intelligible, and the relevance of available space in this problem is especially clear. Nevertheless, this model is simplistic as it does not include considerations such as synergies between species, importance of species, and whether species could coexist in a parcel without major disturbance of the ecosystem.

# MSCP In Abstract

The graph shows the objective function of Maximum Species Coverage Optimization Problem where the objective value represents the number of species preserved  and the p-value represents the number of parcels selected. One parcel holds, at most, four species (p = 1), and as we allow additional parcels to be selected, the number of species preserved increases at a marginally decreasing rate. By the we reach p = 3, we have preserved the  maximum number of species without creating additional value. 


```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/HW7_1A.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/HW7_1A.png)<!-- -->


This flattened curve is also explained by the table. At 3 parcels and thereafter, we preserve the same 7 species, even when the algorithm chooses additional parcels. In-fact, it did not recognize any necessary parcels to select at p = 4. 


```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/Table1.B.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/Table1.B.png)<!-- -->

The results show that, when examining biodiversity in a given region and not factoring ecological intersectionality or other factors, the MSCP is a useful tool to determine the number of parcels that need to be set aside for species preservation. 

# MSCP in Real-World Application

In the second part of this study, we examine Mecklenburg County, North Carolina, and the species that are endangered in the region. In the graph below we identify a similar curve behavior as the previous exmple. The curve flattens out completely at p = 6, and the number of species preserved increases at a decreasing rate as it approaches said point. 


```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/HW7_2A.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/HW7_2A.png)<!-- -->

The figure below shows a matrix table of all the species preserved at each parcel. A '1' represents successful preservation of the species and the numbers of each column represent the number of parks preserved by the algorithm. 



```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/HW7_2B.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/HW7_2B.png)<!-- -->


The map represents the parcels when P = 5. At this point compared to the previous graph, we could use the algorithm to select one additional parcel to preserve two more species. In this map of the five parks, 24 sampled species are preserved.


```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/HW7MAP.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/HW7MAP.png)<!-- -->

# Conclusion

Maximum Species Coverage Optimization Problem is a useful tool to determine the number of parcels that need to be set aside for species preservation. The model is simplistic and does not include considerations such as synergies between species, importance of species, and whether species could coexist in a parcel without major disturbance of the ecosystem. Nevertheless, it is a good starting point to understand the importance of space in species preservation.



