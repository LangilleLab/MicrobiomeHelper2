---
layout: default
title: "Metagenomic assembly and binning"
subtitle: "SOP: Metagenome 7 - Assembly and binning"
show_sidetoc: true
show_toc: true
header_type: base
permalink: /docs/sop-metagenome-7-assembly-binning/
---

## Introduction

Note that we have put many of the first steps into loops for running these on different sample sets, but they will work fine even if you only have one set of contigs :) It is always worth thinking about the groups of samples that you assemble together and whether you would expect the same genomes to be found in them or not. We have included a later dereplication step so that you will only have one set of MAGs at the end.

## Requirements

## 1. (Co-)Assembly of raw reads with MEGAHIT

## 2. Make an Anvi'o contigs databases

## 3. Identify SCGs and taxonomy of SCGs

## 4. Map samples onto contigs with Bowtie2

### 4.1. Create Bowtie 2 index

### 4.2. Map samples onto contigs

### 4.3. Add coverage and detection statistics to the Anvi'o profile

### 4.4. Merge sample profiles

## 5. Cluster contigs into bins

### 5.1. Using CONCOCT

### 5.2. Using METABAT2

### 5.3. Using Binsanity2

### 5.4. Using MAXBIN2

### 5.5. Combine results using DASTOOL

#### 5.5.1. Estimate taxonomy for all

#### 5.5.2. Run DASTOOL

#### 5.5.3. Summarise final bins

## 5.6. Interactive refinement of bins

### 5.6.1. Refinement

### 5.6.2. Export the final bins into MAGs

## 6. Get taxonomy and quality of MAGs

### 6.1. Get taxonomy of MAGs with GTDB-tk

### 6.2. Get quality of MAGs with CheckM2

## 7. Get non-redundant MAGs

### 7.1. Collate MAG stats

Taxonomic affiliations (phylum, length, stats)

### 7.2. Run skani on MAGs within the same phyla

### 7.3. Group MAGs based on ANI 

Default >= 98% ANI with minimum alignment of 75% of shorter genome

### 7.4. Make new collection of non-redundant MAGs (nrMAGs)

### 7.5. Filter final GTDB-tk and CheckM2 files to include only nrMAGs


**Authors:** Robyn Wright<br>
**First created:** 4 June 2025<br>
**Last updated:** 16 October 2026<br>
**Based on initial versions by:** Robyn Wright ([Microbiome Helper 2 MAG assembly, binning, and curation with Anvi'o](https://github.com/LangilleLab/microbiome_helper/wiki/Microbiome-Helper-2-MAG-assembly,-binning,-and-curation-with-Anvi'o))

<img src="/assets/images/MicrobiomeHelperLogo.png" alt="Microbiome Helper logo" style="width: 50%; height: auto;">
