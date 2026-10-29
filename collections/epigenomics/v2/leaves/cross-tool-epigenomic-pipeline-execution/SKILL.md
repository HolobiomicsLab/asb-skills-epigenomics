---
name: cross-tool-epigenomic-pipeline-execution
description: Use when scChIP-seq or scATAC-seq count matrices with known ground-truth
  labels are available and the goal is to reproduce or extend a multi-method benchmark
  comparing clustering performance via adjusted Rand index.
when_to_use_negative:
- Input is raw FASTQ or single-cell BED files rather than count matrices — use the
  scChIPseq_DataEngineering pipeline instead to generate the count matrix first
- The dataset has no known ground-truth labels, so ARI cannot be computed and a methodological
  ranking is not defensible
- Only a single method is of interest — running the full five-tool benchmark adds
  unnecessary compute and pipeline complexity
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: ChromSCape
  role: Primary method under test; Shiny/R Bioconductor application run with default
    QC, normalization and PCA (50 PCs) settings on the mixed count matrix
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: Cusanovich2018
  role: Competing method; run from scATAC-benchmarking scripts on the same count matrix
    using merged_peaks.5000.bed
  repo: https://github.com/pinellolab/scATAC-benchmarking
- name: SnapATAC
  role: Competing method; run on 50kbp genomic bins with ENCODE blacklist regions
    removed
  repo: https://github.com/pinellolab/scATAC-benchmarking
- name: CisTopic
  role: Competing method; run on pseudo-bulk peaks (merged_peaks.5000.bed) via the
    scATAC-benchmarking pipeline
  repo: https://github.com/pinellolab/scATAC-benchmarking
- name: EpiScanpy
  role: Competing method; run following the scATAC-seq tutorial steps on the same
    count matrix
  repo: https://github.com/colomemaria/epiScanpy
- name: scOrange
  role: Used to run the optimized scChIP workflow (Optimized_scChIP_pipe.ows) reproducing
    Figure 2c
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:b8302a378e63cc38e19f99244e8599b350377e7fe3df2f25d73bfdcbecb97e24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallotlab/ChromSCape_benchmarking
  tools:
  - Bioconductor
  - ChromSCape
  - CisTopic
  - Cusanovich2018
  - EpiScanpy
  - R
  - SnapATAC
  - scOrange
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cross-tool-epigenomic-pipeline-execution

## Summary

Executes a benchmarking comparison of single-cell epigenomic clustering methods — ChromSCape, Cusanovich2018, SnapATAC, CisTopic and EpiScanpy — on the same H3K27me3 scChIP-seq count matrix, reducing each to a feature space and comparing clustering agreement (ARI) against known cell-line labels.

## When to use

Use when scChIP-seq or scATAC-seq count matrices with known ground-truth labels are available and the goal is to reproduce or extend a multi-method benchmark comparing clustering performance via adjusted Rand index.

## When NOT to use

- Input is raw FASTQ or single-cell BED files rather than count matrices — use the scChIPseq_DataEngineering pipeline instead to generate the count matrix first
- The dataset has no known ground-truth labels, so ARI cannot be computed and a methodological ranking is not defensible
- Only a single method is of interest — running the full five-tool benchmark adds unnecessary compute and pipeline complexity

## Inputs

- H3K27me3 scChIP-seq count matrices in 50kbp bins (GSE117309 for Jurkat, Ramos, HBCx-22; GSE152502 for MDA-MB-468)
- Ground-truth cell-line labels for the 1287 QC-passing cells
- merged_peaks.5000.bed pseudo-bulk peaks (required for CisTopic, Cusanovich2018, SCRAT-optimized)
- hg38-blacklist.v2.bed ENCODE blacklisted regions (removed for SnapATAC)
- MM468_identified_CNA.bed MDA-MB-468 copy-number aberrant regions to exclude

## Outputs

- Reduced feature spaces from ChromSCape (50 PCs), SnapATAC, CisTopic, Cusanovich2018 and EpiScanpy
- Hierarchical cluster assignments per method
- ari_results.csv with five adjusted Rand index values
- Figures 2a and 2b reproduced by cross_analysis.R in the Cross_analysis folder

## How to apply

Assemble the in-silico H3K27me3 mix by downloading count matrices for Jurkat, Ramos and HBCx-22 from GSE117309 and MDA-MB-468 from GSE152502, restricting to the 1287 cells passing QC thresholds (minimum coverage 1600 unique reads per cell, top 5% coverage cells filtered out). Run ChromSCape with default settings: QC filtering, normalization by total read count and region size, then PCA keeping the 50 first PCs. Run the competing tools using the scATAC-benchmarking scripts — Cusanovich2018, SnapATAC on 50kbp genomic bins, and CisTopic on merged_peaks.5000.bed pseudo-bulk peaks — and EpiScanpy following its scATAC-seq tutorial. Apply hierarchical clustering to each method's reduced feature space and compute the adjusted Rand index against the cell-line labels; remove the 77 Mbp of known MDA-MB-468 CNV segments (MM468_identified_CNA.bed) before analysis. Expected ARIs are approximately 0.998 (ChromSCape), 0.996 (Cusanovich2018), 0.822 (SnapATAC), 0.996 (CisTopic) and 0.940 (EpiScanpy), each recomputed value must match within ±0.05.

## Related tools

- **ChromSCape** (Primary method under test; Shiny/R Bioconductor application run with default QC, normalization and PCA (50 PCs) settings on the mixed count matrix) — https://github.com/vallotlab/ChromSCape_benchmarking
- **Cusanovich2018** (Competing method; run from scATAC-benchmarking scripts on the same count matrix using merged_peaks.5000.bed) — https://github.com/pinellolab/scATAC-benchmarking
- **SnapATAC** (Competing method; run on 50kbp genomic bins with ENCODE blacklist regions removed) — https://github.com/pinellolab/scATAC-benchmarking
- **CisTopic** (Competing method; run on pseudo-bulk peaks (merged_peaks.5000.bed) via the scATAC-benchmarking pipeline) — https://github.com/pinellolab/scATAC-benchmarking
- **EpiScanpy** (Competing method; run following the scATAC-seq tutorial steps on the same count matrix) — https://github.com/colomemaria/epiScanpy
- **scOrange** (Used to run the optimized scChIP workflow (Optimized_scChIP_pipe.ows) reproducing Figure 2c) — https://github.com/vallotlab/ChromSCape_benchmarking

## Examples

```
Rscript cross_analysis.R
```

## Evaluation signals

- Each recomputed ARI matches the reported value within ±0.05: ChromSCape ≈0.998, Cusanovich2018 ≈0.996, SnapATAC ≈0.822, CisTopic ≈0.996, EpiScanpy ≈0.940
- The count matrix contains exactly the 1287 cells passing QC thresholds before subsampling for methods like the ChromSCape 'subsample - 500 cells' option
- SnapATAC feature space is computed on 50kbp bins with hg38-blacklist.v2.bed regions removed
- ari_results.csv contains all five method values with a consistent row schema
- cross_analysis.R regenerates Figures 2a and 2b in the Cross_analysis output folder

## Limitations

- Benchmark restricted to a subsampled set of 1287 cells with random sub-sampling of 500 MDA-MB-468 cells, limiting comparison scope
- Comparison required prior manual removal of 77 Mbp of known MDA-MB-468 CNV segments, an extra curation step
- SnapATAC's lower ARI (0.822) reflects its run on 50kbp bins rather than peaks; results are feature-resolution dependent
- Transitory objects needed for CisTopic and SnapATAC were stored as RData, tying reproduction to specific R versions

## Evidence

- [discussion] a set of 1287 cells passing the quality control thresholds were selected to be used as input for all methods: "a set of 1287 cells passing the quality control thresholds were selected to be used as input for all methods"
- [discussion] We show that ChromSCape performs as well or better than state of the art single-cell epigenomic analytic tools: "We show that ChromSCape performs as well or better than state of the art single-cell epigenomic analytic tools"
- [discussion] principle component analysis (PCA) is applied to the matrix, with centering, and the 50 first PCs are kept for further analysis: "principle component analysis (PCA) is applied to the matrix, with centering, and the 50 first PCs are kept for further analysis"
- [readme] Four scripts are made to produce the reduced feature space of SnapATAC, CisTopic, Cusanovich2018 and EpiScanpy from count H3K27me3 scChIP-seq data: "Four scripts are made to produce the reduced feature space of SnapATAC, CisTopic, Cusanovich2018 and EpiScanpy from count H3K27me3 scChIP-seq data obtained from an in-silico mix of Jurkat, Ramos,"
- [readme] ‘MM468_identified_CNA.bed’ contains previously identifed Copy Number Abberant regions in the MDA-MB-468 cell line: "‘MM468_identified_CNA.bed’ contains previously identifed Copy Number Abberant regions in the MDA-MB-468 cell line"
