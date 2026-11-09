---
name: modularity-index-calculation
description: Use when scoring the structural quality of predicted TAD/TLD partitions
  on single-cell Hi-C contact matrices (e.g., Tan GM12878, Nagano mouse ES, Lee PFC
  datasets), to compare predictors on modularity alongside 2D structural entropy and
  AMI.
when_to_use_negative:
- Input is bulk Hi-C or already-imputed/sc Hi-C data — this benchmark skill evaluates
  predictors specifically on raw, non-imputed single-cell contact matrices since deDoc2
  'decodes TLD from single-cell Hi-C data with no need of data imputation'
- Cells from stages with too few callable bins (e.g., '2n DNA' stage in Nagano's data)
  where an insufficient number of TLDs can be identified — such data were excluded
  from analysis
- The goal is predictor runtime/memory benchmarking rather than TLD structural quality
  — that requires CPU-time and memory profiling (e.g., SuperTAD's orders-of-magnitude
  higher CPU time, Table S4) instead of modularity
tools:
- name: deDoc2
  role: Primary TLD predictor whose calls (deDoc2.w higher-level, deDoc2.s lower-level)
    are scored by the modularity index
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Comparison TLD predictor scored by modularity on the same scHi-C matrices
  repo: https://github.com/yinxc/structural-information-minimisation
- name: scHiCluster
  role: Imputation-based comparison predictor included in the modularity benchmark
- name: deTOKI
  role: NMF-based comparison predictor included in the modularity benchmark
- name: Higashi
  role: Comparison predictor run with nbr = 4, embedding_epoch = 1, no_nbr_epoch =
    20, then scored by modularity
- name: HiCS
  role: Comparison predictor (HiCS_de) whose modularity values are reported among
    the largest
- name: scikit-learn
  role: Provides adjusted_mutual_info_score used alongside modularity in the benchmark
    aggregation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:791593386b2cc384f2cae811a3c9b93c2e002e0397cf25595476cbf75e18f733
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - HiCS
  - Higashi
  - deDoc
  - deDoc2
  - deTOKI
  - scHiCluster
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# modularity-index-calculation

## Summary

Benchmark-quality metric for TAD-like domain (TLD) predictions in single-cell Hi-C: a modularity index computed on the log2-processed contact matrix, with each gap-region locus treated as a singleton community. Used in the deDoc2 paper to score structural quality of TLDs from deDoc2.w, deDoc2.s, deDoc, scHiCluster, deTOKI, Higashi, and HiCS on experimental scHi-C datasets (Tan, Nagano, Lee, Flyamer).

## When to use

Use when scoring the structural quality of predicted TAD/TLD partitions on single-cell Hi-C contact matrices (e.g., Tan GM12878, Nagano mouse ES, Lee PFC datasets), to compare predictors on modularity alongside 2D structural entropy and AMI.

## When NOT to use

- Input is bulk Hi-C or already-imputed/sc Hi-C data — this benchmark skill evaluates predictors specifically on raw, non-imputed single-cell contact matrices since deDoc2 'decodes TLD from single-cell Hi-C data with no need of data imputation'
- Cells from stages with too few callable bins (e.g., '2n DNA' stage in Nagano's data) where an insufficient number of TLDs can be identified — such data were excluded from analysis
- The goal is predictor runtime/memory benchmarking rather than TLD structural quality — that requires CPU-time and memory profiling (e.g., SuperTAD's orders-of-magnitude higher CPU time, Table S4) instead of modularity

## Inputs

- Raw single-cell Hi-C intra-chromosome contact matrices (n x n tab-split contact matrix format or sparse format: bin count line plus from_bin to_bin edge_weight lines)
- Predicted TLD partitions per predictor (one TLD per line, node ids space-split, numbering from 1)
- Gap-region annotations per contact matrix (loci to be treated as singleton communities)

## Outputs

- Per-cell, per-predictor modularity index values
- Aggregated benchmark table (e.g., tld_benchmark_results.csv) with modularity alongside 2D structural entropy and adjusted mutual information per predictor

## How to apply

For each cell's raw (non-imputed) contact matrix, first replace all zero diagonal elements of the matrix, then log2-transform the remaining entries before computing modularity. Treat each locus falling in a gap region as its own singleton community so that uncovered bins do not inflate the score. Compute modularity for the TLD calls of every predictor (deDoc2.w, deDoc2.s, deDoc, scHiCluster, deTOKI, Higashi, HiCS_de), running predictors with default settings except Higashi (nbr = 4, embedding_epoch = 1, no_nbr_epoch = 20 for multicell datasets). Pair the modularity values with 2D structural entropy computed on a height-2 encoding tree and adjusted mutual information (sklearn.metrics.adjusted_mutual_info_score) to aggregate into a benchmark table such as tld_benchmark_results.csv. Rationale: modularity complements structural entropy because deDoc2.w/HiCS_de were reported among the largest for both structural entropy and modularity on experimental scHi-C data.

## Related tools

- **deDoc2** (Primary TLD predictor whose calls (deDoc2.w higher-level, deDoc2.s lower-level) are scored by the modularity index) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Comparison TLD predictor scored by modularity on the same scHi-C matrices) — https://github.com/yinxc/structural-information-minimisation
- **scHiCluster** (Imputation-based comparison predictor included in the modularity benchmark)
- **deTOKI** (NMF-based comparison predictor included in the modularity benchmark)
- **Higashi** (Comparison predictor run with nbr = 4, embedding_epoch = 1, no_nbr_epoch = 20, then scored by modularity)
- **HiCS** (Comparison predictor (HiCS_de) whose modularity values are reported among the largest)
- **scikit-learn** (Provides adjusted_mutual_info_score used alongside modularity in the benchmark aggregation)

## Evaluation signals

- Zero diagonal elements are replaced before log2-processing the remaining entries — the computed modularity must use the log2-processed matrix, not raw counts
- Every gap-region locus appears as a singleton community in the modularity computation
- Reproduced pattern: deDoc2.w and HiCS_de fall among the largest modularity (and structural entropy) values on experimental scHi-C datasets
- Predictors run with default settings, except Higashi with nbr = 4, embedding_epoch = 1, no_nbr_epoch = 20 for multicell datasets
- Per-predictor modularity values aggregate cleanly into tld_benchmark_results.csv alongside structural entropy and AMI columns

## Limitations

- Cells from stages with insufficient TLDs (e.g., '2n DNA') must be excluded from subsequent analysis
- No version numbers stated for deDoc2 or any compared tool, affecting exact reproducibility
- Supporting information (e.g., Table S4, Figures S2 and S3) with additional quantitative benchmarks is not included in the source document

## Evidence

- [other] modularity on log2-processed matrix with gap singletons: [evidence span withheld — anchor status: not_found]
- [results] modularity tested on Tan's GM12878 cells: "We tested the structural entropy and modularity of TLDs on 16 GM12878 cells from Tan's dataset"
- [readme] deDoc2 predictors definition: "The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher level, larger and lower level, smaller TLDs, respectively."
- [discussion] no imputation rationale: [evidence span withheld — anchor status: not_found]
- [methods] default settings caveat: "All test TLD predictors were run with default settings, except for Higashi"
