---
layout: default
title: "CBW 2026 module 5 - assigning functions to metagenomic data"
show_sidetoc: true
header_type: base
permalink: /docs/tutorials/cbw-2026-module5/
---

## Introduction

This tutorial is part of the 2026 CBW Microbiome Analysis (held in Guelph, ON, September 15-17). It is based on the metagenomics workflows available on the [Microbiome Helper](https://microbiomehelper.ca/).

**Author:** Robyn Wright

## Overview

The goal here is to introduce students to the different types of functional annotation that we can do, as well as the different things that we can annotate. This is not comprehensive and there are many different types of functional databases out there. Many steps of this can also stand alone, so you can choose which is most applicable to you. We'll cover:
- Read-based functional annotation:
    - General functional annotation of reads using MMSeqs2 with the UniRef 90 database (and linking this with the Kraken 2 taxonomy that we generated in module 3)
    - Annotation of AMR genes in reads using CARD RGI
- Functional annotation of our MAGs in Anvi'o:
    - General functional annotation of MAGs in our Anvi'o database using the NCBI Clusters of Orthologous Groups (COGs)
    - Annotation of MAGs in our Anvi'o database using the CARD RGI to identify AMR genes
    - Visualisation of NCBI COGs and AMR genes in Anvi'o (and combining this with our previous phylogenetic tree and taxonomy information on our MAGs)
- Functional annotation of our MAGs in stand-alone programs:
    - General functional annotation of MAG fasta files using Bakta
    - Annotation of AMR genes in MAGs using CARD RGI
    
> <i class="fa-solid fa-circle-exclamation"></i> NOTE<br>
> Each of these can be run independently, so feel free to choose the one that is of most use/interest to you to start with! It may be a lot to get through in this lab.
{: .alert .alert-primary .p-3}

> <i class="fa-solid fa-circle-exclamation"></i> Throughout this module, there are some questions aimed to help your understanding of some of the key concepts. You’ll find the answers at the bottom of this page, but no one will be marking them.
{: .alert .alert-success .p-3}

## 5.1. MMSeqs initial setup

```
conda activate mmseqs2-18.8cc5c
```

```
ln -s ~/CourseData/UniRef90_2026-01/ .
```

## 5.2. Run MMSeqs

```
mkdir mmseqs_U90_out

parallel -j 4 --progress 'mmseqs createdb {} mmseqs_U90_out/mmseqs-{/.}-queryDB' ::: cat_reads/*

parallel -j 1 --progress 'mmseqs search mmseqs_U90_out/mmseqs-{/.}-queryDB UniRef90_2026-01/UniRef90 mmseqs_U90_out/mmseqs-{/.}-resultDB tmp --db-load-mode 3 --threads 4 --max-seqs 25 -s 1 -a -e 1e-5' ::: cat_reads/*
```

```
cp
```

## 5.3. Get MMSeqs top hits

## 5.4. Combine Kraken taxonomy and MMSeqs functions

## 5.5. AMR annotation of reads using CARD RGI

```
conda activate card-rgi-6.0.8
```

```
cd workspace/metagenome
```

```
ln -s ~/CourseData/card_data/ .
```

```
mkdir card_out
cd card_data

FOLDER='/home/ubuntu/workspace/metagenome/'
export FOLDER
#so all the subshells spawned by parallel can find it

parallel -j 1 --eta "rgi bwt \
                    -1 ${FOLDER}kneaddata_out/{}_R1_subsampled_kneaddata_paired_1.fastq \
                    -2 ${FOLDER}kneaddata_out/{}_R1_subsampled_kneaddata_paired_2.fastq \
                    -n 4 \
                    -o ${FOLDER}card_out/{} \
                    --local \
                    --clean" :::: ${FOLDER}sample_ids.txt
                            
cd ..
ls card_out
```

## 5.6. Functional annotation of MAGs using Anvi'o NCBI COGs

```
anvi-run-ncbi-cogs -c anvio_full/anvio_databases/CONTIGS.db -T 4
```

```
anvi-db-info -c anvio_full/anvio_databases/CONTIGS.db
```

## 5.7. AMR annotation of MAGs with CARD RGI

```
conda activate anvio-9

anvi-get-sequences-for-gene-calls -c anvio_full/anvio_databases/CONTIGS.db \
                                  --get-aa-sequences \
                                  -o contigs_amino_acids.faa
                                  
mkdir card_out_contigs
conda activate card-rgi-6.0.8

ln -s ~/CourseData/card_data/ .

cd card_data

rgi main \
                     -i ${FOLDER}contigs_amino_acids.faa \
                     -o ${FOLDER}card_out_contigs/contigs \
                     -t contig \
                     -a DIAMOND \
                     -n 4 \
                     --include_loose \
                     --local \
                     --clean \
                     --input_type protein
cd ..

echo -e "gene_callers_id\tsource\taccession\tfunction\te_value" > contigs_card_rgi_ARO.txt
tail -n +2 card_out_contigs/contigs.txt | awk -F'\t' '{print $1"\tCARD-ARO\t"$11"\t"$9"\t"$8}' >> contigs_card_rgi_ARO.txt

echo -e "gene_callers_id\tsource\taccession\tfunction\te_value" > contigs_card_rgi_drug-class.txt
tail -n +2 card_out_contigs/contigs.txt | awk -F'\t' '{print $1"\tCARD-drug-class\t"$11"\t"$15"\t"$8}' >> contigs_card_rgi_drug-class.txt

echo -e "gene_callers_id\tsource\taccession\tfunction\te_value" > contigs_card_rgi_gene-family.txt
tail -n +2 card_out_contigs/contigs.txt | awk -F'\t' '{print $1"\tCARD-gene-family\t"$11"\t"$17"\t"$8}' >> contigs_card_rgi_gene-family.txt

conda activate anvio-9 
anvi-import-functions -c anvio_full/anvio_databases/CONTIGS.db \
                      -i contigs_card_rgi_ARO.txt

anvi-import-functions -c anvio_full/anvio_databases/CONTIGS.db \
                      -i contigs_card_rgi_drug-class.txt

anvi-import-functions -c anvio_full/anvio_databases/CONTIGS.db \
                      -i contigs_card_rgi_gene-family.txt
```

```
anvi-db-info -c anvio_full/anvio_databases/CONTIGS.db
```

```
AVAILABLE FUNCTIONAL ANNOTATION SOURCES
===============================================
* CARD-ARO (11,786 annotations)
* CARD-drug-class (11,786 annotations)
* CARD-gene-family (11,786 annotations)
* COG24_CATEGORY (104,084 annotations)
* COG24_FUNCTION (104,084 annotations)
* COG24_PATHWAY (26,425 annotations)
```

## 5.7 Visualisation of MAG functional annotations

```
anvi-script-gen-genomes-file -c anvio_full/anvio_databases/CONTIGS.db \
                             -p anvio_full/anvio_databases/merged_profiles/PROFILE.db \
                             -C "FINAL_dastool" \
                             --output-file internal-genomes-final.txt
```

```
anvi-display-functions -i internal-genomes-final.txt \
                       --annotation-source COG24_PATHWAY \
                       --profile-db COG24_PATHWAY-PROFILE.db \
                       --server-only \
                       -P 8081
                       
anvi-display-functions -i internal-genomes-final.txt \
                       --annotation-source CARD-drug-class \
                       --profile-db CARD-drug-class-PROFILE.db \
                       --server-only \
                       -P 8081
                       
anvi-display-functions -i internal-genomes-final.txt \
                       --annotation-source CARD-drug-class \
                       --profile-db CARD-drug-class-PROFILE-26.db \
                       --min-occurrence 26 \
                       --server-only \
                       -P 8081
                       
anvi-interactive -p CARD-drug-class-PROFILE.db \
                 --manual \
                 --server-only \
                 -P 8081
```

How many drug classes are present in all of our MAGs? 10

```
anvi-display-functions -i internal-genomes-final.txt \
                       --annotation-source CARD-gene-family \
                       --profile-db CARD-gene-family-PROFILE-26.db \
                       --min-occurrence 26 \
                       --server-only \
                       -P 8081
```

How many gene families? 8

Finally, let's make it so we can view these with the other information about our MAGs and the phylogenetic tree:
```{bash}
anvi-script-gen-function-matrix-across-genomes -i internal-genomes-final.txt \
                                               --annotation-source CARD-drug-class \
                                               --output-file-prefix CARD-drug-class-MAGs-final
                                               
anvi-script-gen-function-matrix-across-genomes -i internal-genomes-final.txt \
                                               --annotation-source COG24_PATHWAY \
                                               --output-file-prefix COG24_PATHWAY-MAGs-final
```

You can see that I've chosen the CARD drug classes and COG24 pathways. If you want to choose something different, then you are welcome to. Remember you can see the options like this: `anvi-db-info -c anvio_full/anvio_databases/CONTIGS.db`

If you look at the length of these files like this: `less CARD-drug-class-MAGs-final-PRESENCE-ABSENCE.txt | wc -l` and `COG24_PATHWAY-MAGs-final-PRESENCE-ABSENCE.txt` you'll see that we have 102 drug classes and 76 COG pathways. This is obviously going to be a big much to view, so let's just take a few. I've just selected a few more-or-less at random, so you can changes these if you like. Note that something like this is where our final module on finding functional significance would be useful! 

```{bash}
awk -F'\t' 'NR==1 || $28 == "macrolide antibiotic" || $28 == "carbapenem" || $28 == "nucleoside antibiotic"' CARD-drug-class-MAGs-final-FREQUENCY.txt > CARD-drug-class-MAGs-final-FREQUENCY-filtered.txt

awk -F'\t' 'NR==1 || $28 == "Type X secretion system" || $28 == "Lipid A biosynthesis" || $28 == "Type V secretion system" || $28 == "Asparagine biosynthesis" || $28 == "Pyruvate oxidation" || $28 == "TCA cycle"' COG24_PATHWAY-MAGs-final-FREQUENCY.txt > COG24_PATHWAY-MAGs-final-FREQUENCY-filtered.txt
```

Now let's make a single file with all of the additional data that we want to show, so that we can add it with the `--additional-layers` flag. We're going to use Python for this seeing as we have a few modifications to make. Open it up by typing in `python` and pressing enter. 

Now paste in:
```
import pandas as pd

tax = pd.read_csv('scg_taxonomy_FINAL_dastool_reduced_fixed.txt', index_col=0, header=0, sep='\t')
tax = tax.loc[:, ['t_class', 't_species']]

card = pd.read_csv('CARD-drug-class-MAGs-final-FREQUENCY-filtered.txt', index_col=0, header=0, sep='\t').transpose()
card.columns = card.loc['CARD-drug-class', :]
card = card.drop('CARD-drug-class', axis=0)

cog = pd.read_csv('COG24_PATHWAY-MAGs-final-FREQUENCY-filtered.txt', index_col=0, header=0, sep='\t').transpose()
cog.columns = cog.loc['COG24_PATHWAY', :]
cog = cog.drop('COG24_PATHWAY', axis=0)

combined = pd.concat([tax, card, cog], axis=1)
combined = combined.drop(['HMP2_Bin_00027', 'HMP2_Bin_00028', 'HMP2_Bin_00029', 'HMP2_Bin_00030'], axis=0)

combined.index.name = "bin_name"
combined.to_csv('anvio_taxonomy_card_cog.txt', sep='\t')
```
Once it's finished, type in `quit()` to go back to the regular command line.


And let's view this!
```
anvi-interactive -c anvio_full/anvio_databases/CONTIGS.db \
                 -p anvio_full/anvio_databases/merged_profiles/PROFILE.db \
                 -C "FINAL_dastool" \
                 --additional-layers anvio_taxonomy_card_cog.txt \
                 --tree gtdbtk.bac120.unrooted.filtered.tree \
                 --server-only \
                 -P 8081
```

As you did previously, you can change to the GTDB tree view and change things like the class and species to text, and then sort the other layers however you would like to view them. 

Some other things I personally like to do:
- Change the completion/redundancy to intensity and change the minimum/maximum to reflect your data (i.e. 50-100 for completion and 0-10 for redundancy)
- Reorder the layers so that bin name, class and species are all next to the tree
- Change the colours for antibiotics and the COG groups

## 5.9. General functional annotation of MAG fasta files using Bakta

```
conda activate bakta-1.12.1
export BAKTA_DB=/media/cbwdata/CourseData/tools/bakta/db-light
```

Run:
```
cd workspace/metagenome
mkdir bakta_out
parallel -j 1 'bakta \
               --db $BAKTA_DB \
               --output bakta_out/{/.} \
               --threads 4 \
               {}' ::: MAG_fasta/*
```

## 5.10. AMR annotation in MAGs using CARD RGI

```
conda activate card-rgi-6.0.8
```

```
ln -s ~/CourseData/card_data/ .
```

```
mkdir card_out_MAGs
cd card_data

parallel -j 1 --eta "rgi main \
                     -i {} \
                     -o ${FOLDER}card_out_MAGs/{/.} \
                     -t contig \
                     -a DIAMOND \
                     -n 4 \
                     --include_loose \
                     --local \
                     --clean" ::: ${FOLDER}MAG_fasta/*
```

## 5.11. Other things we're often interested in

## Answers