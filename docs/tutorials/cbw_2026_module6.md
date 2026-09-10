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
1    CD  0.9025269     0.006677512               PWY0_1297_SP_of_purine_dns_deg
2    UC -0.8210926     0.011590329           DAPLYSINESYN_PWY_L_lysine_biosyn_I
3    UC -0.7258358     0.017441839           PWY_5345_SP_of_L_methionine_biosyn
4    CD -0.7788738     0.019584636         PWY_7383_anaerobic_energy_metabolism
5    CD -0.1221061     0.024377110 PWY_7111_pyruvate_fermentation_to_isobutanol
```

The **x-axis** will display the model coefficient, indicating the direction and magnitude of the association. Positive coefficients represent higher pathway abundance in the diagnosis group compared with the `nonIBD` reference group, whereas negative coefficients represent lower pathway abundance. The **y-axis** will display the negative logarithm of the adjusted *p*-value, `-log10(adjusted p-value)`, with larger values representing stronger statistical evidence. The **color** will represent whether the association is comparing `nonIBD` to `CD` or `UC`. 

```
diagnosis_dna_pathway_abundance %>% ggplot(aes(x=coef, y=-log10(pval_individual), color=value)) + geom_point() +
  theme_bw(base_size=12)
```

<img width="1400" height="865" alt="image" src="https://github.com/user-attachments/assets/73632458-412b-4524-bf8f-757334f6d328" />


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


**As with the other models this will run slowly. The results are already saved on your instance so we suggest skipping this command and loading the preserved results.**
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

The summary plot will be saved in the specified output folder above and should look like below.

<img width="5400" height="3300" alt="image" src="https://github.com/user-attachments/assets/51db1383-41b2-43d1-a48c-348a88f50b60" />


**How could we improve our MTX models in the future to be more robust to differences in MTX read depth?**

For more information on MaAsLin 3 check out its GitHub page: 

https://github.com/biobakery/maaslin3

## Supervised Learning with Random Forests and `caret`

In this section, we will introduce **Random Forest** models for classifying samples based on their microbiome features. We will use MGX pathway abundance data from week 0 samples to predict whether each sample belongs to the `nonIBD` or `CD` diagnosis group. 

```
classification_data <- HMP2_metadata %>% filter(week_num==0) %>% filter(diagnosis!="CD")

# Remove samples that lack MGX pathway data
classification_data$Sample <- rownames(classification_data)

classification_data <- classification_data %>% filter(Sample %in% rownames(MGX_pathway))
table(classification_data$diagnosis)
```

```
nonIBD     CD     UC 
    23      0     22 
```

**Why might we not want to use the entire dataset to train our model? Should we be concerned about data leakage?**

We will explore two different data-splitting strategies.

First, we will randomly divide the data into separate training and test sets one time. The training set will be used to fit the Random Forest model, while the test set will be held out and used to evaluate the model’s performance on previously unseen samples. This approach provides a straightforward estimate of model performance. However, because the result depends on a single random split, the estimated performance may vary depending on which samples are assigned to each set.

Secondly, we will train and evaluate a second Random Forest model using *k*-fold cross-validation with the `caret` package. Cross-validation allows us to assess model performance across multiple training and validation splits, providing a more robust estimate of how well the model may perform on new data. 

### Data splitting

We will split the data using the `createDataPartition()` function from the `caret` package. This function randomly selects samples for the training set while using the diagnosis groups to maintain similar proportions of `nonIBD` and `CD` samples in both the training and test datasets.

This type of split is called a **stratified split**. Stratification helps ensure that both datasets contain representative samples from each diagnosis group, allowing us to train the model and evaluate its performance more fairly.

```
# Set a seed so the split can be reproduced
set.seed(123)

# Create an 70/20 stratified split based on diagnosis
training_index <- createDataPartition(
  classification_data$diagnosis,
  p = 0.70,
  list = FALSE
)
```

In the above command `p` represents the proportion of data that should be included in the training dataset.

```
# Create the training and test sets
training_data <- classification_data[training_index, ]
test_data <- classification_data[-training_index, ]

table(training_data$diagnosis)
```

```
nonIBD     CD     UC 
    17      0     16 
```

```
table(test_data$diagnosis)
```

```
nonIBD     CD     UC 
     6      0      6 
```

We now need to subset our MGX pathway abundance data so that it matches with the training and test metadata we created. We can do this by subsetting the table based on the `rownames()` of the test and training data.

```
MGX_training <- MGX_pathway[rownames(training_data),]
MGX_test <- MGX_pathway[rownames(test_data),]
```

### Training the model

Now that we have divided the data into training and test sets, we can train a **Random Forest** classification model. The model will use the diagnosis labels in the training metadata as the response variable and the metagenomic (MGX) pathway abundances as the predictor variables.

We will fit the model using the `randomForest()` function from the `randomForest` package. Before training the model, we need to ensure that the diagnosis variable is stored as a factor and that any unused factor levels have been removed. This is important because a classification model can only be trained on response classes that are actually represented in the training data.

We can remove unused levels using `droplevels()`:

```
training_data$diagnosis <- droplevels(training_data$diagnosis)
```

Now we can train our model.
```
model1 <- randomForest(x = MGX_training, y=training_data$diagnosis,
                       ntree=100, mtry=128)
```

In the model above, `x` represents the predictor variables—in this case, the MGX pathway abundances. The argument `y` represents the response variable, which is the diagnosis group for each sample.

The `ntree` argument specifies the number of decision trees generated in the Random Forest model. Random Forest models combine the predictions from many individual decision trees to improve classification performance and reduce the influence of any single tree. In practice, `ntree` is often set between 500 and 1,000 trees, depending on the size and complexity of the dataset. To reduce computational time in this tutorial, we will use `ntree = 100`.

The `mtry` argument is a **hyperparameter** that controls the number of predictor variables randomly considered as candidates at each split in a Random Forest decision tree. Changing `mtry` can affect both the model’s performance and the level of correlation among the individual trees.

### Predicting on the test set

We can now apply our model to the unseen test dataset to see how well our model performs. 

```
model1_predictions <- predict(model1, MGX_test)
```

```
CSM5MCTZ_P CSM6J2H9_P CSM79HQR_P HSM67VDT_P   HSM67VDT MSM6J2JH_P MSM79H94_P   MSM79HBZ MSM79HD6_P MSM79HF1_P 
    nonIBD     nonIBD     nonIBD     nonIBD     nonIBD         UC         UC     nonIBD         UC     nonIBD 
MSM9VZJF_P   PSM6XBW3 
    nonIBD         UC 
Levels: nonIBD UC
```

You can see that this returns a vector with the sample name and the diagnosis label that the model predicts.

We can examine the accuracy of our model by looking at its confusion matrix.

```
test_data$diagnosis <- droplevels(test_data$diagnosis)
confusionMatrix(test_data$diagnosis, model1_predictions)
```

```
Confusion Matrix and Statistics

          Reference
Prediction nonIBD UC
    nonIBD      3  3
    UC          5  1
                                          
               Accuracy : 0.3333          
                 95% CI : (0.0992, 0.6511)
    No Information Rate : 0.6667          
    P-Value [Acc > NIR] : 0.9961                       
```

Overall we can see our model did pretty poorly. This isn't entirely surprising given the small number of trees we trained our model with along with the small number of samples we are working with.

We could potentially improve our model by perform feature selection on the training data but this is outside the scope of this tutorial. For more information on randomForest feature selection I would suggest checking out this [paper](https://arxiv.org/pdf/1201.1587), or ask ask your favorite local AI agent.  


### Training a model with K-fold cross validation using `caret`

To obtain a more robust estimate of model performance, we can train and evaluate the Random Forest model using *k*-fold cross-validation. In this approach, the training data are divided into *k* approximately equal-sized subsets, called folds.

The model is trained on *k* − 1 folds and evaluated on the remaining fold. This process is repeated *k* times so that each fold is used as the validation set once. The performance measurements from all folds are then combined to estimate how well the model is likely to perform on new, unseen samples.

Compared with a single training–test split, *k*-fold cross-validation reduces the influence of any one random split and provides a more reliable assessment of model performance.

```
set.seed(786)

# Define 3-fold cross-validation
control <- trainControl(
  method = "cv",
  number = 3,
  classProbs = TRUE,
  savePredictions = "final"
)
```

In `caret`, we define the training and resampling strategy using the `trainControl()` function. In the example above, we set `method = "cv"` to specify that the model should use *k*-fold cross-validation.

The `number` argument specifies the number of folds. For example, `number = 3` divides the data into three folds. The model is trained on two folds and evaluated on the remaining fold, and this process is repeated until each fold has been used for validation.

We also set `classProbs = TRUE` to save the predicted probability for each classification class. These probabilities indicate how confident the model is that a sample belongs to each diagnosis group. Finally, `savePredictions = "final"` tells `caret` to save the final class predictions generated during cross-validation, allowing us to examine the predictions and evaluate model performance after training.

```
# Bind the previous training and test datasets into a single dataset to be used for k-fold cross validation.
MGX_full_data <- rbind(MGX_training, MGX_test)

# Drop unused levels
classification_data$diagnosis <- droplevels(classification_data$diagnosis)

# Here we set caret to train the model without
fixed_mtry <- 128

rf_grid <- data.frame(
  mtry = fixed_mtry
)
```

Here, we specify a fixed value for the `mtry` tuning parameter. By setting `fixed_mtry <- 128`, we tell `caret` to use 128 randomly selected predictors at every tree split. We then place this value into a one-row data frame called `rf_grid`, with the column named `mtry`.

Providing a tuning grid with only one value prevents `caret` from testing multiple `mtry` values. Cross-validation will therefore estimate the model’s performance while keeping `mtry` fixed, rather than optimizing this hyperparameter.

With a larger dataset, we could evaluate several candidate `mtry` values using cross-validation on the training data. After selecting the value that performs best, we would train the final model using that value and evaluate it on a separate test set that was not used during model training or hyperparameter selection.

```
set.seed(128)

rf_cv_model <- train(
  x = mgx_pathway_data,
  y = classification_data$diagnosis,
  method = "rf",
  trControl = control,
  tuneGrid = rf_grid,
  ntree = 100
)

```

The command above trains a Random Forest model in `caret` using the cross-validation strategy defined with `trainControl()`.

Within the `train()` function:

- `x` specifies the predictor variables. In this analysis, these are the MGX pathway abundances.
- `y` specifies the response variable, or class labels. Here, the response is the diagnosis of each sample.
- `method = "rf"` specifies that `caret` should train a Random Forest model using the `randomForest` package.
- `trControl = control` provides the cross-validation settings defined previously, including the number of folds and the predictions to save.
- `tuneGrid = rf_grid` provides the mtry variable we want each of our training folds to use.
- `ntree = 100` specifies the number of decision trees to grow for each Random Forest model trained during cross-validation.

We can get some information on the models by just calling the saved `rf_cv_model` variable.

```
rf_cv_model
```

```
Random Forest 

 45 samples
254 predictors
  2 classes: 'nonIBD', 'UC' 

No pre-processing
Resampling: Cross-Validated (3 fold) 
Summary of sample sizes: 30, 29, 31 
Resampling results:

  Accuracy   Kappa    
  0.6478175  0.3004658

Tuning parameter 'mtry' was held constant at a value of 128
```

Here we can see that across our model we achieved an accuracy of 0.605. This is better than our previous model but still fairly poor and again not surprising given the small dataset we are working with. 

We can extract the predictions from `rf_cv_model` using the following code:

```
cv_predictions <- rf_cv_model$pred

cv_predictions[1:5, 1:5]
```

```
  mtry   pred    obs nonIBD   UC
1  128     UC     UC   0.24 0.76
2  128     UC     UC   0.22 0.78
3  128 nonIBD nonIBD   0.56 0.44
4  128 nonIBD nonIBD   0.63 0.37
5  128     UC nonIBD   0.39 0.61
```

We can then use the `pred` column and the `obs` column to create a confusion matrix like we did previously. 

```
confusionMatrix(
  data = cv_predictions$pred,
  reference = cv_predictions$obs,
  positive = "UC"
)
```

```
Confusion Matrix and Statistics

          Reference
Prediction nonIBD UC
    nonIBD     17 10
    UC          6 12
                                          
               Accuracy : 0.6444          
                 95% CI : (0.4878, 0.7813)
    No Information Rate : 0.5111          
    P-Value [Acc > NIR] : 0.04975         
                                          
                  Kappa : 0.2857          
                                          
 Mcnemar's Test P-Value : 0.45325         
                                          
            Sensitivity : 0.5455          
            Specificity : 0.7391          
         Pos Pred Value : 0.6667          
         Neg Pred Value : 0.6296          
             Prevalence : 0.4889          
         Detection Rate : 0.2667          
   Detection Prevalence : 0.4000          
      Balanced Accuracy : 0.6423          
                                          
       'Positive' Class : UC   
```

The tutorial and code presented above provide an initial introduction to supervised learning with microbiome data. We covered the basic steps involved in developing and evaluating a classification model, including preparing the data, splitting samples into training and test sets, training a Random Forest model, generating predictions, and assessing model performance using cross-validation and confusion matrices.

These examples are intended to establish a foundation for applying supervised learning methods to microbiome datasets. In practice, additional considerations may be necessary, including feature preprocessing, class imbalance, hyperparameter tuning, model interpretation, and independent validation using an external dataset. To get more information on these we suggest taking a look at this [paper](https://www.nature.com/articles/s41579-023-00984-1).

## Authors

**Authors:** Jacob T. Nearing<br>
