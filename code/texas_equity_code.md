---
title: "Texas Equity Metrics"
author: "Dashiell Young-Saver, Jared Knowles"
date: "Jun 30, 2018"
output: 
  html_document:
    theme: simplex
    css: ../docs/styles.css
    highlight: NULL
    keep_md: true
    toc: true
    toc_depth: 3
    toc_float: true
    number_sections: false
---

# Creating Equity Metrics
*Using Texas state testing data files*
*Programmed in R*

## Getting Started



<div class="navbar navbar-default navbar-fixed-top" id="logo">
<div class="container">
<img src="../img/open_sdp_logo_red.png" style="display: block; margin: 0 auto; height: 115px;">
</div>
</div>

### Objective

In this guide, you will be able to use standard statistics and data visuals to investigate equity in student testing outcomes along lines of race, income, gender, learning differences, English proficiency, and migrancy status.

### Using this Guide

This guide utilizes mock data from Texas's standardized exams (STAAR test, grades 3-8). Therefore, Texas districts can use this code to directly analyze the testing data files they have received from the Texas Education Agency. However, the code can be adapted to any other state or district context in which student testing outcomes are analyzed. 

Once you have identified analyses that you want to try to replicate or modify, click the 
"Download" buttons to download R code and sample data. You can make changes to the 
charts using the code and sample data, or modify the code to work with your own data. If 
you are familiar with Github, you can click "Go to Repository" and clone the entire repository to your own computer. 

Go to the Participate page to read about more ways to engage with the OpenSDP community or reach out for assistance in adapting this code for your specific context.

### About the Data

The data used in this guide was synthetically generated, and it was formatted to match the Texas Education Agency's state test file formats (Texas's file formats can be found here: https://tea.texas.gov/student.assessment/datafileformats). The data has one record per student. Out of the hundreds of features reported for each student by the Texas Education Agency, we used (and retained) the following features in our analysis: student grade level (3-8), school code, student ID, gender, race-ethnicity, economic disadvantage level, Limited English Proficiency level, and scale scores in reading, math, and writing (for the STAAR state test). We also used indicators of whether or not the student attended a Title 1 school, was a migrant, and was enrolled in special education. Here is a key of the features and their variable names in our simulated dataset:

The Texas Education Agency data files have hundreds of features attached to each student record. Coding our analyses with all of these features could get unwieldy, so the best practice is to select the key features we will need for our analyses. If you would like to directly use your own data with the code from this guide, it is best to delete unecessary features and change the headers to the feature names we chose. Below is a legend to the features we chose and how we named them. A more detailed data definition guide can be found in the `man` folder on the Github repository:

| Feature name    | Feature Description                                 |
|:-----------     |:------------------                                  |
| `grade_level`   | Grade level of exam student took (3-8)              |
| `school_code`   | School ID number                                    |
| `sid`           | Student ID number                                   |
| `male`          | Student gender                                      |
| `race_ethnicity`| Student race/ethnicity                              |
| `eco_dis`       | Student level of economic disadvantage              |
| `title_1`       | Indicator if student attends Title 1 school         |
| `migrant`       | Indicator if student is a migrant                   |
| `lep`           | Level of Limited English Proficiency                |
| `iep`           | Indicator if student enrolled in special education  |
| `rdg_ss`        | Scale score for reading exam                        |
| `math_ss`       | Scale score for math exam                           |
| `wrtg_ss`       | Scale score for writing exam                        |  
| `composition`   | Score on writing composition exam                   |

#### Loading the OpenSDP Dataset and R Packages

This guide takes advantage of the OpenSDP synthetic dataset and several key R packages. The first chunk of code below loads the R packages (make sure to install first!), and the second chunk loads the dataset and provides us with variable labels.


```r
library(tidyverse) # main suite of R packages to ease data analysis
library(magrittr) # allows for some easier pipelines of data
library(tidyr) #
library(plyr)
library(dplyr)
library(FSA)
library(ggplot2) # to plot
library(scales) # to format
library(grid)
library(gridExtra) # to plot
# Read in some R functions that are convenience wrappers
source("../R/functions.R")
#pkgTest("devtools")
#pkgTest("OpenSDPsynthR")
```


```r
# // Step 1: Read in csv file of our dataset, naming it "texas.data"
texas.data <- read.csv("../data/synth_texas.csv")  

# // Step 2: View data file
View(texas.data)

# // Step 3: Create a vector of labels for feature names in our dataset 
#These labels will appear in visualizations and tables
labels <- c("Grade","School ID","Student ID","Gender", "Race-Ethnicity",
            "Econ Disadvantage Status","Title 1 Status","Migrancy Status",
            "LEP Status","Spec Ed Enrolled","Reading Score",
            "Math Score","Writing Score","Writing Comp Score")


#Pairs labels with feature names from file
names(labels) <- c("grade_level","school_code","sid","male","race_ethnicity",
                   "eco_dis","title_1","migrant",
                   "lep","iep","rdg_ss",
                   "math_ss","wrtg_ss","composition")
```





### About the Analyses

Especially in larger districts, students may have systematically differing educational outcomes in relation to their various identity markers (race, class, gender, English Language Learner status, etc.). These gaps often present themselves in standardized testing data, at the national, state, and local levels. The following analyses will assist your organization in seeing where gaps may exist and how wide they are, in order to spur and focus the conversation around why the gaps exist and what to do about them.

### Sample Restrictions

One of the most important decisions in running each analysis is defining the sample. 


```r
# Read in global variables for sample restriction
# Agency name
#agency_name <- "Agency"
```

### Giving Feedback on this Guide
 
This guide is an open-source document hosted on Github and generated using R Markdown. We welcome feedback, corrections, additions, and updates. Please visit the OpenSDP equity metrics repository to read our contributor guidelines.

## Analyses

### Descriptive Statistics

**Purpose:** Descriptive statistics give your agency a quick snapshot of current achievement gaps among students, identifying areas for further investigation and analysis.

**Required Analysis File Variables:**

- `grade_level` 
- `school_code` 
- `male`        
- `race_ethnicity` 
- `eco_dis`      
- `title_1`     
- `migrant`     
- `lep`
- `iep`
- `rdg_ss`
- `math_ss`
- `wrtg_ss`
- `composition`

**Ask Yourself**

- How do different study subpopulations in your organization perform on standardized tests? How do they compare?
- Why do these differences occur?
- What differences do you want to explore further?

**Analytic Technique:** Calculate the summary statistics for exam performance, for all 5th and 8th grade exam takers. Note: Once you set which tests and which grade levels you would like to analyze here, the code (as written) will analyze those grade levels and tests for the rest of the guide.

```r
# // Step 1: Set which tested grade levels to analyze (5th and 8th here)
grades <- c("5","8")

# // Step 2: Set which tested subjects to analyze (math and reading here)
subjects <- c("rdg_ss","math_ss")

# // Step 3: Calculate summary stats for each grade level
# Loop over grade level
for(grade in grades){
  
  data = texas.data[texas.data$grade_level == grade,] #Isolates grade level
  print(paste("grade level: ",grade))
  a <- summary(data[,subjects]) #Summary stats table for grade
  colnames(a) <- labels[subjects]  #Label table
  print(a) #Print summary table
  
} #End loop over grade level
```

```
[1] "grade level:  5"
 Reading Score    Math Score       
 Min.   :-545.4   Min.   :-2678.8  
 1st Qu.: 130.8   1st Qu.:  199.8  
 Median : 264.2   Median :  536.8  
 Mean   : 265.0   Mean   :  615.1  
 3rd Qu.: 400.3   3rd Qu.:  954.2  
 Max.   :1224.7   Max.   : 4606.0  
[1] "grade level:  8"
 Reading Score    Math Score       
 Min.   :-497.9   Min.   :-2481.2  
 1st Qu.: 177.6   1st Qu.:  270.0  
 Median : 311.9   Median :  625.5  
 Mean   : 312.4   Mean   :  710.2  
 3rd Qu.: 448.6   3rd Qu.: 1065.5  
 Max.   :1289.5   Max.   : 4821.9  
```

**Analytic Technique:** In order to give us a further sense of the distributions behind these summary statistics, we will create some visualizations of the data. Specifically, here we use box plots and histograms


```r
# // Visualization 1: Box plots
#Loop over tested subjects
for(subject in subjects){
  
  data = texas.data[texas.data$grade_level == grades,] #Isolates 5th and 8th graders
  
    #Set variables and parameters for our boxplot
    p <- ggplot(data, aes(x=as.factor(grade_level), y=data[,subject])) + 
          geom_boxplot() +
          ggtitle(paste(labels[subject], ", by Grade Level")) +
          scale_y_continuous(name=labels[subject]) +
          scale_x_discrete(name="Grade Level")
    print(p) 
    

} #End loop over tested subjects
```

<img src="../figure/E_VisualizeTotals-1.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeTotals-2.png" style="display: block; margin: auto;" />

```r
# // Comparison 2: Histograms of scores for eco_dis students
#Loop over grade levels
for(grade in grades){
  
  data = texas.data[texas.data$grade_level == grade,] #Isolates grade level
  
  #Loop over tested subject
  for(subject in subjects){ 
    #Set variables and parameters for our boxplot
    p <- ggplot(data, aes(x=data[,subject])) + 
          ggtitle(paste("Grade: ",grade,", ",labels[subject], " (all students)"))+
          geom_histogram(alpha = 0.5, binwidth = 50, fill = "dodgerblue", color = "dodgerblue") + 
          scale_x_continuous(name=paste(labels[subject])) 
    print(p) 
    
  } #End loop over tested subject
} #End loop over grade level
```

<img src="../figure/E_VisualizeTotals-3.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeTotals-4.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeTotals-5.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeTotals-6.png" style="display: block; margin: auto;" />

**Analytic Technique:** Now that we have some measures for the performance on exams among all of our students, we can create those same measures for our various subpopulations of students. We will start by comparing descriptive statistics among different student demographic populations (income, race, and gender).


```r
# // Step 1: Initialize Demographics to Analyze
#Vector with features we want to analyze: Econ Disadvantage, Race Ethnicity, Gender
dems <- c("eco_dis","race_ethnicity","male")

# // Comparisons: Generating summary stats for Eco Dis, Race Ethnicity, and Gender
#Loop over grade levels
for(grade in grades){
  
  data = texas.data[texas.data$grade_level == grade,] #Isolates grade level
  
  #Loop over tested subject
  for(subject in subjects){
    
    print(paste("Grade: ",grade,", ",labels[subject])) #Print subject and grade level
    
    #Loop over demographic features
    for(dem in dems){
      
      a<-Summarize(data[,subject] ~ data[,dem]) #Makes comparison table
      colnames(a)[1] <- labels[dem] #Labels comparison table
      print(a) #Prints comparison table
      
    } #End loop over demogrpahic features
    
  } #End loop over tested subject
  
} #End loop over grade level
```

```
[1] "Grade:  5 ,  Reading Score"
  Econ Disadvantage Status     n     mean       sd    min  Q1 median    Q3  max
1                        0 31650 266.0917 201.5706 -534.0 132  264.4 401.9 1135
2                        1 18537 263.0128 201.2255 -545.4 128  264.1 397.4 1225
                             Race-Ethnicity     n     mean       sd     min
1          American Indian or Alaska Native   354 257.5892 196.4762 -465.70
2                                     Asian  2451 278.8205 208.7169 -460.30
3                 Black or African American  6199 259.3710 203.0171 -545.40
4        Demographic Race Two or More Races  1066 269.0954 198.2896 -365.30
5              Hispanic or Latino Ethnicity  8272 267.0831 201.8443 -456.00
6 Native Hawaiian or Other Pacific Islander    72 317.6098 188.0882  -16.59
7                                     White 31773 264.2438 200.5958 -534.00
     Q1 median    Q3    max
1 111.3  258.3 381.3  830.3
2 139.5  277.1 421.6  945.5
3 121.1  258.2 397.5 1129.0
4 132.0  269.1 406.2 1225.0
5 133.4  268.1 400.1  987.1
6 183.7  294.6 416.9  800.0
7 130.9  263.2 398.9 1170.0
  Gender     n     mean       sd    min    Q1 median    Q3  max
1 Female 24344 265.7453 201.0299 -534.0 131.5  264.9 401.2 1170
2   Male 25843 264.2095 201.8396 -545.4 129.8  263.5 399.0 1225
[1] "Grade:  5 ,  Math Score"
  Econ Disadvantage Status     n     mean       sd   min    Q1 median    Q3
1                        0 31650 617.1197 604.9358 -1866 199.8  538.7 958.8
2                        1 18537 611.7395 596.4073 -2679 200.0  533.4 945.4
   max
1 4445
2 4606
                             Race-Ethnicity     n     mean       sd     min
1          American Indian or Alaska Native   354 585.6512 565.5531 -1340.0
2                                     Asian  2451 649.2518 620.1701 -1236.0
3                 Black or African American  6199 603.5025 599.2157 -2679.0
4        Demographic Race Two or More Races  1066 619.4379 587.7379 -1062.0
5              Hispanic or Latino Ethnicity  8272 616.0075 598.8102 -1895.0
6 Native Hawaiian or Other Pacific Islander    72 730.2251 582.7930  -153.2
7                                     White 31773 614.4649 602.4646 -2010.0
     Q1 median     Q3  max
1 199.8  492.5  927.0 2627
2 229.2  564.2  986.6 3762
3 185.0  523.7  952.3 3888
4 213.3  532.1  941.7 3044
5 208.6  538.5  950.0 4272
6 325.1  624.0 1005.0 2364
7 197.8  536.6  953.5 4606
  Gender     n     mean       sd   min    Q1 median    Q3  max
1 Female 24344 613.5182 600.0064 -2010 198.9  536.2 949.7 4606
2   Male 25843 616.6530 603.4913 -2679 200.9  537.5 958.9 4445
[1] "Grade:  8 ,  Reading Score"
  Econ Disadvantage Status     n     mean       sd    min    Q1 median    Q3
1                        0 32290 314.0757 202.4790 -497.1 179.0  313.0 449.8
2                        1 17898 309.4325 202.0173 -497.9 174.9  309.9 446.0
   max
1 1159
2 1290
                             Race-Ethnicity     n     mean       sd     min
1          American Indian or Alaska Native   356 305.9883 196.8412 -433.20
2                                     Asian  2449 325.1233 209.1188 -412.80
3                 Black or African American  6196 307.0588 203.6341 -497.90
4        Demographic Race Two or More Races  1065 315.8640 198.8154 -300.60
5              Hispanic or Latino Ethnicity  8263 314.1816 202.8339 -415.50
6 Native Hawaiian or Other Pacific Islander    72 363.1799 188.3806   31.07
7                                     White 31787 311.8698 201.5648 -497.10
     Q1 median    Q3    max
1 167.7  303.6 421.3  855.8
2 186.0  322.4 467.8  987.9
3 167.9  309.3 445.6 1169.0
4 183.5  318.6 453.7 1290.0
5 180.4  314.7 448.8 1019.0
6 238.9  335.3 470.5  843.6
7 177.7  310.4 447.8 1195.0
  Gender     n     mean       sd    min    Q1 median    Q3  max
1 Female 24356 313.2493 201.9476 -497.1 178.4    313 449.5 1192
2   Male 25832 311.6378 202.6803 -497.9 176.6    311 447.5 1290
[1] "Grade:  8 ,  Math Score"
  Econ Disadvantage Status     n     mean       sd   min    Q1 median   Q3  max
1                        0 32290 714.8031 630.5780 -1788 270.2  628.3 1072 4655
2                        1 17898 702.0119 615.9986 -2481 269.9  620.9 1054 4822
                             Race-Ethnicity     n     mean       sd     min
1          American Indian or Alaska Native   356 681.7163 588.2992 -1280.0
2                                     Asian  2449 741.7295 643.0427 -1146.0
3                 Black or African American  6196 700.6656 622.4377 -2481.0
4        Demographic Race Two or More Races  1065 713.2967 612.2081  -883.1
5              Hispanic or Latino Ethnicity  8263 709.7394 622.1026 -1701.0
6 Native Hawaiian or Other Pacific Islander    72 822.6888 611.4625  -174.3
7                                     White 31787 709.7751 626.3405 -1788.0
     Q1 median   Q3  max
1 265.9  585.5 1060 2705
2 303.5  654.7 1095 3987
3 256.1  613.2 1067 4125
4 283.0  634.2 1062 3179
5 276.3  630.3 1053 4517
6 410.5  736.5 1143 2513
7 268.0  625.0 1065 4822
  Gender     n     mean       sd   min    Q1 median   Q3  max
1 Female 24356 708.1860 623.8962 -1788 269.2  624.7 1060 4822
2   Male 25832 712.1797 626.9010 -2481 271.0  625.9 1071 4655
```

**Analytic Technique:** In addition to these raw numbers, it will help to visualize these comparisons, to give us a sense of scale and shape in these gaps. To do so, we will use both box plots and histograms.


```r
# // Step 1: Initialize a set of distinguishable colors for graphics
colors <- c("red","dodgerblue3","green","coral","violet","burlywood2","grey68")

# // Comparison: Box plots and histograms of scores for Econ Disadvantage, Race, and Gender
#Loop over grade levels
for(grade in grades){
  
  data = texas.data[texas.data$grade_level == grade,] #Isolates grade level
  
  #Loop over tested subject
  for(subject in subjects){
    
    #Loop over demographic features
    for(dem in dems){
      
        #Set variables and parameters for our boxplot
        bp <- ggplot(data, aes(x=as.factor(data[,dem]), y=data[,subject])) + 
              geom_boxplot() +
              ggtitle(paste("Grade: ",grade,", ", labels[subject],", by ",labels[dem])) +
              scale_y_continuous(name=labels[subject]) +
              scale_x_discrete(name=labels[dem])
        print(bp) #Print box plot
        
        #Set variables and parameters for our histogram
        h <- ggplot(data, aes(x=data[,subject], fill = as.factor(data[,dem]))) + 
              ggtitle(paste("Grade: ",grade,", ", labels[subject],", by ",labels[dem]))+
              geom_histogram(alpha = 0.5, binwidth = 50) + 
              scale_fill_manual(name=labels[dem],
                                values=colors[1:length(levels(as.factor(data[,dem])))])+
              scale_x_continuous(name=paste(labels[subject], ", Grade", grade)) 
        print(h) #Print histogram
      
    }#End loop over demographic features
    
  } #End loop over tested subject
  
} #End loop over grade level
```

<img src="../figure/E_VisualizeEcoDis-1.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-2.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-3.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-4.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-5.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-6.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-7.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-8.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-9.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-10.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-11.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-12.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-13.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-14.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-15.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-16.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-17.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-18.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-19.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-20.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-21.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-22.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-23.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeEcoDis-24.png" style="display: block; margin: auto;" />


**Analytic Technique:** Next we will extend these comparisons to different student status populations, in relation to special education, migrancy status, and LEP status. It's the same code, but now we just change our demographic indicators to the new ones we'd like to analyze (how convenient!). We start with the descriptive statistics in this first chunk, then move to visualizations in the next chunk.


```r
# // Step 1: Initialize Demographics to Analyze
#Vector with features we want to analyze: Special Education, Migrancy, and LEP Status
##NOTE MIGRANCY DATA JUST 'NAs' SO NOT INCLUDED YET IN ANALYSIS##
dems <- c("iep","lep")

# // Comparisons: Generating summary stats for Spec Ed, Migrancy, and LEP
#Loop over grade levels
for(grade in grades){
  
  data = texas.data[texas.data$grade_level == grade,] #Isolates grade level
  
  #Loop over tested subject
  for(subject in subjects){
    
    print(paste("Grade: ",grade,", ",labels[subject])) #Print subject and grade level
    
    #Loop over demographic features
    for(dem in dems){
      
      a<-Summarize(data[,subject] ~ data[,dem]) #Makes comparison table
      colnames(a)[1] <- labels[dem] #Labels comparison table
      print(a) #Prints comparison table
      
    } #End loop over demogrpahic features
    
  } #End loop over tested subject
  
} #End loop over grade level
```

```
[1] "Grade:  5 ,  Reading Score"
  Spec Ed Enrolled     n     mean       sd    min    Q1 median    Q3  max
1                0 41035 265.3136 201.1999 -534.0 131.4  264.9 400.6 1225
2                1  9152 263.3444 202.5527 -545.4 127.5  262.1 398.8 1017
  LEP Status     n     mean       sd    min    Q1 median    Q3  max
1          0 46610 264.9881 201.4374 -545.4 130.9  264.2 400.3 1225
2          1  3577 264.5164 201.5948 -529.7 127.6  264.4 400.2 1170
[1] "Grade:  5 ,  Math Score"
  Spec Ed Enrolled     n     mean       sd   min    Q1 median    Q3  max
1                0 41035 615.2123 600.9117 -2010 200.2  536.8 954.4 4606
2                1  9152 614.7744 605.7967 -2679 197.2  537.4 953.0 3762
  LEP Status     n     mean       sd   min    Q1 median    Q3  max
1          0 46610 615.0614 601.9573 -2679 199.8  537.4 953.9 4606
2          1  3577 616.0583 599.8229 -1289 200.3  527.7 958.8 3467
[1] "Grade:  8 ,  Reading Score"
  Spec Ed Enrolled     n     mean       sd    min    Q1 median    Q3  max
1                0 39161 312.6652 202.0395 -497.1 178.3  312.1 448.4 1290
2                1 11027 311.5484 203.3411 -497.9 175.5  311.1 449.3 1073
  LEP Status     n     mean       sd    min    Q1 median    Q3  max
1          0 46066 312.1641 202.2760 -497.9 177.5  311.6 448.2 1290
2          1  4122 315.2784 202.8703 -438.6 179.8  315.5 454.4 1192
[1] "Grade:  8 ,  Math Score"
  Spec Ed Enrolled     n     mean       sd   min    Q1 median   Q3  max
1                0 39161 709.7893 624.3217 -1788 269.4  624.9 1066 4822
2                1 11027 711.8476 629.4282 -2481 272.0  627.8 1064 4517
  LEP Status     n     mean       sd   min    Q1 median   Q3  max
1          0 46066 709.8335 625.5590 -2481 269.0  625.9 1065 4822
2          1  4122 714.8019 624.1848 -1129 280.7  621.1 1073 4517
```

Now for visualizations:


```r
# // Comparison: Box plots and histograms of scores for Special Education, Migrancy, and LEP Status
#Loop over grade levels
for(grade in grades){
  
  data = texas.data[texas.data$grade_level == grade,] #Isolates grade level
  
  #Loop over tested subject
  for(subject in subjects){
    
    #Loop over demographic features
    for(dem in dems){
      
        #Set variables and parameters for our boxplot
        bp <- ggplot(data, aes(x=as.factor(data[,dem]), y=data[,subject])) + 
              geom_boxplot() +
              ggtitle(paste("Grade: ",grade,", ", labels[subject],", by ",labels[dem])) +
              scale_y_continuous(name=labels[subject]) +
              scale_x_discrete(name=labels[dem])
        print(bp) #Print box plot
        
        #Set variables and parameters for our histogram
        h <- ggplot(data, aes(x=data[,subject], fill = as.factor(data[,dem]))) + 
              ggtitle(paste("Grade: ",grade,", ", labels[subject],", by ",labels[dem]))+
              geom_histogram(alpha = 0.5, binwidth = 50) + 
              scale_fill_manual(name=labels[dem],
                                values=colors[1:length(levels(as.factor(data[,dem])))])+
              scale_x_continuous(name=paste(labels[subject], ", Grade", grade)) 
        print(h) #Print histogram
      
    }#End loop over demographic features
    
  } #End loop over tested subject
  
} #End loop over grade level
```

<img src="../figure/E_VisualizeStatus-1.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeStatus-2.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeStatus-3.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeStatus-4.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeStatus-5.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeStatus-6.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeStatus-7.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeStatus-8.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeStatus-9.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeStatus-10.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeStatus-11.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeStatus-12.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeStatus-13.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeStatus-14.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeStatus-15.png" style="display: block; margin: auto;" /><img src="../figure/E_VisualizeStatus-16.png" style="display: block; margin: auto;" />

**Possible Next Steps or Action Plans:** Do this for more and/or different grade levels.