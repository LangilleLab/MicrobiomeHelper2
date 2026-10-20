---
layout: default
title: "Metagenome"
subtitle: "SOP: Metagenome"
header_type: base
permalink: /docs/metagenome/
---

<img src="/assets/images/Workflows_metagenome.png" alt="Metagenome overview" style="width: 100%; height: auto;">

This page will contain more, but for now, these are all of the parts:

#### [Metagenome 1: initial quality-control](/docs/sop-metagenome-1-initial-qc/)

#### [Metagenome 2: taxonomic annotation](/docs/sop-metagenome-2-taxonomy/)

#### [Metagenome 3: functional annotation with MMSeqs 2](/docs/sop-metagenome-3-function-mmseqs/)

#### [Metagenome 4: functional annotation with custom HMM](/docs/sop-metagenome-4-function-hmm/)

#### [Metagenome 5: functional annotation of AMR genes with CARD RGI](/docs/sop-metagenome-5-function-rgi/)

#### [Metagenome 6: linking taxonomy and function](/docs/sop-metagenome-6-link-taxonomy-function/)

#### [Metagenome 7: assembly and binning with Anvi'o](/docs/sop-metagenome-7-assembly-binning/)

#### [Metagenome 8: generating multi-tier profiles (combining annotations from MAGs, contigs, and reads)](/docs/sop-metagenome-8-multitier/)

## Notes before starting

- I would highly recommend using something like `tmux` or `screen` so that things can stay running even if you get logged out of your server. You can see some information on `tmux` [here](/docs/cheatsheet/#keeping-things-running-even-if-you-get-disconnected-from-your-server)
- See our [cheatsheet](/docs/cheatsheet/) for other useful commands that we'll use
- We have some information on setting up a server [here](/docs/tutorials/2026-server-setup/), but generally we are assuming that you are already somewhat familiar
- In time, we will have an Amazon image and instructions on using this available, if you would like to go right ahead with analysing your data without needing to set anything up for yourself

<br><br>


<img src="/assets/images/MicrobiomeHelperLogo.png" alt="Microbiome Helper logo" style="width: 50%; height: auto;">
