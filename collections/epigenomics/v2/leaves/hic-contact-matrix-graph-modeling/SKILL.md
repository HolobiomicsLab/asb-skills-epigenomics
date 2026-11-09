---
name: hic-contact-matrix-graph-modeling
description: Use when raw (non-imputed) bulk or single-cell Hi-C contact matrices
  must be converted into weighted-graph inputs for structural-entropy-based TLD/TAD
  prediction, including sparse single-cell data.
when_to_use_negative:
- The task requires imputation-based single-cell TLD predictors (scHiCluster, deTOKI,
  Higashi, HiCS) rather than imputation-free structural-entropy decoding — deDoc2
  decodes TLDs with no need of data imputation.
- The input is already a normalized or processed contact map and the goal is metric
  computation (AMI, WS, structural entropy, modularity) rather than TLD prediction.
- Genome-wide inter-chromosomal contacts are the target — the format is n×n intra-chromosome
  matrices only.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: deDoc2
  role: Predicts TLDs from contact matrices modeled as weighted graphs via dynamic-programming
    minimization of 2D structural entropy (deDoc2.w whole-matrix, deDoc2.s sliding-window)
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Predecessor structural-entropy minimization tool for bulk Hi-C TAD prediction
    using the same hicGraph adjacency-matrix input format
  repo: https://github.com/yinxc/structural-information-minimisation
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:e8104559dbe280606816134e71c21d5d820ea2c6ba5044e79e6efd115ec5dd0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - GRiNCH
  - Higashi
  - IS
  - SpectralTAD
  - deDoc
  - deDoc2
  - deTOKI
  - scHiCluster
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# hic-contact-matrix-graph-modeling

## Summary

Model raw intra-chromosomal Hi-C contact maps as weighted graphs so that TAD-like domains (TLDs) can be predicted by globally minimizing 2D structural entropy with deDoc2 (deDoc2.w on the whole matrix, deDoc2.s in 10 Mb sliding windows), with no data imputation required.

## When to use

Use when raw (non-imputed) bulk or single-cell Hi-C contact matrices must be converted into weighted-graph inputs for structural-entropy-based TLD/TAD prediction, including sparse single-cell data.

## When NOT to use

- The task requires imputation-based single-cell TLD predictors (scHiCluster, deTOKI, Higashi, HiCS) rather than imputation-free structural-entropy decoding — deDoc2 decodes TLDs with no need of data imputation.
- The input is already a normalized or processed contact map and the goal is metric computation (AMI, WS, structural entropy, modularity) rather than TLD prediction.
- Genome-wide inter-chromosomal contacts are the target — the format is n×n intra-chromosome matrices only.

## Inputs

- Intra-chromosome Hi-C contact matrix (n×n, tab-split) at a chosen binsize (e.g., 40 kb), raw and unnormalized
- Sparse-format Hi-C graph file: first line = number of bins, subsequent lines = from_bin to_bin edge_weight with node ids starting at 1
- Binsize in kb (default 40 kb) and optional sliding-window / max TLD size (default 10 Mb)

## Outputs

- *.TAD file: deDoc2.w higher-level TLD partitions (one TLD per line as space-split node ids, numbering from 1)
- *.window.TAD file: deDoc2.s lower-level TLD partitions from 10 Mb sliding windows
- Optionally, NDI (normalized decoding information) values for binsize determination via -NDI

## How to apply

Interpret each n×n intra-chromosome contact matrix as a weighted graph (adjacency matrix), accepting either the dense tab-split contact-matrix format or the sparse format whose first line is the bin count followed by 'from_bin to_bin edge_weight' edges with node ids starting at 1. Choose a binsize (default 40 kb as in the deDoc2 benchmarks on downsampled Rao et al. data), optionally add the self-loop value r = 2m/n0 for isolated vertices as specified for deDoc2.w, and supply raw (unnormalized) data since deDoc2 needs no normalization. Run deDoc2.w (dynamic-programming global minimization of 2D structural entropy, TLD length capped at maxTLDsize = 10 Mb) for higher-level TLDs and deDoc2.s (same minimization within 10 Mb sliding windows) for lower-level TLDs; optionally pass -rwr (restart probability rp = 0.5) for random-walk-restart imputation when desired. Drop single-bin lines from the .TAD outputs as non-TAD regions before downstream comparison or benchmarking.

## Related tools

- **deDoc2** (Predicts TLDs from contact matrices modeled as weighted graphs via dynamic-programming minimization of 2D structural entropy (deDoc2.w whole-matrix, deDoc2.s sliding-window)) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Predecessor structural-entropy minimization tool for bulk Hi-C TAD prediction using the same hicGraph adjacency-matrix input format) — https://github.com/yinxc/structural-information-minimisation

## Examples

```
java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40
```

## Evaluation signals

- Output .TAD lines contain only space-separated node ids numbering from 1; single-bin lines are removed as non-TAD regions before use.
- On downsampled Rao et al. bulk data (0.1%, 0.05%, 0.025% sampling at 40 kb binsize), deDoc2.w should achieve AMI and WS at or above every competitor (deDoc, IS, deTOKI, scHiCluster, SpectralTAD, GRiNCH, Higashi).
- On experimental single-cell Hi-C datasets (Tan, Nagano, Flyamer, Lee), deDoc2.w and deDoc2.s should rank first and second for smallest structural entropy of predicted TLDs.
- Height-2 encoding trees built from TLD calls (gap-region vertices placed directly under root λ) should yield favorable (low) 2D structural entropy and high modularity after replacing zero diagonal elements and log2-processing entries.
- Sparse-format files must be symmetric and must begin with the bin count as the first line; deDoc prints the 2D-nSE of the hicGraph, which can be checked against expected values.

## Limitations

- deDoc2 requires a Java runtime environment and is distributed as a jar package.
- SuperTAD requires several orders of magnitude higher CPU time and memory space for high-dimensional structural entropy — runtime cost is a known consideration in this family of tools.
- Cells with insufficient identifiable TLDs (e.g., '2n DNA' stage) must be excluded from downstream analysis.
- No explicit version numbers, installation instructions, or dependency details are provided beyond the Java requirement.

## Evidence

- [readme] deDoc2 is a TAD-like domain(TLD) prediction tool using structural information theory, it treats the Hi-C contact map as a weighted graph: "deDoc2 is a TAD-like domain(TLD) prediction tool using structural information theory, it treats the Hi-C contact map as a weighted graph"
- [readme] sparse format: the first line is the number of bins, following lines are edges formed as from_bin to_bin edge_weight: "sparse format: the first line is the number of bins, following lines are edges formed as from_bin to_bin edge_weight, space split."
- [methods] deDoc2 employed a dynamic programming algorithm to minimize 2D structural entropy.: "deDoc2 employed a dynamic programming algorithm to minimize 2D structural entropy."
- [results] the Hi‐C contact map was regarded as a network‐connected matrix, and the global optimal partition of the network with minimal 2D structural entropy was detected by a dynamic programming algorithm: "the Hi‐C contact map was regarded as a network‐connected matrix, and the global optimal partition of the network with minimal 2D structural entropy was detected by a dynamic programming algorithm"
- [discussion] deDoc2 decodes TLD from single‐cell Hi‐C data with no need of data imputation: "deDoc2 decodes TLD from single‐cell Hi‐C data with no need of data imputation, while most state‐of‐the‐art TLD predictors do"
- [results] sampling rates of 0.1%, 0.05%, and 0.025% and set default binsize = 40 kb: "sampling rates of 0.1%, 0.05%, and 0.025% and set default binsize = 40 kb"
