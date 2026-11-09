---
name: sparse-region-detection-filtering
description: Use when a single-cell epigenomic count matrix (bins, peaks, or TSS regions
  from scChIP-seq/scATAC-seq) contains sparse regions and you must filter lowly detected
  regions before dimensionality reduction or differential enrichment.
when_to_use_negative:
- Input is already a feature table that has passed region filtering (e.g., count matrices
  from figshare or a pre-filtered peak count matrix from GEO) — reapplying would silently
  drop valid regions
- Analysis targets bulk ChIP-seq data, where the low-coverage sparsity assumptions
  of single-cell region filtering do not apply
- The goal is peak calling from BAM files (which requires MACS2 and pseudo-bulk signal),
  not filtering an existing count matrix
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3168
tools:
- name: ChromSCape
  role: Interactive Shiny/R Bioconductor application that implements the region-detection
    filtering (>=1% of cells, user-defined percentage of cells with coverage >1000
    reads) within its count-matrix QC module
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: R
  role: Language in which ChromSCape's filtering and normalization of the count matrix
    are executed
- name: Bioconductor
  role: Distribution channel for the ChromSCape package that performs the filtering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:fbe86a9ae3f3024bb3772f966e27f71f23f517ce526c25c57c985297ffe7ffdc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallotlab/ChromSCape_benchmarking
  tools:
  - Bioconductor
  - ChromSCape
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Sparse region detection filtering

## Summary

ChromSCape's region-level QC step that removes sparse or lowly-supported genomic bins/peaks from a single-cell count matrix by keeping only regions detected in at least 1% of cells and dropping regions not supported by a minimum user-defined percentage of cells with coverage greater than 1000 reads. This reduces sparsity in scChIP-seq/scATAC-seq matrices before PCA and downstream subpopulation analysis.

## When to use

Use when a single-cell epigenomic count matrix (bins, peaks, or TSS regions from scChIP-seq/scATAC-seq) contains sparse regions and you must filter lowly detected regions before dimensionality reduction or differential enrichment.

## When NOT to use

- Input is already a feature table that has passed region filtering (e.g., count matrices from figshare or a pre-filtered peak count matrix from GEO) — reapplying would silently drop valid regions
- Analysis targets bulk ChIP-seq data, where the low-coverage sparsity assumptions of single-cell region filtering do not apply
- The goal is peak calling from BAM files (which requires MACS2 and pseudo-bulk signal), not filtering an existing count matrix

## Inputs

- Single-cell count matrix (cells x genomic regions: bins, peaks, or TSS regions) produced from BAM/BED files or downloaded from GEO (e.g., GSE105012 gzipped single-cell BED files)
- User-defined minimum percentage of cells threshold for region support
- Optional BED file of regions to exclude (e.g., CNV regions such as MM468_identified_CNA.bed)

## Outputs

- Filtered count matrix retaining only regions detected in >=1% of cells and supported by the user-defined percentage of cells with >1000 reads
- Normalized count matrix (normalized by total read count and region size) for downstream PCA/t-SNE/UMAP and differential enrichment

## How to apply

After generating the count matrix by aggregating single-cell signal into successive genomic bins (e.g., 50,000 bp bins for H3K27me3, 5,000 bp bins or TSS regions from Gencode_hg38_v26 for sharper marks) or into user-provided peaks, apply region-level filtering prior to cell QC or normalization. Keep only regions detected in at least 1% of cells, and additionally filter out regions not supported by a minimum user-defined percentage of cells that have a coverage greater than 1000 reads. This complements, but does not replace, cell-level QC (minimum of 1600 unique reads per cell, removal of the top 5% coverage cells). Rationale: existing bulk ChIP-seq tools are not fit for the low coverage and sparsity of single-cell histone modification datasets, so sparse features degrade downstream PCA (50 PCs kept) and differential enrichment. The filtered matrix is then normalized by total read count and region size. Optionally exclude user-provided BED regions such as known CNV segments (e.g., the 77 Mbp of MDA-MB-468 CNAs) before this step.

## Related tools

- **ChromSCape** (Interactive Shiny/R Bioconductor application that implements the region-detection filtering (>=1% of cells, user-defined percentage of cells with coverage >1000 reads) within its count-matrix QC module) — https://github.com/vallotlab/ChromSCape_benchmarking
- **R** (Language in which ChromSCape's filtering and normalization of the count matrix are executed)
- **Bioconductor** (Distribution channel for the ChromSCape package that performs the filtering)

## Evaluation signals

- Retained regions should all be detected in at least 1% of cells; any region detected in fewer cells indicates the filter was not applied
- No retained region should lack support from the user-defined percentage of cells with coverage greater than 1000 reads
- The filtered matrix should have fewer columns (regions) than the raw binned matrix (e.g., fewer than the full genome-wide 50,000 bp bin set), while cell count should be unchanged by region filtering alone
- After subsequent normalization, values should reflect scaling by total read count and region size, verifiable in count_matrix_normalized.tsv
- Cell-level QC should be independently verified: cells kept must pass the 1600 unique reads minimum and exclude the top 5% coverage cells

## Limitations

- The percentage-of-cells threshold is user-defined, so results are not fully standardized across runs
- Filtering assumes sparse detection reflects noise, but extremely low-coverage datasets may lose biologically real regions; existing tools are not fit for the low coverage and sparsity of single-cell histone modification datasets
- Region filtering operates on the chosen feature space (50kbp bins vs 5kbp bins vs TSS±2500bp); results are not comparable across binning choices
- Known CNV regions (e.g., 77 Mbp in MDA-MB-468) must be removed separately via a user-provided BED file, or they may bias the analysis

## Evidence

- [discussion] Region filtering: keep regions detected in at least 1% of cells: "keeping regions detected in at least 1% of cells"
- [discussion] Region support threshold with >1000 reads: "Regions not supported by a minimum user-defined percentage of cells that have a coverage greater than 1000 reads are filtered out"
- [discussion] Count matrix aggregation into bins and TSS regions: "ChromSCape creates the count matrix by aggregating the signal into successive genomic bins, peaks (BED file must be provided by user) or into regions around genes Transcription Start Sites"
- [readme] 50kbp bins used for H3K27me3 count matrices in benchmarking: "Count matrices of each sample in 50kbp bins can be inputted directly in ChromSCape"
- [readme] CNV region exclusion as a prerequisite curation step: "'MM468_identified_CNA.bed' contains previously identifed Copy Number Abberant regions in the MDA-MB-468 cell line"
