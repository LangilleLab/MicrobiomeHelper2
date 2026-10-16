---
layout: default
title: "Metagenomic first steps"
subtitle: "SOP: Metagenome 1 - Initial QC"
show_sidetoc: true
show_toc: true
header_type: base
permalink: /docs/sop-metagenome-1-initial-qc/
---

## Introduction

* The below options are not necessarily best for your data; it is important to explore different options, especially at the read QC stage.
* You should check that only the FASTQs of interest are being specified at each step (e.g. with the `ls` command). If you are re-running commands multiple times or using optional steps the input filenames and/or folders could differ.
* The tool `parallel` comes up several times in this workflow. Be sure to check out our tutorial on this tool [here](https://github.com/LangilleLab/microbiome_helper/wiki/Quick-Introduction-to-GNU-Parallel).
* You can always run parallel with the `--dry-run` option to see what commands are being run to double-check they are doing what you want!

## Requirements

* **This workflow starts with raw paired-end sequencing data in demultiplexed FASTQ format assumed to be located within a folder called `raw_data`.**
* `concat_lanes.pl` script ([here](https://github.com/LangilleLab/microbiome_helper))
* `fastqc`
* `multiqc`
* `kneaddata`
* `bowtie2`
* `Trimmomatic`
* `trf`
* `parallel`

You can see how we set up a conda environment called `kneaddata-v0.12.2` for carrying out these steps [here](https://github.com/LangilleLab/microbiome_helper/wiki/Microbiome-Helper-2-Setting-up-environments-for-analysis).

## 1. First Steps

### 1.1. Join lanes together (only some Illumina)

Concatenate multiple lanes of sequencing together (e.g. for NextSeq550 data; NS2000 does not need this). If you do **NOT** do this step, remember to change ```cat_lanes``` to ```raw_data``` in the further commands below that have assumed you are working with the most common type of metagenome data.

```bash
concat_lanes.pl raw_data/* -o cat_lanes -p 4
```

### 1.2. Inspect read quality

Run ```fastqc``` to allow manual inspection of the quality of sequences.

```bash
mkdir fastqc_out
fastqc -t 4 cat_lanes/* -o fastqc_out/
```

And run ```multiqc``` on this output.

```bash
multiqc fastqc_out/ -o multiqc
```

This allows you to look at the `fastqc` output for all files together, and will generate a html file in `multiqc/multiqc_report.html`.

Note that these steps do not filter out any low quality reads, they simply allow you to look at the quality of your samples. The aim of this step is to see whether anything has gone wrong with your samples; the subsequent ```kneaddata``` steps will allow you to remove low quality reads.

### 1.3. Stitch F+R reads

Stitch paired-end reads together with PEAR (summary of stitching results are written to "pear_summary_log.txt"). Note: it is important to check the % of reads assembled. It may be better to concatenate the forward and reverse reads together if the assembly % is too low (see step 2.2).

```bash
run_pear.pl -p 4 -o stitched_reads cat_lanes/*
```

> <i class="fa-solid fa-bell"></i> <b>This is optional!</b><br>
> If you don't stitch your reads together at this step you will need to unzip your FASTQ files or modify the commands before continuing with the below commands.
{: .alert .alert-info .p-3}

## 2. Read Quality-Control and Contaminant Screens

### 2.1 Running KneadData

Use ```kneaddata``` to run pre-processing tools. First Trimmomatic is run to remove low quality sequences. Then Bowtie2 is run to screen out contaminant sequences. Below we are screening out reads that map to the human or PhiX genomes. Note KneadData is being run below on all unstitched FASTQ pairs with ```parallel```, you can see our quick tutorial on this tool [here](https://github.com/LangilleLab/microbiome_helper/wiki/Quick-Introduction-to-GNU-Parallel). For a detailed breakdown of the options in the below command see [this page](https://github.com/LangilleLab/microbiome_helper/wiki/Metagenomics-Sequencing-Pre-processing). The forward and reverse reads will be specified by "_1" and "_2" in the output files, ignore the "R1" in each filename. Note that the ```\``` characters at the end of each line are just to split the command over multiple lines to make it easier to read.

Depending on what you are sequencing, you will need to choose a different Bowtie2 database. This should contain the PhiX genome and, if you are looking at a host-associated microbiome, the host genome (or something close to it, if none is available). There are several available on the Bowtie2 website [here](https://benlangmead.github.io/aws-indexes/bowtie), or we have some optional instructions on constructing one here.

### 2.2 (Optional) construct Bowtie 2 database

This is an example for creating a database with the chicken *Gallus gallus* and PhiX genomes as well as common vectors and contaminants of sequencing.

First, download the NCBI RefSeq assembly summary:
```bash
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/refseq/assembly_summary_refseq.txt
```

Have a look in this file. You'll see that it contains information on all of the genomes in NCBI RefSeq - we often need reference genomes for constructing various kinds of databases, so it's useful to get familiar with what is available on NCBI. You'll see that each assembly/accession has information on the NCBI taxonomy ID, whether it's a complete genome, whether it's a reference or representative genome, and a link to where we can find the associated files. Because the samples we are using are from chickens, we're going to look for the files that are associated with the chicken Gallus gallus:
```bash
grep "Gallus gallus" assembly_summary_refseq.txt
```

The first one has "reference genome" in the first column, so that's what we'll use. Copy the link that starts `https://ftp.ncbi/.....` into a new browser window. You will see a bunch of different file types, so find the one that ends `_genomic.fna.gz`, right click, and click "Copy Link Address". Now use the `wget` command to download this.

Now we're going to do the same with PhiX. PhiX is added as a sequencing control and is often removed before we're given back our samples, but it's good to make sure.
1. Use grep to search for "phiX174"
2. Go to the link that you find
3. Download the _genomic.fna.gz file using wget

Now we'll combine these two files into one:
```bash
cat GCF_016699485.2_bGalGal1.mat.broiler.GRCg7b_genomic.fna.gz GCF_000819615.1_ViralProj14015_genomic.fna.gz > chicken_Gallusgallus_phiX174.fna.gz
```

And finally, we'll build our Bowtie2 database:
```bash
mkdir bowtie2_db
bowtie2-build chicken_Gallusgallus_phiX174.fna.gz bowtie2_db/chicken_Gallusgallus_phiX174 --threads 1
```

This will take about ten minutes to run.

### 2.3 Running Kneaddata

Run kneaddata:
```bash
parallel -j 1 --link --eta 'kneaddata -i {1} -i {2} -o kneaddata_out/ \
    -db BOWTIE2_DB -t 4 --trimmomatic-options "SLIDINGWINDOW:4:20 MINLEN:50" \
    --bowtie2-options "--very-sensitive --dovetail" --remove-intermediate-output' \
    ::: cat_lanes/*_R1.fastq ::: cat_lanes/*_R2.fastq
```

> <i class="fa-solid fa-triangle-exclamation"></i> Make sure that you replace `BOWTIE2_DB` with the name of the database that you are using! This should have no file extension, so if my folder containing my database was named `bowtie2_databases` had the following files: ```GRCh38_PhiX.1.bt2  GRCh38_PhiX.2.bt2  GRCh38_PhiX.3.bt2  GRCh38_PhiX.4.bt2  GRCh38_PhiX.rev.1.bt2  GRCh38_PhiX.rev.2.bt2``` <br>
> Then you would replace `BOWTIE2_DB` with `bowtie2_databases/GRCh38_PhiX`<br>
> <br>
> If you have unpaired reads, as you would when using the tutorial data (or if you joined them as above), you will need to use the `-un` flag instead of `-i`. 
{: .alert .alert-warning .p-3}

Version for joined reads:
```
parallel -j 1 --link 'kneaddata -un {} -o kneaddata_out/ \
    -db BOWTIE2_DB -t 4 --trimmomatic-options "SLIDINGWINDOW:4:20 MINLEN:50" \
    --bowtie2-options "--very-sensitive --dovetail" --remove-intermediate-output' \
    ::: stitched_reads/*.assembled.fastq
```

Clean up the output directory (helps downstream commands) by moving the discarded sequences to a subfolder:

```
mkdir kneaddata_out/contam_seq
mkdir kneaddata_out/unmatched_seq
mv kneaddata_out/*_contam*.fastq kneaddata_out/contam_seq
mv kneaddata_out/*_unmatched*.fastq kneaddata_out/unmatched_seq
```

You can produce a logfile summarizing the kneadData output with this command:

```
kneaddata_read_count_table --input kneaddata_out --output kneaddata_read_counts.txt
```

> <i class="fa-solid fa-triangle-exclamation"></i> If you are having issues with running Kneaddata and all of your reads are being output as unmatched despite the fastq headers in both files matching, you may want to try using an older version of Kneaddata. It seems that this issue has been [around for a while](https://forum.biobakery.org/t/all-paired-end-read-unmatched/2895/32), and that the simplest fix is to use an older version of Kneaddata. I have found that version 0.7.4 works as expected.
{: .alert .alert-warning .p-3}

### 2.4 (Optional) Rename kneaddata output files

This is just some code that you can use if you would like to. In this example, I am replacing `_1.fastq` with `_R1.fastq`, `_2.fastq` with `_R2.fastq`, and `_L001_R1_001_` with `_` (note that if your file formats are different then you may want/need to modify this, or not use it at all!):
```
cd kneaddata_out

for f in *.fastq; do
    newf="${f//"_1.fastq"/"_R1.fastq"}"
    mv $f $newf
    done
    
for f in *.fastq; do
    newf="${f//"_2.fastq"/"_R2.fastq"}"
    mv $f $newf
    done
    
for f in *.fastq; do
    newf="${f//"_L001_R1_001_"/"_"}"
    mv $f $newf
    done

cd ..
```

I tend to like to rename my samples at this point so that I don't need to do this with every single output from this point on, but it is not necessary.

### 2.5 Concatenate unstitched output (omit if data stitched above)

If you did not stitch your paired-end reads together with PEAR, then you can concatenate the forward and reverse FASTQs together now. Note this is done after quality filtering so that both reads in a pair are either discarded or retained. It's important to only specify the "paired" FASTQs outputted by ```kneaddata``` in the below command.

```
concat_paired_end.pl -p 4 -o cat_reads kneaddata_out/*_paired_*.fastq 
```

You should check over the commands that are printed to screen to make sure the correct FASTQs are being concatenated together.

## Authors

**Authors:** Robyn Wright, Dhwani Desai, Gina Filloramo, Gavin Douglas & André Comeau<br>
**First created:** January 30 2018<br>
**Last updated:** 16 October 2026<br>
**Based on initial versions by:** Gavin Douglas ([Metagenomics standard operating procedure v2](https://github.com/LangilleLab/microbiome_helper/wiki/Metagenomics-standard-operating-procedure-v2)) and André Comeau ([Metagenomics Standard Operating Procedure v3](https://github.com/LangilleLab/microbiome_helper/wiki/Metagenomics-Standard-Operating-Procedure-v3)).

<img src="/assets/images/MicrobiomeHelperLogo.png" alt="Microbiome Helper logo" style="width: 50%; height: auto;">
