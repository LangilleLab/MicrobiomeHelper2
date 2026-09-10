---
layout: default
title: "CBW 2026 module 6 - visualization and finding functional significance"
show_sidetoc: true
header_type: base
permalink: /docs/tutorials/cbw-2026-module6/
---

## Introduction

In this lab, we will build on the concepts introduced in the Module 6 lecture to further explore differential abundance testing with MaAsLin3. In particular, we will learn how to incorporate covariates and random effects into our models to account for potential confounding factors and repeated or correlated measurements. We will also examine how MaAsLin3 can be applied to metatranscriptomic data to explore differential expression or differential abundance.

In the second part of the lab, we will introduce the basic concepts of supervised machine learning in R using random forest models. We will begin by exploring how to divide data into training and testing sets, then compare this approach with k-fold cross-validation using the `caret` package. These exercises will provide a foundation for evaluating model performance and applying machine-learning methods to microbiome data.

### Libraries we will use:


- **`ggplot2`**  
  A data visualization package based on the *Grammar of Graphics*. It allows you to create customizable and publication-quality plots, including scatterplots, boxplots, bar charts, and faceted visualizations.

- **`dplyr`**  
  A data manipulation package that provides functions for filtering, selecting, arranging, summarizing, and transforming data frames.

- **`maaslin3`**  
  A statistical modeling package for microbiome and other high-dimensional biological data. It can be used to identify differentially abundant or differentially expressed features while accounting for covariates, random effects, and other study-design factors.

- **`caret`**  
  A machine-learning framework that provides tools for data preprocessing, model training, parameter tuning, performance evaluation, and cross-validation.

- **`randomForest`**  
  An R package for fitting random forest models for classification. Random forests combine multiple decision trees to make predictions and evaluate the importance of predictor variables.

```
library(ggplot2)
library(dplyr)
library(maaslin3)
library(caret)
library(randomForest)
```


## Data loading

In this tutorial we will use data from the Human Microbiome Project 2 already processed with HUMAnN 4.0. The main data components include pathway abundances at the DNA and RNA level, and sample metadata.

### Metatranscriptomic data (MTX)
```
#load MTX profiles
MTX_pathways <- read.table("../CourseData/Module6_lab/HMP2_pwyRNA.tsv", header=T, sep="\t",
                           check.names = F, row.names=1, stringsAsFactors = F )

MTX_pathways[1:5, 1:2]
```

```
           X1CMET2_PWY_N10_formyl_tetrahydrofolate_biosyn ANAEROFRUCAT_PWY_homolactic_fermentation
CSM5FZ3T_P                                     0.03156540                               0.00114574
CSM5FZ46_P                                     0.00000000                               0.00000000
CSM5FZ4C_P                                     0.01669700                               0.00000000
CSM5FZ4G_P                                     0.01153230                               0.00833768
CSM5FZ4K_P                                     0.00899462                               0.01277590
```


This shows the first 5 samples along with the RNA abundances of the first two pathways. How might you determine the total number of pathways detected across all samples?

### Metagenomic data (MGX)
```
MGX_pathway <- read.table("../CourseData/Module6_lab/HMP2_pwyDNA.tsv", header=T, sep="\t",
                           check.names = F, row.names=1, stringsAsFactors = F )
MGX_pathway[1:5, 1:2]
```

```
         X1CMET2_PWY_N10_formyl_tetrahydrofolate_biosyn ANAEROFRUCAT_PWY_homolactic_fermentation
CSM5FZ4M                                      0.0158099                               0.00946321
CSM5MCUO                                      0.0101701                               0.00440300
CSM5MCVL                                      0.0167429                               0.00611800
CSM5MCVN                                      0.0180019                               0.00710437
CSM5MCW6                                      0.0153125                               0.00257452
```

Above we can see that the same pathways are listed in the DNA and RNA pathway tables. This is great as we would expect
that pathways that are expressed in a community should be encoded. Are there cases where this might not be the case?


### Metadata

```
#load metadata
HMP2_metadata <- read.table("../CourseData/Module6_lab/HMP2_metadata.tsv", sep="\t", header=T,
                            row.names=1, stringsAsFactors = F)
HMP2_metadata[1:5, 1:4]
```

```
           participant_id    site_name week_num    reads
CSM5FZ3N_P          C3001 Cedars-Sinai        0  9961743
CSM5FZ3R_P          C3001 Cedars-Sinai        2 16456391
CSM5FZ3T_P          C3002 Cedars-Sinai        0 10511448
CSM5FZ3V_P          C3001 Cedars-Sinai        6 17808965
CSM5FZ3X_P          C3002 Cedars-Sinai        2 13160893
```


Here we can see the first few metadata columns represent the participant's ID the site where the sample was collect, the week it was collected and the number of reads in the sample after quality filtering. 

How can we see what other metadata is contained within this file?

```
colnames(HMP2_metadata)
```

```
 [1] "participant_id"  "site_name"       "week_num"        "reads"           "diagnosis"       "dysbiosis_state"
 [7] "antibiotics"     "age"             "sex"             "race"            "education"       "probiotic"      
[13] "red_meat"        "sweets"        
```

Let's explore the diagnosis column by tabulating the number of each response. 

```
table(HMP2_metadata$diagnosis)
```

```
    CD nonIBD     UC 
   685    405    437 
```

So we can see that we have fairly even numbers with CD being the most abundant sample type. 


#### Factors and levels in R

In R, a **factor** is a data type used to represent categorical variables, such as treatment groups, disease status, or sampling locations. Factors consist of a set of possible values called **levels**. When a factor is included in a linear model, R typically uses one level as the **reference group** and compares the remaining levels against it. By default, the reference level is usually the first level of the factor, although it can be changed explicitly using functions such as `relevel()` or by specifying the factor levels in the desired order.

With this in mind, we will set `nonIBD` as the **reference group** for the `diagnosis` variable. MaAsLin 3 will then compare the abundance and prevalence of each feature in individuals with Crohn’s disease (`CD`) or ulcerative colitis (`UC`) to those in individuals without inflammatory bowel disease (`nonIBD`).

```
HMP2_metadata$diagnosis <- factor(HMP2_metadata$diagnosis, levels=c("nonIBD", "CD", "UC"))
```

Using `nonIBD` as the reference group makes the model coefficients easier to interpret. A positive coefficient indicates that a feature is more abundant or prevalent in the comparison group (`CD` or `UC`) than in the `nonIBD` group, whereas a negative coefficient indicates that it is less abundant or prevalent.

We can do the same for the variable `antibiotics` so that the base level is `No`. 

```
HMP2_metadata$antibiotics <- factor(HMP2_metadata$antibiotics, levels=c("No", "Yes"))
```

With `No` being the reference group what would a positive coefficient mean? What about a negative?

## Advanced Modeling with MaAsLin 3

### Using formulas in R
Now that we have explored the metadata, we can identify several variables that could potentially confound our analysis of whether pathway abundance and prevalence are associated with diagnosis. These variables include `antibiotics`, `age`, and `sex`.

To address this potential confounding, we will include these variables as covariates in the MaAsLin3 model. This allows us to estimate the association between diagnosis and pathway abundance or prevalence while accounting for differences in antibiotic use, age, and sex among participants. In other words, the model will assess whether pathway features are associated with diagnosis after adjusting for the potential effects of these additional variables. 

Fixed effects are separated in the MaAsLin 3 formula with a `+`

```
formula = " ~ age + sex + antibiotics"
```

**However, our analysis is not complete.** During our inspection of the metadata, we may have noticed that several samples were collected from the same participant. This is known as **repeated sampling**. Because samples from the same individual are likely to be more similar to one another than samples from different individuals, they cannot be treated as completely independent observations.

We can check the number of samples from each individual using the `table()` function:

```
table(HMP2_metadata$participant_id)
```

```
C3001 C3002 C3003 C3004 C3005 C3006 C3008 C3009 C3010 C3011 C3012 C3013 C3015 C3016 C3017 C3019 C3020 C3021 C3022 C3023 
   16    15    10    24    12    11    13    12    14    22    14    22    23    19    23     1     1     9    21    12 
C3024 C3028 C3029 C3030 C3032 C3033 C3034 C3035 C3036 C3037 E5001 E5002 E5003 E5004 E5008 E5009 E5013 E5019 E5022 H4001 
    1    12     9    10    12     1    10    11     1    11    13     5     1    12     3    11    13     1     1    12 
H4004 H4006 H4007 H4008 H4009 H4010 H4011 H4012 H4013 H4014 H4015 H4016 H4017 H4018 H4019 H4020 H4022 H4023 H4024 H4027 
   11    22    12    23    20    13     1     1    14    12    20    14    18    12    22    23    14    21    23    10 
H4028 H4030 H4031 H4032 H4035 H4038 H4039 H4040 H4042 H4043 H4044 H4045 M2008 M2010 M2014 M2021 M2024 M2025 M2026 M2027 
    9    13    11    13    24    13    13    11     7     8     5    13    17     2    13     8     1    10    16    13 
M2028 M2034 M2039 M2041 M2042 M2047 M2048 M2060 M2064 M2068 M2069 M2071 M2072 M2075 M2077 M2079 M2081 M2083 M2084 M2085 
   18    20    14    13    23    14    10    10    20    25    25    10    24    11    14    14     1    18    23    14 
M2086 M2091 M2097 M2103 P6005 P6009 P6010 P6012 P6013 P6014 P6016 P6017 P6018 P6024 P6025 P6028 P6033 P6035 P6037 P6038 
    1     1    11     6    15    22    22    14    20     9    14    10    25    10     7     8    11    11     7    12 
```

To account for this lack of independence, we will include the individual identifier as a **random effect** in the MaAsLin3 model. A random effect allows the model to account for subject-specific differences while estimating the associations between our variables of interest and pathway abundance or prevalence.

Random effects are specified using the syntax `1|variable`, where `variable` identifies the grouping factor. In this analysis, we will use the participant identifier:

```
formula=" ~ age + sex + antibiotics + (1|participant_id)
```

**Note For MaAsLin 3 specifically we also need to add the fixed effect reads to account for differences in sequencing depth between the samples**.

### Running MaAsLin 3 with both fixed effects and random effects

Now that we have figured out the formula that we want to use for our model we can now run MaAsLin 3 on our DNA pathway data in a similar manner to what we did in the **module 2 lab**



**This model will take about 15 minutes to run if you would like to save time the outputs are already saved in workspace**
```
HMP2_diagnosis <- maaslin3(input_data = MGX_pathway, input_metadata = HMP2_metadata, 
                           formula = "~ diagnosis + age + sex + antibiotics + reads + (1|participant_id)", 
                           output = "Module6/maaslin3_pathway_DNA/", 
                           normalization = "TSS", 
                           transform = "LOG"
                           )
```

We can now load the results from our MaAsLin3 analysis and use them to create a **volcano plot**. This plot will examine the relationship between the model coefficients and the adjusted *p*-values for DNA pathways associated with diagnosis.

```
#load in results table
dna_pathway_res <- read.table("Module6/maaslin3_pathway_DNA/all_results.tsv", sep="\t", header=T)

##volcano plot for abundance values
diagnosis_dna_pathway_abundance <- dna_pathway_res %>% 
  #filter to only the results for the abundance model
  filter(model=="abundance") %>% 
  #only keep models that didn't report any errors
  filter(is.na(error)) %>% 
  #only keep coefficents that are associated with diagnosis
  filter(metadata=="diagnosis")


diagnosis_dna_pathway_abundance[1:5, c("value", "coef", "qval_individual", "feature")]
```

```
  value       coef qval_individual                                      feature
1    CD  0.9269134      0.01127269               PWY0_1297_SP_of_purine_dns_deg
2    UC -0.8053247      0.01956326           DAPLYSINESYN_PWY_L_lysine_biosyn_I
3    CD -0.1267587      0.02236974                   VALSYN_PWY_L_valine_biosyn
4    CD -0.1223755      0.02325975 PWY_7111_pyruvate_fermentation_to_isobutanol
5    CD -0.1268310      0.02405732            ILEUSYN_PWY_L_isoleucine_biosyn_I
```

The **x-axis** will display the model coefficient, indicating the direction and magnitude of the association. Positive coefficients represent higher pathway abundance in the diagnosis group compared with the `nonIBD` reference group, whereas negative coefficients represent lower pathway abundance. The **y-axis** will display the negative logarithm of the adjusted *p*-value, `-log10(adjusted p-value)`, with larger values representing stronger statistical evidence. The **color** will represent whether the association is comparing `nonIBD` to `CD` or `UC`. 

```
diagnosis_dna_pathway_abundance %>% ggplot(aes(x=coef, y=-log10(pval_individual), color=value)) + geom_point() +
  theme_bw(base_size=12)
```

<img width="1400" height="865" alt="image" src="https://github.com/user-attachments/assets/acc0e4aa-b117-43bc-a9a9-5172744b225f" />

You can also inspect the results table using the `View()` function within Rstudio.

```
View(dna_pathway_res)
```

**Try building your own heat maps First, create a volcano plot showing the associations between pathway prevalence and diagnosis. Then, create a second volcano plot showing the associations between pathway abundance or prevalence and one of the control covariates, such as `sex`, `age`, or antibiotic use.**


## Metatranscriptomic analysis with MaAsLin 3

Now that we have covered how to incorporate fixed and random effects into MaAsLin3 models, we will turn our attention to modeling metatranscriptomic data. Specifically, we will examine how MaAsLin3 can be used to analyze metatranscriptomic feature abundance and identify pathways or transcripts whose activity is associated with variables of interest, such as diagnosis.

Unlike metagenomic data, which describe the genetic potential of a microbial community, metatranscriptomic data provide information about genes and pathways that are actively being transcribed. This allows us to investigate not only which functions are present, but also which functions are being expressed under specific conditions.

There are two models that we can explore in this analysis. The first simple looks at transcript abundance without consideration for the abundance of the underlying DNA copy number. This allows use to identify transcripts that are highly abundant in one community compared to another but does not allow us to identify whether they are differential **expressed**. 

### Unadjusted MTX model

We can now run the unadjusted model using the metatranscriptomic (MTX) pathway data. We will use the same model formula as before, but apply it to the MTX feature table.

In this analysis, we will use MaAsLin3’s default settings for normalization and transformation: **total-sum scaling (TSS)** normalization and a **logarithmic (LOG)** transformation. These options are not included explicitly in the function call because they are the default values used by `maaslin3()` when no alternative settings are specified.

You can view the available arguments and default settings for the function by running:
```
?maaslin3()
```

**This model will take about 15 minutes to run if you would like to save time the outputs are already saved in workspace**

```
RNA_model <- maaslin3(
    input_data = MTX_pathways,
    input_metadata = HMP2_metadata,
    output = 'Module6/maaslin3_pathway_raw_RNA',
    formula = "~ diagnosis + age + sex + antibiotics + (1|participant_id)")
```

Try loading in the results of this model yourself. If you have time, try to create a volcano plot like we did for the MGX pathway results.

### DNA adjusted MTX model

Finally, we will run the differential expression MTX model in MaAsLin 3. We first put the DNA and RNA abundance files into the MaAsLin 3 function preprocess_dna_mtx to total sum scale the abundances of both and apply the proper transformation to the DNA abundances. For each sample in each feature, this function:

1. Log 2 transforms the DNA abundance if the DNA abundance is >=0.
2. Sets the DNA abundance to log2([minimum non-zero relative abundance in the dataset] / 2) if the corresponding RNA abundance is non-zero but the DNA abundance is zero.
3. Sets the DNA abundance to NA if both are zero, which excludes the sample when fitting the model for the feature.

Now, we will switch the input_data to the preprocessed RNA table preprocess_out$dna_table and include the pre-processed DNA as the feature-specific covariate with `feature_specific_covariate = preprocess_out$dna_table`. We also set the name of the covariate for model fitting with `feature_specific_covariate_name = 'DNA'` and we specify that we do not want to record the associations with the DNA in the outputs and plots by setting `feature_specific_covariate_record = FALSE.` 

```
preprocess_out <- preprocess_dna_mtx(MGX_pathway, MTX_pathways)

RNA_expression_model <- maaslin3(
    input_data = preprocess_out$rna_table,
    input_metadata = HMP2_metadata,
    output = 'Module6/maaslin3_pathway_expression_RNA',
    formula="~ diagnosis + age + sex + antibiotics + (1|participant_id)",
    feature_specific_covariate = preprocess_out$dna_table,
    feature_specific_covariate_name = 'DNA',
    feature_specific_covariate_record = FALSE)
```

We can now check out the summary plot to see which features are differentially expressed due to our covariates. 


## Supervised Learning with Random Forests and `caret`

### Data splitting

### Training the model

### Predicting on the test set

### Training a model with K-fold cross validation using `caret`


## Authors

**Authors:** Jacob T. Nearing<br>
