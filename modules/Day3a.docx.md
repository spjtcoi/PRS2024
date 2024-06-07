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
For this practical we will use PRSice-2. PRSice-2 is one of the dedicated PRS calculation and analysis programs that makes use of a sequence of PLINK functions. The tools utilises the standard clumping and thresholding (C+T) approach.

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


|**Phenotype**|**Source**|**Description**|
|:---:|:---:|:---:|
|Base dataset (EUR, AFR)|simulated |GWAS summary stats of SBP|
|Target dataset (EUR, AFR)|simulated |EUR (n = 500), AFR (n = 650)| 


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

##### Key parameters to the code

Note: The parameters listed in this table remain consistent across various scenarios, but the specific values may change based on the dataset and analysis scenario. For illustrative purposes, this table uses the scenario of an African base and African target population:  

|**Parameter**|**Value**|**Description**|
|:---:|:---:|:---:|
|prsice|PRSice_xxx|Informs PRSice.R that the location of the PRSice binary|
|base|Height.QC.gz|Specifies the GWAS summary statistics file for input|
|A1|A1|Column name for the effect allele in the GWAS summary statistics|
|p value|P|Column name for the p-values of SNPs in the GWAS summary statistics|
|no clump|-|Instructs PRSice to skip the clumping process, which is used to remove SNPs in linkage disequilibrium|
|beta|-|Indicates that the effect sizes are given in beta coefficients (linear regression coefficients)|
|snp|SNP|Column name for SNP identifiers in the GWAS summary statistics|
|score|sum|Specifies that the score calculation should sum the product of SNP effect sizes and their genotype counts|
|target|AFR_1kg.hm3.only.csx|The genotype data file for the target sample|
|binary-target|F|Indicates that the phenotype of interest is not binary (e.g., quantitative trait), F for no|
|pheno|sbp_afr_1kg.sim_pheno|Specifies the file containing phenotype data|
|pheno-col|pheno100|Column name in the phenotype file that contains the phenotype data to be used for PRS calculation|
|cov|EUR.covariate|Specifies the file containing covariate data for the analysis|
|base-maf|MAF:0.01|Filter out SNPs with MAF < 0.01 in the GWAS summary statistics, using information in the MAF column|
|base-info|INFO:0.8|Filter out SNPs with INFO < 0.8 in the GWAS summary statistics, using information in the INFO column|
|stat|OR|Column name for the odds ratio (effect size) in the GWAS summary statistics|
|or|-|Inform PRSice that the effect size is an Odd Ratio|
|thread|8|Specifies the number of computing threads to use for the analysis|
|out|SBP_trial.afr.afr|Specifies the name for the output files generated by PRSice|

This will automatically perform "high-resolution scoring" and generate the "best-fit" PRS (e.g. in EUR.best), with associated plots of the results.

##### Code: Predicting from African training to African target data
```sh
Rscript /Volumes/Chris-1/PRS24/Data_Day4/software/PRSice.R \
--prsice /Volumes/Chris-1/PRS24/Data_Day4/software/PRSice_mac \
--base /Volumes/Chris-1/PRS24/Data_Day4/data/AFR-SBP-simulated.sumstats.prscsx \
--A1 A1 \
--pvalue P \
--no-clump \
--beta \
--snp SNP \
--score sum \
--target /Volumes/Chris-1/PRS24/Data_Day4/data/AFR_1kg.hm3.only.csx \
--binary-target F \
--pheno /Volumes/Chris-1/PRS24/Data_Day4/data/sbp_afr_1kg.sim_pheno \
--pheno-col pheno100 \
--thread 8 \
--out /Volumes/Chris-1/PRS24/Data_Day4/out/SBP_trial.afr.afr
```
##### Code: Predicting from European training to African target data
```sh
Rscript /Volumes/Chris-1/PRS24/Data_Day4/software/PRSice.R \
--prsice /Volumes/Chris-1/PRS24/Data_Day4/software/PRSice_mac \
--base /Volumes/Chris-1/PRS24/Data_Day4/data/EUR-SBP-simulated.sumstats.prscsx \
--A1 A1 \
--pvalue P \
--no-clump \
--beta \
--snp SNP \
--score sum \
--target /Volumes/Chris-1/PRS24/Data_Day4/data/AFR_1kg.hm3.only.csx \
--binary-target F \
--pheno /Volumes/Chris-1/PRS24/Data_Day4/data/sbp_afr_1kg.sim_pheno \
--pheno-col pheno100 \
--thread 8 \
--out /Volumes/Chris-1/PRS24/Data_Day4/out/SBP_trial.afr.by.eur
```

##### Code: Predicting from European training to European target data
```sh
Rscript /Volumes/Chris-1/PRS24/Data_Day4/software/PRSice.R \
--prsice /Volumes/Chris-1/PRS24/Data_Day4/software/PRSice_mac \
--base /Volumes/Chris-1/PRS24/Data_Day4/data/EUR-SBP-simulated.sumstats.prscsx \
--A1 A1 \
--pvalue P \
--no-clump \
--beta \
--snp SNP \
--score sum \
--target /Volumes/Chris-1/PRS24/Data_Day4/data/EUR_1kg.hm3.only.csx \
--binary-target F \
--pheno /Volumes/Chris-1/PRS24/Data_Day4/data/sbp_eur_1kg.sim_pheno \
--pheno-col pheno100 \
--thread 8 \
--out /Volumes/Chris-1/PRS24/Data_Day4/out/SBP_trial.eur.eur  
```

##### Code: Predicting from African training to European target data
```sh
Rscript /Volumes/Chris-1/PRS24/Data_Day4/software/PRSice.R \
--prsice /Volumes/Chris-1/PRS24/Data_Day4/software/PRSice_mac \
--base /Volumes/Chris-1/PRS24/Data_Day4/data/AFR-SBP-simulated.sumstats.prscsx \ 
--A1 A1 \
--pvalue P \
--no-clump \
--beta \
--snp SNP \
--score sum \
--target /Volumes/Chris-1/PRS24/Data_Day4/data/EUR_1kg.hm3.only.csx \
--binary-target F \
--pheno /Volumes/Chris-1/PRS24/Data_Day4/data/sbp_eur_1kg.sim_pheno \
--pheno-col pheno100 \
--thread 8 \
--out /Volumes/Chris-1/PRS24/Data_Day4/out/SBP_trial.eur.by.afr
 ```
---
> 
> ‼️ Note that all target phenotype data in this worshop are **simulated**. They have no specific biological meaning and are for demonstration purposes only. 
> 
---
<a href="#top">[Back to Top](#table-of-contents)</a>

