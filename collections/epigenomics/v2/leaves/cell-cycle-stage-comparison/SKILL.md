---
name: cell-cycle-stage-comparison
description: Use when you have single-cell Hi-C contact matrices resolved by cell-cycle
  stage and need to test whether nested TLD hierarchy (modularity, TLD-adjR2, weighted
  similarity WS) changes across stages such as G1 vs S phase vs G2.
when_to_use_negative:
- Cells with insufficient TLDs identified by deDoc2 (as with the '2n DNA' stage, which
  was excluded from subsequent analysis in the article)
- Input is bulk Hi-C or already-imputed/aggregated data rather than per-cell contact
  matrices — the skill targets cell-to-cell hierarchy dynamics, not population-level
  TADs
- The question is TLD boundary accuracy or binsize selection rather than cross-stage
  dynamics of TLD hierarchy
tools:
- name: deDoc2
  role: Predicts hierarchical (nested) TLDs from each single-cell Hi-C contact matrix
    via dynamic programming minimization of 2D structural entropy; deDoc2.w yields
    higher-level TLDs and deDoc2.s lower-level TLDs that feed the modularity and WS
    comparisons across cell-cycle stages
  repo: https://github.com/zengguangjie/deDoc2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:5dc09b069dc5178c79ab6d301e043f3ec4ef066ad8f99cb3dd5b4ccf42a7d51d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - deDoc2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# cell-cycle-stage-comparison

## Summary

Quantify and compare the hierarchical TAD-like domain (TLD) structure of single-cell Hi-C contact matrices across cell-cycle stages (G1, early/mid/late S, G2) using deDoc2-derived nested TLDs, per-cell modularity Q, weighted similarity WS, and TLD-adjR2. It reveals how nested TLD hierarchy weakens through S phase (Nagano mouse ES cell scHi-C).

## When to use

Use when you have single-cell Hi-C contact matrices resolved by cell-cycle stage and need to test whether nested TLD hierarchy (modularity, TLD-adjR2, weighted similarity WS) changes across stages such as G1 vs S phase vs G2.

## When NOT to use

- Cells with insufficient TLDs identified by deDoc2 (as with the '2n DNA' stage, which was excluded from subsequent analysis in the article)
- Input is bulk Hi-C or already-imputed/aggregated data rather than per-cell contact matrices — the skill targets cell-to-cell hierarchy dynamics, not population-level TADs
- The question is TLD boundary accuracy or binsize selection rather than cross-stage dynamics of TLD hierarchy

## Inputs

- Cell-cycle-resolved single-cell Hi-C contact matrices (G1, early S, middle S, late S, G2 stages), e.g. from Nagano's mouse ES cell scHi-C dataset
- deDoc2 sparse or n×n contact-matrix input files with a specified binsize (default 40 kb)
- Stage labels for each single cell

## Outputs

- Nested TLD partitions per cell (.TAD and .window.TAD files from deDoc2.w and deDoc2.s)
- Per-cell TLD modularity Q averaged over all nested TLDs
- Per-stage weighted similarity WS of nested TLD pairs between stage groups
- Stage-aggregated summary table (cell_cycle_tld_metrics.csv) of modularity and WS by cell-cycle stage

## How to apply

Load the cell-cycle-resolved single-cell Hi-C contact matrices (G1, early S, middle S, late S, G2) and run deDoc2 with default settings and no imputation: deDoc2.w for higher-level TLDs and deDoc2.s for lower-level TLDs on each single-cell contact matrix to obtain nested TLD partitions per cell. For each cell, treat every nested TLD as an independent matrix, replace zero diagonal elements, log2-transform the remaining elements, and compute modularity Q = (1/2m)Σ(M_ij − k_i k_j/2m)δ(σ_i,σ_j), averaging Q over all nested TLDs in the cell. Compute the weighted similarity WS(T,K) = Σ_j S_K(T_j)*K_j / Σ_j K_j between nested TLD pairs of stage groups (raw-data TLDs in T, comparison TLDs in K, excluding intermediate windows), averaging WS over subTLDs within matched nested TLD pairs. Aggregate per-cell modularity and WS by stage into a summary table (e.g. cell_cycle_tld_metrics.csv) and check for the reported gradual decrease from G1 into early and middle S stages.

## Related tools

- **deDoc2** (Predicts hierarchical (nested) TLDs from each single-cell Hi-C contact matrix via dynamic programming minimization of 2D structural entropy; deDoc2.w yields higher-level TLDs and deDoc2.s lower-level TLDs that feed the modularity and WS comparisons across cell-cycle stages) — https://github.com/zengguangjie/deDoc2

## Examples

```
java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40
```

## Evaluation signals

- Per-stage modularity and WS of nested TLDs show the reported gradual decrease from G1 into early and middle S stages
- Every nested TLD matrix used for Q has zero diagonal elements replaced and remaining elements log2-transformed before computing Q = (1/2m)Σ(M_ij − k_i k_j/2m)δ(σ_i,σ_j)
- WS computation excludes intermediate windows and averages over subTLDs within matched nested TLD pairs
- Cells lacking a sufficient number of deDoc2 TLDs are excluded (mirroring the '2n DNA' stage exclusion in the article)
- deDoc2 outputs contain .TAD (deDoc2.w, higher level) and .window.TAD (deDoc2.s, lower level) files, with single-bin lines removed as non-TAD regions

## Limitations

- deDoc2 failed to identify a sufficient number of TLDs from the '2n DNA' stage, so such data must be excluded from subsequent analysis
- deDoc2 is run with default settings and no data imputation, so extremely sparse cells may yield too few TLDs for meaningful stage comparison
- A single bin in one output line of the .TAD files should be removed as non-TAD regions before downstream analysis

## Evidence

- [abstract] Hierarchy dynamics across cell cycle: "the hierarchy of TLDs in single cells is highly dynamic during cell cycle, as well as among human brain cortex cells"
- [results] Nested TLD hierarchy weakens through S phase: [evidence span withheld — anchor status: not_found]
- [results] deDoc2 predictors for hierarchy levels: "The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher and lower level TLDs"
- [discussion] No imputation required: "deDoc2 decodes TLD from single‐cell Hi‐C data with no need of data imputation, while most state‐of‐the‐art TLD predictors do"
- [results] Exclusion of stage with too few TLDs: "As we failed to identify a sufficient number of TLDs from “2n DNA” stage, we excluded those data from subsequent analysis"
- [readme] README output file conventions: "file suffixed `.TAD` is the result of deDoc2.w, the higher level of TLD. - file suffixed `.window.TAD` is the result of deDoc2.s, the lower level of TLD."
