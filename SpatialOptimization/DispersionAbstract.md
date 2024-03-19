---
title: '**Dispersion Theory**'
author: "Trevor Kapuvari, Nohman Akhtari"
date: "03/14/2024"
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
The p-dispersion problem is a technique to maximize the minimum distance between the two closest pair of facilities in a network. Even though this might sound not productive at first glance, there are quite a few evident use cases for p-dispersion optimization problems. Think of facilities that can pose a threat to each other, like in some chemical factories- the gases and other byproducts might be hazardous, could interact, and therefore, are best located far from each other. For a more tangible business case, take a savvy entrepreneur who wants to open a service business- where should he best open his shop? It only makes sense to avoid competition, and therefore, maximize the minimum distance between his stores or to those of the competition.

In this assignment, we will take the p-dispersion problem and display how computation time significantly varies as parameters change. 


# Methodology
For the purpose of this assignment, we will employ the p-dispersion maximization problem that maximizes the minimum distance between any pair of facilities at two locations. As this formulation seems a bit unintuitive, we will first introduce the mathematical formulation and then explain each term in the optimization problem. 


$$
\text{max } Z \\
\text{s.t. } Z - m (2-X_i-X_j) \leq d_{ij}\\
\sum_{i} X_i = p \\
X_i \in \{0,1\} \\
Z \geq 0
$$
where $Z$ is the minimum distance between two facilities,$d_{ij}$ is the distance between locations $i$ and $j,$ $p$ is the number of facilities to be sited, and $m$ is a large number that is used to establish what is called an upper effective bound on $Z$. Note that if facilities at both site $i$ and $j$ are chosen, we are left with $Z \leq d_{ij}$ implying that we must chose $d_{ij}$ as distance. The upper bound on $Z$ only comes into action if none or either locations are chosen, giving us $Z \leq d_{ij}+2m$ or $Z \leq d_{ij}+m,$ respectively.



# Code
Below is the Python code used to generate the LP file for the Dispersion problem. The code generates a set of candidate locations and calculates the distance between each pair of candidate locations. The code then writes the LP file for the Dispersion problem. To change the number of facilities chosen, change 'p='. To change the number of candidate locations, change 'minX', 'maxX', 'minY', 'maxY' (only for boundaries), or 'interval' (for spacing). The results are plotted below.

```python
import random, string, os, math, time, sys
import matplotlib.pyplot as plt   # only necessary if you want to plot
import numpy                          # only necessary if you want to plot

def distance(x1, y1, x2, y2):
    dx = x2 - x1
    dy = y2 - y1
    d = math.sqrt(dx**2 + dy**2)
    return d

## parameters
p=15            #num facilities
M=100000        # a very large number

## Generate candidate locations (uniform)
ID=[]
X=[]
Y=[]
minX=0
maxX=100
minY=0
maxY=100 
interval =5 # interval between candidate locations

k=0
for i in range (minX, maxX, interval):
    for j in range(minY, maxY, interval):
        ID.append(k)
        X.append(i + random.random())
        Y.append(j + random.random())
        k+=1

print('there are ' + str(len(X)) + ' decision variables')
OD = []
i=0
while i<len(X):
    j=0
    while j<len(Y):
        OD.append(str(distance(X[i], Y[i], X[j], Y[j])))
        j=j+1
    i=i+1


## activate the next lines only if you want to plot the points
plt.scatter(X, Y)
plt.show()

################# WRITING L-P file ################# 

###   Objective function headings
outputFile = open('pdispersion.txt', 'w')
outputFile.write("Maximize D\n")
outputFile.write("\nSubject to")
outputFile.write("\n")

#   Constraint 2 
i=0
while i< len(X):
    j=i+1
    while j < len(X):
        Dij = OD[i*len(X) + j]
        outputFile.write('D+' + str(M - float(Dij))+ 'X' +str(i) + '+' + str(M- float(Dij)) + 'X' +str(j) +'<= ' +str((2*M) - float(Dij)) + '\n') 
        j+=1
    i+=1


#   Constraint 3- max facilities = p
outputFile.write("\n")
j=0
while j < len(ID):
    outputFile.write("+X")
    outputFile.write(str(int(ID[j])))
    j=j+1
outputFile.write("="+str(p)+"\n")

#   Constraint 4 - integer requirements
outputFile.write("\n") 
outputFile.write("Integers\n")

i=0
while i<len(X):
    outputFile.write('X' + str(i) +'\n')
    i=i+1

outputFile.write("\n")
outputFile.write("END\n")
outputFile.close()
```


```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/998862e2b8c02a1738a5787e077c54fa56018b97/SpatialOptimization/PydispersionResult.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/998862e2b8c02a1738a5787e077c54fa56018b97/SpatialOptimization/PydispersionResult.png)<!-- -->

# Results

The results of the run times in seconds based on the amount of facilities the code is expected to identify. The scale is logarithmic because of the sheer run time and data processing required to calculate the optimal number of facilities. We notice here thatit can identify a single optimal facility in less than a second yet takes over 10,000 seconds (2.7 hours) to find 5 facilities. 

The runtime for $p = 1$ is very short for obvious reasons. However, it might seem unintuitive that run time decreases as the complexity of the problem increases. Nevertheless, there is a quite simple explanation. We start with $p = 20$ and realize that the total amount of possible sitings for 20 facilities given our constraints is radically lower than that for, say, $p = 10.$ This combinatorial fact also explains why runtimes decrease with increasing sample size.


```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/FacilityCountP4.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/FacilityCountP4.png)<!-- -->



Another observation is that our objective value $Z$ decreases as facility count increases.

```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/FacilityCountvOV4.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/FacilityCountvOV4.png)<!-- -->

