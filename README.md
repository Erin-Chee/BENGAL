## BENGAL: BENchmarking inteGration strAtegies for cross-species anaLysis of single-cell transcriptomics data ##


Author: Yuyao Song <ysong@ebi.ac.uk>

A Nextflow DSL2 pipeline to perform cross-species single-cell RNA-seq data integration and assessment of integration results.

[check our preprint](https://www.biorxiv.org/content/10.1101/2022.09.27.509674v1).

## This repo includes:

1) the Nextflow program
2) conda environment configurations and containers
3) an example config file for running the Nextflow pipeline

## System requirements
#### hardware:
This workflow is written to be executed on HPC clusters with LSF job scheduler. It could be easily adapted to other shcedulers via syntax modifications. If the GPU inplementation of scVI is to be used (massively speed up the integration), GPU computing nodes are required and pleasr refer to [scVI](https://scvi-tools.org/). 

#### OS:
Development of this workflow was done on Rocky Linux 8.5 (RHEL), while in theory this can be run on any linux distribution.

## Installation:

#### Pull the code for BENGAL
`git clone git@github.com:Functional-Genomics/CrossSpeciesIntegration.git`
#### If nextflow or singularity is not installed in your cluster, install them, [go to nextflow documentation](https://www.nextflow.io/docs/latest/getstarted.html), [go to singularity documentation](https://singularity-tutorial.github.io/01-installation/)

## Inputs:
The nextflow script takes one input file: a tab-seperated metadata file mapping species to the paths of raw count AnnData objects, in the form of .h5ad files. See example: `example_metadata_nf.tsv`.

#### Input Requirements:

The raw count AnnData objects need to have the following row or column annotations. Note that the exact column name of each key is specified in the config file.

1) a `species_key` in adata.obs to store species identity. Naming should be in line with the short name in ENSEMBL, such as hsapiens; mmusculus; drerio etc.
2) a `cluster_key` in adata.obs to store cell types. If assessment is performed, this column will be used to match homologous cell types across species. Preferably, use [Cell Ontology annotation](https://obofoundry.org/ontology/cl.html). 
3) `mean_counts` in adata.var computed by `sc.pp.calculate_qc_metrics` from [scanpy](https://github.com/scverse/scanpy).

The .var_names of the raw count AnnData file should be ENSEMBL gene ids.
The .X of the raw count AnnData file should be stored in dense matrix format, if SeuratDisk is used for .h5ad/.h5seurat conversion.


## Run instructions:

Perpare the conda environment for .h5ad/.h5seurat conversion. In principle, you can use any program to perform the conversion. Here we used [SeuratDisk](https://github.com/mojaveazure/seurat-disk). SeuratDisk is only available from github and prior installation is required for smoother troubleshoot. Note that the environments for other packages are included in the nextflow program and will be created upon execution. [Mamba](https://github.com/mamba-org/mamba) is recommended as a substitute for conda. 

First create a conda environment for the conversion:

    conda env create -f envs/h5ad_h5seurat_convert.yml
    conda activate hdf5_1820
    R

Then install the package in the R session:

    if (!requireNamespace("remotes", quietly = TRUE)) {
      install.packages("remotes")
    }
    remotes::install_github("mojaveazure/seurat-disk", upgrade = 'never')



Note, the key for this environment to work is the compatible R, hdf5r and hdf5 version. Do not change this from the defined version in envs/h5ad_h5seurat_convert.yml.

### To run BENGAL:

1) `nextflow -C config/example.config run concat_by_homology_multiple_species.nf`
2) Convert concatenated files from .h5ad to .h5seurat using SeuratDisk (do not remove the .h5ad files). This is for running R-based methods that requires .h5seurat as input. The `envs/h5ad_h5seurat_convert.yml` is a working environment to perform the conversion (see above). 
3) `nextflow -C config/example.config run cross_species_integration_multiple_species.nf`
4) Convert methods that output .h5seurat files to .h5ad files (do not remove the .h5seurat files), this is for calculating benchmarking metrics.
5) `nextflow -C config/example.config run cross_species_assessment_multiple_species.nf`

## Outputs:

1) Concatenated raw count AnnData objects containing cells from all species, in the form of .h5ad files. Objects are concatenated by matching genes between species using gene homology annotation from ENSEMBL.  
2) Integration result from different algorithms including: [fastMNN](https://bioconductor.org/packages/release/bioc/html/batchelor.html), [harmony](https://github.com/slowkow/harmonypy), [LIGER](https://github.com/welch-lab/liger), [LIGER-UINMF](https://github.com/welch-lab/liger), [scanorama](https://github.com/brianhie/scanorama), [scVI](https://scvi-tools.org/), [SeuratV4CCA](https://satijalab.org/seurat/) and [SeuratV4RPCA](https://satijalab.org/seurat/), in the form of AnnData (.h5ad) or Seurat (.h5seurat) objects, and respective UMAP visualization.
3) Assessment metrics for each integrated results. There are 4 batch correction metrics and 6 biology conservation metrics. Plots associated for the metrics are also generated for visual inspection. 
4) Cross-species cell type annotation transfer results with [SCCAF](https://github.com/SCCAF/sccaf).


LICENSE: MIT license


