---
name: hic-contact-matrix-tld-detection
description: Use when input is a raw intra-chromosome Hi-C contact matrix (n×n or
  sparse edge format) from bulk or single-cell data and hierarchical TAD/TLD boundaries
  must be predicted, especially for sparse scHi-C where imputation is undesirable.
when_to_use_negative:
- When the goal is bulk TAD calling on dense, well-covered Hi-C and higher-dimensional
  structural entropy tools suffice — deDoc2 is designed for reliable hierarchical
  TLD prediction in sparse single cells
- When input is already a TLD/TAD boundary table rather than a contact matrix — deDoc2
  requires raw Hi-C contact data as a weighted graph
- When a sufficient number of TLDs cannot be identified from the cell's data (e.g.,
  the '2n DNA' stage in the cell-cycle analysis) — such cells should be excluded rather
  than forced through the pipeline
edam_operation: http://edamontology.org/operation_3901
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: deDoc2
  role: Primary TLD predictor; deDoc2.w for higher-level and deDoc2.s for lower-level
    TLDs via 2D structural entropy minimization with dynamic programming
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Predecessor tool for bulk Hi-C TAD prediction using high-dimensional structural
    entropy; deDoc2 extends it with 2D structural entropy and dynamic programming
  repo: https://github.com/yinxc/structural-information-minimisation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:3b48abb767a313c12076723ab4c0a2bfddcbab6e19049a5f2298d4db11172f6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - deDoc
  - deDoc2
  edam_operation: http://edamontology.org/operation_3901
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# hic-contact-matrix-tld-detection

## Summary

Detect hierarchical TAD-like domains (TLDs) from single-cell or sparse Hi-C contact matrices using deDoc2, a structural-information-theory tool that minimizes 2D structural entropy via dynamic programming, without requiring data imputation.

## When to use

Use when input is a raw intra-chromosome Hi-C contact matrix (n×n or sparse edge format) from bulk or single-cell data and hierarchical TAD/TLD boundaries must be predicted, especially for sparse scHi-C where imputation is undesirable.

## When NOT to use

- When the goal is bulk TAD calling on dense, well-covered Hi-C and higher-dimensional structural entropy tools suffice — deDoc2 is designed for reliable hierarchical TLD prediction in sparse single cells
- When input is already a TLD/TAD boundary table rather than a contact matrix — deDoc2 requires raw Hi-C contact data as a weighted graph
- When a sufficient number of TLDs cannot be identified from the cell's data (e.g., the '2n DNA' stage in the cell-cycle analysis) — such cells should be excluded rather than forced through the pipeline

## Inputs

- Intra-chromosome Hi-C contact matrix, n×n tab-split format or sparse format (first line: number of bins; following lines: from_bin to_bin edge_weight), raw/unnormalized data suitable
- Binsize of the input data in kb (default 40 kb)
- Single-cell Hi-C contact matrices per cell/stage (e.g., Nagano mouse ES cell scHi-C: G1, early S, middle S, late S, G2)

## Outputs

- .TAD file from deDoc2.w: higher-level TLD partitions, one TLD per line as space-split node ids numbered from 1
- .window.TAD file from deDoc2.s: lower-level TLD partitions from sliding windows (default 10 Mb)
- Per-cell modularity and nested-TLD weighted similarity (WS) summary tables (e.g., cell_cycle_tld_metrics.csv) when used in dynamics workflows

## How to apply

Treat the Hi-C contact map as a weighted network-connected graph and run deDoc2, which uses a dynamic programming algorithm to minimize 2D structural entropy for global optimization of genome partitions. The package provides two predictors: deDoc2.w, which minimizes structural entropy over the whole contact matrix to predict higher-level, larger TLDs (output suffixed .TAD), and deDoc2.s, which applies deDoc2.w within sliding windows along the genome to predict lower-level, smaller TLDs (output suffixed .window.TAD; default windowsize 10 Mb). Run with a required -binsize (default 40 kb); deDoc2 needs no normalization and raw Hi-C data is suitable, with optional -rwr (random walk with restart, restart probability -rp 0.5) imputation if desired. For cell-cycle dynamics analyses, apply deDoc2.w and deDoc2.s per single-cell contact matrix (e.g., Nagano mouse ES cell stages G1, early/mid/late S, G2), then compute per-cell modularity Q = (1/2m)Σ(M_ij − k_i k_j/2m)δ(σ_i,σ_j) averaged over nested TLDs, and weighted similarity WS(T,K) = Σ_j S_K(T_j)*K_j / Σ_j K_j of nested TLD pairs between stage groups. In output files each line is a TLD containing space-split node ids numbered from 1; single bins in one line should be removed as non-TAD regions.

## Related tools

- **deDoc2** (Primary TLD predictor; deDoc2.w for higher-level and deDoc2.s for lower-level TLDs via 2D structural entropy minimization with dynamic programming) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Predecessor tool for bulk Hi-C TAD prediction using high-dimensional structural entropy; deDoc2 extends it with 2D structural entropy and dynamic programming) — https://github.com/yinxc/structural-information-minimisation

## Examples

```
java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40
```

## Evaluation signals

- Both .TAD (deDoc2.w) and .window.TAD (deDoc2.s) output files exist; each line contains space-split node ids numbered from 1, and single-bin lines are filtered as non-TAD regions
- Normalized decoding information (NDI, via -NDI flag) can be computed to confirm the chosen binsize maximizes NDI of the Hi-C contact matrix
- For cell-cycle scHi-C (Nagano mouse ES cells), per-stage modularity and WS of nested TLDs should reproduce the reported gradual decrease from G1 into early and middle S stages
- The input Hi-C matrix should be a symmetric intra-chromosome contact matrix; the 2D structural entropy of the partition should be the global minimum achieved by dynamic programming
- Cells yielding an insufficient number of TLDs are identified and excluded (as done for the '2n DNA' stage) rather than producing degenerate partitions

## Limitations

- deDoc2 relies on a Java environment (java -jar deDoc2.jar); dependencies and version numbers are not stated in the article
- TLD detection can fail on very sparse cells — a sufficient number of TLDs may not be identified (e.g., '2n DNA' stage excluded from subsequent analysis)
- Compared tools such as SuperTAD require orders of magnitude higher CPU time and memory; deDoc2 trades this off, but quantitative benchmark values were not included in the discussion section
- Optional RWR imputation requires sqrtVC normalization beforehand to ensure convergence, stopping when ∥Qt − Qt−1∥2 ≤ 10−2

## Evidence

- [results] The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher and lower level TLDs: "The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher and lower level TLDs"
- [discussion] deDoc2 decodes TLD from single-cell Hi-C data with no need of data imputation: "deDoc2 decodes TLD from single‐cell Hi‐C data with no need of data imputation, while most state‐of‐the‐art TLD predictors do"
- [readme] deDoc2 supports both contact matrix format and sparse format as in deDoc. deDoc2 needs no normalization process, raw Hi-C data is suitable.: "deDoc2 supports both contact matrix format and sparse format as in deDoc. deDoc2 needs no normalization process, raw Hi-C data is suitable."
- [methods] A sliding window is used to split the Hi‐C contact matrix and deDoc2.w is applied to the split matrix: "A sliding window is used to split the Hi‐C contact matrix and deDoc2.w is applied to the split matrix"
- [readme] file suffixed `.TAD` is the result of deDoc2.w, the higher level of TLD.: "- file suffixed `.TAD` is the result of deDoc2.w, the higher level of TLD.
- file suffixed `.window.TAD` is the result of deDoc2.s, the lower level of TLD."
