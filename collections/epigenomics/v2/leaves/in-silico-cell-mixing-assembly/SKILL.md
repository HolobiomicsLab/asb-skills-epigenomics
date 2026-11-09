---
name: in-silico-cell-mixing-assembly
description: Use when benchmarking single-cell epigenomic clustering methods on scChIP-seq/scATAC-seq
  data where no natural labels exist and a ground-truth cell-type mix with known labels
  is required to compute ARIs.
when_to_use_negative:
- The dataset already contains biological labels or annotated cell types, so clustering
  performance can be evaluated directly without constructing a synthetic mix
- The analysis targets discovery of novel sub-populations rather than method benchmarking
  — an in-silico mix imposes artificial, perfectly separable cell identities
- Input data are not count matrices from sparse single-cell epigenomic assays (e.g.
  they are bulk ChIP-seq or scRNA-seq); the assembly procedure and QC thresholds are
  calibrated for scChIP-seq/scATAC-seq sparsity
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3173
tools:
- name: ChromSCape
  role: Ingests the 50kbp-bin count matrices of each sample (with 'subsample - 500
    cells' option) and provides QC filtering, normalization and PCA; source of the
    benchmarked default workflow
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: SnapATAC
  role: Competing method run on the in-silico mix using 50kbp genomic bins after removing
    ENCODE blacklisted regions
  repo: https://github.com/pinellolab/scATAC-benchmarking
- name: CisTopic
  role: Competing method run on the in-silico mix using pseudo-bulk peaks from merged_peaks.5000.bed
  repo: https://github.com/pinellolab/scATAC-benchmarking
- name: Cusanovich2018
  role: Competing method run on the in-silico mix following the scATAC-benchmarking
    reference pipeline
  repo: https://github.com/pinellolab/scATAC-benchmarking
- name: EpiScanpy
  role: Competing method run on the in-silico mix following its scATAC-seq tutorial
    steps
  repo: https://github.com/colomemaria/epiScanpy
- name: scChIPseq_DataEngineering
  role: Upstream pipeline converting raw paired-end fastq reads into exploitable count
    matrices (barcode mapping, trimming, RT/PCR/window duplicate removal, counting)
  repo: https://github.com/vallotlab/scChIPseq_DataEngineering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:a113fd3b5f3f3240a385e2a942f17b2a8f20ce76557154a7be6bb1617ad02aea
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
  - scChIPseq_DataEngineering
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# in-silico-cell-mixing-assembly

## Summary

Assemble a controlled in-silico mixture of single-cell H3K27me3 scChIP-seq count matrices from four known cell lines (Jurkat, Ramos, HBCx-22, MDA-MB-468) so that clustering/benchmarking methods can be evaluated against a known ground-truth label vector using the adjusted Rand index.

## When to use

Use when benchmarking single-cell epigenomic clustering methods on scChIP-seq/scATAC-seq data where no natural labels exist and a ground-truth cell-type mix with known labels is required to compute ARIs.

## When NOT to use

- The dataset already contains biological labels or annotated cell types, so clustering performance can be evaluated directly without constructing a synthetic mix
- The analysis targets discovery of novel sub-populations rather than method benchmarking — an in-silico mix imposes artificial, perfectly separable cell identities
- Input data are not count matrices from sparse single-cell epigenomic assays (e.g. they are bulk ChIP-seq or scRNA-seq); the assembly procedure and QC thresholds are calibrated for scChIP-seq/scATAC-seq sparsity

## Inputs

- H3K27me3 scChIP-seq single-cell count matrices from GEO GSE117309 (Jurkat, Ramos, HBCx-22)
- H3K27me3 scChIP-seq single-cell count matrices from GEO GSE152502 (MDA-MB-468)
- BED file of ENCODE blacklisted regions (hg38-blacklist.v2.bed)
- BED file of MDA-MB-468 copy-number-aberrant regions (MM468_identified_CNA.bed, 77 Mbp)
- BED file of pseudo-bulk-called peaks (merged_peaks.5000.bed)
- Cell-line origin labels for every cell (ground truth)

## Outputs

- A pooled in-silico mixed count matrix covering 1287 QC-passing cells across four cell lines
- A ground-truth label vector mapping each cell to its cell line
- Per-method ARI values (e.g. ari_results.csv) comparing clustering assignments to cell-line labels

## How to apply

Download H3K27me3 single-cell count matrices for Jurkat, Ramos and HBCx-22 from GEO accession GSE117309 and for MDA-MB-468 from GSE152502, and pool them into a single mixed count matrix restricted to the 1287 cells passing the quality-control thresholds (minimum 1600 unique reads per cell, filtering out cells with the top 5% coverage). Use the R and Python scripts in the vallotlab/ChromSCape_benchmarking repository, which ships the supporting BED files: 'hg38-blacklist.v2.bed' (ENCODE blacklisted regions for SnapATAC), 'MM468_identified_CNA.bed' (77 Mbp of known copy-number-aberrant regions in MDA-MB-468 that must be excluded), and 'merged_peaks.5000.bed' (pseudo-bulk peaks for CisTopic, Cusanovich2018 and SCRAT-optimized). Keep the per-cell-line origin of every cell as the ground-truth label vector; after each method produces a reduced feature space, apply hierarchical clustering and compute the adjusted Rand index against these labels. Optionally use ChromSCape's 'subsample - 500 cells' option on 50kbp-bin matrices to reproduce sub-sampled variants of the benchmark.

## Related tools

- **ChromSCape** (Ingests the 50kbp-bin count matrices of each sample (with 'subsample - 500 cells' option) and provides QC filtering, normalization and PCA; source of the benchmarked default workflow) — https://github.com/vallotlab/ChromSCape_benchmarking
- **SnapATAC** (Competing method run on the in-silico mix using 50kbp genomic bins after removing ENCODE blacklisted regions) — https://github.com/pinellolab/scATAC-benchmarking
- **CisTopic** (Competing method run on the in-silico mix using pseudo-bulk peaks from merged_peaks.5000.bed) — https://github.com/pinellolab/scATAC-benchmarking
- **Cusanovich2018** (Competing method run on the in-silico mix following the scATAC-benchmarking reference pipeline) — https://github.com/pinellolab/scATAC-benchmarking
- **EpiScanpy** (Competing method run on the in-silico mix following its scATAC-seq tutorial steps) — https://github.com/colomemaria/epiScanpy
- **scChIPseq_DataEngineering** (Upstream pipeline converting raw paired-end fastq reads into exploitable count matrices (barcode mapping, trimming, RT/PCR/window duplicate removal, counting)) — https://github.com/vallotlab/scChIPseq_DataEngineering

## Examples

```
Rscript cross_analysis.R  # produces Figures 2a and b in the 'Cross_analysis' folder of vallotlab/ChromSCape_benchmarking
```

## Evaluation signals

- The mixed matrix contains exactly the 1287 cells that pass the QC thresholds (>=1600 unique reads per cell, top 5% coverage cells removed)
- Every cell in the mix retains a verifiable cell-line label traceable to GSE117309 or GSE152502
- Recomputed ARIs match the reported values within ~±0.05: ChromSCape ~0.998, Cusanovich2018 ~0.996, CisTopic ~0.996, EpiScanpy ~0.940, SnapATAC ~0.822 (50kbp bins)
- All five ARI values are written to ari_results.csv with method names matching those benchmarked
- Known MDA-MB-468 CNA segments (77 Mbp) are excluded from the feature space before running the methods

## Limitations

- Benchmark comparison restricted to a subsampled set of 1287 cells (with random sub-sampling of 500 MDA-MB-468 cells), a potentially limiting comparison scope
- Benchmark required prior removal of known CNV segments of MDA-MB-468 (77 Mbp), an extra manual curation step based on bulk ChIP-seq of the same cells
- In-silico mixing creates perfectly separable, homogeneous cell types, so ARIs near 1.0 may overestimate performance on real heterogeneous tissues

## Evidence

- [discussion] 1287 QC-passing cells define the benchmark input: "a set of 1287 cells passing the quality control thresholds were selected to be used as input for all methods"
- [discussion] Source GEO accessions for the four cell lines: [evidence span withheld — anchor status: not_found]
- [discussion] MDA-MB-468 data source: [evidence span withheld — anchor status: not_found]
- [discussion] Manual CNV curation step for MDA-MB-468: "corresponding to a total of 77Mbp, previously found by analyzing the input of bulk ChIP-seq of the same cells"
- [readme] README describes the in-silico mix and supporting BED files: "from count H3K27me3 scChIP-seq data obtained from an in-silico mix of Jurkat, Ramos, HBCx-22 and MDA-MB-468 cells"
- [readme] README lists blacklist and CNA BED files: "'hg38-blacklist.v2.bed' contains ENCODE blacklisted regions to remove from the analysis for SnapATAC, 'MM468_identified_CNA.bed' contains previously identifed Copy Number Abberant regions"
- [discussion] QC threshold of 1600 unique reads per cell: "a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage"
