---
layout: default
title: "CBW 2026 module 4 - metagenomic assembly and binning"
show_sidetoc: true
header_type: base
permalink: /docs/tutorials/cbw-2026-module4/
---

## Introduction

This tutorial is part of the 2026 CBW Microbiome Analysis (held in Guelph, ON, September 15-17). It is based on the metagenomics workflows available on the [Microbiome Helper](https://microbiomehelper.ca/) and is based on previous versions written for the [2024 CBW Advanced Microbiome Analysis workshop](https://github.com/LangilleLab/microbiome_helper/wiki/CBW%E2%80%90ICG%E2%80%90AMB%E2%80%90Module2).

**Author:** Robyn Wright

## Overview

The main goal of this tutorial is to introduce students to the assembly of genomes from metagenomic reads (Metagenome Assembled Genomes/MAGs). There is not a one-size-fits-all pipeline for assembling MAGs. MAG assembly is incredibly computationally intensive with a lot of differen options at many steps, and so the approach here is to demonstrate the main steps involved and give you some familiarity with the methods used. At the end of this tutorial we've provided a few other pipelines for MAG assembly that you may wish to look into if you are looking to assemble MAGs with your own metagenome data.

> <i class="fa-solid fa-circle-exclamation"></i> Throughout this module, there are some questions aimed to help your understanding of some of the key concepts. You’ll find the answers at the bottom of this page, but no one will be marking them.
{: .alert .alert-success .p-3}

### Anvi'o

[Anvi'o](https://anvio.org/) is an open-source, community-driven `an`alysis and `vi`sualization platform for microbial `'o`mics. It packages together many different tools used for genomics, metagenomics, metatranscriptomics, phylogenomics, etc. and has great interactive visualisations that can be used to help this. We are just touching the surface of what Anvi'o can do today, but the website has great tutorials and learning resources for all of its capabilities - I recommend browsing through to get some inspiration!

## 4.1. Initial setup

Hopefully, at the end of module 3 you were able to get MEGAHIT started. If you were, go back into your `tmux` session to see how it is going: `tmux a`
This usually takes about 2 hours to run with this data, so hopefully it is finished now! In any case, go to the next step where I explain what it is that we did there. 

If you didn't get here

If you didn't get here, you will need to copy over the data as we won't have enough time for the assembly to run now. 


> <i class="fa-solid fa-circle-info"></i> This page is currently still a work in progress. 
{: .alert .alert-primary .p-3}

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

## Authors

**Authors:** Robyn Wright<br>