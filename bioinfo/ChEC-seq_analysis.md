# ChEC-seq Analysis

Last updated July 2022

## Introduction

ChEC-seq, for Chromatin Endogenous Cleavage and sequencing, combines fusion of a target protein to micrococcal nuclease with high-throughput sequencing to determine where the target binds DNA or is in close proximity to the chromatin.  See the [original paper](https://pubmed.ncbi.nlm.nih.gov/26490019/) by Zentner *et al* and the [methods chapter](https://pubmed.ncbi.nlm.nih.gov/34382196/) for details.

Here, we give an overview of the basic ChEC-seq analysis workflow using common bioinformatic tools, which you can elaborate on for your own studies.  As the data are similar to those obtained from ChIP, these processing steps and analyses are also broadly applicable to ChIP-seq analysis as well.

This tutorial assumes you are working on one of [IU's HPC](https://kb.iu.edu/d/alde) clusters, but should work in general on any GNU/linux-like system which has the appropriate sofware installed.  I recommend you use RED (see below) to get started, as it will give you access to all of Carbonate's resources with the convenience of a desktop environment.

## Setup

You should already have an account and access to [carbonate](https://kb.iu.edu/d/aolp), [RED](https://kb.iu.edu/d/apum).  If you don't, see the [HPC Setup](../hpc_setup.md) guide for assistance.

Once you are logged into a cluster, you can load required [modules](https://kb.iu.edu/d/bcwy) using the `module load [your_module]` command, load them on the fly in your processing/analysis script, or place them in a `~/.modules` file which will load each time you log in or open a new shell.

## Initial Processing Overview

The data must first be processed (and analyzed for quality) before you can perform any subsequent analyses.  The general steps and programs to use are:
  1. **Download** the raw data archive (`wget` or `curl`)
  2. **Unpack** the data (`tar` and/or `gzip`/`gunzip`)
  3. Run **quality control** on the reads (`fastqc` and `multiqc`)
  4. **Align** the reads to the appropriate genome (`bowtie2`)

#### Looking at the steps in more detail:

You should have access to your raw sequencing data, either from a public repository or the sequencing center.   Talk to Jason, Jay, or another experienced lab member if you need access credentials.  Once you have access, you can use a tool like `wget` or `curl` to **download** the data to the cluster.  An template download command would be:
  ```bash
  wget  --user [your_user_name] --password [your_password] https://your.sequence.source.com/path/to/the_archive.tar.gz
  ```
...which will download the archive to your currend working directory for further processing.  See `wget --help` for further command line options if you want to play with fancier behavior.

Once downloaded, you must **unpack** the archive.  If it is a "tarball", which is a (usually) gzipped tar archive file ending in `.tar.gz`, you can simply run:
  ```bash
    tar -xzvf the_archive.tar.gz
  ```
...to unpack its contents.  If you'd like to write the sequence files to a certain directory, you can add the `-C your/dir` flag (see `tar --help` for more command details).  Occasionally you will find archives whose contents were zipped before `tar`; in that case you would omit the `-z` flag to unpack the archive and then would subsequently run `gunzip *.fastq` on the sequence files.

With the sequences unpacked and decompressed, you can then perform **quality control**.  We like to just use [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) followed by [MultiQC](https://multiqc.info/) to amalgamate all of the FastQC output into one easily browsable page.  Make sure you have both programs loaded (`module load fastqc multiqc`) and run:
    ```bash
    fastqc [path/to/your/*.fastq] -t [number_of_threads] -o [path/to/your/qc-output/]
    multiqc [path/to/your/qc-output/]
    ```
The `*.fastq` wildcard will run `fastqc` on every file ending with `.fastq` in that directory. Examine the QC output to gauge how well the run went and if it's worthwhile to continue with the data.

If you are reasonably confident your sequences are sound, it's time to **align** them to the genome.  We use [Bowtie 2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml) to align DNA sequences to the genome, though there are many other aligners out there.  Before alignment, we need to have an *index* on hand or build one, which is a genome-derived data structure that `bowtie2` uses to quickly find and score alignments.  To build an index, we need a genome, available fromt the usual sources like [Ensembl](https://ensemblgenomes.org/).  For example, to fetch the fly genome ([Drosophila melanogaster](https://metazoa.ensembl.org/Drosophila_melanogaster/Info/Index)), we could run the command:
  ```bash
  wget http://ftp.ensemblgenomes.org/pub/metazoa/release-53/fasta/drosophila_melanogaster/dna/Drosophila_melanogaster.BDGP6.32.dna_sm.toplevel.fa.gz
  ```
Then unzip it, and put it somewhere convenient.  To build the index, run:
  ```bash
  bowtie2-build --threads [number_of_threads] [path/to/your/genome_assembly.fa] path/to/output/bt2_index
  
  ```
With the index built, you can then run the alignment command:
  ```bash
  bowtie2 --threads "$threads" --no-unal --dovetail --no-mixed \
    --no-discordant --minins 40 --maxins 750 -x [path/to/your/bt2_index] \
    -1 [path/to/R1.fq] -2 [path/to/R2.fq] -S [path/to/output/sample_alignment.sam]
  ```
...runs `bowtie2` with several paired-end-specific options on your read files and index, outputting the result as a [SAM](https://github.com/samtools/hts-specs/blob/master/SAMv1.pdf) file.  There are many more `bowtie2` options available--see the manual linked above or `bowtie2 --help`.  If you have many samples to align, it's probably easieast to write a shell loop (`for R1_path in path/to/your/fastq-dir/*R1_001.fasq; do bowtie2 ... ; done`) -- see the sample script below for an example.

### Further Processing, Initial Analysis

Once you have your samples aligned, you probably want to get a look at the data tracks before proceeding with any quantification.  This can be a good way to compare your samples by eye and gauge how well they worked.  There are several ways to further process the data for track viewing, among them applying some of the [HOMER](http://homer.ucsd.edu/homer/index.html) suite tools, which take your files across a progression of conversions (including normalization) and formats which are useful for subsequent analysis:
  1. `makeTagDirectory` converts the SAM output to dirs and files countaining information about where your reads (tags) pile up.
  2. `makeUCSCfile` normalizes the tag dir data, typically to counts per million (CPM), and writes a `bedgraph` file indicating the number of reads overlapping at each position in the genome.  These `.bedgraph.gz` files must then be unzipped (`gunzip`).
  3. `bedGraphToBigWig` simply converts the large `.bedgraph` file to a compressed `bigWig` format which is track viewer friendly.
  4. `IGV` can then be run on your desktop to view the tracks.

You may find the various bioinformatic file formats confusing.  Indeed, this is a common issue with the ad-hoc nature of bioinformatic software output--only SAM really has some sort of agreed-upon [specification](https://github.com/samtools/hts-specs/blob/master/SAMv1.pdf).  UCSC has some pages which may help you understand what the formats are supposed to be and their contents:
  - https://www.genome.ucsc.edu/ENCODE/fileFormats.html
  - https://genome.ucsc.edu/goldenPath/help/bedgraph.html
  - https://genome.ucsc.edu/goldenPath/help/wiggle.html
  - https://genome.ucsc.edu/goldenPath/help/bedgraph.html

#### In more detail

First, you need to [install HOMER](https://genome.ucsc.edu/goldenPath/help/bedgraph.html).  It is not currently available as a module on IU HPC systems, and it has a number of other useful utilities and analyses programs even if there are other, newer options out there (see below) and you decide not to use it for track processing.

To make the **tag directories**,  run:
  ```bash
  makeTagDirectory [/path/to/output/sample-tag-dir/] [path/to/your/alignment_file.sam] 
  ```
See the [tagDir documentation](http://homer.ucsd.edu/homer/ngs/tagDir.html) for more details (HOMER includes some handy QC output and filtering options here).

For the **bedgraphs**, run:
  ```bash
  makeUCSCfile [path/to/your/sample-tag-dir/] -norm [normalization_factor] -o [path/to/your/output/sample.bedgraph]
  gunzip [path/to/your/bedgraph-dir/*.bedgraph.gz]
  ```
We usually use 1e6 (i.e. CPM) as the normalization factor. See the [makeUCSC documentation](http://homer.ucsd.edu/homer/ngs/ucsc.html) for details.  The `gunzip` call is necessary because homer like to output these files compressed, and it's easiest for the user to just inflate them before use.

To run the final **bigWig** step, we must first make sure we have a utility from UCSC installed.  Download the [appropriate version](https://hgdownload.cse.ucsc.edu/admin/exe/)of `bedGraphToBigWig` and make sure it's in your PATH; it's probably easiest just to put it in your `homer/bin/` dir.  The command below uses HOMER's perl wrapper; if you use the UCSC utility directly, it also needs a "`chrom.sizes`" file to indicate how big your chromosomes are.  The commands for linux and Drosophila would be:
   ```bash
   wget -P [path/to/homer/bin/] https://hgdownload.cse.ucsc.edu/admin/exe/linux.x86_64.v385/bedGraphToBigWig
   chmod +x [path/to/homer/bin/bedGraphToBigWig] # makes the program executable if it isn't already
   makeBigWig.pl [path/to/your/sample-tag-dir/] [your_genome] [/path/to/your/sample.bedgraph]
   # or
   bedGraphToBigWig [path/to/your/sample-tag-dir/] [path/to/appropriate/chrom.sizes] [path/to/output/sample_bigWig.bw]
   ```
See further down the HOMER `makeUCSCfile` documentation linked above for more details. You can find the appropriate `chrom.sizes`files at UCSC, `http://hgdownload.soe.ucsc.edu/goldenPath/<db>/bigZips/<db>.chrom.sizes`, if your assembly `<db>` is hosted. 

**Note** that many of these HOMER utilities are only single-threaded, so can take some time with large (e.g. human) input files, and you will have to loop over your samples at each step (see the example script further below).

Now we are finally ready to **view** the normalized tracks.  Our favorite desktop track browser is [IGV](https://software.broadinstitute.org/software/igv/); see the instructions on how to [download and install](https://software.broadinstitute.org/software/igv/download) it for your system.  Once you have it installed and open, you should first load a genome and annotation before importing your tracks (make sure they are evenly scaled!).  Then, you can start "peak gazing" around your favoriate genes, as Gabe Zentner and Steve Henikoff like to call it.  The annotations are extremely useful to visualize where your genes are as the bottom track, and also make them searchable, so make sure to include those.  See the [IGV documentation](https://software.broadinstitute.org/software/igv/UserGuide) for further details on how to load or build genomes + annotations, and usage of the software in general.

#### Alternative Processing Workflow with DeepTools

TO BE ADDED

### Example Processing Script

Here's a basic tar-to-bigWig processing script template that will bring you all the way from fetching the raw data to bigWig files ready for viewing and further analyses; it is heavily commented for the beginner's understanding:

```bash
#!/bin/bash
# simple script to process a ChEC seq project for visualization and further analysis

set -euo pipefail 

```` 

## Further Analysis and Quantification

We're only getting started once the data are processed and tracks examined.  You're going to want to further analyze, quantify, plot, and visualize the data for sharing, presentation, and publication.  What exactly you need to do will depend on your project and is beyond the purview of this document; there are many tools out there with their own use cases and documentation you will need to explore.

TO BE EXPOUNDED ON MORE:

To get you started, here are a few suggesstions:
  - HOMER findPeaks.pl and annotatePeaks.pl, motif finding
  - MACS2 to call peaks
  - DeepTools will generate heatmaps for you directly