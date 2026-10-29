---
name: chromscape-workflow-execution
description: Use when the input is a sparse single-cell epigenomic count matrix (e.g.
  gzipped reads-in-peaks from GEO like GSE99172) and the goal is to cluster cells
  into subpopulations with common epigenomes and validate clustering against known
  labels such as cell line or replicate.
when_to_use_negative:
- Input is raw FASTQ/BAM reads rather than a count matrix — use the scChIPseq_DataEngineering
  preprocessing pipeline to generate the count matrix first
- The analysis goal is motif/k-mer level featurization or cross-method benchmarking
  of SnapATAC/CisTopic/Cusanovich2018 rather than a single ChromSCape run
- Data are single-cell transcriptomic (scRNA-seq) — ChromSCape is designed for sparse
  single-cell epigenomic (ChIP-seq/ATAC-seq) datasets
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3173
tools:
- name: ChromSCape
  role: Bioconductor Shiny/R application that performs QC filtering, normalization,
    PCA (50 PCs), clustering, t-SNE/UMAP visualization and differential enrichment
    analysis of single-cell epigenomic count matrices
  repo: github.com/vallotlab/ChromSCape_benchmarking
- name: Shiny
  role: Interactive graphical interface through which the ChromSCape workflow steps
    are driven
- name: R
  role: Language in which the count matrix is loaded and metrics (assignment scores,
    Fisher's exact test) are recomputed
- name: Bioconductor
  role: Distribution channel for the ChromSCape package installation
- name: SnapATAC
  role: Benchmark comparator whose clustering performance on scATAC-seq data is contrasted
    with ChromSCape results
  repo: https://github.com/pinellolab/scATAC-benchmarking
- name: CisTopic
  role: Benchmark comparator on the scATAC-seq benchmarking datasets
  repo: https://github.com/pinellolab/scATAC-benchmarking
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:fef7903136e61a40037ad9c973e0574e2972e4ad2f83a40633fa05db40819aec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - ChromSCape
  - CisTopic
  - R
  - Shiny
  - SnapATAC
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# chromscape-workflow-execution

## Summary

Run the ChromSCape interactive Shiny/R pipeline on sparse single-cell epigenomic count matrices (scATAC-seq or scChIP-seq), from reads-in-peaks or binned inputs through QC filtering, PCA, clustering and t-SNE/UMAP visualization to reproduce sub-population structure and reported metrics such as assignment scores.

## When to use

Use when the input is a sparse single-cell epigenomic count matrix (e.g. gzipped reads-in-peaks from GEO like GSE99172) and the goal is to cluster cells into subpopulations with common epigenomes and validate clustering against known labels such as cell line or replicate.

## When NOT to use

- Input is raw FASTQ/BAM reads rather than a count matrix — use the scChIPseq_DataEngineering preprocessing pipeline to generate the count matrix first
- The analysis goal is motif/k-mer level featurization or cross-method benchmarking of SnapATAC/CisTopic/Cusanovich2018 rather than a single ChromSCape run
- Data are single-cell transcriptomic (scRNA-seq) — ChromSCape is designed for sparse single-cell epigenomic (ChIP-seq/ATAC-seq) datasets

## Inputs

- gzipped single-cell BED-derived reads-in-peaks count matrix (e.g. GEO accession GSE99172)
- count matrices in 50kbp genomic bins for scChIP-seq (e.g. H3K27me3 in-silico cell mix of Jurkat, Ramos, HBCx-22, MDA-MB-468)
- cell/sample annotation table with cell line, replicate, or patient labels for computing assignment scores

## Outputs

- k-means/unsupervised cell cluster assignments (e.g. k=5 clusters)
- t-SNE and UMAP two-dimensional embeddings computed on the first 50 PCs
- replicate assignment scores and average correct-cluster assignment percentages
- Fisher's exact test p-value comparing cluster proportions between samples
- metrics table such as gse99172_chromscape_metrics.csv

## How to apply

Load the count matrix (reads-in-peaks or 50kbp bins) in R and apply ChromSCape QC: minimum coverage of 1600 unique reads per cell, removal of cells in the top 5% coverage, and keeping regions detected in at least 1% of cells. Normalize the filtered matrix by total read count and region size, then apply PCA with centering, retaining the first 50 PCs. Cluster cells (e.g. k=5 for the GSE99172 scATAC-seq benchmarking dataset) and visualize with t-SNE and UMAP on the PCA embedding. Compute assignment scores per cell line/replicate (expected: K562 99.7%, GM12878 98.3%, average correct-cluster assignment for TF1/K562/GM12878 of 99.4%) and, for multi-sample comparisons such as SU070 vs SU353, run a Fisher's exact test on cluster proportions (expected p=0.0025) and sub-cluster the dominant cluster. Rationale: these defaults come from the ChromSCape paper's optimized workflow, distributed as a Bioconductor package and benchmarked against SnapATAC, CisTopic and Cusanovich2018.

## Related tools

- **ChromSCape** (Bioconductor Shiny/R application that performs QC filtering, normalization, PCA (50 PCs), clustering, t-SNE/UMAP visualization and differential enrichment analysis of single-cell epigenomic count matrices) — github.com/vallotlab/ChromSCape_benchmarking
- **Shiny** (Interactive graphical interface through which the ChromSCape workflow steps are driven)
- **R** (Language in which the count matrix is loaded and metrics (assignment scores, Fisher's exact test) are recomputed)
- **Bioconductor** (Distribution channel for the ChromSCape package installation)
- **SnapATAC** (Benchmark comparator whose clustering performance on scATAC-seq data is contrasted with ChromSCape results) — https://github.com/pinellolab/scATAC-benchmarking
- **CisTopic** (Benchmark comparator on the scATAC-seq benchmarking datasets) — https://github.com/pinellolab/scATAC-benchmarking

## Examples

```
Rscript cross_analysis.R
```

## Evaluation signals

- Recomputed K562 technical-replicate assignment score matches 99.7% and GM12878 matches 98.3% within a small tolerance
- Average correct-cluster assignment across TF1, K562 and GM12878 cells is 99.4%
- Fisher's exact test comparing monocyte-cluster proportions between SU070 and SU353 yields p ≈ 0.0025
- QC metrics confirm filtering at ≥1600 unique reads/cell, top 5% coverage removal, and retention of regions detected in ≥1% of cells
- PCA embedding retains 50 principal components and t-SNE/UMAP show separated clusters consistent with k=5 cell-type groupings

## Limitations

- Batch effects may require the optional FastMNN mutual-nearest-neighbors correction from the batchelor package
- Benchmark comparison scope was restricted (e.g. 1287 cells after QC, 500-cell subsampling; CNV regions of 77 Mbp in MDA-MB-468 had to be manually excluded)
- Most R dependency versions are unreported, limiting environment reproducibility; the peak calling module additionally requires MACS2 and user-supplied BAM files that are not deposited for all benchmarking datasets
- The demo app is on third-party shinyapps.io with no long-term availability guarantee, and no versioned archive (e.g. Zenodo DOI) exists for the code

## Evidence

- [discussion] a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage: "a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage"
- [discussion] keeping regions detected in at least 1% of cells: "keeping regions detected in at least 1% of cells"
- [discussion] principle component analysis (PCA) is applied to the matrix, with centering, and the 50 first PCs are kept for further analysis: "principle component analysis (PCA) is applied to the matrix, with centering, and the 50 first PCs are kept for further analysis"
- [discussion] The count matrix of reads in peaks was downloaded from GEO accession number “GSE99172”: "The count matrix of reads in peaks was downloaded from GEO accession number “GSE99172”"
- [readme] The ‘cross_analysis.R’ will produce the Figures 2a and b in the ‘Cross_analysis’ folder.: "The ‘cross_analysis.R’ will produce the Figures 2a and b in the ‘Cross_analysis’ folder."
