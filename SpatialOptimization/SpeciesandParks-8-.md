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
Natural Resources take many forms and governments have the responsibility between balancing preservation and economic development. One natural resource that is overlooked is endangered species, which are critical to the ecosystem. Protecting endangered species and preserving their habitats are essential to maintaining biodiversity, yet there are foregone economic opportunity costs when setting aside this land, or a recurring cost to maintain the land. There are techniques to measure the optimal balance between preserving land for protecting species and maintain parkland for public & environmental use. The Maximum Species Cover Optimization Problem (MSCP) is a mathematical model that can help determine the optimal number of parcels to set aside for species preservation. 

We will first examine how many species are preserved given a certain budget, dictating the amount of parks opened. Then, we will examine how the budgetary constraints alter the number of species preserved when there are lower park maintenance costs. 


# Python Script 
For this study, we will calculate the Maximum Species Coverage Optimization Problem (MSCP) in ArcGIS Pro through a Python script. 


```python

import random, string, os,  math, time, sys
import arcpy
from arcpy import env
arcpy.OverWriteOutput = True

    
## user specific ########################################
parcels = "C:\\Users\\Owner\\Downloads\\hwk8data\\Newfolder\\parks.shp" 
weightCostObj = 0
cost = "SUM_ACRES"
species = "species"
weightFile= "C:\\Users\\Owner\\Downloads\\hwk8data\\Newfolder\\speciesWeights.dbf"
weightField= 'weight'
p = 10
solutionFile = "C:\\Users\\Owner\\Downloads\\hwk8data\\Newfolder\\solution.txt"


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


The graph......


# Costs and Species Rarity

In order to incorporate cost into our model, we will incorporate 'weightCostObj' (found in the Python script) as a parameter for the "cost objective". Weighted Cost functions as the price per acre to maintain a park. At each cost we measure how many species are preserved and at what cost per species. At each cost we compare the economic cost-benefit analysis between the species preserved and the cost per species. 

### Figure 2

```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/HW8P2A.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/HW8P2A.png)<!-- -->

In Figure 2.....

### Figure 3

```r
knitr::include_graphics("https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/HW8P2B.png")
```

![](https://raw.githubusercontent.com/TrevorKap/Classes_MUSA/main/SpatialOptimization/images/HW8P2B.png)<!-- -->

Figuer 3....


# Conclusion

We notice...



