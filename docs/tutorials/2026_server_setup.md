---
layout: default
title: "2026 server setup"
show_sidetoc: true
header_type: base
permalink: /docs/tutorials/2026-server-setup/
---

This is not a module that we have previously run, but is a page that contains instructions for installing all of the programs that we use throughout these tutorials. If you do not plan to run all of the tutorials, you can see which environments are needed for each of the tutorials at the top of the page and install only these.

## Introduction

If you have not run through the [Introduction to the command line](/docs/tutorials/2026-command-line/) then I highly recommend doing this first! This page assumes that you have all knowledge from there already. 

We will be creating `conda` environments that we have used for each of the tutorial sections and installing the necessary programs to these, as well as any necessary databases. 

All of these environment installs will follow the format:

1. Create environment: ```conda create -y --name env_name```
2. Activate environment: ```conda activate env_name```
3. Install required programs: ```conda install -y package1 package2```

I recommend creating and changing into a directory called `tools`. Many of these will not create any folders in here, but if they do then it is nice to keep them together.
```bash
mkdir tools
cd tools
```

> <i class="fa-solid fa-circle-info"></i><b> A note on databases</b><br>
> If you are on a shared server, you may wish to designate a spot for all databases to go to avoid storing unnecessary duplicates (especially when these may be several hundred GB in size). On our lab servers, this is typically something like `/home/shared/MH2/databases/`, but this is something to think about if you are setting up your own server.<br>
> We will assume that you will change into this folder before downloading any databases within these tutorials.
{: .alert .alert-info .p-3}

Note that here we have tried to note which programs you are using each time, but if you use the same workflow for your own analysis then it is really important to cite all of the tools that you use!

## Quality control

This environment has:

- [`fastqc`](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
- [`multiqc`](https://github.com/multiqc/multiqc)

Create the environment:
```bash
conda create -y --name quality_control
```

Activate the environment:
```bash
conda activate quality_control
```

Install the programs:
```bash
conda install -y bioconda::fastqc bioconda::multiqc
```

Deactivate the environment (optional):
```bash
conda deactivate
```

## QIIME2

This environment has:

- [`QIIME2`](https://qiime2.org/)

First set default channel priority within conda:
```bash
conda config --set channel_priority flexible
conda clean --all -y
```

Create the environment and install QIIME2:
```bash
conda env create \
  --name rachis-qiime2-2026.7 \
  --file https://raw.githubusercontent.com/qiime2/distributions/refs/heads/dev/2026.7/qiime2/released/rachis-qiime2-linux-64-conda.yml
```

Activate the environment:
```bash
conda activate rachis-qiime2-2026.7
```

Test environment:
```bash
qiime info
```

If you run into R errors, you may need to run:
```bash
mkdir -p $CONDA_PREFIX/etc/conda/activate.d
mkdir -p $CONDA_PREFIX/etc/conda/deactivate.d
echo 'export R_PROFILE_USER=""' > $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh
echo 'export R_ENVIRON_USER=""' >> $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh
echo 'export RENV_CONFIG_SANDBOX=true' >> $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh
echo 'export R_LIBS_SITE=""' >> $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh
echo 'export R_LIBS_USER=""' >> $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh
echo "R_PROFILE_INITIAL=''" >> $CONDA_PREFIX/lib/R/etc/Renviron
echo "R_LIBS_SITE=''" >> $CONDA_PREFIX/lib/R/etc/Renviron
```

And then close your terminal window. Reopen a new one and run:
```bash
conda activate rachis-qiime2-2026.7
qiime info
```

Only run these last two steps if you got an error when running ```qiime info```!!

Deactivate the environment (optional):
```bash
conda deactivate
```

## Kneaddata

This environment has:

- [Kneaddata](https://huttenhower.sph.harvard.edu/kneaddata/) (and all dependencies)
- [Bowtie2 version 2.5.4](https://bowtie-bio.sourceforge.net/bowtie2/index.shtml) (we have specified this version because the latest version prints an annoying warning message every time it is run)
- [GNU parallel](https://www.gnu.org/software/parallel/) (a program that allows us to run multiple samples at once)

Create the environment:
```bash
conda create -y --name kneaddata-0.12.4
```

Activate the environment:
```bash
conda activate kneaddata-0.12.4
```

Install the programs:
```bash
conda install -y bioconda::kneaddata parallel bioconda::bowtie2=2.5.4
```

If you'd like to install the default database, you can do that like so:
```bash
kneaddata_database --download human_genome bowtie2 human_bt2db
```

If you'd like to install a custom database (i.e. for a different host organism) then you can see details on doing that [here](https://github.com/LangilleLab/microbiome_helper/wiki/Microbiome-Helper-2-Metagenomics-initial-Steps#construct-bowtie2-database).

Deactivate the environment (optional):
```bash
conda deactivate
```

## Kraken

Note that Kraken is a little more complicated as it seems to need the Github installation to ensure that it all works as expected. But we still make a conda environment and activate it:
```bash
conda create -y --name kraken-2.17.1
conda activate kraken-2.17.1
```

Then, make sure you are in your `tools` directory that you made above and run the following:
```bash
git clone https://github.com/DerrickWood/kraken2
cd kraken2
./install_kraken2.sh .
```

Now, we want to make sure that the programs we need are added to our environment and can be found:
```bash
mkdir $CONDA_PREFIX/bin
cp kraken2{,-build,-inspect} $CONDA_PREFIX/bin/
```

And install the other programs that we will need:
```bash
conda install -y bioconda::bracken parallel
```

For this step, you will want to change into the directory that you have decided your databases will live in. If this isn't a shared area, then just somewhere that you can remember.

For Kraken 2, it is important to have the largest database that your server can handle. You can see all available Kraken 2 databases [here](https://benlangmead.github.io/aws-indexes/k2) - note that if you choose a different one than I have, you will need to modify these commands.

Change to the directory that your database will live in:
```bash
cd database_directory
```

Standard smallest database:
```bash
wget https://genome-idx.s3.amazonaws.com/kraken/k2_standard_08_GB_20260626.tar.gz
mkdir k2_standard_08_GB_20260626
tar -xvf k2_standard_08_GB_20260626.tar.gz -C k2_standard_08_GB_20260626
```

If you are happy that the database is downloaded correctly, remove the tar file:
```bash
rm k2_standard_08_GB_20260626.tar.gz
```

Standard plus protozoa and fungi:
```bash
wget https://genome-idx.s3.amazonaws.com/kraken/k2_pluspf_20260626.tar.gz
mkdir k2_pluspf_20260626
tar -xvf k2_pluspf_20260626.tar.gz -C k2_pluspf_20260626
```

If you are happy that the database is downloaded correctly, remove the tar file:
```bash
rm k2_pluspf_20260626.tar.gz
```

Deactivate the environment (optional):
```bash
conda deactivate
```

## MetaPhlAn

```bash
conda create -y --name metaphlan-4.2.6
conda activate metaphlan-4.2.6
conda install -y bioconda::metaphlan bioconda::bowtie2=2.5.4 parallel
```

Get the database:
```bash
metaphlan --install --db_dir metaphlan_databases
```

If you - like me - need to try to install these in a shared location that you need `sudo` access to copy to, you can download them as above to an accessible area and then run the following:
```bash
sudo cp -r metaphlan_databases/* /home/shared/MH2/databases/metaphlan_databases/
```
Note that you may need to specify the database location when you are running MetaPhlAn.

Deactivate the environment (optional):
```bash
conda deactivate
```

## Anvi'o

This follows the instructions [here](https://anvio.org/install/).

Create and activate the Anvi'o environment:
```bash
conda create -y --name anvio-9 python=3.10
conda activate anvio-9
```

Install some dependencies:
```bash
conda install -y -c conda-forge -c bioconda python=3.10 \
        sqlite=3.46 prodigal idba mcl muscle=3.8.1551 famsa hmmer diamond \
        blast megahit spades bowtie2=2.5.4 bwa graphviz "samtools>=1.9" \
        trimal iqtree trnascan-se fasttree vmatch r-base r-tidyverse \
        r-optparse r-stringi r-magrittr bioconductor-qvalue meme ghostscript \
        nodejs=20.12.2 llvmlite numba
        
conda install -y -c bioconda fastani
```
Note that if `fastani` doesn't install properly then this is likely fine.

Now get and install Anvi'o:
```bash
curl -L https://github.com/merenlab/anvio/releases/download/v9/anvio-9.tar.gz \
        --output anvio-9.tar.gz
        
pip install anvio-9.tar.gz
```

Remove the Anvi'o tar file:
```bash
rm anvio-9.tar.gz
```

Download the SCG taxonomy and NCBI COGs (functional information):
```bash
anvi-setup-scg-taxonomy
anvi-setup-ncbi-cogs
```

Install the binners for clustering contigs:
```bash
#CONCOCT
cd tools
git clone https://github.com/merenlab/CONCOCT.git

cd CONCOCT
pip install cython
python setup.py build
python setup.py install

#other binners
conda install -y bioconda::metabat2 bioconda::maxbin2 bioconda::das_tool bioconda::binsanity
#conda install -y -c bioconda metabat2
#conda install -y -c bioconda maxbin2
#conda install -y -c bioconda das_tool
#conda install -y -c bioconda binsanity
```

Install some specific versions of things:
```bash
pip install "setuptools<82"
pip install nose
conda install -y -c bioconda -c conda-forge "scikit-learn==1.1.0" usearch diamond
```

Test Anvi'o:
```bash
anvi-self-test --suite mini --no-interactive

#pip install gffutils - unused
```

Install a couple of other packages that we will use:
```bash
conda install -y -c bioconda gotree
conda install -y parallel
```

Deactivate the environment (optional):
```bash
conda deactivate
```

## Checkm2

Create and activate the environment:
```bash
conda create -y --name checkm2-1.1.0
conda activate checkm2-1.1.0
```

Install CheckM2:
```bash
conda install -y bioconda::checkm2
```

Install the CheckM2 database:
```bash
checkm2 database --download --path .
```

Check that it is working:
```bash
checkm2 testrun
```

Deactivate the environment (optional):
```bash
conda deactivate
```

## GTDB-tk

Create and activate the environment:
```bash
conda create -y --name gtdbtk-2.7.2
conda activate gtdbtk-2.7.2
```

Install CheckM2:
```bash
conda install -y -c conda-forge -c bioconda gtdbtk=2.7.2
```

Deactivate the environment (optional):
```bash
conda deactivate
```

## GeCoCheck

```bash
conda create --name gecocheck-1.0 python==3.13
conda activate gecocheck-1.0

pip install setuptools==66.1.1
conda install conda-forge::pandas conda-forge::matplotlib conda-forge::biopython bioconda::samtools bioconda::bowtie2==2.5.4 bioconda::minimap2

git clone https://github.com/R-Wright-1/GeCoCheck
cd GeCoCheck
pip install --editable .
```

Deactivate the environment (optional):
```bash
conda deactivate
```

## PICRUSt2

Download the package:
```bash
wget https://github.com/picrust/picrust2/archive/v2.6.3.tar.gz
tar xvzf  v2.6.3.tar.gz
cd picrust2-2.6.3/
```

I then modified the `picrust2-env.yaml` file so that the environment would be called `picrust2-v2.6.3` instead of `picrust2` - this is up to you, but you will just need to remember to change the environment name when activating this environment later if you don't do this.

Install:
```bash
conda env create -f picrust2-env.yaml
conda activate picrust2
pip install --editable .
```

Run the tests to verify the installation:
```bash
pytest
```

Deactivate the environment (optional):
```bash
conda deactivate
```

## R packages

These will need to be installed in RStudio instead of on the command line! If you have RStudio server then we recommend using this so that you don't need to copy any files across.

Install necessary R packages:
```bash
install.packages("vctrs")
install.packages("dplyr")
install.packages("tidyr")
install.packages("ggplot2")
install.packages("vegan")
library("devtools")
install_github("biobakery/maaslin3")
install.packages("GUniFrac")
install.packages("ape")
install.packages("data.table", type = "source")
```





## Authors

**Author:** Robyn Wright<br>
**Modifications by:** NA<br>
**Based on initial versions by:** NA

<img src="/assets/images/MicrobiomeHelperLogo.png" alt="Microbiome Helper logo" style="width: 50%; height: auto;">