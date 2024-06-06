# Advanced Polygenic Risk Score Analyses

## Day 3 - Polygenic Risk Score Analyses Workshop 2024

## Table of Contents

  1. [Key Learning Outcomes](#Key-learning-outcomes)
  2. [Method for calculating PRS](#Method-for-calculating-PRS)
  3. [Base and target datasets](#Base-and-target-datasets)
  4. [Downloading Datasets](#Downloading-Datasets) 
  5. [Exercise 1 Estimating R<sup>2</sup> ](#exercise-1-estimating-r2)
  6. [Exercise 2 Overfitting caused by model optimisation](#exercise-2-Overfitting-caused-by-model-optimisation)
  7. [Introduction to cross-ancestry PRS computation](#Cross-ancestry-PRS-computation)
  8. [Cross-ancestry PRS analysis using PRS-CSx](#Cross-ancestry-PRS-computation)

     
## Day 3a practical
## Key Learning Outcomes
After completing this practical, you should be able to:
  1. Compute and analyse ancestry-matched and unmatched PRS using PRSise-2.
  2. Understand and interpret the results from PRSise-2 derived scores. 
  3. Understand and identify the impact of ancestry on the predictive utility of PRS.
  4. Understand and identify the impact of sample size on the predictive utility of PRS.
  5. Understand the challenges and limitations of applying PRS in populations with diverse genetic backgrounds.


## Method for calculating PRS
For this practical we will use PRSice-2. PRSice-2 is one of the dedicated PRS calculation and analysis programs that makes use of a sequence of PLINK functions.
The tools utilises the standard clumping and thresholding (C+T) approach.

## Base and target datasets 
In this practical, we will compute a PRS for systolic blood pressure (SBP) and assess it performance across European and African ancestry datasets to clearly illustrate the portability problem. 
We will assess the predictive utility of 4 scores:

**ANCESTRY-MATCHED**
1. EUR base - EUR target: Utilise European summary statistics as the base data and individual-level genotyped data from Europeans as the target dataset. 
2. AFR base - AFR target: Utilise African summary statistics as the base data and individual-level genotyped data from Africans as the target dataset.

**ANCESTRY-UNMATCHED**
1. EUR base - AFR target: Utilise European summary statistics as the base data and individual-level genotyped data from Africans as the target dataset. 
2. AFR base - EUR target: Utilise African summary statistics as the base data and individual-level data from genotyped European as the target dataset.

Please note that the sample sizes of the individual-level target data are as follows: Europeans (n = 500) and Africans (n = 650). 
Also note that this is simulated data with no real-life biological meaning or implication. 


|**Phenotype**|**Source**|**Description**|**Download Link**|
|:---:|:---:|:---:|:---:|
|Base dataset (EUR)|[Link](https://replace)|GWASsumary stats of SBP| [Link](https://change)|
|Target dataset (EUR, AFR)|simulated |EUR (n = 500), AFR (n = 650)| [Link](http://change)|


## Downloading Datasets

The required datasets for this part of the practical can be downloaded at:       

[Download Backup WeTransfer](https://we.tl/t-YO8YbDjDK7)        

The data will be downloaded into your "Downloads" folder. You will need to move it to right directory, using the following commands.



```
cd data
mv ~/Downloads/Day_3.zip .
unzip Day_3.zip
cd Day_3
```

All required material for this practical is found in the **data/Day_3a** directory. Relevant materials that you should see there at the start of the practical are as follows:

 📂: Base_Data
  - GIANT_Height.txt,
  - cad.add.txt,
  - cad.add.readme.

 📂: Target_Data
  - TAR.fam
  - TAR.bim
  - TAR.bed
  - TAR.height
  - TAR.cad 
  - TAR.covariate
    
  📁: Reference files
   - Homo_sapiens.GRCh38.86.gt
   - Sets.gmt
     
 🛠️: Software
  - PRSice.R 
  - PRSice_linux
  - nagelkerke.R
  - Quantile.R
    
---
> 
> ‼️ Note that all target phenotype data in this worshop are **simulated**. They have no specific biological meaning and are for demonstration purposes only. 
> 
---
<a href="#top">[Back to Top](#table-of-contents)</a>

