---
layout: default
title: "Module 2: Alpha Diversity, Beta Diversity, and Differential Abundance"
show_sidetoc: true
header_type: base
permalink: /docs/tutorials/cbw-2026-module2/
---

## Introduction

**Author**: Jacob T. Nearing

In the previous lab, we learned how to process raw amplicon sequencing data into feature tables. In this module, we will apply the concepts introduced in lecture to perform several common downstream analyses of amplicon sequencing data.

Specifically, we will cover:

- Loading amplicon sequencing data into R
- Calculating and comparing alpha diversity
- Calculating and comparing beta diversity
- Performing differential abundance testing with MaAsLin3

All analyses in this module will be conducted in R/RStudio.

> <i class="fa-solid fa-circle-info"></i> Connecting to Rstudio <br><br>
> Connecting to Rstudio on your workshop instances is covered in the pre-work shop material
> for a refresher checkout the link below.
> https://bioinformaticsdotca.github.io/MIC_Gue-2609/r.html
{: .alert .alert-info .p-3}

## Setup

First, load the R packages needed for this lab.

- **dplyr** and **tidyr**: Tools for cleaning, transforming, reshaping, and summarizing data
- **ggplot2**: Tools for creating plots
- **vegan**: Tools for ecological and community analysis, including diversity measures and ordination
- **Maaslin3**: A statistical modeling tool for identifying associations between microbial features and metadata
- **GUniFrac**: Methods for calculating UniFrac distances in R
- **ape**: Tools for loading and manipulating phylogenetic trees in R

```r
library(dplyr)
library(tidyr)
library(ggplot2)
library(vegan)
library(Maaslin3)
library(GUniFrac)
library(ape)
```

## Loading the Data

> <i class="fa-solid fa-circle-info"></i> BEFORE YOU START <br><br>
> Below, we load the amplicon sequencing data used in this lab. As with the previous lab there are three choices 16S, ITS, and 18S data. The below commands are provided for the 16S data, however, those up for an extra challenge may choose one of the other datasets and edit the code accordingly.
> Additionally The code chunks below contain comments to help explain what each command is doing. Examine them closely!


```r
# Read in the feature table containing taxonomy information
counts <- read.table(
  "amplicon_data/16S_Blueberry/final_output_exported/feature-table_w_tax.txt",
  sep = "\t",
  header = TRUE,
  comment.char = "",
  skip = 1,
  check.names = FALSE
)

# Set the row names of the count table to the ASV IDs
rownames(counts) <- counts[, 1]

# Remove the taxonomy and ASV ID columns from the abundance table
counts_no_taxa <- counts %>%
  select(-c(taxonomy, `#OTU ID`))

# Read in the sample metadata
metadata <- read.table(
  "amplicon_data/16S_Blueberry/metadata.tsv",
  sep = "\t",
  header = TRUE,
  check.names = FALSE
)

# Rename the first metadata column to "Sample"
colnames(metadata)[1] <- "Sample"

head(metadata)
```


## Alpha Diversity

Alpha diversity describes the diversity within individual samples. In this section, we will examine sequencing depth, calculate several alpha diversity metrics, and compare diversity between sample categories.

### Rarefaction

The first step in examining alpha diversity is to evaluate the relationship between sequencing depth and ASV detection. Rarefaction curves allow us to determine whether samples are approaching a plateau in the number of observed ASVs as sequencing depth increases.

The `vegan` package expects samples to be rows and features or taxa to be columns. We therefore transpose the feature table.

```r
# Transpose the feature table so that samples are rows
counts_no_taxa_t <- as.data.frame(t(counts_no_taxa))

# Determine the minimum sequencing depth across all samples
min_depth <- min(rowSums(counts_no_taxa_t))

min_depth
```

We can now generate rarefaction curves for each sample.

```r
rarecurve(
  x = counts_no_taxa_t,
  step = 20,
  sample = min_depth
)
```

<img width="1400" height="865" alt="image" src="https://github.com/user-attachments/assets/528ebc87-2b4f-4c6a-80ad-d61f8999ff73" />


At the minimum sequencing depth, the rarefaction curves appear to plateau to some degree. This suggests that the selected depth is fairly reasonable for comparing diversity among samples.

In the literature, rarefaction is often performed using a single random subsample. However, because rarefaction is stochastic, a more robust approach is to repeat the process multiple times at the selected sequencing depth.

We will first go through the code to perform rarefaction with a single subsample.

```r
# Set a seed to make the results reproducible
set.seed(123)

# Perform one rarefaction for each sample
counts_no_taxa_t_rare <- rrarefy(
  counts_no_taxa_t,
  sample = min_depth
)

# Confirm that all samples have the same read depth
rowSums(counts_no_taxa_t_rare)
```

We can calculate Shannon diversity from the rarefied count table.

```r
shannon_div <- data.frame(
  Sample = rownames(counts_no_taxa_t_rare),
  Shannon = diversity(
    counts_no_taxa_t_rare,
    index = "shannon"
  )
)

head(shannon_div)
```

We can repeat the rarefaction process 100 times and calculate multiple diversity metrics for each iteration to reduce some of the randomness caused by subsampling.

```r
# Save the sample names
sample_names <- rownames(counts_no_taxa_t)

set.seed(123)

# Perform 100 independent rarefactions
results <- lapply(seq_len(100), function(i) {

  # Rarefy the count table once
  rarefied <- vegan::rrarefy(
    counts_no_taxa_t,
    sample = min_depth
  )

  # Calculate diversity metrics
  data.frame(
    Sample = sample_names,
    Iteration = i,
    Observed_Richness = rowSums(rarefied > 0),
    Shannon = vegan::diversity(
      rarefied,
      index = "shannon"
    ),
    Simpson = vegan::diversity(
      rarefied,
      index = "simpson"
    )
  )
})

# Look at the top of the first table in results
head(results[[1]])

# If we wanted to look at the second table, we could call:
# head(results[[2]])
```
<img width="754" height="202" alt="image" src="https://github.com/user-attachments/assets/c2b63543-f542-4591-a87f-b5de68acd620" />

The object `results` is a list containing 100 data frames. Each data frame contains diversity metrics calculated from one subsampled dataset.

We can combine the results into a single data frame.

```r
rareified_div <- bind_rows(results)

head(rareified_div)
```


We can now examine how observed richness varies across rarefaction iterations for each sample.

```r
rareified_div %>%
  ggplot(aes(x = Sample, y = Observed_Richness)) +
  geom_boxplot() +
  geom_jitter(
    width = 0.15,
    alpha = 0.4
  ) +
  theme_bw(base_size = 12) +
  theme(
    axis.text.x = element_text(
      angle = 45,
      hjust = 1
    )
  ) +
  labs(
    x = "Sample",
    y = "Observed richness"
  )
```
<img width="1400" height="865" alt="image" src="https://github.com/user-attachments/assets/c01bef97-e483-4d8b-8299-93938a26ce16" />

For each sample, the variation in observed richness across rarefaction iterations is relatively small. We can therefore calculate the mean diversity value for each sample across all iterations to get an estimate of alpha diversity in each sample.

```r
rareified_div_means <- rareified_div %>%
  group_by(Sample) %>%
  summarise(
    Richness = mean(Observed_Richness),
    Shannon = mean(Shannon),
    Simpson = mean(Simpson),
    .groups = "drop"
  )

head(rareified_div_means)
```

<img width="645" height="204" alt="image" src="https://github.com/user-attachments/assets/fac89069-21f7-466f-b7be-b91258964d17" />


### Testing for differences in alpha diversity

Next, combine the mean diversity estimates with the sample metadata.

```r
metadata_div <- rareified_div_means %>%
  left_join(
    metadata,
    by = "Sample"
  )

head(metadata_div)
```

We can visualize Shannon diversity across sample categories.

```r
metadata_div %>%
  ggplot(aes(x = category, y = Shannon)) +
  geom_boxplot() +
  theme_bw(base_size = 12) +
  labs(
    x = "Sample category",
    y = "Mean Shannon diversity"
  )
```
<img width="1400" height="865" alt="image" src="https://github.com/user-attachments/assets/2d370ee8-a652-4e38-81f7-5675608fd7df" />

We can test for a difference in Shannon diversity between categories using a Wilcoxon rank-sum test.

```r
wilcox.test(
  Shannon ~ category,
  data = metadata_div
)
```

The test indicates whether Shannon diversity differs significantly between the sample categories.

**Try this on your own:**

1. Create boxplots for observed richness and Simpson diversity.
2. Perform Wilcoxon tests to compare these metrics between sample categories.

## Beta Diversity

Beta diversity describes differences in community composition among samples. In microbiome research, beta diversity is often used to test whether the overall microbial community composition differs between groups.

### Calculating Distance and Dissimilarity Metrics

Several distance and dissimilarity metrics can be calculated from a microbiome feature table. Different metrics emphasize different aspects of community composition.

In this section, we will introduce:

- Jaccard distance
- Bray–Curtis dissimilarity
- Weighted UniFrac distance

### Jaccard Distance

Jaccard distance is based on the presence or absence of features. It measures differences in community membership while ignoring the relative abundance of each feature.

We can calculate Jaccard distance using the `vegan` package. Here, `avgdist()` calculates an average distance across repeated subsampling events. As such, we need to specify the sampling depth determined above.

```r
# x is the feature table
# sample is the sampling depth
# dmethod is the distance/dissimilarity method

jaccard_dist <- avgdist(
  x = counts_no_taxa_t,
  sample = min_depth,
  dmethod = "jaccard"
)

head(as.matrix(jaccard_dist))
```

**Question:** Why is the diagonal all 0s?

### Principal Coordinates Analysis

We can use principal coordinates analysis (PCoA) to visualize differences among samples based on the Jaccard distance matrix.

We will do this using the `cmdscale()` function.

```r
jaccard_pcoa <- cmdscale(
  jaccard_dist,
  k = 4,
  eig = TRUE
)

# Calculate the percentage of variation explained by the first two axes.
# Variation explained is the component eigenvalue divided by the sum of
# all positive eigenvalues across each component.

positive_eigenvalues <- jaccard_pcoa$eig[
  jaccard_pcoa$eig > 0
]

pc1_percent <- 100 * jaccard_pcoa$eig[1] /
  sum(positive_eigenvalues)

pc2_percent <- 100 * jaccard_pcoa$eig[2] /
  sum(positive_eigenvalues)

pc1_percent
pc2_percent
```

The coordinates of each sample along the PCoA axes are stored in `jaccard_pcoa$points`. Before combining these coordinates with the metadata, ensure that the metadata are in the same order as the PCoA results.

```r
# Set sample names as the row names of the metadata
rownames(metadata) <- metadata$Sample

# Reorder metadata to match the PCoA sample order
metadata_ordered <- metadata[
  rownames(jaccard_pcoa$points),
  ,
  drop = FALSE
]

# Create a data frame containing PCoA coordinates and sample categories
jaccard_df <- data.frame(
  Sample = rownames(jaccard_pcoa$points),
  PC1 = jaccard_pcoa$points[, 1],
  PC2 = jaccard_pcoa$points[, 2],
  category = metadata_ordered$category
)
```

We can visualize the PCoA results using `ggplot2`.

```r
jaccard_df %>%
  ggplot(aes(
    x = PC1,
    y = PC2,
    color = category
  )) +
  geom_point(size = 3) +
  theme_bw(base_size = 16) +
  labs(
    x = paste0("PCoA 1 (", round(pc1_percent, 2), "%)"),
    y = paste0("PCoA 2 (", round(pc2_percent, 2), "%)"),
    color = "Category"
  )
```
<img width="1400" height="865" alt="image" src="https://github.com/user-attachments/assets/62cbb594-1a01-45d8-b265-b61603e3c365" />


### PERMANOVA

We can test whether community composition differs between sample categories using PERMANOVA, implemented in the `adonis2()` function.

```r
adonis2(
  jaccard_dist ~ category,
  data = metadata_ordered,
  permutations = 999
)
```

<img width="640" height="201" alt="image" src="https://github.com/user-attachments/assets/99f38971-3332-4d32-8fba-93aaf3732bb7" />


The PERMANOVA output includes an estimate of the amount of variation explained by sample category and a permutation-based significance test.

Our results indicate that category explains 25.3% of the total community variance in Jaccard distances.

## Bray–Curtis Dissimilarity

Bray–Curtis dissimilarity incorporates the relative abundance of features and is commonly used to compare microbial community composition.

We can calculate Bray–Curtis dissimilarity in the same way we did above for Jaccard by adjusting the `dmethod` parameter in `avgdist()`. Try this yourself!

## Weighted UniFrac Distance

Weighted UniFrac incorporates both feature abundance and phylogenetic relationships among features.

Calculating weighted UniFrac requires a phylogenetic tree in addition to the feature table and taxonomy information.

We will therefore load the tree using the `ape` package.

```r
phylo_tree <- read.tree(
  "amplicon_data/16S_Blueberry/final_output_exported/tree.nwk"
)

phylo_tree
```

Next, we can calculate the weighted UniFrac distance of a feature table and its accompanying phylogenetic tree using `GUniFrac()`.

```r
# This function calculates multiple UniFrac distances based on an alpha parameter.
# The alpha parameter controls how abundance is weighted during the UniFrac calculation.
# Below, we will explore the traditional weighted UniFrac value.
# Feel free to explore the other values as well.

unifracs <- GUniFrac(
  counts_no_taxa_t_rare,
  tree = phylo_tree
)$unifracs

w_unifrac <- unifracs[, , "d_1"]
```

We can now complete both PCoA visualization and PERMANOVA testing in the same manner as we did previously. 

**Try to write the code out yourself!**

## Differential Abundance Testing with MaAsLin3

MaAsLin3 can be used to identify microbial features whose abundances or prevalences are associated with sample metadata variables.

Before running MaAsLin3, ensure that:

- The feature table contains samples as rows and features as columns.
- The metadata table contains samples as rows.
- Sample identifiers match between the feature table and metadata.
- Features with extremely low prevalence or abundance have been removed if appropriate.
- The metadata variables have the correct data types.

The first step in MaAsLin3 is to calculate the read depth of each sample. This is done to control for differences in read depth without the need for rarefaction.

```r
# Read depth is equal to the total number of reads detected
read_depths <- data.frame(
  Sample = rownames(counts_no_taxa_t),
  read_depth = rowSums(counts_no_taxa_t)
)

metadata <- metadata %>%
  left_join(read_depths)

# MaAsLin3 requires metadata row names to be the sample names
rownames(metadata) <- metadata$Sample

maas_results <- maaslin3::maaslin3(
  input_data = counts_no_taxa_t,
  input_metadata = metadata,
  output = "amplicon_data/16S_Blueberry/maaslin3_out",
  formula = "~ category + read_depth",
  normalization = "TSS",
  transform = "LOG",
  max_pngs = 100
)
```

We can now look at the summary heatmap.

<img width="3800" height="3330" alt="image" src="https://github.com/user-attachments/assets/8426fa99-af4c-4df7-af8e-6fdb55fcdf43" />


The color of the dots indicates significance, whereas their placement on the x-axis shows the magnitude of the effect. As expected with our small dataset of 10 samples, we found no significant features.

We can also examine the results table by opening the saved `.tsv` file with Excel or by examining the saved variables in the current R session.

```r
# Prevalence results
head(maas_results$$fit_data_prevalence$$results)
```

<img width="753" height="227" alt="image" src="https://github.com/user-attachments/assets/c26fa047-0b86-4e15-b372-015c277f0404" />



```r
head(maas_results$$fit_data_abundance$$results)
```



### Explanation of MaAsLin3 Outputs

A full explanation of the outputs is provided below.

#### 1. Data Output Files

- **`all_results.tsv`**
  - `feature` and `metadata` are the feature and metadata names.
  - `value` and `name` are the value of the metadata and variable name from the model.
  - `coef` and `stderr` are the fit coefficient and standard error from the model.
  - In abundance models, a one-unit change in the metadata variable corresponds to a `2^coef` fold change in the relative abundance of the feature.
  - In prevalence models, a one-unit change in the metadata variable corresponds to a `coef` change in the log-odds of a feature being present.
  - `null_hypothesis` is the value of the null hypothesis against which the coefficient is tested.
  - `pval_individual` is the p-value of the individual association.
  - `qval_individual` is the false discovery rate-corrected q-value of the individual association.
  - FDR correction is performed over all p-values without errors in the abundance and prevalence modeling together.
  - `pval_joint` and `qval_joint` are the p-value and q-value of the joint prevalence and abundance association.
  - The joint p-value comes from plugging the minimum of the association's abundance and prevalence p-values into the Beta(1,2) cumulative distribution function.
  - `error` lists any errors from the model fitting.
  - `model` specifies whether the association is abundance or prevalence.
  - `N` and `N_not_zero` are the total number of data points and the total number of non-zero data points for the feature.

- **`significant_results.tsv`**
  - This file is a subset of the results in `all_results.tsv`.
  - It only includes associations with joint or individual q-values less than or equal to the significance threshold.

- **`features`**
  - This folder includes the filtered, normalized, and transformed versions of the input feature table.
  - These steps are performed sequentially in the above order.
  - If an option is set such that a step does not change the data, the resulting table will still be output.

- **`models_linear.rds` and `models_logistic.rds`**
  - These files contain a list with every model fit object.
  - They are generated only if `save_models` is set to `TRUE`.

- **`residuals_linear.rds` and `residuals_logistic.rds`**
  - These files contain a data frame with residuals for each feature.

- **`fitted_linear.rds` and `fitted_logistic.rds`**
  - These files contain a data frame with fitted values for each feature.

- **`ranef_linear.rds` and `ranef_logistic.rds`**
  - These files contain a data frame with extracted random effects for each feature when random effects are specified.

- **`maaslin3.log`**
  - This file contains all log information for the run.
  - It includes all settings, warnings, errors, and steps run.

#### 2. Visualization Output Files

- **`summary_plot.pdf`**
  - This file contains a combined coefficient plot and heatmap of the most significant associations.
  - In the heatmap, one star indicates that the individual q-value is below the `max_significance` parameter.
  - Two stars indicate that the individual q-value is below `max_significance / 10`.

- **`association_plots/[metadatum]/[association]/[metadatum]_[feature]_[association].png`**
  - A plot is generated for each significant association up to `max_pngs`.
  - Scatter plots are used for continuous metadata abundance associations.
  - Box plots are used for categorical metadata abundance associations.
  - Box plots are used for continuous metadata prevalence associations.
  - Grids are used for categorical metadata prevalence associations.
  - Data points plotted are after filtering, normalization, and transformation, so the scale in the plot is the scale used during model fitting.

At the top right of each association plot is the name of the significant association in the results file, the FDR-corrected q-value for the individual association, the number of samples in the dataset, and the number of samples with non-zero abundances for the feature.

In plots with categorical metadata variables, the reference category is on the left. The significant q-values and coefficients in the top right are in the order of the values specified above.

Because the displayed coefficients correspond to the full fitted model with potentially scaled metadata variables, the marginal association plotted might not match the coefficient displayed. However, the plots are intended to provide an interpretable visual while usually agreeing with the full model.

## If There Is Time

### Collapsing Tables by Taxa

What happens if you want to run a differential abundance analysis at a level other than the ASV level?

```r
#define the taxonomic hierarchy
taxa_order <- c(
  "Domain",
  "Phylum",
  "Class",
  "Order",
  "Family",
  "Genus",
  "Species"
)

#save the sample names
Samples <- metadata$Sample

#split the taxonomy column by ";" into the taxonomic order we defined above.
counts_taxa_split <- separate(
  counts,
  col = taxonomy,
  sep = ";",
  into = taxa_order
)

# Aggregate counts by Genus
counts_by_genus <- counts_taxa_split %>%
  mutate(
    # If Genus is empty, set it to unclassified
    Genus = if_else(
      is.na(Genus) | Genus == "",
      "Unclassified",
      Genus
    )
  ) %>%
  group_by(Genus) %>%
  summarise(
    across(
      .cols = all_of(Samples),
      .fns = ~ sum(.x, na.rm = TRUE)
    ),
    .groups = "drop"
  ) %>%
  data.frame(
    check.names = FALSE,
    check.rows = FALSE
  )

# Add genus names as the row names
rownames(counts_by_genus) <- counts_by_genus$Genus

# Remove the genus column
counts_by_genus <- counts_by_genus[, -1]

# Flip the table so samples are rows and genera are columns
counts_by_genus <- data.frame(
  t(counts_by_genus),
  check.names = FALSE,
  check.rows = FALSE
)
```
Now that the samples have rows as samples and rows as Genus we can run MaAsLin3 on them to see if any Genera are associated our metadata.

```r
maas_results <- maaslin3::maaslin3(
  input_data = counts_by_genus,
  input_metadata = metadata,
  output = "amplicon_data/16S_Blueberry/maaslin3_out_genus",
  formula = "~ category + read_depth",
  normalization = "TSS",
  transform = "LOG",
  max_pngs = 100
)
```
<img width="3549" height="3330" alt="image" src="https://github.com/user-attachments/assets/602dbd31-b68b-487b-947c-13495d52a46f" />


