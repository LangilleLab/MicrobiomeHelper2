---
layout: default
title: "Metagenomic taxonomic annotation workflow"
subtitle: "SOP: Metagenome 2 - Taxonomic annotation"
show_sidetoc: true
show_toc: true
header_type: base
permalink: /docs/sop-metagenome-2-taxonomy/
---

## Introduction

## Requirements

## 1. Taxonomic assignment of sequences using Kraken 2 and complete database

We recommend Kraken 2 + all NCBI genomes if possible. Kraken non-redundant database is next-best.

Our in-lab database uses an 803 GB database built using RefSeq V214 genomes (803GB; updated version of the best database described in [our paper](https://doi.org/10.1099/mgen.0.000949)): 

- all bacteria + archaea + viruses  
- only complete eukaryotic genomes  
- other adapter + plasmid sequences  

### 1.1. Build or acquire Kraken 2 database

### 1.2. Run Kraken 2

> <i class="fa-solid fa-bell"></i> <b>Confidence thresholds</b><br>
> Some more of a note to go here!
{: .alert .alert-info .p-3}

## 2. Separation of sequences into different domains

### 2.1. Get bacterial, archaeal, and unclassified sequences

### 2.2. Get eukaryotic sequences

### 2.2.1. Get all eukaryotic sequences

### 2.2.2 Get non-metazoan sequences

### 2.3. Get viral sequences

## 3. Classify bacteria, archaeal, and unclassified with Kraken 2 + GTDB database

### 3.1. Run Kraken 2

### 3.2. Run GeCoCheck

### 3.3. Filter results based on GeCoCheck output

## 4. Classify non-metazoan eukaryotes with Kraken 2 + microEuk database

### 4.1. Run Kraken 2

### 4.2. Run GeCoCheck

### 4.3. Filter results based on GeCoCheck output

## 5. Collate all output

## Authors

**Authors:** Robyn Wright, Dhwani Desai, Gina Filloramo & André Comeau<br>
**First created:** 2 June 2021<br>
**Last updated:** 16 October 2026<br>
**Based on initial versions by:** André Comeau ([Metagenomics Standard Operating Procedure v3](https://github.com/LangilleLab/microbiome_helper/wiki/Metagenomics-Standard-Operating-Procedure-v3)) and Robyn Wright ([Microbiome Helper 2 Annotation of reads with Kraken2 (and taxonomic profiling with Bracken)](https://github.com/LangilleLab/microbiome_helper/wiki/Microbiome-Helper-2-Annotation-of-reads-with-Kraken2-(and-taxonomic-profiling-with-Bracken)))

<img src="/assets/images/MicrobiomeHelperLogo.png" alt="Microbiome Helper logo" style="width: 50%; height: auto;">
