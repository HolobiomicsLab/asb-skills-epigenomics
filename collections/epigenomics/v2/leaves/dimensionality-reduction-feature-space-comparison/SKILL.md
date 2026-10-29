---
name: dimensionality-reduction-feature-space-comparison
description: Use when you need to evaluate whether one single-cell epigenomic clustering
  method (e.g. ChromSCape on scChIP-seq H3K27me3 data) performs as well or better
  than alternatives on data with known ground-truth cell labels.
when_to_use_negative:
- The dataset has no known ground-truth cell labels, making ARI-based evaluation impossible
- The input is raw sequencing reads (FASTQ) rather than a count matrix — use the scChIPseq_DataEngineering
  preprocessing pipeline first
- The goal is scRNA-seq rather than sparse single-cell epigenomic (scChIP-seq/scATAC-seq)
  data
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_2229
tools:
- name: ChromSCape
  role: Produce the reference reduced feature space via QC filtering, count normalization
    and PCA keeping 50 PCs, then hierarchical clustering
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: SnapATAC
  role: Competitor method run on 50 kbp genomic bins to generate a reduced feature
    space for ARI comparison
  repo: https://github.com/pinellolab/scATAC-benchmarking
- name: CisTopic
  role: Competitor method run on pseudo-bulk called peaks (merged_peaks.5000.bed)
    to generate a reduced feature space
  repo: https://github.com/pinellolab/scATAC-benchmarking
- name: Cusanovich2018
  role: Competitor method generating a reduced feature space for ARI comparison
  repo: https://github.com/pinellolab/scATAC-benchmarking
- name: EpiScanpy
  role: Competitor method run following its scATAC-seq tutorial steps on the same
    count matrix
  repo: https://github.com/colomemaria/epiScanpy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:e06ade8ba07ae98c725772e9f8c24cd59090ee6035958dd0e564230906acc218
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# dimensionality-reduction-feature-space-comparison

## Summary

Benchmark clustering performance of multiple single-cell epigenomic dimensionality-reduction methods (ChromSCape, Cusanovich2018, SnapATAC, CisTopic, EpiScanpy) on the same count matrix by comparing adjusted Rand indices (ARI) of hierarchical clustering against known cell-line labels.

## When to use

Use when you need to evaluate whether one single-cell epigenomic clustering method (e.g. ChromSCape on scChIP-seq H3K27me3 data) performs as well or better than state-of-the-art alternatives on data with known ground-truth cell labels.

## When NOT to use

- The dataset has no known ground-truth cell labels, making ARI-based evaluation impossible
- The input is raw sequencing reads (FASTQ) rather than a count matrix — use the scChIPseq_DataEngineering preprocessing pipeline first
- The goal is scRNA-seq rather than sparse single-cell epigenomic (scChIP-seq/scATAC-seq) data

## Inputs

- H3K27me3 scChIP-seq count matrices (50 kbp bins) for Jurkat, Ramos, HBCx-22 (GSE117309) and MDA-MB-468 (GSE152502)
- In-silico cell mix count matrix restricted to 1287 QC-passing cells with known cell-line labels
- hg38-blacklist.v2.bed (ENCODE blacklisted regions) and MM468_identified_CNA.bed (77 Mbp MDA-MB-468 CNV regions)
- merged_peaks.5000.bed (peaks called from pseudo-bulk) for CisTopic, Cusanovich2018 and SCRAT-optimized

## Outputs

- Reduced feature space matrices for each of the five methods (ChromSCape PCA with 50 PCs, SnapATAC, CisTopic, Cusanovich2018, EpiScanpy)
- ARI values per method saved to ari_results.csv (expected ~0.998 ChromSCape, 0.996 Cusanovich2018, 0.822 SnapATAC, 0.996 CisTopic, 0.940 EpiScanpy)
- Comparison figures (Figures 2a and 2b) produced by cross_analysis.R

## How to apply

Assemble the in-silico mix of H3K27me3 scChIP-seq count matrices (Jurkat, Ramos, HBCx-22 from GSE117309; MDA-MB-468 from GSE152502), restricting to the 1287 cells passing quality control. Run ChromSCape with default settings (QC filtering, count normalization, PCA keeping 50 PCs) and produce reduced feature spaces for the competitors using scripts from vallotlab/ChromSCape_benchmarking and pinellolab/scATAC-benchmarking (SnapATAC on 50 kbp genomic bins, CisTopic and Cusanovich2018 on pseudo-bulk called peaks from merged_peaks.5000.bed, EpiScanpy following its scATAC tutorial); remove ENCODE blacklist regions (hg38-blacklist.v2.bed) and MDA-MB-468 CNV regions (MM468_identified_CNA.bed) where required. Apply hierarchical clustering on each reduced feature space and compute the adjusted Rand index against the four known cell-line labels. The rationale is that ARI against ground truth isolates clustering quality of each feature space from downstream interpretation, allowing a fair cross-method comparison.

## Related tools

- **ChromSCape** (Produce the reference reduced feature space via QC filtering, count normalization and PCA keeping 50 PCs, then hierarchical clustering) — https://github.com/vallotlab/ChromSCape_benchmarking
- **SnapATAC** (Competitor method run on 50 kbp genomic bins to generate a reduced feature space for ARI comparison) — https://github.com/pinellolab/scATAC-benchmarking
- **CisTopic** (Competitor method run on pseudo-bulk called peaks (merged_peaks.5000.bed) to generate a reduced feature space) — https://github.com/pinellolab/scATAC-benchmarking
- **Cusanovich2018** (Competitor method generating a reduced feature space for ARI comparison) — https://github.com/pinellolab/scATAC-benchmarking
- **EpiScanpy** (Competitor method run following its scATAC-seq tutorial steps on the same count matrix) — https://github.com/colomemaria/epiScanpy

## Examples

```
Rscript cross_analysis.R  # in vallotlab/ChromSCape_benchmarking: computes ARI of each method's feature space and produces Figures 2a and 2b
```

## Evaluation signals

- Each recomputed ARI matches the reported value within ±0.05 (ChromSCape ~0.998, Cusanovich2018 ~0.996, SnapATAC ~0.822, CisTopic ~0.996, EpiScanpy ~0.940)
- All five ARI values are written to ari_results.csv with method names and ground-truth labels used
- The mixed dataset contains exactly 1287 cells after QC filtering, matching the benchmark input
- ChromSCape's ARI is ≥ each competitor's ARI, confirming it performs as well or better than state-of-the-art tools

## Limitations

- Benchmark restricted to a subsampled set of 1287 cells with random sub-sampling of 500 MDA-MB-468 cells, limiting comparison scope
- Required prior removal of known CNV segments of MDA-MB-468 (77 Mbp), an extra manual curation step
- Very large transitory objects needed for CisTopic and SnapATAC were stored as RData rather than regenerated, complicating full end-to-end reproduction

## Evidence

- [discussion] ChromSCape performs as well or better than state-of-the-art single-cell epigenomic tools on an in-silico H3K27me3 scChIP-seq mix: "We show that ChromSCape performs as well or better than state of the art single-cell epigenomic analytic tools"
- [discussion] 1287 QC-passing cells used as input for all methods: "a set of 1287 cells passing the quality control thresholds were selected to be used as input for all methods"
- [discussion] PCA keeping 50 PCs after normalization: "principle component analysis (PCA) is applied to the matrix, with centering, and the 50 first PCs are kept for further analysis"
- [readme] Benchmarking repo description of scripts and additional files: "Scripts and dataset to reproduce benchmarking of SnapATAC, CisTopic, Cusanovich2018 and EpiScanpy &amp; scOrange on single-cell H3K27me3 cell mix"
- [readme] Additional BED files for blacklist, CNV regions and pseudo-bulk peaks: "‘hg38-blacklist.v2.bed’ contains ENCODE blacklisted regions to remove from the analysis for SnapATAC, ‘MM468_identified_CNA.bed’ contains previously identifed Copy Number Abberant regions"
- [readme] cross_analysis.R produces Figures 2a and 2b: "The ‘cross_analysis.R’ will produce the Figures 2a and b in the ‘Cross_analysis’ folder"
