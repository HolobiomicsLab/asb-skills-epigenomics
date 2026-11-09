---
name: nested-tld-hierarchy-assessment
description: Use when single-cell or sparse Hi-C contact matrices must be decomposed
  into hierarchical TLDs and the hierarchy compared across conditions (e.g., cell-cycle
  stages G1, early/mid/late S, G2) via modularity and similarity metrics.
when_to_use_negative:
- The data are bulk Hi-C with deep coverage — deDoc (structural-information-minimisation)
  or hierarchical bulk TAD callers are the appropriate predecessors, not the single-cell
  TLD workflow
- The analysis goal is cell-type clustering or 2D embedding from TLDs (e.g., TF-IDF
  + TruncatedSVD + t-SNE on Lee's PFC dataset) rather than quantifying hierarchy strength
  via modularity/WS
- Cells yield an insufficient number of TLDs (as with the '2n DNA' stage, which the
  authors excluded from subsequent analysis)
edam_operation: http://edamontology.org/operation_3901
edam_topics:
- http://edamontology.org/topic_3957
tools:
- name: deDoc2
  role: Detects hierarchical TLDs from single-cell Hi-C by dynamic-programming minimization
    of 2D structural entropy; deDoc2.w gives higher-level TLDs, deDoc2.s lower-level
    TLDs; runs on raw data without imputation
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Predecessor tool for bulk Hi-C TAD detection via structural entropy minimization;
    not used for the single-cell nested TLD scoring itself
  repo: https://github.com/yinxc/structural-information-minimisation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:0e5c4f3ffc94e83e78bf19a230bd7de0b16cb272d12aadff37661b119239cff3
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
yes_rate: 0.9
n_claims: 10
---

# nested-tld-hierarchy-assessment

## Summary

Quantifies the nested (hierarchical) TAD-like domain (TLD) structure in single-cell Hi-C by running deDoc2 (deDoc2.w for higher-level, deDoc2.s for lower-level TLDs) on raw single-cell contact matrices and scoring each cell with TLD modularity Q and weighted TLD similarity WS(T,K).

## When to use

Use when single-cell or sparse Hi-C contact matrices must be decomposed into hierarchical TLDs and the hierarchy compared across conditions (e.g., cell-cycle stages G1, early/mid/late S, G2) via modularity and similarity metrics.

## When NOT to use

- The data are bulk Hi-C with deep coverage — deDoc (structural-information-minimisation) or hierarchical bulk TAD callers are the appropriate predecessors, not the single-cell TLD workflow
- The analysis goal is cell-type clustering or 2D embedding from TLDs (e.g., TF-IDF + TruncatedSVD + t-SNE on Lee's PFC dataset) rather than quantifying hierarchy strength via modularity/WS
- Cells yield an insufficient number of TLDs (as with the '2n DNA' stage, which the authors excluded from subsequent analysis)

## Inputs

- Raw single-cell Hi-C intra-chromosomal contact matrices (n × n tab-split matrix or sparse format: bin count line plus 'from_bin to_bin edge_weight' edges, node ids from 1), e.g. Nagano mouse ES cell cycle-resolved cells (G1, early S, middle S, late S, G2)
- Binsize of the input data in kb (default 40 kb)

## Outputs

- .TAD files from deDoc2.w (higher-level TLD partitions; one TLD per line, space-split bin ids)
- .window.TAD files from deDoc2.s (lower-level nested TLD partitions)
- Per-cell TLD modularity Q values and weighted similarity WS(T,K) values, aggregated by cell-cycle stage (e.g., cell_cycle_tld_metrics.csv)

## How to apply

Load the raw intra-chromosomal single-cell Hi-C contact matrices and run deDoc2 with no imputation and default settings (40 kb binsize): deDoc2.w minimizes 2D structural entropy over the whole contact map for higher-level TLDs, and deDoc2.s applies deDoc2.w within sliding windows (default 10 Mb) for lower-level TLDs, giving nested TLD partitions per cell. For each cell, treat every nested TLD as an independent matrix, replace zero diagonal elements, log2-transform the remaining elements, and compute modularity Q = (1/2m)Σ(M_ij − k_i k_j/2m)δ(σ_i,σ_j); the TLD modularity is the average Q over all nested TLDs in the cell. To compare hierarchies across cell groups, compute the weighted similarity WS(T,K) = Σ_j S_K(T_j)*K_j / Σ_j K_j over nested TLD pairs, using raw-data TLDs in T and comparison TLDs in K, excluding intermediate windows, and average WS over subTLDs within matched nested TLD pairs. The rationale is that 2D structural entropy minimization via dynamic programming reliably decodes hierarchical TLDs from sparse single-cell data without imputation. Finally aggregate per-cell modularity and WS by group (e.g., cell-cycle stage) and inspect trends; in the Nagano mouse ES cell cycle data both metrics decrease gradually from G1 into early and middle S phase.

## Related tools

- **deDoc2** (Detects hierarchical TLDs from single-cell Hi-C by dynamic-programming minimization of 2D structural entropy; deDoc2.w gives higher-level TLDs, deDoc2.s lower-level TLDs; runs on raw data without imputation) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Predecessor tool for bulk Hi-C TAD detection via structural entropy minimization; not used for the single-cell nested TLD scoring itself) — https://github.com/yinxc/structural-information-minimisation

## Examples

```
java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40
```

## Evaluation signals

- Output files end with .TAD (deDoc2.w) and .window.TAD (deDoc2.s); each line lists bin ids (numbering from 1) of one TLD, and single-bin lines should be removed as non-TAD regions
- Per-cell modularity Q is the average over all nested TLDs after replacing zero diagonals and log2-transforming remaining elements; verify the aggregation step matches this definition
- WS(T,K) uses raw-data TLDs in T and comparison TLDs in K, excludes intermediate windows, and averages over subTLDs within matched nested TLD pairs
- On the Nagano cell-cycle dataset, per-stage modularity and WS should show a gradual decrease from G1 into early and middle S stages, reproducing the published finding
- deDoc2 should run on raw, unnormalized, unimputed matrices — if imputation (-rwr) was required to obtain a sufficient number of TLDs, the workflow deviates from the published no-imputation protocol

## Limitations

- Cells/stages with too few detectable TLDs (e.g., the '2n DNA' stage) must be excluded from the hierarchy analysis
- Downstream metrics depend on binsize; deDoc2 provides -NDI (normalized decoding information) to choose the binsize that maximizes NDI, and the reference workflow used 40 kb
- deDoc2 does not require normalization or imputation, but optional RWR imputation (-rwr, restart probability 0.5, converged when ∥Qt − Qt−1∥2 ≤ 10−2) changes results if enabled
- A single bin in an output line marks a non-TAD region and must be filtered before downstream scoring

## Evidence

- [readme] deDoc2.w and deDoc2.s, to predict higher level, larger and lower level, smaller TLDs, respectively: "The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher level, larger and lower level, smaller TLDs, respectively."
- [readme] deDoc2 needs no normalization process, raw Hi-C data is suitable: "deDoc2 supports both contact matrix format and sparse format as in deDoc. deDoc2 needs no normalization process, raw Hi-C data is suitable."
- [methods] deDoc2 employed a dynamic programming algorithm to minimize 2D structural entropy.: "deDoc2 employed a dynamic programming algorithm to minimize 2D structural entropy."
- [results] As we failed to identify a sufficient number of TLDs from “2n DNA” stage, we excluded those data from subsequent analysis: "As we failed to identify a sufficient number of TLDs from “2n DNA” stage, we excluded those data from subsequent analysis"
- [results] nested TLD hierarchy weakens progressively through S phase: [evidence span withheld — anchor status: not_found]
