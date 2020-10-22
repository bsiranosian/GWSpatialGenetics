# Pipeline for processing next-generation sequencing (fastq) files from raw sequencing.

The Institute of Disease Modeling has been part of an interdisciplinary  collaboration with Elizabeth Thiele (Vassar) and James Cotton (Wellcome Sanger Institute) to maximize the value of epidemiological and genetics data to understand Guinea worm transmission in Chad. Preliminary analyses by IDM has shown whole mitochondrial genome data can give higher resolution information about genetic relatedness in a population than the current three-locus method. 
This pipeline is intented to process NGS sequencing from whole mitochondrial DNA to variants, with additional feautres to count primers and coverage. 


# Download publicly available genomes:
I downloaded the data corresponding to https://www.biorxiv.org/content/10.1101/808923v1. The data are available from the European Nucleotide Archive project ERP117282. This analysis focused solely on the samples collected in Chad as specified in Supplementary Table 1.

I attempted to follow the pipeline from the preprint as best as possible. The GEM masking program was difficult to get running and mitochondrial mask regions were defined manually by the authors, thus some undesirable regions may be included in the SNPs. Instead of including a known variants file from masking, I followed GATK4 guidelines to call variants, filter HQ (QD > 30), and use those in the base quality recalibration step until convergence.

# Set-up
Pipelines are wrapped into Conda virtual environments and organized by Snakemake. These must be run on a Linux based system.

Step 1: Download and set up Miniconda (https://docs.conda.io/en/latest/miniconda.html)
Step 2: Run install to create a virtual environment with the necessary programs.
```
conda env create -f ngs_align.yml
```

# Step 3: Remove adapters
Sequencing data from repositories have the adapters used to map reads to a sample. Leaving adapters in a read can diminish to ability to align to a reference since it increases the number of mismatches. To create adapter free sequencing files, run the following command:
```
snakemake -s path/to/git/clone/wg_processing.snakefile --configfile path/to/project/yaml/project.yaml --until post_multiqc --jobs 20
```

# Step 4: Call and filter variants

Calling variants without a high quality set of SNPs to recalibrate sequencing errors is an iterative process. To avoid errors from cyclic dependencies in rules, first run an initial round (iteration=0). Then rerun the command with a number of iterations to run (recommended 2-3).

```
snakemake -s gw_processing.snakefile --configfile configGW_mtAll.yaml --rerun-incomplete --jobs 100 --latency-wait 30  --config iteration='0'
snakemake -s gw_processing.snakefile --configfile configGW_mtAll.yaml --rerun-incomplete --jobs 100 --latency-wait 30  --config iteration='3'
```

The BSQR comparisons to the initial round of variant calling will guide if 3 iterations are sufficient to normalize read errors.

That's it! At the end you will have VCF files with genotypes at positions. 