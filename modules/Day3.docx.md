# Advanced Polygenic Risk Score Analyses

## Day 3 - Polygenic Risk Score Analyses Workshop 2024

## Table of Contents

  1. [Key Learning Outcomes](#key-learning-outcomes)
  2. [Resources you will be using](#resources-you-will-be-using)
  3. [Datasets](#data-sets) 
  4. [Exercise 1 Estimating R<sup>2</sup> in case and control studies](#exercise-1-estimating-r2-in-case-and-control-studies)
  5. [Exercise 2 Overfitting caused by model optimisation](#exercise-2-Overfitting-caused-by-model-optimisation)
     1. [Out of Sample Validation](#out-of-sample-validation)
  7. [Exercise 3 Distribution of PRS](#exercise-3-distribution-of-prs)
  8. [Gene Set Analysis](#gene-set-analysis)
     1. [Molecular Signatures Database MSigDB](#molecular-signatures-Database-msigdb)
     2. [General Transfer Format file](#general-transfer-format-file)
     3. [Browser Extensible Data BED](#browser-extensible-data-bed)
  9. [Gene Set Enrichment Analysis](#gene-set-enrichment-analysis)
  10. [Exercise 4 Gene Set Based PRS Analysis](#exercise-4-gene-set-based-prs-analysis)
      

## Key Learning Outcomes
After completing this practical, you should be able to:
  1. Know how to adjust for ascertainment bias in case-control analysis
  2. Know how over-fitting aﬀects PRS results and how to handle it 
  3. Understand xxxxhsdlgbl b
  4. understand distribution of PRS
  5. understand diﬀerent file formats involved in gene-set analysis
  6. understand diﬀerence between self-contained and competitive gene-set analyses
     

## Base and target datasets 
In this practical, we will compute a PRS for systolic blood pressure (SBP) and assess it performance across European and ancestry datasets to clearly illustrate the portability problem. The base dataset will be summary statistics from Europeans and target will be individual level genotyped data from Europeans (n = 500) and Africans (n = 650). Please be reminded that this is simulated data with no real-life biological meaning or implication. 

|**Phenotype**|**Provider**|**Description**|**Download Link**|
|:---:|:---:|:---:|:---:|
|Base dataset (EUR)|[GIANT Consortium](https://portals.broadinstitute.org/collaboration/giant/index.php/GIANT_consortium_data_files)|GWASsumary stats of SBP| [Link](https://portals.broadinstitute.org/collaboration/giant/images/0/01/GIANT_HEIGHT_Wood_et_al_2014_publicrelease_HapMapCeuFreq.txt.gz)|
|Target dataset (EUR, AFR)|[CARDIoGRAMplusC4D Consortium](http://www.cardiogramplusc4d.org/)|EUR (n = 500), AFR (n = 650)| [Link](http://www.cardiogramplusc4d.org/media/cardiogramplusc4d-consortium/data-downloads/cad.additive.Oct2015.pub.zip)|


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


