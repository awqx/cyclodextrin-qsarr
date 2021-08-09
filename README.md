# Cyclodextrin QSAR Comparison

This repository will detail the process of creating and comparing QSARs that predict binding affinity of small molecules to  α-, β-, and γ-cyclodextrin.

## Workflow

Overall, the workflow can be followed by running the R Markdown files in order. However, the "zeroth" file, `00-dwnld.Rmd`, can be skipped. This is the least important file for reproducibility and the data generated is already loaded in the repository. Additionally, loading the file is likely to be difficult without an academic VPN or academic credentials to download the data. 

## Steps

### 00. Downloading experimental observations

File: [`00-dwnld.Rmd`](00-dwnld.html)

- Downloads data from Rekharsky and Inoue (1997), Suzuki (2001), and Singh et al (2015)
- Converted Singh data from constant of association to Gibbs free energy change
- Abbreviated as `ri`, `suzuki`, and `singh`
- Basic wrangling of data into data frames
- Cyclodextrin is categorized as "alpha", "beta", "gamma" rather than by Greek letter
- Raw data is stored in `affinity/raw`, derived data is stored in `affinity/derived`

### 01. Cleaning experimental observations

File: [`01-clean.Rmd`](01-clean.html)

- Removed special or unconventional characters
- Cleaned by solvent conditions
- Rekharsky and Inoue data required most cleaning as it comes from multiple compiled sources
  - Additional cleaning is necessary in order to pass the data to `02-sdf.Rmd`
  - Typos in chemical names corrected

### 02. Downloading ligand structures as SDFs

File: [`02-sdf.Rmd`](02-sdf.html)

- Structure data files (SDFs) downloaded into `sdf/`
  - Subdirectories for each data source
  - Additional subdirectory for compiled data from each source
- Queried Chemical Identifier Resolver from NCI https://cactus.nci.nih.gov/chemical/structure
- All observations successfully downloaded
- Observations compiled into single SDFs
- The directories of individual SDFs is not uploaded onto GitHub for convenience
  - Combined SDF file is backed up in GitHub

### 03. Calculating chemical descriptors using CDK for R

File: [`03-cdk.Rmd`](03-cdk.html)

- Using package `rcdk` to obtain chemical descriptors
  - https://cran.r-project.org/web/packages/rcdk/vignettes/using-rcdk.html
- Calculates 281 descriptors for each molecule
- Does not calculate 3D descriptors
  - 3D descriptors may not be reliable due to lack of optimization

### 04. Other sources for chemical descriptors

File: [`04-desc_external.Rmd`](04-desc_external.html)

- Incorporation of descriptors from other sources
- PaDEL-Descriptor
  - http://yapcwsoft.com/dd/padeldescriptor/
- Online chemical database with modeling environment
  - https://ochem.eu
- Mordred
  - https://github.com/mordred-descriptor/mordred

### 05. Joining affinity data to descriptors

File: [`05-join.Rmd`](05-join.html)

- Combining affinity data and the descriptors
- Requires addition of guest names for OCHEM and Mordred descriptors
- Requires renaming of the column `Name` in results from PaDEL

### 06. Creating external validation sets

File: [`06-extval.Rmd`](06-extval.html)

- 15% of the data is set aside for external validation
- Data for external validation in directory `extval/`
- Data for model building and training in directory `trn/`
  
### 07. Preprocessing chemical descriptors

File: [`07-preprocess.Rmd`](07-preprocess.html)

- Common preprocessing functions applied to data
  - Centering and scaling
  - Removal of variables with near zero variance
  - Removal of molecules with large amounts of missing data
- If more than 5% of the chemical descriptors of a molecule are `NA` or `NaN`, the molecule is removed from the data. Otherwise, the descriptors are filled in using a simple mean.
- These steps are performed on the data saved in `trn`
- Removal of X-outliers 
  - Use of algorithm discussed here: https://doi.org/10.1016/j.chemolab.2015.04.013
  - Assumptions of standard deviation of descriptors
  
### 08. Random forest proof of concept

File: [`08-rf_poc.Rmd`](08-rf_poc.html)

- Building of a random forest
- Model built on all variables as well as variables selected by `caret::rfe`
- Tuning with new functions and existing `caret::train` functions
- Analyzing model performance over repetitions
- Decision that building models every time with RFE is too time intensive, leading to step 09
- Models saved in `model/rf_pov/`

### 09. Feature selection

File: [`09-rfe.Rmd`](09-rfe.html)

- Use of `caret::rfe` to select features
- Running a single iteration with random forest functions only
  - 10 repeats of 10-fold cross-validation
- Requires multiple core processing to cut down on time
- Top 16 variables saved
- `"rfe"` objects saved in `rfe/`
- Variables saved in `rfe_var/`

### 10. Random forest (RF)

File: [`10-rf.Rmd`](10-rf.html)

- Random forest model built
- Tuning for two sets of variables: full variable set and selected variables
- Results from 10 repeats of 10-fold cross-validation

### 11. Support Vector Machine (SVM)

#### a. Linear Kernel

File: [`11a-svm_linear.Rmd`](11a-svm_linear.html)

- Built support vector machine (SVM) using the linear kernel
- SVMs take a relatively long time to tune due to the number of available variables


## Notes on workflow

Because the data sources already completed screening of molecule names, there was no cleaning step to verify uniqueness of all ligands. In the future, this would be an appropriate step to take in order to curate the data.

## Acknowledgements

This research was performed at Horst von Recum's lab at Case Western Reserve University under the mentorship of Edgardo Rivera-Delgado. 