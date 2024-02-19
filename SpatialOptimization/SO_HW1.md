---
title: '**Spatial Optimization: Maximum Covering Location Problem**'
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
In this document, we use ArcGis Pro and CPLEX to solve problems involving the maximum covering location problem. In Part A, facility siting will be optimized, whereas in Part B, bus stop access is maximized.

# Methodology
The Maximum Location Coverage Problem (MLCP) is a NP-hard maximization problem in computational complexity theory that optimizes to cover a maximum of demand subject to constraints. A typical use for MLCP is the siting of public service facilities, as public funds should be deployed to their maximum efficiency. Mathematically speaking, the MLCP has the form

$$
\text{max} \sum_{i \in I} g_iY_i \\


\text{s.t.} \sum_{j \in N_i}x_j \geq Y_i \ \forall i \in I, \ (1)\\

\sum_{j \in J} x_j \leq p, \ (2) \\

x_j, y_i \in \{0,1\}, \\

y_i = \begin{cases} 1 & \text{if } i \text{ is covered by at least one facility} \\
                    0 & \text{otherwise }\end{cases}.
$$
In this form, $$g_i$$ is the demand at a location and $$Y_i$$ is a decision variable that is constrained to be either zero or one. Thus, the objective functions aims to capture the most amount of demand possible given that we can place a maximum of $p$ facilities (constraint (2), also called budget constraint in Game Theory)) and that a demand node can only be covered if at least one facility is covering it (constraint (1)).

It should be noted that the MCLP can he highly sensitive to the input variables. This is to say that slight changes in the constraint can lead to utteerly distinct optimization results, and that therefore, thorough prior research is required for a proper use of MCLP.

After solving the maximization problem with CPLEX, the MCLP will give us the ideal location of facilities given our optimization problem. These type of MCLP analyses are especially useful in cases where the real world distribution of facilities is compared to the ideal distribution of facilities, since policy makers can clearly draw conclusions on what locations to potentially close, at least, to know where and where not to open new public facilities.
# Part A

# Part B

As alluded to in the methodology part, the constraints of the optimization problem can greatly affect the ideal location of sites. Below, we see how a willingness to walk 250m vs 500m to a bus stop can significantly increase coverage. 


```r
knitr::include_graphics("https://github.com/TrevorKap/Classes_MUSA/raw/07c1cc0cfadea9492a46b1b9558bdf460171e594/SpatialOptimization/SpatialOptHW1New.png")
```

![](https://github.com/TrevorKap/Classes_MUSA/raw/07c1cc0cfadea9492a46b1b9558bdf460171e594/SpatialOptimization/SpatialOptHW1New.png)<!-- -->

We first look at the Pareto-Front for different budget constraints and see the typical decreasing margins. This is because in an optimization problem, the "best" locations are picked first, and the "worst" locations last. In this context, best and worse is measured as in demand coverage. Each line, regardless of willingness to travel, eventually 'flatlines' because the problem no longer becomes how many people can be reached per station, but the lack of stations capable of covering new distances. Regardless, having the double in willing distance (250m to 500m) results in a double in accessible population. The greatest increase in change (slope) is between when adding 1 to 6 bus stops, where 500m bus stops are more than double in outreach than 250m ones. 




```r
knitr::include_graphics("https://github.com/TrevorKap/Classes_MUSA/raw/07c1cc0cfadea9492a46b1b9558bdf460171e594/SpatialOptimization/Optimized%20Bus%20Stop.png")
```

![](https://github.com/TrevorKap/Classes_MUSA/raw/07c1cc0cfadea9492a46b1b9558bdf460171e594/SpatialOptimization/Optimized%20Bus%20Stop.png)<!-- -->
We then see how the effect plays out visually and realize that that doubling the willingness to walk genuinely makes a big difference. One policy implication for this could be to motivate inhabitants to walk more, e.g., through advertising the health benefits of doing a certain amount of exercise or walk per day.

There are three main shortcomings which we want to address. Firstly, as was demonstrated in class, Euclidean and Network Distance can severely differ in the sense that the Euclidean metric overestimates the reachable population. Secondly, the model doe not take into account whether the stops offered to the population actually satisfy their travel needs. It could be that accessibility is high, but the routes offered are rarely used. This also includes issues of directionality. Lastly, the true geography of the network is not taken into account. It could be that in some areas, there are severe elevation differences making routes that are officially walkable extremely difficult for elderly or disabled population. One prime example for a city where this would apply is Lisbon that is built on several hills.


# Conclusion
(to be done)
