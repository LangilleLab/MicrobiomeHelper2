---
layout: default
title: "CBW 2026 AWS setup"
show_sidetoc: true
header_type: base
permalink: /docs/tutorials/cbw-2026-aws-setup/
---

This page just contains the commands used for installing programs on the AWS instances used for the 2026 CBW Microbiome Analysis workshop (held in Guelph, ON, September 15-17). 

**Author**: Robyn Wright

# Quality control

```
conda create --name quality_control
conda activate quality_control
conda install bioconda::fastqc
conda install bioconda::multiqc
```

# QIIME2

```
conda env create \
  --name rachis-qiime2-2026.7 \
  --file https://raw.githubusercontent.com/qiime2/distributions/refs/heads/dev/2026.7/qiime2/released/rachis-qiime2-linux-64-conda.yml
```

# Kneaddata

```
conda create --name kneaddata-0.12.4
conda activate kneaddata-0.12.4
conda install bioconda::kneaddata
conda install parallel

kneaddata_database --download human_genome bowtie2 human_bt2db
```

# Kraken

```
conda create --name kraken-2.17.1
conda activate kraken-2.17.1
cd CourseData
mkdir tools
cd tools/

git clone https://github.com/DerrickWood/kraken2
cd kraken2
./install_kraken2.sh .

cp kraken2{,-build,-inspect} /home/ubuntu/CourseData/.conda/envs/kraken-2.17.1/bin/
conda install bioconda::bracken
```

```
cd workspace/metagenome
wget https://genome-idx.s3.amazonaws.com/kraken/k2_pluspf_08_GB_20260626.tar.gz
mkdir k2_pluspf_08_GB_20260626
tar -xvf k2_pluspf_08_GB_20260626.tar.gz -C k2_pluspf_08_GB_20260626
rm k2_pluspf_08_GB_20260626.tar.gz
```

# MetaPhlAn

```
conda create --name metaphlan-4.2.6
conda activate metaphlan-4.2.6
conda install bioconda::metaphlan
conda install parallel
```

# Anvi'o

```
conda create -y --name anvio-9 python=3.10
conda activate anvio-9

conda install -y -c conda-forge -c bioconda python=3.10 \
        sqlite=3.46 prodigal idba mcl muscle=3.8.1551 famsa hmmer diamond \
        blast megahit spades bowtie2 bwa graphviz "samtools>=1.9" \
        trimal iqtree trnascan-se fasttree vmatch r-base r-tidyverse \
        r-optparse r-stringi r-magrittr bioconductor-qvalue meme ghostscript \
        nodejs=20.12.2 llvmlite numba
        
conda install -y -c bioconda fastani

curl -L https://github.com/merenlab/anvio/releases/download/v9/anvio-9.tar.gz \
        --output anvio-9.tar.gz
        
pip install anvio-9.tar.gz

anvi-setup-scg-taxonomy
anvi-setup-ncbi-cogs

#CONCOCT
cd CourseData/tools
mkdir -p ~/github/ && cd ~/github/
git clone https://github.com/merenlab/CONCOCT.git

cd CONCOCT
pip install cython
python setup.py build
python setup.py install

#other binners
conda install -c bioconda metabat2
conda install -c bioconda maxbin2
conda install -c bioconda das_tool
conda install -c bioconda binsanity

pip install "setuptools<82"
pip install nose
conda install "scikit-learn==1.1.0"

#conda install -c bioconda usearch
#conda uninstall usearch
conda install -c bioconda -c conda-forge diamond

anvi-self-test --suite mini --no-interactive

#pip install gffutils - unused

conda install -c bioconda gotree
conda install parallel
```

# Checkm2

```
conda rename -n checkm2 checkm2-1.1.0
conda activate checkm2-1.1.0
conda install bioconda::checkm2
```





