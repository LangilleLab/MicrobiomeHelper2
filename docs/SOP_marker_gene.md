---
layout: default
title: "Marker gene workflow"
subtitle: "SOP: Marker gene workflow"
show_sidetoc: true
show_toc: true
header_type: base
permalink: /docs/marker-gene/
---

## Introduction

## Requirements

## QIIME2 basics

## 1. First steps

### 1.1. Activate QIIME2 environment

### 1.2. Set number of cores

### 1.3. Inspect read quality

### 1.4. 

## 2. Import, trim primers, and denoise

### 2.1. Import FASTQs as QIIME 2 artifact

> <i class="fa-solid fa-bell"></i> <b>Choose one of these!</b><br>
> Either import with the expected Illumina format *or* import with a manifest file. 
{: .alert .alert-info .p-3}

#### 2.1.1. Importing with the expected Illumina format

#### 2.1.2. Importing with a manifest file 

PacBio, Arctic Ocean tutorial data, other SRA downloads, etc

### 2.2. Check the imported .qza file

### 2.3. Summarize raw FASTQs

### 2.4. Trim primers and denoise

> <i class="fa-solid fa-bell"></i> <b>Choose one of these!</b><br>
> Either the Illumina or the PacBio trimming/denoising.
{: .alert .alert-info .p-3}

### 2.4.1. Illumina trim primers and denoise

### 2.4.2. PacBio trim primers and denoise

## 3. Assign taxonomy to ASVs

### 3.1. Build or acquire taxonomic classifier

> <i class="fa-solid fa-bell"></i> <b>Choose one of these!</b><br>
> Either download a taxonomic classifier from somewhere or build one yourself.
{: .alert .alert-info .p-3}

#### 3.1.1. Download a taxonomic classifier

#### 3.1.2. Build a taxonomic classifier

### 3.2. Run taxonomic classification

### 3.3. Assess a subset of taxonomic assignments with BLAST

## 4. Filtering resultant table

### 4.1. Filter out rare ASVs

### 4.2. Filter out contaminant and unclassified ASVs

### 4.3. Exclude low-depth samples

> <i class="fa-solid fa-bell"></i> <b>This is optional!</b><br>
> We often investigate several different cut-offs in our downstream analysis and wait until then. If you skip this here, remember that it is important to check that you don't have any very low depth samples remaining in your analysis!
{: .alert .alert-info .p-3}

### 4.4. Subset and summarize filtered table

## 5. Build tree

> <i class="fa-solid fa-bell"></i> <b>Choose one of these!</b><br>
> Either insert your sequences into an existing tree (recommended for 16S) or build a new tree. 
{: .alert .alert-info .p-3}

### 5.1. Insertion into existing tree with SEPP

### 5.2. Building a tree

## 6. Exporting the final files

### 6.1. Export the fasta file of your ASVs

### 6.2. Export your feature table

### 6.3. Export your phylogenetic tree

## 7. Basic statistics and visualisation within QIIME2

> <i class="fa-solid fa-bell"></i> <b>This is optional!</b><br>
> We'd recommend doing something in R/Python - or as described in our [downstream analysis](/docs/sop-downstream-analysis/) - but if you'd like a quick overview of your results then this is nice :) 
{: .alert .alert-info .p-3}

### 7.1. Format metadata file

### 7.2. Generate rarefaction curves

### 7.3. Calculating diversity metrics and generating ordination plots

### 7.4. Generate stacked bar chart of taxa relative abundances

### 7.5. Identifying differentially abundant features with ANCOM

### 7.6. Phylogenetic Robust Aitchison PCA






## Authors

**Authors:** Robyn Wright, Monica Alvaro Fuss, Nidhi Gohil, André Comeau & Gavin Douglas<br>
**First created:** 3 October 2018<br>
**Last updated:** 16 October 2026<br>
**Based on initial versions by:** Gavin Douglas ([Amplicon SOP v2 (qiime2 2018.6)](https://github.com/LangilleLab/microbiome_helper/wiki/Amplicon-SOP-v2-(qiime2-2018.6)), André Comeau ([PacBio CCS Amplicon SOP v1 (qiime2)](https://github.com/LangilleLab/microbiome_helper/wiki/PacBio-CCS-Amplicon-SOP-v1-(qiime2))), and Robyn Wright [Microbiome Helper 2 Marker gene workflow](https://github.com/LangilleLab/microbiome_helper/wiki/Microbiome-Helper-2-Marker-gene-workflow)

<img src="/assets/images/MicrobiomeHelperLogo.png" alt="Microbiome Helper logo" style="width: 50%; height: auto;">
