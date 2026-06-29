---
layout: default
title: "Marker gene workflow"
show_toc: true
header_type: base
permalink: /docs/marker-gene/
---

> <i class="fa-solid fa-circle-info"></i> This page is currently still a work in progress. You can see the current version on our [Github Wiki](https://github.com/LangilleLab/microbiome_helper/wiki/Microbiome-Helper-2-Marker-gene-workflow).
{: .alert .alert-primary .p-3}

## Introduction

In this workflow, we will go from our raw reads (as they come off a sequencer), and take these all the way through to denoised Amplicon Sequence Variants (ASVs), a feature table containing representative sequences, and a phylogenetic tree.

The pipeline described is embedded in the latest version of QIIME2 (Quantitative Insights into Microbial Ecology version 2025.4), which is a popular microbiome bioinformatics platform for microbial ecology built on user-made software packages called plugins that work on QIIME2 artifact or QZA files. Documentation for these plugins can be found in the [QIIME 2 user documentation](https://qiime2.org/), along with tutorials and other useful information. QIIME2 also provides interpretable visualizations that can be accessed by opening any generated QZV files within [QIIME2 View](https://view.qiime2.org/).

Many steps in this workflow are the same whether you have sequenced your samples using Illumina or PacBio. The steps that differ are split to Illumina/PacBio. 

There are some questions throughout that are designed to make you think about what the steps you are running are doing. The answers provided are for the Arctic Ocean [tutorial data](https://github.com/LangilleLab/microbiome_helper/wiki/Microbiome-Helper-2-Tutorial-data). 

If you use this workflow make sure to keep track of the commands you use locally as this page will be updated over time (see "revisions" above for earlier versions). 

## QIIME 2 basics

To standardize QIIME 2 analyses and to keep track of provenance (i.e. a list of what commands were previously run to produce a file) a special format is used for all QIIME 2 input and output files called an "artifact" (with the extension QZA). In many of these steps, we will also create QIIME 2 files that can be viewed on their [QIIME 2 View](https://view.qiime2.org/) webpage (with the extension QZV). 

## 1. First steps

> <i class="fa-solid fa-circle-info"></i> It is assumed that you already have a folder containing your raw data that is called `raw_data/`. This will contain either single- or paired-end reads. I would recommend creating this inside a folder that describes your project - if you are using the [tutorial data](https://github.com/LangilleLab/microbiome_helper/wiki/Microbiome-Helper-2-Tutorial-data), then this could be inside a folder called `arctic_ocean_illumina` or `arctic_ocean_pacbio` so that you have *e.g.*, `arctic_ocean_illumina/raw_data`
{: .alert .alert-info .p-3}

It is also assumed that these reads are in demultiplexed FASTQ format. QIIME 2 accepts many different formats so if your files are not already in this format (e.g. not demultiplexed) you would just need to use slightly different commands for importing your data.

### 1.1 Activate QIIME 2 environment

You should run the rest of the workflow in a conda environment, which makes sure the correct version of the Python packages required by QIIME 2 are being used. Our [Setting up environments for analysis page](https://github.com/LangilleLab/microbiome_helper/wiki/Microbiome-Helper-2-Setting-up-environments-for-analysis) contains details on how to download and install QIIME2. Note that this workflow should work with the same QIIME2 version as we are using (*i.e.*, in the name of the environment below) - there are not usually major changes between QIIME2 versions, but there may be small changes in the commands needed. 

You can activate this conda environment with this command (you may need to swap in `source` for `conda` if you get an error):
  
```
conda activate qiime2-amplicon-2026.1
```

### 1.2 Set number of cores

Several commands throughout this workflow can run on multiple cores in parallel. How many cores to use in these cases will be saved to the `NCORES` variable defined below. We set this variable to 1 below, but you can change this to be however many cores you would like to use.

```
NCORES=1
```

### 1.3 Inspect read quality

Visualize sequence quality across raw reads. This is important as a sanity check that your reads are of reasonable quality and to determine how your reads should be trimmed in downstream steps. QIIME 2 comes with a plugin for visualizing read quality, which we will use at a later step. However, when dealing with raw reads the easiest method to use is a combination of [FASTQC][15] and [MultiQC][16]. Note that these tools are not packaged with QIIME 2 so you will need to install them separately.

This is an important step for identifying outlier samples with especially low quality, read sizes, read depth, and other metrics. 

> <i class="fa-solid fa-circle-info"></i> Note that we don't actually show the steps for removing any samples here. There are other steps later on where low-quality sequences will be removed, however, if a large number of your samples show very low quality here, this may be an indication that something went wrong with your sequencing and you may want to investigate this further before carrying on with your analysis. 
{: .alert .alert-info .p-3}

You can run FASTQC with this command (after creating the output directory).

```
mkdir fastqc_out
fastqc -t $NCORES raw_data/*.fastq.gz -o fastqc_out
```

If you receive the error `Value "FASTQ" invalid for option threads (number expected)` (where "FASTQ" is an input filename) then make sure you have defined the `NCORES` variable correctly and re-run the command.

FASTQC generates a report for each individual file. To aggregate the summary files into a single report we can run MultiQC with these commands:

```
multiqc fastqc_out --filename multiqc_report.html
```

The full report is found within `multiqc_report.html`. You can view this report in a web-browser on your local computer. The most important reason to visualize this report is to ensure that your samples are of high-quality (based largely on whether the per-base quality is >30 across most of the reads) and that there are no outlier samples.

Remember that you can use the `scp` command to download these files to your local computer. 

### 1.4 Format metadata table

You can format the metadata file (which is in a compatible format to the QIIME 1 mapping files) in a spreadsheet and validate the format using a google spreadsheet plugin as described [on the QIIME 2 website](https://docs.qiime2.org/2022.11/tutorials/metadata/#metadata-formatting-requirements). The only required column is the sample id column, which should be first. All other columns should correspond to sample metadata. 

This can be done like so (for example if your file is called metadata.txt and is found in the folder /home/user):

```
METADATA="/home/user/project_name/metadata.txt"
```

If you are working from the directories/folders that we have suggested, this will likely look like this:

```
METADATA="/home/user/arctic_ocean_illumina/arctic_study_metadata.txt"
```

> <i class="fa-solid fa-circle-info"></i> If you are using the tutorial data that we have provided then you will need to run the following commands to make a new metadata file that has the column names that QIIME2 is expecting, in the right place:
{: .alert .alert-info .p-3}

```
cut -f2- arctic_study_metadata_pacbio.txt > arctic_study_metadata.txt
sed -i 's/sample_rename/sampleid/g' arctic_study_metadata.txt
```
These commands are: (1) removing the first column, and (2) renaming the first column from `sample_rename` to `sampleid`.

> <i class="fa-solid fa-circle-info"></i> Note that we don't actually use this metadata file in the workflow shown on this page! It is used in the next step (QIIME2 statistics and visualisation) and it is always a good idea to have an idea of what your samples are so that you can interpret things about them more easily, but if you don't have one made at this point then that is not a deal-breaker!
{: .alert .alert-info .p-3}

> <i class="fa-solid fa-circle-exclamation"></i> A primary alert
{: .alert .alert-primary .p-3}

> <i class="fa-solid fa-bell"></i> A secondary alert
{: .alert .alert-secondary .p-3}

> <i class="fa-solid fa-bell"></i> A green alert
{: .alert .alert-success .p-3}

> <i class="fa-solid fa-triangle-exclamation"></i> A warning
{: .alert .alert-warning .p-3}

> <i class="fa-solid fa-triangle-exclamation"></i> Danger!
{: .alert .alert-danger .p-3}

> <i class="fa-solid fa-circle-info"></i> Extra information
{: .alert .alert-info .p-3}

> <i class="fa-solid fa-bell"></i> A light alert
{: .alert .alert-light .p-3}

> <i class="fa-solid fa-bell"></i> A dark alert
{: .alert .alert-dark .p-3}

#### Authors

**Authors:** Robyn Wright, Monica Alvaro Fuss, Nidhi Gohil<br>
**Modifications by:** André Comeau<br>
**Based on initial versions by:** Gavin Douglas ([Amplicon SOP v2 (qiime2 2018.6)](https://github.com/LangilleLab/microbiome_helper/wiki/Amplicon-SOP-v2-(qiime2-2018.6)) and André Comeau ([PacBio CCS Amplicon SOP v1 (qiime2)](https://github.com/LangilleLab/microbiome_helper/wiki/PacBio-CCS-Amplicon-SOP-v1-(qiime2)))
