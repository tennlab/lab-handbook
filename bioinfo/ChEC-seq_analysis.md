# ChEC-seq Analysis

Last updated July 2022

## Introduction

ChEC-seq, for Chromatin Endogenous Cleavage and sequencing, combines fusion of a target protein to micrococcal nuclease with high-throughput sequencing to determine where the target binds DNA or is in close proximity to the chromatin.  See the [original paper](https://pubmed.ncbi.nlm.nih.gov/26490019/) by Zentner *et al* and the [methods chapter](https://pubmed.ncbi.nlm.nih.gov/34382196/) for details.

Here, we give an overview of the basic ChEC-seq analysis workflow using common bioinformatic tools, which you can elaborate on for your own studies.  As the data are similar to those obtained from ChIP, these processing steps and analyses are also broadly applicable to ChIP-seq analysis as well.

This tutorial assumes you are working on one of [IU's HPC](https://kb.iu.edu/d/alde) clusters, but should work in general on any GNU/linux-like system which has the appropriate sofware installed.  I recommend you use RED to get started, as it will give you access to all of Carbonate's resources with the convenience of a desktop environment.

## Setup

You should already have an account and access to [carbonate](https://kb.iu.edu/d/aolp), [RED](https://kb.iu.edu/d/apum).  If you don't, see the [HPC Setup](../hpc_setup.md) guide for assistance.

Once you are logged into a cluster, you can load required [modules](https://kb.iu.edu/d/bcwy) using the `module load [your_module]` command, load them on the fly in your processing/analysis script, or place them in a `~/.modules` files which will load each time you log in or open a new shell.

## Processing Overview

The data must first be processed (and analyzed for quality) before you can perform any subsequent analyses.  The general steps and programs to use are:
  1. Download the raw data archive (`wget` or `curl`)
  2. Unpack the data (`tar` and/or `gzip`/`gunzip`)
  3. Run quality control on the reads (`fastqc` and `multiqc`)
  4. Align the reads to the appropriate genome (`bowtie2`)

We now look at the steps in more detail below.

You should have access to your raw sequencing data, either from a public repository or the sequencing center.   Talk to Jason, Jay, or another experienced lab member if you need access credentials.  Once you have access, you can use a tool like `wget` or `curl` to download the data to the cluster.  An template download command would be:
  ```bash
  wget  --user [your_user_name] --password [your_password] https://your.sequence.source.com/path/to/the_archive.tar.gz
  ```
...which will download the archive to your currend working directory for further processing.  See `wget --help` for further command line options if you want to play with fancier behavior.

Once downloaded, you must unpack the archive.  If it is a "tarball", which is a (usually) gzipped tar archive file ending in `.tar.gz`, you can simply run:
    ```bash
    tar -xzvf the_archive.tar.gz
    ```
...to unpack its contents.  If you'd like to write the sequence files to a certain directory, you can add the `-C your/dir` flag (see `tar --help` for more command details).  Occasionally you will find archives whose contents were zipped before `tar`; in that case you would omit the `-z` flag to unpack the archive and then would run `gunzip *.fastq` on the sequence files.

With the sequences unpacked and decompressed, you can then perform quality control.  We like to just use [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) followed by [MultiQC](https://multiqc.info/) to amalgamate all of the FastQC output into one easily browsable page.  Make sure you have both programs loaded (`module load fastqc multiqc`) and run:
    ```bash
    fastqc path/to/your/*.fastq -t [number_of_threads] -o [path/to/your/qc-output/]
    multiqc path/to/your/qc-output/
    ```
The `*.fastq` wildcard will run `fastqc` on every file ending with `.fastq` in that directory. Examine the QC output to gauge how well the run went and if it's worthwhile to continue with the data.

If you are reasonably confident your sequences are sound, it's time to align them to the genome.  We use [(Bowtie 2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml) to align DNA sequences to the genome, though there are many other aligners out there.  Before alignment, we need to have an index on hand or build one, which is a genome-derived data structure that `bowtie2` uses to quickly find and score alignments.  To build an index, we need a genome, available fromt the usual sources like [Ensembl](https://ensemblgenomes.org/).  For example, to fetch the fly genome ([Drosophila melanogaster](https://metazoa.ensembl.org/Drosophila_melanogaster/Info/Index)), we could run the command `wget http://ftp.ensemblgenomes.org/pub/metazoa/release-53/fasta/drosophila_melanogaster/dna/Drosophila_melanogaster.BDGP6.32.dna_sm.toplevel.fa.gz`.