# TdGD Pop80 GWAS - Pipeline Description
## Phenotypic data preparation and analysis
**1. Raw_data_standardization.Rmd:** Within-environment standardization of raw phenotypic data

**1.1:**
- INPUT: Raw phenotypic dataset
- OUTPUT: Cleaned raw dataset with appropriate column types (factor and numeric)
- PARAMETER MODIFICATION: Update input file path and column names for factor conversion as needed
- Reads the raw phenotypic data, convert specified columns to factors, and convert remaining columns with phenotypes to numeric

**1.2:**
- INPUT: Cleaned raw dataset from step 1.1
- OUPUT: Summary table for all numeric traits containing minimum, maximum, mean, and standard deviations
- Identifies all numeric columns in the dataset and calculates basic summary statistics for each trait for inspection of trait scales prior to standardization

**1.3:**
- INPUT: Cleaned raw dataset from step 1.1
- OUTPUT:
  - Dataset with new standardized trait columns prefixed by std_
  - Console output verifying that the standardization worked correctly, reporting trait means of 0 and trait standard deviations of 1
- PARAMETER MODIFICATION: Update the list of traits to standardize
- For each listed trait, performs within-environment standardization so that observations are centered and scaled separately within each environment, with standardized values added as new columns, retaining original raw trait values
