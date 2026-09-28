---
layout: default
title: "Tutorials"
show_sidetoc: true
header_type: base
permalink: /docs/tutorials/
---

We frequently run in-person workshops where participants work through some data that we provide and we are available to give help. These have often been through the [Canadian Bioinformatics Workshops](https://bioinformatics.ca/) (CBW). In the CBW workshops, we will typically set up an Amazon Web Services server with all of the commands and data needed for the tutorials, and CBW hosts the code and tutorials that we have used afterwards.

The most recent workshops that we ran was in Guelph, ON, between September 15-17 2026. You can find the workshop pages [here](https://bioinformaticsdotca.github.io/MIC_Gue-2609/index.html), but we have also put together a set of tutorials that are based very heavily on these, but have additional instructions for you to be able to run these on your own servers. 

> <i class="fa-solid fa-circle-info"></i> 
> If you run into issues with any of the steps in these tutorials, please feel free to post in the [Github Issues page](https://github.com/LangilleLab/MicrobiomeHelper2/issues).
{: .alert .alert-info .p-3}

### Before starting

In order to run these tutorials, you will need:

* Access to a server with:
    * ~50GB storage (more like 300GB if you would like to run a larger Kraken 2 database and the GTDB-toolkit yourself)
    * 32GB RAM (~110GB if you would like to run the larger Kraken 2 database and ~150GB if you'd like to run GTDB-toolkit yourself)
* [R Studio](https://docs.posit.co/ide/user/#rstudio-ide-oss-downloads) installed either on your server or on your computer

### Tutorials

We recommend following through these tutorials in the order that they are listed here as the tutorials often require data from one of the previous tutorials to work, although obviously this is up to you. 

* **[Introduction to the command line](/docs/tutorials/2026-command-line)**: familiarise yourself with how the command line works prior to attempting any of the subsequent modules
* **[Installing the environments needed](/docs/tutorials/2026-server-setup)**: this contains all instructions for the installation of the programs that we will need for the tutorials. If you are in the Langille lab then these will already be installed on our lab server kronos (although you are welcome to also try installing these for yourself).
* **[1: Marker gene profiling](/docs/tutorials/2026-1-marker-gene-profiling/)**: workflow for processing 16S, 18S or ITS marker gene sequencing data using QIIME2. Includes all commands necessary to go from raw sequencing data to a feature table with counts of ASVs across your samples with their taxonomic classifications, a fasta file containing your ASV sequences, and a phylogenetic tree with your ASVs.
* **[2: Alpha Diversity, Beta Diversity, Differential Abundance and functional prediction with PICRUSt2](/docs/tutorials/2026-2-alpha-beta-diff-abun/)**: workflow for calculating alpha diversity, beta diversity, and differential abundance of the 16S data. This may also be applied to the 18S and ITS data, although these commands are not provided for you. An additional section is functional prediction on 16S data using PICRUSt2
* **[3: Metagenomics and read-based profiling](/docs/tutorials/2026-3-metagenomics-read-based/)**: workflow for processing short-read shotgun metagenomic sequencing data using KneadData (quality control of reads and removal of host sequences), Kraken 2 (taxonomic profiling of reads), GeCoCheck (confirmation of Kraken 2 taxonomic annotations), MetaPhlAn 4 (alternative for taxonomic profiling), and R (visualisation of taxonomic profiles).
* **[4: Metagenomic assembly and binning](/docs/tutorials/2026-4-metagenomic-assembly-binning/)**: workflow for MAG generation within the Anvi'o ecosystem, including assembly of reads with MEGAHIT, running HMMs to identify single-copy genes, taxonomic annotation of contigs, generating contig sample profiles, clustering contigs into bins, refinement of bins, and visualisation of MAGs.
* **[5: Assigning functions to metagenomic data](/docs/tutorials/2026-5-metagenomic-functions/)**: functional assignment to contigs/MAGs within Anvi'o (using NCBI COGs and CARD RGI), reads using MMSeqs 2, or MAGs with Bakta.
* **[6: Visualisation and finding functional significance](/docs/tutorials/2026-6-visualisation-functional-significance/)**: workflow for incorporating covariates and random effects into statistical models, using MaAsLin 3 with metatranscriptomic data, and the basic concepts of supervised machine learning using Random Forests.

<img src="/assets/images/MicrobiomeHelperLogo.png" alt="Microbiome Helper logo" style="width: 50%; height: auto;">
