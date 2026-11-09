---
name: structural-entropy-graph-partitioning
description: Use when single-cell or sparse bulk Hi-C contact matrices need hierarchical
  TAD/TLD boundary detection without imputation, or when TLD hierarchy dynamics (e.g.,
  across cell-cycle stages) must be quantified via modularity or structural-entropy-based
  partitions.
when_to_use_negative:
- The input is not a Hi-C contact map/graph adjacency structure (e.g., already a feature
  table or expression matrix) — deDoc2 requires a weighted contact graph
- You need bulk-level TAD calling with high-dimensional structural entropy where the
  predecessor deDoc/SuperTAD workflow is already established and hierarchy in single
  cells is not the goal
- Cells yield too few contacts to identify a sufficient number of TLDs (as with the
  '2n DNA' stage, which the authors excluded from subsequent analysis)
edam_operation: http://edamontology.org/operation_3901
edam_topics:
- http://edamontology.org/topic_3957
tools:
- name: deDoc2
  role: 'Primary predictor: deDoc2.w globally minimizes 2D structural entropy for
    higher-level TLDs; deDoc2.s does so in sliding windows for lower-level TLDs, via
    dynamic programming on the contact graph'
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Predecessor structural-entropy-minimisation tool for bulk Hi-C TAD prediction;
    its graph input format (sparse edges with node ids from 1) is shared by deDoc2
  repo: https://github.com/yinxc/structural-information-minimisation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:7f4dead1dc2e88c8d05f44415256b58e1bdc239f6cf32f8be1e0dcec61aece30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - deDoc
  - deDoc2
  edam_operation: http://edamontology.org/operation_3901
  edam_topics:
  - http://edamontology.org/topic_3957
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# structural-entropy-graph-partitioning

## Summary

Partition Hi-C contact maps into hierarchical TAD-like domains (TLDs) by minimizing 2D structural entropy of the contact graph via dynamic programming, as implemented in deDoc2 (deDoc2.w for higher-level, deDoc2.s for lower-level TLDs), with no data imputation required.

## When to use

Use when single-cell or sparse bulk Hi-C contact matrices need hierarchical TAD/TLD boundary detection without imputation, or when TLD hierarchy dynamics (e.g., across cell-cycle stages) must be quantified via modularity or structural-entropy-based partitions.

## When NOT to use

- The input is not a Hi-C contact map/graph adjacency structure (e.g., already a feature table or expression matrix) — deDoc2 requires a weighted contact graph
- You need bulk-level TAD calling with high-dimensional structural entropy where the predecessor deDoc/SuperTAD workflow is already established and hierarchy in single cells is not the goal
- Cells yield too few contacts to identify a sufficient number of TLDs (as with the '2n DNA' stage, which the authors excluded from subsequent analysis)

## Inputs

- Single-cell Hi-C intra-chromosome contact matrices (n x n tab-split, raw/unnormalized) or sparse graph format (bin count line + 'from_bin to_bin edge_weight' edges, node ids from 1)
- Binsize of the input data in kb (default 40 kb)
- Optionally downsampled bulk Hi-C matrices (e.g., sampling rates 0.1%, 0.05%, 0.025%)

## Outputs

- .TAD file: higher-level TLD partitions from deDoc2.w (one TLD per line, space-separated node ids)
- .window.TAD file: lower-level TLD partitions from deDoc2.s via sliding windows
- Per-cell TLD modularity Q and nested-TLD weighted similarity WS values (e.g., aggregated as cell_cycle_tld_metrics.csv)

## How to apply

Treat each intra-chromosome Hi-C contact matrix as a weighted graph and run deDoc2 with dynamic programming to find the global partition minimizing 2D structural entropy: deDoc2.w on the whole contact map for higher-level TLDs and deDoc2.s with a sliding window (default 10 Mb) for lower-level TLDs, using a binsize of 40 kb by default. Supply raw, unnormalized contact matrix (n x n tab-split) or sparse format (first line = number of bins, then 'from_bin to_bin edge_weight' lines, node ids starting from 1); no normalization or RWR imputation is needed unless -rwr is explicitly requested (restart probability 0.5, convergence at ||Qt − Qt−1||2 ≤ 10^-2). Collect the .TAD (deDoc2.w) and .window.TAD (deDoc2.s) outputs, where each line lists the space-separated node ids of one TLD; remove single-bin lines as non-TAD regions. For downstream quantification, treat each nested TLD as an independent matrix, replace zero diagonal elements, log2-transform remaining elements, and compute modularity Q = (1/2m)Σ(M_ij − k_i k_j/2m)δ(σ_i,σ_j); average over nested TLDs per cell. Optionally compute normalized decoding information (-NDI) to choose the binsize, and compare nested TLDs across stage groups with weighted similarity WS(T,K) = Σ_j S_K(T_j)*K_j / Σ_j K_j.

## Related tools

- **deDoc2** (Primary predictor: deDoc2.w globally minimizes 2D structural entropy for higher-level TLDs; deDoc2.s does so in sliding windows for lower-level TLDs, via dynamic programming on the contact graph) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Predecessor structural-entropy-minimisation tool for bulk Hi-C TAD prediction; its graph input format (sparse edges with node ids from 1) is shared by deDoc2) — https://github.com/yinxc/structural-information-minimisation

## Examples

```
java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40
```

## Evaluation signals

- Output .TAD/.window.TAD files parse as lines of contiguous, space-separated bin ids numbered from 1, with single-bin lines removed as non-TAD regions
- Reported 2D-nSE (normalized structural entropy) of the graph decreases relative to the unpartitioned contact matrix, indicating a valid entropy-minimizing partition
- Per-stage TLD modularity and WS of nested TLDs reproduce the reported gradual decrease from G1 into early and middle S stages on the Nagano cell-cycle scHi-C data
- TLD boundaries are reproducible across replicate single cells and consistent with ChIP-seq structural protein/histone marks (e.g., data from Yue et al. for Nagano's dataset)
- Binsize choice is justified by maximal normalized decoding information (NDI) when the -NDI option is used

## Limitations

- SuperTAD-style high-dimensional structural entropy is orders of magnitude more expensive; deDoc2 restricts to 2D structural entropy, trading generality for speed
- Cells/stages with too few contacts yield insufficient TLDs and must be excluded (e.g., the '2n DNA' stage)
- No version numbers, installation instructions, or data accession IDs are documented; supporting tables/figures (e.g., Table S4, Figures S2–S3) are not included in the retrieved text
- All benchmark comparisons used default settings, except for Higashi, so non-default parameter behavior may be under-characterized

## Evidence

- [abstract] deDoc2 seeks genome partitions with global minimal structure entropy for both whole and local contact matrix: "With dynamic programming, deDoc2 seeks genome partitions with global minimal structure entropy for both whole and local contact matrix"
- [readme] The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher level, larger and lower level, smaller TLDs, respectively.: "The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher level, larger and lower level, smaller TLDs, respectively."
- [readme] deDoc2 needs no normalization process, raw Hi-C data is suitable.: "deDoc2 supports both contact matrix format and sparse format as in deDoc. deDoc2 needs no normalization process, raw Hi-C data is suitable."
- [readme] Each line represents a TLD, containg the node id of this TLD, space split, numbering from 1. A single bin in one line should be removed as non-TAD regions.: "Each line represents a TLD, containg the node id of this TLD, space split, numbering from 1.
A single bin in one line should be removed as non-TAD regions."
- [methods] deDoc2 employed a dynamic programming algorithm to minimize 2D structural entropy.: "deDoc2 employed a dynamic programming algorithm to minimize 2D structural entropy."
- [results] As we failed to identify a sufficient number of TLDs from “2n DNA” stage, we excluded those data from subsequent analysis: "As we failed to identify a sufficient number of TLDs from “2n DNA” stage, we excluded those data from subsequent analysis"
