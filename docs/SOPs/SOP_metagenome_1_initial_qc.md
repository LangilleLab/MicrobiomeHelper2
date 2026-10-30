---
layout: default
title: "Metagenomic first steps"
subtitle: "SOP: Metagenome 1 - Initial QC"
show_sidetoc: true
show_toc: true
header_type: base
permalink: /docs/sop-metagenome-1-initial-qc/
---

<img src="/assets/images/Workflows_metagenome.png" alt="Metagenome overview" style="width: 100%; height: auto;">

> <i class="fa-solid fa-circle-exclamation"></i> <b>Workflow status:</b> everything that you see here should be working and has been tested with real data. 
{: .alert .alert-primary .p-3}

## Introduction

* The below options are not necessarily best for your data; it is important to explore different options, especially at the read QC stage.
* You should check that only the FASTQs of interest are being specified at each step (e.g. with the `ls` command). If you are re-running commands multiple times or using optional steps the input filenames and/or folders could differ.
* The tool `parallel` comes up several times in this workflow. Be sure to check out our tutorial on this tool [here](https://microbiomehelper.ca/docs/cheatsheet/#running-the-same-command-on-multiple-files---a-crash-course-in-gnu-parallel).
* You can always run parallel with the `--dry-run` option to see what commands are being run to double-check they are doing what you want!

## Requirements

We are assuming here that you are already familiar with the command line and can install these packages for yourself. You will need the following conda environments:

* `quality_control_sep2026` - environment containing `fastqc`, `multiqc` and `kneaddata`

See our [server setup tutorial](/docs/tutorials/2026-server-setup/) for more information. 

**This workflow starts with raw paired-end sequencing data in demultiplexed FASTQ format assumed to be located within a folder called `raw_data`.**

## Tools used

* `fastqc`
* `multiqc`
* `kneaddata`
* `bowtie2`
* `Trimmomatic`
* `trf`
* `parallel`

## 1. First Steps

### 1.1. Set number of cores

Several commands throughout this workflow can run on multiple cores in parallel. How many cores to use in these cases will be saved to the `NCORES` variable defined below. We set this variable to 1 below, but you can change this to be however many cores you would like to use.

```bash
export NCORES=1
```

For metagenomic analyses, you will find many of these steps very slow if you are only using 1 core, so we would highly recommend increasing this if possible!

### 1.1. Join lanes together (only some Illumina)

Concatenate multiple lanes of sequencing together (e.g. for NextSeq550 data; NS2000 does not need this). If you do this step, remember to change ```raw_data``` to ```cat_lanes``` in the further commands below that have assumed you are working with the most common type of metagenome data.

You can download our `concat_lanes.pl` script [here](https://kronos.pharmacology.dal.ca:8080/public_files/MH2/scripts/concat_lanes.pl) like so:
```bash
wget https://kronos.pharmacology.dal.ca:8080/public_files/MH2/scripts/concat_lanes.pl
```

And then run the script:
```bash
concat_lanes.pl raw_data/* -o cat_lanes -p $NCORES
```

### 1.2. Inspect read quality

Run ```fastqc``` to allow manual inspection of the quality of sequences.

```bash
mkdir fastqc_out
fastqc -t $NCORES raw_data/* -o fastqc_out/
```

And run ```multiqc``` on this output.

```bash
multiqc fastqc_out/ -o multiqc
```

This allows you to look at the `fastqc` output for all files together, and will generate a html file in `multiqc/multiqc_report.html`.

Note that these steps do not filter out any low quality reads, they simply allow you to look at the quality of your samples. The aim of this step is to see whether anything has gone wrong with your samples; the subsequent ```kneaddata``` steps will allow you to remove low quality reads.

## 2. Read Quality-Control and Contaminant Screens

### 2.1. Running KneadData

Use ```kneaddata``` to run pre-processing tools. First Trimmomatic is run to remove low quality sequences. Then Bowtie2 is run to screen out contaminant sequences. 

Before running Kneaddata, you will want a Bowtie2 database that contains any reads that you expect to be contaminants in your samples. If you were sequencing human-associated samples, you would want to remove human reads from your analysis and would therefore include the human genome in your Bowtie2 database. We have also found it to be beneficial to include common vectors and contaminants in our Bowtie2 database, along with the PhiX genome (commonly used as a sequencing positive control). 

There are several available databases on the Bowtie2 website [here](https://benlangmead.github.io/aws-indexes/bowtie), or we have some optional instructions on constructing one here.

### 2.1.1. Construct Bowtie 2 database

> <i class="fa-solid fa-bell"></i> <b>This is optional!</b><br>
> The alternative is to download a database from the Bowtie2 website, as described above.
{: .alert .alert-info .p-3}

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

I would also download the common vectors and contaminants of sequencing:
```bash
wget https://ftp.ncbi.nlm.nih.gov/pub/UniVec/UniVec -O Univec.fasta
```

And zip it so that it matches the format of the other ones:
```bash
gzip Univec.fasta
```

Now we'll combine these three files into one:
```bash
cat GCF_016699485.2_bGalGal1.mat.broiler.GRCg7b_genomic.fna.gz GCF_000819615.1_ViralProj14015_genomic.fna.gz Univec.fasta.gz > chicken_Gallusgallus_phiX174_univec.fna.gz
```

And finally, we'll build our Bowtie2 database:
```bash
mkdir bowtie2_db
bowtie2-build chicken_Gallusgallus_phiX174_univec.fna.gz bowtie2_db/chicken_Gallusgallus_phiX174_univec --threads $NCORES
```

This will take about ten minutes to run.

### 2.1.2. Running Kneaddata

Run kneaddata:
```bash
parallel -j 1 --link --eta 'kneaddata \
                            -i1 {1} \
                            -i2 {2} \
                            -o kneaddata_out/ \
                            -db BOWTIE2_DB \
                            -t $NCORES \
                            --trimmomatic-options "SLIDINGWINDOW:4:20 MINLEN:50" \
                            --bowtie2-options "--very-sensitive --dovetail" --remove-intermediate-output' \
                            ::: raw_data/*_R1*.fastq.gz \
                            ::: raw_data/*_R2*.fastq.gz
```

> <i class="fa-solid fa-triangle-exclamation"></i> Make sure that you replace `BOWTIE2_DB` with the name of the database that you are using! This should have no file extension, so if my folder containing my database was named `bowtie2_databases` had the following files: ```GRCh38_PhiX.1.bt2  GRCh38_PhiX.2.bt2  GRCh38_PhiX.3.bt2  GRCh38_PhiX.4.bt2  GRCh38_PhiX.rev.1.bt2  GRCh38_PhiX.rev.2.bt2``` <br>
> Then you would replace `BOWTIE2_DB` with `bowtie2_databases/GRCh38_PhiX`<br>
> <br>
> If you have unpaired reads, as you would when using the tutorial data (or if you joined them as above), you will need to use the `-un` flag instead of `-i`. 
{: .alert .alert-warning .p-3}

Version for joined reads:
```
parallel -j 1 --link --eta 'kneaddata \
                            -un {} \
                            -o kneaddata_out/ \
                            -db BOWTIE2_DB \
                            -t 4 \
                            --trimmomatic-options "SLIDINGWINDOW:4:20 MINLEN:50" \
                            --bowtie2-options "--very-sensitive --dovetail" --remove-intermediate-output' \
                            ::: unpaired_reads/*.fastq
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

### 2.2. Rename kneaddata output files

I tend to like to rename my samples at this point so that I don't need to do this with every single output from this point on, but it is not necessary.

This is just some code that you can use if you would like to. In this example, I am replacing `_1.fastq` with `_R1.fastq`, `_2.fastq` with `_R2.fastq`, `_L001_R1_001_` with `_`, and `_kneaddata_paired_` with `_` (note that if your file formats are different then you may want/need to modify this, or not use it at all!):
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
    
for f in *.fastq; do
    newf="${f//"_kneaddata_paired_"/"_"}"
    mv $f $newf
    done

cd ..
```

### 2.3. Concatenate unstitched output (omit if data stitched above)

You can concatenate the forward and reverse FASTQs together now. Note this is done after quality filtering so that both reads in a pair are either discarded or retained. It's important to only specify the "paired" FASTQs outputted by ```kneaddata``` in the below command (we moved the other files into other folders, so these are the only ones remaining in this folder).

```
concat_paired_end.pl \
        -p $NCORES \
        -o cat_reads \
        kneaddata_out/*.fastq 
```

You should check over the commands that are printed to screen to make sure the correct FASTQs are being concatenated together.

### 2.4. Run Nonpareil

> <i class="fa-solid fa-bell"></i> <b>This is optional!</b><br>
> The aim of this step is to estimate the sequencing coverage that you likely have (i.e. if you had sequenced to a higher read depth, would you see higher diversity?)
{: .alert .alert-info .p-3}

Make an output directory:
```bash
mkdir nonpareil_out
```

And run Nonpareil on samples:
```bash
parallel -j 1 --eta 'nonpareil \
                    -s {} \
                    -T kmer \
                    -f fastq \
                    -b nonpareil_out/{/.} \
                    -t $NCORES' \
                    ::: cat_reads/*
```

We can then plot these in R using the `nonpareil` R package. To enter R in the command line, type `R` and press enter. Then run the following:
```R
install.packages("Nonpareil") #note that this may prompt you to select a mirror for the session. Choose whichever you like. 
library(Nonpareil)

files <- list.files("nonpareil_out", pattern = "\\.npo$", full.names = TRUE)
np <- Nonpareil.set(files, labels = sub("\\.npo$", "", basename(files)))
write.csv(summary(np), "nonpareil_summary.csv")
```

When you are done, type `quit()` and enter to exit. A summary of the Nonpareil results should now be saved to `nonpareil_summary.csv`.

## Authors

**Authors:** Robyn Wright, Dhwani Desai, Gina Filloramo, Gavin Douglas & André Comeau<br>
**First created:** January 30 2018<br>
**Last updated:** 29 October 2026<br>
**Based on initial versions by:** Gavin Douglas ([Metagenomics standard operating procedure v2](https://github.com/LangilleLab/microbiome_helper/wiki/Metagenomics-standard-operating-procedure-v2)) and André Comeau ([Metagenomics Standard Operating Procedure v3](https://github.com/LangilleLab/microbiome_helper/wiki/Metagenomics-Standard-Operating-Procedure-v3)).

<img src="/assets/images/MicrobiomeHelperLogo.png" alt="Microbiome Helper logo" style="width: 50%; height: auto;">
