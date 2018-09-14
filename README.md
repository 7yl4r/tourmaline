<img src="https://upload.wikimedia.org/wikipedia/commons/0/00/Tourmaline-121240.jpg" height=200> <img src="http://melissabessmonroe.com/wp-content/uploads/2014/03/20140303_TourmalineSurfPark128.jpg" height=200>

<!--[![Build Status](https://travis-ci.org/cuttlefishh/tourmaline.svg?branch=master)](https://travis-ci.org/cuttlefishh/tourmaline)

See https://docs.travis-ci.com/user/getting-started/ and https://github.com/biocore/oecophylla/blob/master/.travis.yml for setting up Travis.-->

# tourmaline

Amplicon sequencing is a metagenetics method whereby a single DNA locus in a community of organisms is PCR-amplified and sequenced. Tourmaline is an amplicon sequence processing workflow for Illumina sequence data that uses [QIIME 2](https://qiime2.org) and the software packages it wraps. Tourmaline manages commands, inputs, and outputs using the [Snakemake](https://snakemake.readthedocs.io/en/stable/) workflow management system.

Two methods of amplicon sequence processing are supported, both of which generate ASVs (amplicon sequence variants, which approximate the "true" or "exact" sequences in a sample) rather than OTUs (operational taxonomic units, which blur sequencing errors and microdiversity through clustering):

* [Deblur](https://github.com/biocore/deblur) is a greedy deconvolution algorithm based on known Illumina read error profiles ([Amir et al., 2017](https://doi.org/10.1128/mSystems.00191-16)).
* [DADA2](https://github.com/benjjneb/dada2) implements a quality-aware model of Illumina amplicon errors to infer sample composition by dividing amplicon reads into partitions consistent with the error model ([Callahan et al., 2016](https://doi.org/10.1038/nmeth.3869)).

Tourmaline is an alternative amplicon pipeline to [Banzai](https://github.com/jimmyodonnell/banzai), which was developed for [MBON](https://github.com/marinebon/MBON) and uses [Swarm](https://github.com/torognes/swarm) for OTU picking.

## Installation

Tourmaline has the following dependencies (installation instructions below):

* QIIME 2 (version `2018.6` or later)
* Snakemake
* Perl and `fastaLengths.pl` (included)

First, if you haven't already, install QIIME 2 using the instructions at [qiime2.org](https://docs.qiime2.org/2018.6/install/native/). For example, on macOS these commands will install QIIME 2 inside a Conda environment called `qiime2-2018.6`:

```
wget https://data.qiime2.org/distro/core/qiime2-2018.6-py35-osx-conda.yml
conda env create -n qiime2-2018.6 --file qiime2-2018.6-py35-osx-conda.yml
```

Second, activate your QIIME 2 environment and install Snakemake:

```
source activate qiime2-2018.6
conda install snakemake
```

Third, clone the Tourmaline repository and rename it to the working directory for your project (replace the directories in ALL CAPS with your directories):

```
cd /PATH/TO
git clone https://github.com/cuttlefishh/tourmaline.git
mv tourmaline PROJECT
```

Now you are ready to start analyzing your amplicon sequence data.

## Execution

This section describes how to prepare your data and then run the workflow.

### Prepare data

Tourmaline steps covered in this section (logic described below):

* Step 0: Assess and format data

Note: Currently, Tourmaline does not do demultiplexing and quality filtering (support for this may be added later). You should demultiplex and quality filter your fastq data yourself, then gzip the resulting per-sample fastq files.

#### Format metadata

Your metadata should be a tab-delimited text file (e.g., exported from Excel) with samples as rows and metadata categories as columns. In addition to basic sample information like collection date, latitude, and longitude, your metadata should include categories that describe treatment groups and environmental metadata relevant to your samples. See [Metadata in QIIME 2](https://docs.qiime2.org/2018.6/tutorials/metadata/), the [EMP Metadata Guide](http://www.earthmicrobiome.org/protocols-and-standards/metadata-guide/), and [QIIMP](https://qiita.ucsd.edu/iframe/?iframe=qiimp) for help formatting your metadata.

#### Format sequence data

Currently, Tourmaline supports amplicon sequence data that is already demultiplexed. Using the sample names in your mapping file and the paths to the forward and reverse demultiplexed sequence files (`.fastq.gz`) for each sample, create a fastq manifest file. See [Fastq Manifest Formats](https://docs.qiime2.org/2018.6/tutorials/importing/#fastq-manifest-formats) (QIIME 2) for instructions for creating this file. While `qiime tools import` supports both `.fastq` and `.fastq.gz` formats, using `.fastq.gz` format is strongly recommended because it is ~5x faster and minimizes disk usage. (Hint: Gzipped files can still be viewed using `zcat` with `less` or `head`.)

#### Set up data directory

Create a directory for data inside your working directory:

```
cd /PATH/TO/PROJECT
mkdir 00-data
```

Move your metadata file and fastq manifest file(s) to `/PATH/TO/PROJECT/00-data`.

#### Edit the configfile

The configuration file or `configfile` is `config.yaml`. It must be edited to contain the paths to your data and the parameters you want to use. `Snakefile` and `config.yaml` should describe all the inputs, parameters, and commands needed to produce the desired output.

### Run Snakemake

Tourmaline steps covered in this section (logic described below):

* Step 1: Import data
* Step 2: Denoising
* Step 3: Representative sequence curation
* Step 4: Core diversity analyses
* Step 5: Quality control report

#### Rules

Snakemake works by executing rules, defined in the `Snakefile`. Rules specify commands and outputs but most critically inputs, which dictate which other rules must be run beforehand to generate those inputs. By defining pseudo-rules at the beginning of the `Snakefile`, we can specify desired endpoints as "inputs" that force execution of the whole workflow or just part of it. When a snakemake command is run, only those rules that need to be executed to produce the requested inputs will be run.

Tourmaline provides Snakemake rules for Deblur (single-end) and DADA2 (single-end and paired-end). For each type of processing, the `denoise` rule runs steps 1 and 2 (import data and denoising), the `diversity` rule runs steps 3 through 5 (representative sequence curation, core diversity analyses, and QC report), and the `stats` rule runs group significance and other tests. Pausing after step 2 allows you to filter your biom table and representative sequences before proceeding. For example, if your amplicon is 16S rRNA, you may want to filter out chloroplast/mitochondria sequences.

##### Deblur (single-end)

```
# steps 1-2
snakemake deblur_se_denoise

# steps 3-5
snakemake deblur_se_diversity

# statistical analyses
snakemake deblur_se_stats
```

##### DADA2 (single-end)

```
# steps 1-2
snakemake dada2_se_denoise

# steps 3-5
snakemake dada2_se_diversity

# statistical analyses
snakemake dada2_se_stats
```

##### DADA2 (paired-end)

```
# steps 1-2
snakemake dada2_pe_denoise

# steps 3-5
snakemake dada2_pe_diversity

# statistical analyses
snakemake dada2_pe_stats
```

That's it. Just run one of these commands and let Snakemake do its magic. The results will be placed in organized directories inside your working directory:

```
01-imported
02-denoised
03-repseqs
04-diversity
05-reports
```

## Logic

In plain English, this is the logic behind Tourmaline. Starting with Step 1, these steps correspond to the rules (commands) in the `Snakefile`.

### Step 0: Assess and format data

Answer the following questions to determine the best parameters for processing and to be able to evaluate the success of your completed workflow.

#### Assess amplicon locus

* What is the locus being amplified, and what are the primer sequences?
* How much sequence variation is expected for this locus (and primer sites) and dataset?
* Is the expected sequence variation enough to answer my question?
* What is the expected amplicon size for this locus and dataset?

#### Assess sequence data

* What type and length of sequencing was used? (e.g., MiSeq 2x150bp)
* Do I have long enough sequening to do paired-end analysis, or do I have to do single-end analysis only?
* What sequence pre-processing has been done already: Demultiplexing? Quality filtering and FastQC? Primer removal? Merging of paired reads?

#### Assess sample set and metadata

* Is my metadata file complete? Are the relevant parameters of my dataset present as numeric or categorical variables?
* Do I have enough samples in each group of key metadata categories to determine an effect?

#### Format metadata and sequence data

* Is my metadata file properly formatted? See [Metadata in QIIME 2](https://docs.qiime2.org/2018.6/tutorials/metadata/), the [EMP Metadata Guide](http://www.earthmicrobiome.org/protocols-and-standards/metadata-guide/), and [QIIMP](https://qiita.ucsd.edu/iframe/?iframe=qiimp) for help formatting your metadata.
* Is my sequence data demultiplexed, in `.fastq.gz` format, and described in a QIIME 2 fastq manifest file? See [Fastq Manifest Formats](https://docs.qiime2.org/2018.6/tutorials/importing/#fastq-manifest-formats) from QIIME 2 for instructions for creating this file.
* Are my reference sequences and taxonomy properly formatted for QIIME 2?
* Is my config file updated with the file paths and parameters I want to use?

### Step 1: Import data

Import sequence data and metadata for QIIME 2 processing. It is assumed that any preprocessing and formatting of data have already been done.

* Amplicon sequence data: import into QIIME 2 artifact.
* Reference sequence data: import into QIIME 2 artifact.
* Metadata: import into QIIME 2 artifact.

### Step 2: Denoising

Run Deblur or DADA2 to generate ASV feature tables and representative sequences. This is akin to OTU picking.

* Denoise amplicon data using Deblur or DADA2 to generate ASV feature tables (BIOM).
* Use paired-end mode (DADA2 only) if sequence and amplicon lengths permit.

Perform quality control and add to QC summary (Step 6) before proceeding.

* Summarize tables and representative sequences (ASVs).
* Use table summary to determine appropriate rarefaction depth.
* Use representative sequences summary to determine length distribution of sequences.

### Step 3: Representative sequence curation

Generate a phylogenetic tree of ASV sequences, and identify the taxonomy (phylum, class, order, etc.) of each ASV.

* Build a phylogenetic tree of ASV sequences, or insert ASV sequences into an existing tree for your amplicon locus.
* Assign taxonomy to ASVs using a reference database for your amplicon locus.

### Step 4: Core diversity analyses

First consult table summary and run alpha rarefaction to decide on a rarefaction depth. Then do the major alpha/beta diversity analyses and taxonomy summary.

* Alpha diversity: alpha rarefaction, diversity metrics (evenness, Shannon, Faith's PD, observed sequences), alpha group significance.
* Beta diversity: distance matrices (un/weighted UniFrac, Bray-Curtis, Jaccard), principal coordinates, Emperor plots, beta group significance.
* Taxonomy barplots.

### Step 5: Quality control report

After completing processing and core analyses, determine if the results make sense.

* How many sequences did I start with, and how many are left after denoising?
* Are the representative sequences of similar length or of very different lengths?
* Do the sequence alignment and tree look reasonable?
* Do samples cluster in an expected way in PCoA space?
* Do the taxonomic profiles match expected taxonomic compositions?
