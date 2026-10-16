---
layout: default
title: "Metagenomic functional annotation of sequences with custom HMM"
subtitle: "SOP: Metagenome 4 - Functional annotation with custom HMM"
show_sidetoc: true
show_toc: true
header_type: base
permalink: /docs/sop-metagenome-4-function-hmm/
---

## Introduction

This workflow is simple in theory, as we are just creating a Hidden Markov Model (HMM) with a known gene of interest and using that to search our sequences (reads, contigs, whatever you would like to search), but there are a few options:
- Validation in representative genomes - we assume that you have some idea of how common you expect this to be in representative genomes, so using it on these first will give an idea of how the HMM with your gene is working (you could also just use whichever genomes you took the genes from to begin with to validate that you get the same ones back)
- Normalisation with Single Copy Core Genes (SCGs) - these are a set of genes that that are known to appear as single copies in most genomes (YchF [COG0012], PheS [COG0016], ArgS [COG0018], SerS [COG0172], CysS [COG0215], LeuS [COG0495], ValS [COG0525], TsaD [COG0533], Ffh [COG0541], and FtsY [COG0552]; as used by [Delmont et al. 2018](https://www.nature.com/articles/s41564-018-0176-9)). If you are looking in reads, in particular, it may be useful to also use these for a rough idea of normalising your gene of interest by the approximate number of genomes in your samples

**Representative genomes**
We have used the GTDB reference database for this, and have taken one genome from each genus at random - hopefully this adequately captures microbial diversity without creating too much of an unweildy database.

We pull out the SCGs from these representative genomes, and you could use similar scripts to also pull out genes of interest - this works best for a function that you are relatively confident in the annotation of. Otherwise, I recommend only using genes with experimentally validated functions.

> <i class="fa-solid fa-bell"></i> If you are using the HMMs for a gene of interest that has already been validated and our SCG HMMs, you can skip straight to step 4.
{: .alert .alert-info .p-3}

## Requirements

### Data needed

### Scripts needed

## 1. Creating the Hidden Markov Model with a gene of interest

### 1.1. Make Multiple Sequence Alignments using Clustal Omega

### 1.2. Build HMMs with HMMER (hmmbuild)

### 1.3. Make phylogenetic trees with gene of interest

## 2. Download representative genomes and create SCG HMMs

### 2.1. Get representative genomes

### 2.2. Get sequences matching SCG names from representative genomes

### 2.3. Make Multiple Sequence Alignments using Clustal Omega

### 2.4. Build HMMs with HMMER (hmmbuild)

## 3. Validate HMM with gene of interest and SCG HMMs

### 3.1. Search in genomes using HMMER (hmmsearch) with SCGs

#### 3.1.1. Run hmmsearch in genomes

#### 3.1.2. Filter hits e-value <10<sup>-25</sup> from output

### 3.2. Search in genomes using HMMER (hmmsearch) with gene of interest

#### 3.2.1. Run hmmsearch in genomes

#### 3.2.2. Filter hits e-value <10<sup>-25</sup> from output

### 3.3. Verify that hits to gene of interest are valid

#### 3.3.1. Align hit sequences against reference sequences using Clustal Omega

#### 3.3.2. Discard hit sequences that don't align well with reference

#### 3.3.3. Insert hit sequences into reference tree with EPA-NG

#### 3.3.4. Get distances to the nearest reference tips in tree using castor

#### 3.3.5. Visualise tree

## 4. Use validated HMMs to search reads/contigs/genomes of interest

### 4.1. Run hmmsearch with SCGs

### 4.2. Filter SCG hits e-value <10<sup>-25</sup> from output

### 4.3. Run hmmsearch with gene of interest

### 4.4. Filter gene of interest hits e-value <10<sup>-25</sup> from output

### 4.5. Align hit sequences against reference sequences

### 4.6. Discard hit sequences that don't align well with reference

### 4.7. Insert hit sequences into reference tree with EPA-NG

### 4.8. Get distances to the nearest reference tips in tree using castor

## 5. Normalise hits to gene of interest by SCG hits


## Authors

**Authors:** Robyn Wright<br>
**First created:** 18 December 2024<br>
**Last updated:** 16 October 2026<br>
**Based on initial versions by:** Robyn Wright ([Microbiome Helper 2 Functional annotation of metagenomic reads with HMMs](https://github.com/LangilleLab/microbiome_helper/wiki/Microbiome-Helper-2-Functional-annotation-of-metagenomic-reads-with-HMMs))

<img src="/assets/images/MicrobiomeHelperLogo.png" alt="Microbiome Helper logo" style="width: 50%; height: auto;">
