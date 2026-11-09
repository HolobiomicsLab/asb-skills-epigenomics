---
name: scchipseq-clustering-benchmarking
description: Use when comparing sub-population identification performance of ChromSCape
  or scATAC/scChIP-seq methods on count matrices where cell-type ground-truth labels
  exist and ARI can be computed.
when_to_use_negative:
- Input data lacks known cell-type ground-truth labels, so an ARI against a reference
  cannot be computed
- Input is raw FASTQ/BAM rather than a count matrix — the data engineering pipeline
  (scChIPseq_DataEngineering) is needed first
- The goal is scRNA-seq or bulk ChIP-seq benchmarking, outside the sparse single-cell
  epigenomic scope of these methods
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0203
tools:
- name: ChromSCape
  role: Method under benchmarking; Shiny/R Bioconductor app performing QC filtering,
    normalization, PCA (50 PCs) and hierarchical clustering of the H3K27me3 mix
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: SnapATAC
  role: Competing method run on 50 kbp genomic bins; transitory objects pre-saved
    as RData; ENCODE blacklist hg38-blacklist.v2.bed removed for this analysis
  repo: https://github.com/pinellolab/scATAC-benchmarking
- name: CisTopic
  role: Competing method run with peaks called from pseudo-bulk (merged_peaks.5000.bed);
    transitory objects saved as RData
  repo: https://github.com/pinellolab/scATAC-benchmarking
- name: Cusanovich2018
  role: Competing method run via the scATAC-benchmarking reference pipeline on the
    same count matrix
  repo: https://github.com/pinellolab/scATAC-benchmarking
- name: EpiScanpy
  role: Competing Python/Scanpy-based method run following the scATAC-seq tutorial
    steps on the same count matrix
  repo: https://github.com/colomemaria/epiScanpy
- name: scOrange
  role: Provides the 'optimized workflow' (Optimized_scChIP_pipe.ows) reproduced for
    Figure 2c
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:28809825e2d02da1020cb04fdc193d1c3244ae461c0de068e3645528b643c51f
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# scchipseq-clustering-benchmarking

## Summary

Benchmark ChromSCape against state-of-the-art single-cell epigenomic clustering methods (Cusanovich2018, SnapATAC, CisTopic, EpiScanpy) on an in-silico H3K27me3 scChIP-seq mix of four cell lines with known ground truth, scoring sub-population recovery by adjusted Rand index.

## When to use

Use when comparing sub-population identification performance of ChromSCape or scATAC/scChIP-seq methods on count matrices where cell-type ground-truth labels exist and ARI can be computed.

## When NOT to use

- Input data lacks known cell-type ground-truth labels, so an ARI against a reference cannot be computed
- Input is raw FASTQ/BAM rather than a count matrix — the data engineering pipeline (scChIPseq_DataEngineering) is needed first
- The goal is scRNA-seq or bulk ChIP-seq benchmarking, outside the sparse single-cell epigenomic scope of these methods

## Inputs

- H3K27me3 scChIP-seq count matrices in 50 kbp bins from GEO GSE117309 (Jurkat, Ramos, HBCx-22) and GSE152502 (MDA-MB-468)
- In-silico mixed count matrix of 1287 QC-passing cells with known cell-line labels
- MM468_identified_CNA.bed (77 Mbp of MDA-MB-468 CNV regions) and hg38-blacklist.v2.bed (ENCODE blacklisted regions)
- merged_peaks.5000.bed (pseudo-bulk called peaks required by CisTopic, Cusanovich2018, SCRAT-optimized)

## Outputs

- ari_results.csv containing five adjusted Rand index values (ChromSCape, Cusanovich2018, SnapATAC, CisTopic, EpiScanpy)
- Per-method reduced feature spaces (e.g. RData transitory objects for CisTopic and SnapATAC)
- Figures 2a and 2b produced by cross_analysis.R in the Cross_analysis folder

## How to apply

Download H3K27me3 scChIP-seq count matrices for Jurkat, Ramos and HBCx-22 from GEO GSE117309 and MDA-MB-468 from GSE152502, then assemble the in-silico mix restricting to the 1287 cells passing QC thresholds using scripts from vallotlab/ChromSCape_benchmarking. Run ChromSCape with default settings — QC filtering (minimum 1600 unique reads per cell, remove top 5% coverage cells), normalization by total read count and region size, then PCA keeping 50 PCs. Run the competing methods from pinellolab/scATAC-benchmarking scripts (Cusanovich2018, SnapATAC on 50 kbp genomic bins, CisTopic using merged_peaks.5000.bed) and EpiScanpy following its scATAC-seq tutorial on the same count matrix. Remove previously identified MDA-MB-468 CNV regions (MM468_identified_CNA.bed, 77 Mbp total) as an extra curation step for SnapATAC/other analyses. Apply hierarchical clustering in each method's reduced feature space and compute the adjusted Rand index against known cell-line labels, expecting ~0.998 for ChromSCape, 0.996 for Cusanovich2018 and CisTopic, 0.940 for EpiScanpy, and 0.822 for SnapATAC.

## Related tools

- **ChromSCape** (Method under benchmarking; Shiny/R Bioconductor app performing QC filtering, normalization, PCA (50 PCs) and hierarchical clustering of the H3K27me3 mix) — https://github.com/vallotlab/ChromSCape_benchmarking
- **SnapATAC** (Competing method run on 50 kbp genomic bins; transitory objects pre-saved as RData; ENCODE blacklist hg38-blacklist.v2.bed removed for this analysis) — https://github.com/pinellolab/scATAC-benchmarking
- **CisTopic** (Competing method run with peaks called from pseudo-bulk (merged_peaks.5000.bed); transitory objects saved as RData) — https://github.com/pinellolab/scATAC-benchmarking
- **Cusanovich2018** (Competing method run via the scATAC-benchmarking reference pipeline on the same count matrix) — https://github.com/pinellolab/scATAC-benchmarking
- **EpiScanpy** (Competing Python/Scanpy-based method run following the scATAC-seq tutorial steps on the same count matrix) — https://github.com/colomemaria/epiScanpy
- **scOrange** (Provides the 'optimized workflow' (Optimized_scChIP_pipe.ows) reproduced for Figure 2c) — https://github.com/vallotlab/ChromSCape_benchmarking

## Examples

```
Rscript cross_analysis.R  # produce Figures 2a and b from the SnapATAC/CisTopic/Cusanovich2018/EpiScanpy/ChromSCape feature spaces and compute ARIs
```

## Evaluation signals

- Each recomputed ARI matches the reported value within ±0.05 (ChromSCape ≈ 0.998, Cusanovich2018 ≈ 0.996, CisTopic ≈ 0.996, EpiScanpy ≈ 0.940, SnapATAC ≈ 0.822)
- The assembled mix contains exactly 1287 cells passing the QC thresholds, matching the paper's benchmark input
- ari_results.csv is produced with all five method scores present
- cross_analysis.R regenerates Figures 2a and 2b in the Cross_analysis folder
- ChromSCape's ARI is equal to or better than every competing method, confirming 'as well or better' performance

## Limitations

- Benchmark restricted to a subsampled set of 1287 cells (with random sub-sampling of 500 MDA-MB-468 cells in ChromSCape), limiting comparison scope
- Required prior removal of 77 Mbp of known MDA-MB-468 CNV segments, an extra manual curation step
- SnapATAC's lower ARI (0.822) reflects its run on 50 kbp bins rather than peak-level features
- Very large transitory files for CisTopic and SnapATAC are stored as RData because full files could not be hosted

## Evidence

- [results] ChromSCape with default parameters achieves near-perfect separation (ARI 0.998), performing on par with Cusanovich2018 and CisTopic (ARI 0.996): [evidence span withheld — anchor status: not_found]
- [discussion] the user sets a threshold on a minimum read count per cell: "the user sets a threshold on a minimum read count per cell and the upper percentile of cells to remove"
- [discussion] principle component analysis (PCA) is applied to the matrix, with centering, and the 50 first PCs are kept: "principle component analysis (PCA) is applied to the matrix, with centering, and the 50 first PCs are kept for further analysis"
- [discussion] a set of 1287 cells passing the quality control thresholds were selected to be used as input for all methods: "a set of 1287 cells passing the quality control thresholds were selected to be used as input for all methods"
- [readme] Four scripts are made to produce the reduced feature space of SnapATAC, CisTopic, Cusanovich2018 and EpiScanpy: "Four scripts are made to produce the reduced feature space of SnapATAC, CisTopic, Cusanovich2018 and EpiScanpy from count H3K27me3 scChIP-seq data"
- [discussion] corresponding to a total of 77Mbp, previously found by analyzing the input of bulk ChIP-seq of the same cells: "corresponding to a total of 77Mbp, previously found by analyzing the input of bulk ChIP-seq of the same cells"
