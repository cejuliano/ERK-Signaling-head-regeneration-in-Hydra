## RNA-seq data analysis for Juanico *et al.* (2025)
The custom scripts and required files contained in this repository are divided into two different sections. The scripts and files in the `cluster` folder were written to be executed in a remote cluster running Ubuntu and the slurm workload manager. These scripts are intended for the trimming and mapping of raw reads from RNA sequencing. The code and files in the `local` folder contains the scripts and files were written to be executed on macOS Sonoma (14.4.1) running R (4.3.3) on RStudio (2024.12.1+563). These scripts perform differential gene expression analysis and differential gene expression patterns analysis.

### Accessing raw RNA-seq files not available on the Github repository

Due to file size limitations on Github, this repository does not contain RNA-seq raw reads which can be found at [Iris: Accession Number].

Descriptions for the files contained in this repository can be found in the `fileDescriptions.txt` file.

#### Publicly available resources

[RSEM](https://github.com/deweylab/RSEM) - required to run the `cluster` folder.

[trimmomatic v 0.36](http://www.usadellab.org/cms/?page=trimmomatic) - required to run the `cluster` folder.

## Cluster Analysis

#### Setup

The first step is downloading the `cluster` folder from this repository. Raw RNA-seq reads from this study need to be downloaded to the `cluster` directory as gzipped fastq files. Raw data for this study can be found at NCBI with the accession [PRJNA1373231] .

The analysis requires the following software pacakges [bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.html), [rsem](https://deweylab.github.io/RSEM/), [samtools](http://www.htslib.org/), [trimmomatic v 0.36](http://www.usadellab.org/cms/?page=trimmomatic) and [jdk](https://www.oracle.com/java/technologies/downloads/) 

#### Indexing reference sequences

Mapping of the raw reads requires using either bowtie2 or rsem to index the transcriptomic reference. If your computing cluster uses slurm, this can be done by executing the following command:

```         
sbatch -p [partition name] resources/slurm_prepare_references.sh
```

On other linux systems, you can execute:

```         
cd resources
./prep_references.sh
```

#### RNA-seq data processing

The RNA read mapping pipeline first removes index sequences and low quality basecalls using trimmomatic, and then maps reads and quantifies transcript read counts using rsem. It will also generate fastqc reports from both the raw and trimommatic-filtered data.

If your computing cluster uses slurm, you can execute:

```         
sbatch -p [partition name] resources/slurm_RNA_processing.sh
```

For a system without slurm, you can run the pipeline on each individual sample using the `RNA_Mapping_Pipeline.sh` script like indicated below:

```         
./resources/RNA_processing.sh [insert prefix here]
```

After the pipeline has finished running for all samples, read count matrices need to be calculated for the data generated in this study. The command below cand be executed directly on your terminal session by running:

```         
cd resources
rsem-generate-data-matrix *RNA.genes.results > regeneration.mat.txt
```

## Local Analysis

#### Setup

The following section describes the `local` analysis, which was designed to be run on a personal computer running MacOS.

For the RNA-seq analyses, we have located `regeneration_mat.txt` file produced by the code in the `cluster` folder to the `local/resources`folder for its analysis.

#### Differential Gene Expression

We use edgeR to identify differentially expressed genes between compared time points using a negative binomial generalized linear model over the fold change values (glmTreat). These scripts output the results both as a RData object and as individual results tables in a csv format for *H. oligactis* and *H. vulgaris*.

```         
Rscript DGE_analysis_U126.Rmd
```


#### maSigPro Analysis

To determine differential gene expression patterns between foot regenerating tissue wit alsterpaullone treatment and foot regeneration controls with DMSO, we used maSigPro pipeline in R. Running this script requires `mat.ALP.OLI.csv` a file containing counts per million of alsterpaullone and DMSO treated samples and `design.ALP.OLI.csv` a file with details about expression time, replicates, and treatments. Both of this files have been provided in the `local/resources` folder. This analysis can be run in either Rstudio or by running `DGE_pattern_anlysis.Rmd` in terminal.

```         
Rscript DGE_pattern_analysis.Rmd
```

#### Gene Ontology enrichment analysis

We used TopGO to look for enrichment of Biological Processes annotated as Gene Ontology terms, within the clusters of genes we uncover with maSigPro analysis. The gene sets tested are included in the resources folder. This analysis can be run in either Rstudio or by running `GO_enrichment.Rmd` interminal.

```
Rscript GO_enrichment.Rmd
```
