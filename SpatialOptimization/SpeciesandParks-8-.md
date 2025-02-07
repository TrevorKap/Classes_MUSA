---
title: '**Integrating Species Preservation and Park Budgets
in Optimization**'
author: "Trevor Kapuvari, Nohman Akhtari, Gabriel Hernandez"
date: "04/02/2024"
output:
  html_document:
    keep_md: yes
    toc: yes
    theme: united
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
Natural Resources take many forms and governments have the responsibility between balancing preservation and economic development. One natural resource that is overlooked is endangered species, which are critical to any ecosystem. Protecting endangered species and preserving their habitats are essential for maintaining biodiversity, yet there are foregone economic benefits when setting aside this land, or a recurring cost to maintain the land. There are techniques to measure the optimal balance between protecting species through preserved parkland & fiscal responsibility. The Maximum Species Cover Optimization Problem (MSCP) is a mathematical model that can help determine the optimal number of parcels to set aside for species preservation. 

We will first examine how many species are preserved given a certain budget, dictating the amount of parks opened. Then, we will examine how the budgetary constraints alter the number of species preserved when there are lower park maintenance costs. 


# Python Script 
For this study, we will calculate the Maximum Species Coverage Optimization Problem (MSCP) in ArcGIS Pro through a Python script. 

The code is designed to optimize land conservation efforts by prioritizing parcels of land based on their preservation value and associated costs. Initially, it gathers information about parcels, species, and their respective costs from specified layers. Using this data, the program sets up a Linear Programming (LP) model to maximize preservation benefits while minimizing costs. The objective function is represented as a balance of preservation value of species and the cost of preserving parcels. Constraints are then applied to limit the number of chosen parcels within a predefined threshold. Finally, it writes the LP script to an output file for further analysis that is conducted in CPLEX. When reading the code, each # next to the variable assists with definition for replication in a broader scope. 


```python

import random, string, os,  math, time, sys
import arcpy
from arcpy import env
arcpy.OverWriteOutput = True

    
## user specific ########################################
parcels = "C:\\Users\\Owner\\Downloads\\hwk8data\\Newfolder\\parks.shp" # parcels hosting species / natural resources
weightCostObj = 0 # cost of each natural resource
cost = "SUM_ACRES" # cost of each parcel by size 
species = "species" # resource in each parcel
weightFile= "C:\\Users\\Owner\\Downloads\\hwk8data\\Newfolder\\speciesWeights.dbf" # value of each resource (species)
weightField= 'weight' # field in the weight file that contains the value of each resource
p = 10 # number of parcels to be selected
solutionFile = "C:\\Users\\Owner\\Downloads\\hwk8data\\Newfolder\\solution.txt" # output file


## 1.Inputs
parcelList=[]
parcelCost=[]
speciesList=[]
for ori in arcpy.SearchCursor(parcels):
    parcelList.append(ori.getValue("FID"))
    a = ori.getValue(species) 
    speciesList.append(list(a))
    parcelCost.append(ori.getValue(cost))
arcpy.AddMessage("parcel information read....")

speciesObsInParcel =[]
speciesSighting =[]
for i in range(0,len(speciesList),1):
    tmpList=[]
    for j in range(0,len(speciesList[i]), 1):
        a = speciesList[i][j]
        speciesSighting.append(a)
        tmpList.append(a)
    speciesObsInParcel.append(tmpList)        
speciesSightingUnique= list(set(speciesSighting))
arcpy.AddMessage("species information read....")

weights=[]
for ori in arcpy.SearchCursor(weightFile):
    a = ori.getValue(species)
    b = ori.getValue(weightField)
    weights.append(str(a) + ","+str(b))
arcpy.AddMessage("species weight information read....")

## 2. Generating LP.
outputFile = open(solutionFile,"w")

##   Model heading
outputFile.write("Maximize\n")

##  Objective function
for i in range(0,len(speciesSightingUnique),1):         #how many different species there are (i)
    if str(speciesSightingUnique[i]) != ' ':
        for j in range(0,len(weights), 1):
            if str(speciesSightingUnique[i]) in str(weights[j][0]):
                outputFile.write("+" + str(int(weights[j][2:]) * (1-float(weightCostObj))) + "Y" + str(speciesSightingUnique[i]))

for i in range(0,len(parcelList),1):
    outputFile.write( "-" +str(float(weightCostObj) * float(parcelCost[i])) +  "X" + str(i))

outputFile.write("\n")
arcpy.AddMessage("objective function written...")

##  Constraints
outputFile.write("\nSubject to")
outputFile.write("\n")

##  A species is preserved only if there is a parcel that can harbor the species
for i in range(0,len(speciesSightingUnique),1):         
   if str(speciesSightingUnique[i]) != ' ':
      outputFile.write("+Y" + str(speciesSightingUnique[i]))
      for j in range(0,len(speciesObsInParcel),1):  
         if speciesSightingUnique[i] in speciesObsInParcel[j][:]:
             outputFile.write("-X" + str(j))
      outputFile.write("<=0 \n")
arcpy.AddMessage("constraint 1 written...")


# we can have p number of parcels
for i in range(0,len(parcelList),1):
    outputFile.write( "+X" + str(i))
outputFile.write("<=" + str(p) +"\n")
arcpy.AddMessage("constraint limiting number of sites written....")


# keeping track of the number of species we are going to have
outputFile.write("\n")
outputFile.write("Z1")
for i in range(0,len(speciesSightingUnique),1):         #how many different species there are (i)
    if str(speciesSightingUnique[i]) != ' ':
        for j in range(0,len(weights), 1):
            if str(speciesSightingUnique[i]) in str(weights[j][0]):
                outputFile.write("-" + "Y" + str(speciesSightingUnique[i]))
outputFile.write("=0\n\n\n")

# keeping track of the cost
outputFile.write("Z2")
for i in range(0,len(parcelList),1):
    outputFile.write( "-" +str(float(parcelCost[i])) +  "X" + str(i))
outputFile.write("=0\n")


# tracking objectives
for i in range(0,len(speciesSightingUnique),1):         
   if str(speciesSightingUnique[i]) != ' ':
      outputFile.write("+Y" + str(speciesSightingUnique[i]))
      for j in range(0,len(speciesObsInParcel),1):  
         if speciesSightingUnique[i] in speciesObsInParcel[j][:]:
             outputFile.write("-X" + str(j))
      outputFile.write("<=0 \n")
arcpy.AddMessage("tracking each subobjective")



##   Integers requirements
outputFile.write("Integers\n")
for i in range(0,len(speciesSightingUnique),1):        
   if str(speciesSightingUnique[i]) != ' ':
      outputFile.write( "Y" + str(speciesSightingUnique[i]) + "\n")
for j in range(0,len(parcelList),1):
    outputFile.write("X" + str(j) + "\n")
arcpy.AddMessage("integer requirements written...")
outputFile.write("End")

outputFile.close()
arcpy.AddMessage("LP file written...")

```


# Species Rarity by Means of Weight

The objective of the first study is to estimate the benefit of maintaining parks for species preservation compared to the number of parks opened. Figure 1 represents the change in the objective value (species preserved) compared to our P value (parks opened). 

### Figure 1

```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/HW8P1.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/HW8P1.png)<!-- -->


We notice here generally, as the number of parks opened increases, the number of species preserved increases. However, this increase stops at P = 10, where the maximum number of species are  preserved as indicated by the green line, 129. This level is not exceed when keeping more than 10 parks opened, leaving 10 as the smallest number of parks to open assuming the objective is to preserve every species. 

# Costs and Species Rarity

In order to incorporate cost into our model, we will incorporate 'weightCostObj' (found in the Python script) as a parameter for the "cost objective". Weighted Cost functions as the price per acre to maintain a park. At each cost we measure how many species are preserved and at what cost per species. At each cost we compare the economic cost-benefit analysis between the species preserved and the cost per species. 

### Figure 2

```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/HW8P2A.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/HW8P2A.png)<!-- -->

The objective function represents the benefit of keeping parks opened
for the purpose of species preservation while the weight represents the cost to preserve each species. The axis of weight decreases along the x-axis meaning, as the cost decreases the benefits of preserving species increases. At a minimal cost of 1 cent an acre, as much as 121 species can be preserved (assuming 15 parks are  opened). Yet still  107 species are protected when only 5 parks are opened. As the cost decreases, the return on investment for less parks opened increases.

### Figure 3

```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/HW8P2B.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/HW8P2B.png)<!-- -->

z1 represents the number of species preserved while z2 represents the cost of preserving each species. The y-axis represents the cost of preserving each species while the x-axis represents the number of species preserved. 

There are three different behaviors for each amount provided. At P = 5, there is a consistent upward trend that flat lines between ~13 and 20. Then marginally increases thereafter. 

At P = 10, there is noticeable spike and dip between 10 and 15. Then at z1 = 25, is almost just as cost-effective as P = 5. 

At P = 15, there is a similar behavior to P = 10 but is much more gradual. At the beginning it is more expensive per species to preserve, but becomes much cheaper as the z1 value progresses. This makes P = 15 the most cost-effective option. 

# Conclusion

We have shown that the number of species preserved is directly correlated with the number of parks opened. The cost of preserving each species is variable when factoring the number of parks opened. This is because the cost of maintaining a park is fixed, but preserving species is not. The other nuance that must be considered is the numeric value we put on life in an ecosystem, as that does not necessarily account for the ecological importance it may have on an ecosystem. Regardless, more parks means more places for species to live and therefore, the cost per each to protect them decreases as well. 


