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
***
**2. BLUEs_BLUPs.Rmd:** Obtain within-environment BLUEs and weights, across-environment BLUPs, visualize trait distributions, and trait correlations

**2.1:**
- INPUT: Standardize phenotypic data from step 1.3 prepared for spatial analysis
- OUTPUT: Within-environment estimates of spatially corrected Genotype performance (BLUEs) and precision weights fo each trait
- PARAMETER MODIFICATION: Spatial correction strucuture may be adjusted
- Use MrBean interface to fit within-environment models for each trait, treating genotype as a fixed effect to obtain BLUEs and weights
  - Uses AR1 x AR1 ASReml row x column correction for most environments (22THD analyzed using SpATS)
  - Within-environment genetic repeatabilities obtained separately by fitting genotype as a random effect in SpATS

**2.2:**
- INPUT: Dataset containing BLUEs and weights across all environments as assembled from step 2.1
- OUTPUT: Formatted dataset for BLUP modeling
- PARAMETER MODIFICATION: Update trait/weight vectors as needed
- Reads the dataset containing BLUEs and weights, ensures factor conversion of Environment and Genotype columns, converts trait and weight columns to numeric, and defines matching vectors of trait BLUE columns and their corresponding weight columns for mixed model fitting

**2.3:**
- INPUT: Formatted BLUE dataset from step 2.2
- OUTPUT:
  - For each trait:
    - Across-environment genotype BLUPs
    - Variance components
    - Fixed effect estimates for environments
    - Repeatability estimates based on genotype and GxE variance
- PARAMETER MODIFICATION: Update ASReml model specifications as needed
- For each trait:
  - Filters the data to non-missing BLUEs and weights and fits an across-environment mixed model in ASReml
    - Environment is modeled as a fixed effect, genotype and GxE are modeled as random effects
    - Precision weights are used to define the scale of variance, fixing residual variance at 1
  - Extracts variance components and fixed effects
  - Calculates repeatability as the proportion of genetic variance relative to total variance
  - Predicts genotype BLUPs

**2.4:**
- INPUT:
  - Within-environment BLUEs from step 2.2
  - Combined BLUP table from step 2.3
- OUTPUT:
  - For each trait:
    - Faceted histogram of BLUE distribution for WILs
    - Additional facet for BLUP distribution
    - Overlaid vertical markers and labels for the RP and performance checks
- PARAMETER MODIFICATION: Names and colors for RP and checks, histogram bin width and boundary, environment order, label placement settings, and export image dimensions
- Creates histogram plots showing distributions of WIL performance for each trait within each environment and in across-environment BLUPs. The RP and performance checks are overlaid as labeled vertical lines so their positions can be visually compared against the WIL distributions.

**2.5:**
- INPUT: Combined BLUP table from step 2.3
- OUTPUT:
  - Pearson correlation matrix across BLUP traits
  - Correlation heatmap plot
- Computes a Pearson correlation matrix among trait BLUPs, writes the matrix to a .csv, and generates a lower-triangle correlation plot with coefficients displayed on the output .png figure

**2.6:**
- INPUT: Within-environment BLUEs (separate tables for each environment created out of step 2.2 output)
- OUTPUT:
  - For each environment:
    - Pearson correlation matrix across traits
    - Correlation heatmap plot
- For each environment-specific BLUE dataset, converts all trait columns to numeric, computes Pearson correlations, writes the matrix to a .csv, and generates a lower-triangle correlation plot with coefficients displayed on the output .png figure
***
## Genotypic data preparation
**3. snpReady.Rmd:** Reformat SNP data, filter, and generate population genetic summaries 

**3.1:**
- INPUT: Raw genotyping results file
- OUTPUT: Reformatted genotype table with cleaned sample names and retained marker metadata
- PARAMETER MODIFICATION: Update the range of sample columns and the index used when parsing sample names from column headers
- Reads the raw genotype table, subsets the sample columns and reformats sample names, recombining with marker metadata columns to produce a reformatted genotype table. Removes markers with unknown chromosome or position. Converts the genotype matrix to wide format for snpReady.

**3.2:**
- INPUT: Wide format genotype matrix from step 3.1
- OUTPUT:
  - Filtered matrix retaining informative markers with call rate > 0.9, marker heterozygosity < 0.1, and individual heterozygosity < 0.2
  - Population genetic summary metrics
- PARAMETER MODIFICATION: Update filtering thresholds as needed
- Uses snpReady to filter matrix and generate population genetic summaries
  - raw.data() with call.rate to filter
  - popgen() to summarize marker, genotype, and population level statistics
  - Use heterozygosity results from popgen() to create lists of markers and individuals to remove based off excess heterozygosity

**3.3:**
- INPUT: Marker and genotype summary statistics from step 3.2
- OUTPUT: Histograms for MAF and heterozygosity
- PARAMETER MODIFICATION: Update plot colors, axis limits, and threshold lines as needed
- Generates histograms summarizing key marker and individual QC metrics, with threshold lines added to indicate cutoffs used during filtering
