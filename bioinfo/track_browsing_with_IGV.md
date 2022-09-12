# Track Browsing with Integrative Genomics Viewer (IGV)

A number of tools are out there, both online and local, to help you visualize your high-throughput sequence reads as *tracks*.  Here I describe installation and basic use of one such tool:  IGV from the [Broad Institute](https://software.broadinstitute.org/software/igv/).


### Rationale

*Tracks* represent overlap and accumulation of your aligned, (usually) normalized reads across any given genomic locus.

In addition to any necessary systematic analysis, it's a good idea to engage in some "peak-gazing" of your data, as Drs. Henikoff and Zentner like to call it.  This exploratory data analysis can help you gauge how well your experiment and sequencing worked, and what might be going on at some of your favorite genes of interest. (NOTE - don't draw too many conclusions without also completed the systematic analyses)

## Installation

IGV is available on the [web](https://igv.org/app) (and as CLI) but I prefer to run the local graphic version.  To install IGV on MacOS (or any other supported system):
1.  Go to the [IGV Downloads page](https://software.broadinstitute.org/software/igv/download) and download the appropriate version of the software (just get the "Java-included" release for convenience).
2.  On MacOS, extract the download and put the IGV application folder in your Applications directory.  On PC, run the installer `.exe` and follow the instructions.
3.  Open IGV from the Launchpad (Mac) or wherever else you installed it.

To use IGV, you need to have a genome (+ annotation) loaded and, of course, data.  When you first open the program, it will have the human hg19 as default, but you can add whatever genomes you want.  Convenently, many of the common genomes are already hosted by IGV/Broad and are available for download.  (You can also manually build one from a reference assembly & annotation).

Since we are interested in flies:
- 4.  Select `Genomes -> Select From Hosted Genome...` from the menu bar, OR click on the current genome in the top left (`Human (GRCh37/hg19)`) to bring up a drop-down menu of availble genomes and select `More...`
- 5.  Search for or select the latest version of the `D. melanogaster (dm6)` genome (I also click the box to download sequence, since the fly assembly isn't too huge). Click `OK`.

That's it!

## Basic Usage

Now that you have the fly genome loaded, you'll notice a track of RefSeq genes has appeared at the bottom; this tells you where annotated genes/exons/introns are and in what orientation.  You can see how some regions are more gene-dense than others.

At the top below the control bar, you see all your chromosomes laid out side by side.  You can limit which reference sequence/chromosome in view using the drop down menu next to the currently loaded genome.

In the search box next to the drop-down menus, you can input coordinates or genes of interest to jump to any given locus.  You'll probably use this feature a lot.

In the chromosome bar, you can click and drag across any region to zoom in on it.  This can be done multiple times at different zoom levels.  Once zoomed, you can drag the view window (red box) around the chromosome span as you see fit, or zoom in and out some more.  You'll see as you zoom in, the annotated gene track also scales to show you what's there.

### Loading Data

IGV will accept multiple different forms of data; I like to feed it `bigWig` files from my data processing workflow.  See the ChEC-seq analysis page in the notebook for details on how to get there, and the [IGV manual](https://software.broadinstitute.org/software/igv/UserGuide) for details.

To load a file(s), simply go to `File -> Load from File...` and select those you wish to view.

Once the files are loaded, you will see the tracks.  A variety of plotting and aesthetic options are available to help visualize and organize the data.  Just right-click on the tracks to see them and make changes.  **Probably the most important option** you should set is `Autoscale / Group Autoscale.`  If you have several samples, timepoints, replicates, etc. that you would like to directly compare, make sure you have selected all the pertinent tracks simulteously (with command-click or Ctrl-click), right-cleck, and make sure `Group Autoscale` is enabled.  This will ensure the heights of the grouped tracks are comparable.  Also pay attention to the Data Range in the top-left corner of each track band, and make sure `Show Data Range` is enabled:  just because tracks have a similar height on the plot, doesn't mean they have the same signal intensity, particularly for tracks in different autoscale groups.

### Exporting Images

You can take pictures of tracks in the current view by right-clicking or going to `File -> Save PNG / SVG Image` and choosing a file name and location.  Note that the PNG images are not particularly high resolution or aesthetic, and so best-suited for informal sharing.  If you need to create publication-quality plots for figures, check out other tools like [`rtracklayer`](https://bioconductor.org/packages/release/bioc/html/rtracklayer.html)





