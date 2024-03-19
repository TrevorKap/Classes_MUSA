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
The p-dispersion problem is a technique to maximize the distance between the two closest pair of points in a network. The function of this optimization technique is to disseminate a value, or object, as far as possible from others of the same type. An example of such can be seen as opening a business in a competitive market. One would want to avoid competition, and therefore, maximize their distance from any other similar business to ensure they are not competing for the same customers. This is a simple example of the p-dispersion problem, but it can be applied to many other scenarios such as the placement of cell towers, fire stations, etc. In abstract, the goal is to maximize distance from similar entities within a region.

We will examine the p-dispersion problem and display how computation time significantly varies as parameters change. 


# Methodology
Here, we break down the p-dispersion maximization problem to the theory and formula. The function is, given a set of pre-existing locations, the formula locates the optimal location for a new facility such that the distance between the new facility and the closest pre-existing facility is maximized, or chooses pre-existing facilities that maximize the distance between those facilities. The p-dispersion problem can be formulated as a linear program as follows:


$$
\text{max } Z \\
\text{s.t. } Z - m (2-X_i-X_j) \leq d_{ij}\\
\sum_{i} X_i = p \\
X_i \in \{0,1\} \\
Z \geq 0
$$
Where $Z$ is the minimum distance between two facilities,$d_{ij}$ is the distance between locations $i$ and $j,$ $p$ is the number of facilities to be sited, and $m$ is a large number that is used to establish what is called an upper effective bound on $Z$. Note that if facilities at both site $i$ and $j$ are chosen, we are left with $Z \leq d_{ij}$ implying that we must chose $d_{ij}$ as distance. The upper bound on $Z$ only comes into action if none or either locations are chosen, giving us $Z \leq d_{ij}+2m$ or $Z \leq d_{ij}+m,$ respectively.



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
The results of the run times in seconds are based on the amount of facilities the code is expected to identify. The scale is logarithmic because of the sheer run time and data processing required to calculate the optimal number of facilities. We notice here that it can identify a single optimal facility in less than a second yet takes over 10,000 seconds (2.7 hours) to find 5 facilities. 

The runtime for $p = 1$ is very short due to its simplicity. However, that run time decreases as the complexity of the problem increases (from 5 onward), and has a sharp increase in its initial additions (as from 1 to 5). We start with $p = 20$ and realize that the total amount of possible sites for 20 facilities given our constraints is radically lower than that of $p = 10.$ This combinatorial fact explains runtimes decreasing with increasing sample size.


```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/FacilityCountP4.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/FacilityCountP4.png)<!-- -->


Another observation is that our objective value $Z$ decreases as facility count increases. This can be explained by the fact that with increasing facility count, the minimum distance between facilities decreases. 


```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/FacilityCountvOV4.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/FacilityCountvOV4.png)<!-- -->

# Conclusion
The p-dispersion problem is simple in its objective yet complex in its computation and processing. Yet the problem in theory translates to many potential practices in planning, transportation, and risk management. This study shows the multifaceted nature of the how 'chosen facility' dissemination and its relationship between runtime, our variable $Z,$ may appear straightforward to the human-mind, yet becomes a large computational process for a computer.

