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

First, install QIIME 2 using the instructions at [qiime2.org](https://docs.qiime2.org/2018.6/install/native/), if you haven't already. For example, on macOS these commands will install QIIME 2 inside a Conda environment called `qiime2-2018.6`:

```
wget https://data.qiime2.org/distro/core/qiime2-2018.6-py35-osx-conda.yml
conda env create -n qiime2-2018.6 --file qiime2-2018.6-py35-osx-conda.yml
```

Second, activate your QIIME 2 environment and install Snakemake:

```
source activate qiime2-2018.6
conda install snakemake
```

Third, clone the Tourmaline repository to the working directory for your project.

```
cd /path/to/project
git clone 
```

## Getting started

## Logic

In plain English, this is the logic behind Tourmaline. Starting with Step 1, these steps correspond to the rules (commands) in `Snakefile`.

### Step 0. Data assessment

Answer the following questions to determine the best parameters for processing and to be able to evaluate the success of your completed workflow.

**Amplicon locus**

* What is the locus being amplified, and what are the primer sequences?
* How much sequence variation is expected for this locus (and primer sites) and dataset?
* Is the expected sequence variation enough to answer my question?
* What is the expected amplicon size for this locus and dataset?

**Sequence data**

* What type and length of sequencing was used? (e.g., MiSeq 2x150bp)
* Do I have long enough sequening to do paired-end analysis, or do I have to do single-end analysis only?
* What sequence pre-processing has been done already: Demultiplexing? Quality filtering and FastQC? Primer removal? Merging of paired reads?

**Sample set and metadata**

* Is my metadata file properly formatted? See the [QIIME 2 documentation](https://docs.qiime2.org/2018.6/tutorials/metadata/) and the metadata tool [QIIMP](https://qiita.ucsd.edu/iframe/?iframe=qiimp).
* Is my metadata file complete? Are the relevant parameters of my dataset present as numeric or categorical variables?
* Do I have enough samples in each group of key metadata categories to determine an effect?

### Step 1. Data preparation

Preprocess and format sequence data and metadata for QIIME 2 processing.

* Amplicon sequence data: preprocess, quality filter, and import into QIIME 2 artifact.
* Reference sequence data: format and import into QIIME 2 artifact.
* Metadata: format and import into QIIME 2 artifact.

### Step 2. Denoising

Run Deblur or DADA2 to generate ASV feature tables and representative sequences. This is akin to OTU picking.

* Denoise amplicon data using Deblur or DADA2 to generate ASV feature tables (BIOM).
* Use paired-end mode (DADA2 only) if sequence and amplicon lengths permit.

Perform quality control and add to QC summary (Step 6) before proceeding.

* Summarize tables and representative sequences (ASVs).
* Use table summary to determine appropriate rarefaction depth.
* Use representative sequences summary to determine length distribution of sequences.

### Step 3. Representative sequence curation

Generate a phylogenetic tree of ASV sequences, and identify the taxonomy (phylum, class, order, etc.) of each ASV.

* Build a phylogenetic tree of ASV sequences, or insert ASV sequences into an existing tree for your amplicon locus.
* Assign taxonomy to ASVs using a reference database for your amplicon locus.

### Step 4. Core diversity analyses

First consult table summary and run alpha rarefaction to decide on a rarefaction depth. Then do the major alpha/beta diversity analyses and taxonomy summary.

* Alpha diversity: alpha rarefaction, diversity metrics (evenness, Shannon, Faith's PD, observed sequences), alpha group significance.
* Beta diversity: distance matrices (un/weighted UniFrac, Bray-Curtis, Jaccard), principal coordinates, Emperor plots, beta group significance.
* Taxonomy barplots.

### Step 5. Quality control

After completing processing and core analyses, determine if the results make sense.

* How many sequences did I start with, and how many are left after denoising?
* Are the representative sequences of similar length or of very different lengths?
* Do the sequence alignment and tree look reasonable?
* Do samples cluster in an expected way in PCoA space?
* Do the taxonomic profiles match expected taxonomic compositions?
