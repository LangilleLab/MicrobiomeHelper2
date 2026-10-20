---
layout: default
title: "Toy/tutorial data"
subtitle: "SOP: Toy/tutorial data"
show_sidetoc: true
show_toc: true
header_type: base
permalink: /docs/sop-toy-data/
---

This page contains links to the studies that we have taken the tutorial data from, links to download the data, and details on how we have made these files.

## Marker gene

We are using samples from this study:
> Pascoal, F., Duarte, P., Assmy, P. & Costa, R. (2024) **[Full-length 16S rRNA gene sequencing combined with adequate database selection improves the description of Arctic marine prokaryotic communities](https://annalsmicrobiology.biomedcentral.com/articles/10.1186/s13213-024-01767-6)**. *Annals of Microbiology*.

This study contains both full-length (PacBio) and V4-V5 region (Illumina) 16S rRNA gene sequencing data for samples collected from the Arctic Ocean.

- [View marker gene data](http://kronos.pharmacology.dal.ca/public_files/MH2/marker_gene/)
- [Download Illumina V4-V5 data](http://kronos.pharmacology.dal.ca/public_files/MH2/marker_gene/illumina/raw_data.tar.gz)
- [Download Illumina metadata](http://kronos.pharmacology.dal.ca/public_files/MH2/marker_gene/illumina/arctic_study_metadata_illumina.txt)
- [Download PacBio data](http://kronos.pharmacology.dal.ca/public_files/MH2/marker_gene/pacbio/raw_data.tar.gz)
- [Download PacBio metadata](http://kronos.pharmacology.dal.ca/public_files/MH2/marker_gene/pacbio/arctic_study_metadata_pacbio.txt)

If you just want to run through the tutorials, then you can download the data to your laptop/server like so.

**Illumina**:
```bash
mkdir arctic_ocean_illumina/
cd arctic_ocean_illumina/

wget http://kronos.pharmacology.dal.ca/public_files/MH2/marker_gene/illumina/raw_data.tar.gz
wget http://kronos.pharmacology.dal.ca/public_files/MH2/marker_gene/illumina/arctic_study_metadata_illumina.txt

tar -xvf raw_data.tar.gz
rm raw_data.tar.gz
```

**PacBio**:
```bash
mkdir arctic_ocean_pacbio/
cd arctic_ocean_pacbio/

wget http://kronos.pharmacology.dal.ca/public_files/MH2/marker_gene/pacbio/raw_data.tar.gz
wget http://kronos.pharmacology.dal.ca/public_files/MH2/marker_gene/pacbio/arctic_study_metadata_pacbio.txt

tar -xvf raw_data.tar.gz
rm raw_data.tar.gz
```

## Authors

**Authors:** Robyn Wright<br>
**First created:** 19 October 2026<br>
**Last updated:** 19 October 2026<br>
**Based on initial versions by:** NA

<img src="/assets/images/MicrobiomeHelperLogo.png" alt="Microbiome Helper logo" style="width: 50%; height: auto;">
