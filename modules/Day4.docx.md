**Step 1: Set up environment**
------------------------------

**AFR**
```
for chr in {1..22}; do \
./software/plink_mac \
	--bfile ./data/AFR_1kg.hm3.only.csx \
	--chr $chr \
	--make-bed \
	--out ./data/hm3_by_ancestry/AFR_1kg.hm3.chr${chr}_only.csx;
done
```

**EUR**
```
for chr in {1..22}; do \
./software/plink_mac \
	--bfile ./data/EUR_1kg.hm3.only.csx \
	--chr $chr \
	--make-bed \
	--out ./data/hm3_by_ancestry/EUR_1kg.hm3.chr${chr}_only.csx;
done
```

**Set up the necessary environment variables for threading and verify they are set correctly.**
```
export N_THREADS=10
export MKL_NUM_THREADS=$N_THREADS
export NUMEXPR_NUM_THREADS=$N_THREADS
export OMP_NUM_THREADS=$N_THREADS
```
**Verify the variables are set**
```
echo $N_THREADS
echo $MKL_NUM_THREADS
echo $NUMEXPR_NUM_THREADS
echo $OMP_NUM_THREADS
```
<br>

Step 2: Run CSX - Derive new SNPs weights trained on European and African summary stats
--------------------------------------------------------------------------------------

**Generate job file containing the threaded PRScsx commands.**
```
for chr in {1..22}; do
  echo "python3 ./software/PRScsx.py \
	--ref_dir=./reference/csx \
	--bim_prefix=./data/hm3_by_ancestry/AFR_1kg.hm3.chr${chr}_only.csx \
	--sst_file=./data/sumstats_by_chr/EUR-SBP-simulated.sumstats.chr${chr},./data/sumstats_by_chr/AFR-SBP-simulated.sumstats.chr${chr} \
	--n_gwas=25732,4855 \
	--chrom=$chr \
	--n_iter=1000 \
	--n_burnin=500 \
	--thin=5 \
	--pop=EUR,AFR \
	--phi=1e-4 \
	--out_dir=./out/csx \
	--out_name=afr.target_chr${chr}.csx" >> multithread_job.sh
done
```
**Check the job file contains the correct commands.**
```
head multithread_job.sh
```
**Run the Job File with GNU Parallel:**
```
parallel --verbose --jobs $N_THREADS < multithread_job.sh
```
**If the above command does not work you can create an alternative version of the same script by running the following:**
```
./script/create_multithread.sh
```
<br>

Step 3: Combine CSX-derived SNP weights across chromosomes
----------------------------------------------------------

**Load necessary library**
```
library(dplyr)
```
**Define the path to the directory containing the PRS-CSX output files**
```
path <- "/Users/iyegbc01/Dropbox/StatGen_OReilly_group/PRS_Summer_School/PRS_Summer_school_Wellcome_Trust_2023/Day4_WCS_2024_draft/out/csx/"
```
**Define the ancestry you want to combine ("EUR" or "AFR")**
```
ancestry <- "EUR"
```
**Initialize an empty data frame to store the combined data**
```
combined_data <- data.frame()
```
**Loop through chromosomes 1 to 22, (currently excluding chromosome 3)**
```
for (chr in setdiff(1:22, 3)) {
  # Construct the file name
  file_name <- paste0("afr.target_chr", chr, ".csx_", ancestry, "_pst_eff_a1_b0.5_phi1e-04_chr", chr, ".txt")
  file_path <- file.path(path, file_name)
  
  # Check if file exists before reading
  if (file.exists(file_path)) {
	# Read the data from the file
	data <- read.table(file_path, header = FALSE, sep = "\t", col.names = c("CHR", "rsid", "pos", "ref", "alt", "beta"))
	
	# Combine the data
	combined_data <- rbind(combined_data, data)
  } else {
	warning(paste("File not found:", file_path))
  }
}

```
**Write the combined data to a new file**
```
output_file <- file.path(path, paste0("combined_", ancestry, "_pst_eff.txt"))
write.table(combined_data, output_file, sep = "\t", row.names = FALSE, col.names = TRUE, quote = FALSE)
```
### Task: Replace 'ancestry <- "EUR" ' with 'ancestry <- "AFR" ' and repeat the subsequent steps above (i.e. from "Initialize an empty data frame" onwards)
<br>

# Step 4: Merge genotype-phenotype data
-------------------------------------

**Prepare data**

```
# Load libraries
library(data.table)
library(ggplot2)
library(snpStats)

# Define the path to the directory containing the PLINK files and phenotypic data
plink_path <- "/Users/iyegbc01/Dropbox/StatGen_OReilly_group/PRS_Summer_School/PRS_Summer_school_Wellcome_Trust_2023/Day4_WCS_2024_draft/data/"

# Read PLINK files and phenotype data into R
bim_file <- file.path(plink_path, "AFR_1kg.hm3.only.csx.bim")
fam_file <- file.path(plink_path, "AFR_1kg.hm3.only.csx.fam")
bed_file <- file.path(plink_path, "AFR_1kg.hm3.only.csx.bed")
pheno_file <- file.path(plink_path, "sbp_afr_1kg.sim_pheno")

# Read the genotype data using snpStats
geno_data <- read.plink(bed_file, bim_file, fam_file)

# Extract SNP IDs from geno_data$map$snp.name
snp_ids <- geno_data$map$snp.name
if (is.null(snp_ids) || !is.character(snp_ids)) {
  stop("SNP IDs are missing or not in the correct format.")
}

# Convert the genotype data to a matrix and then to a data.table
geno_matrix <- as(geno_data$genotypes, "matrix")
geno_df <- data.table(geno_matrix)
setnames(geno_df, snp_ids)

# Add IID column from the fam file
geno_df[, IID := geno_data$fam$member]

# Read the phenotypic data**
pheno_data <- fread(pheno_file, sep = " ", header = TRUE)
```

**Merge phenotype and genotype data**
```
# Do merge
combined_data <- merge(pheno_data, geno_df, by = "IID")

# Keep only genotype columns
geno <- combined_data[, !names(combined_data) %in% c("FID", "IID", "pheno100", "pheno20", "pheno33", "pheno10"), with = FALSE]
phen <- combined_data$pheno100

# Convert geno to numeric matrix**
geno <- as.matrix(geno)
mode(geno) <- "numeric"

# Convert phen to vector
phen <- as.vector(phen)
```
<br>

Step 5: Split data into validation and test sets 
------------------------------------------------
**Specify Proportion**
```
# Here we specify that 40% of all IDs will be used to construct the validation group
set.seed(154)
vali_proportion <- 0.4
vali_size <- round(nrow(geno) * vali_proportion)
vali_indices <- sample(1:nrow(geno), vali_size, replace = FALSE)
test_indices <- setdiff(1:nrow(geno), vali_indices)
```
**Subsetting of individuals**
```
X_vali <- geno[vali_indices, , drop=FALSE]
y_vali <- phen[vali_indices]
X_test <- geno[test_indices, , drop=FALSE]
y_test <- phen[test_indices]
```
<br>

Step 6: Prepare the regression model input using the CSX-derived AFR and EUR weights
-------------------------------------------------------------------------------------
```
# Read the merged CSX output files
AFR_betas <- fread(file.path(plink_path, "../out/csx/combined_AFR_pst_eff.txt"), sep = "\t", header = TRUE)
EUR_betas <- fread(file.path(plink_path, "../out/csx/combined_EUR_pst_eff.txt"), sep = "\t", header = TRUE)

# Assuming the beta files have columns: "CHR", "rsid", "pos", "ref", "alt", "beta"
overlap_prs <- merge(AFR_betas, EUR_betas, by = "rsid", suffixes = c("_afr", "_eur"))

# Filter overlap_prs to include only SNPs present in X_vali
overlap_prs <- overlap_prs[rsid %in% colnames(X_vali)]

# Ensure that X_vali and X_test only contain SNPs present in W_afr and W_eur
common_snps <- intersect(colnames(X_vali), overlap_prs$rsid)
X_vali <- X_vali[, common_snps, drop=FALSE]
X_test <- X_test[, common_snps, drop=FALSE]

# Reorder the columns of X_vali and X_test to match the order of SNPs in overlap_prs
X_vali <- X_vali[, match(overlap_prs$rsid, colnames(X_vali)), drop=FALSE]
X_test <- X_test[, match(overlap_prs$rsid, colnames(X_test)), drop=FALSE]

# Extract the overlapping rsid and their corresponding betas
W_afr <- overlap_prs$beta_afr
W_eur <- overlap_prs$beta_eur

# Convert W_afr and W_eur to numeric vectors
W_afr <- as.numeric(W_afr)
W_eur <- as.numeric(W_eur)
```
<br>

Step 7: Prepare the variant weights matrices as vectors
-------------------------------------------------------
```
# Pre-check the alignment between the different objects
if (ncol(X_vali) != length(W_afr) || ncol(X_vali) != length(W_eur)) {
  stop("Dimensions of X_vali and W_afr/W_eur do not match.")
}

# In the validation sample: 

# (i) Compute XWafr_vali
XWafr_vali <- X_vali %*% W_afr
# (ii) Convert XWafr_vali to have zero mean and unit variance
XWafr_vali_z <- scale(XWafr_vali)

# (iii) Compute XWeur_vali
XWeur_vali <- X_vali %*% W_eur
# (iv) Convert XWeur_vali to have zero mean and unit variance
XWeur_vali_z <- scale(XWeur_vali)

# (v) Combine the normalized matrices
XW_vali <- cbind(XWafr_vali_z, XWeur_vali_z)

# Fit the model
model <- lm(scale(y_vali) ~ XWafr_vali_z + XWeur_vali_z - 1) # '- 1' removes the intercept

# Obtain the regression parameters
a_hat <- coef(model)[1]
b_hat <- coef(model)[2]
print(paste("a_hat =", a_hat))
print(paste("b_hat =", b_hat))
```
<br>

Step 8: Predict phenotype on validation and test dataset
--------------------------------------------------------
**Generate a linear combination of AFR and EUR PRSs for each individual**
```
# Each ancestry component is weighted by the regression coefficient of that ancestry, in the preceding step
y_hat_vali <- a_hat * XWafr_vali_z + b_hat * XWeur_vali_z

# In the test sample: 

# Compute XWafr_test and XWeur_test
XWafr_test <- X_test %*% W_afr
XWafr_test_z <- scale(XWafr_test)

XWeur_test <- X_test %*% W_eur
XWeur_test_z <- scale(XWeur_test)

# y_hat in the test sample
y_hat <- a_hat * XWafr_test_z + b_hat * XWeur_test_z
```
<br>

Step 9: Plot phenotype distributions of validation and test data:
---------------------------------------------------------------------------
**Check that both distributions are approximately normal**
```
library(ggplot2)

# Create data frames for validation and test sets
vali_data <- data.frame(trait = y_vali, dataset = "Validation")
test_data <- data.frame(trait = y_test, dataset = "Test")

# Combine both data frames
combined_data <- rbind(vali_data, test_data)

# Plot the distributions
ggplot(combined_data, aes(x = trait, fill = dataset)) +
  geom_density(alpha = 0.5) +
  labs(title = "Trait Distributions for Validation and Test Sets", x = "Trait Value", y = "Density") +
  theme_minimal()
```
<br>

Step 10: Plot true values against predicted values
--------------------------------------------------
**The next steps use standard normal phenotype data to reduce scale differences between PRS and trait values**
```
min_true <- min(min(y_vali), min(y_test))
max_true <- max(max(y_vali), max(y_test))
min_pred <- min(min(y_hat_vali), min(y_hat))
max_pred <- max(max(y_hat_vali), max(y_hat))

pdf("true_against_pred.pdf", width = 10, height = 5)
par(mfrow = c(1, 2))

plot(scale(y_vali), y_hat_vali, pch = 19, col = rgb(0, 0, 0, 0.5), xlab = 'True Values', ylab = 'Predicted Values', main = 'Validation Dataset')
abline(0, 1, col = 'red', lty = 2)

plot(scale(y_test), y_hat, pch = 19, col = rgb(0, 0, 0, 0.5), xlab = 'True Values', ylab = 'Predicted Values', main = 'Test Dataset')
abline(0, 1, col = 'red', lty = 2)

dev.off()

```
Step 11: Calculate deviance-based _R<sup>2</sup>_
-------------------------------------
```
# Calculate the deviance (SS_res)
deviance <- sum((scale(y_test) - y_hat) ^ 2)
print(paste("deviance =", deviance))

# Calculate the mean of the scaled y_test
y_test_mean <- mean(scale(y_test))

# Calculate the null deviance (SS_tot)
deviance_null <- sum((scale(y_test) - y_test_mean)^ 2)
print(paste("deviance_null =", deviance_null))

# Calculate R2
R2 <- 1 - (deviance / deviance_null)
print(paste("R2 =", R2))
```
