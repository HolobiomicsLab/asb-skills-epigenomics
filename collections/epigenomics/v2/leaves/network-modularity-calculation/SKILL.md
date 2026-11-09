---
name: network-modularity-calculation
description: Use when single-cell Hi-C contact matrices have been partitioned into
  nested TLDs (e.g., by deDoc2.w/deDoc2.s) and you need a per-cell scalar (modularity
  Q) to track how TLD hierarchy changes across cell-cycle stages (G1, early/mid/late
  S, G2) or other groupings.
when_to_use_negative:
- Insufficient TLDs can be identified for the sample (as with the '2n DNA' stage,
  which the article excluded from subsequent analysis) — modularity over too few/empty
  partitions is not meaningful.
- The goal is TLD boundary detection itself — use deDoc2 prediction rather than the
  downstream modularity scoring step.
- Data require imputation or normalization before TLD calling — deDoc2 'needs no normalization
  process, raw Hi-C data is suitable', and this scoring workflow runs on raw contact
  matrices with no imputation.
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: deDoc2
  role: Predicts the nested TLD partitions whose sub-matrices are scored for modularity;
    deDoc2.w for higher-level and deDoc2.s for lower-level TLDs, default settings,
    no RWR imputation (-rwr omitted).
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Predecessor structural-entropy-minimisation tool for bulk Hi-C TAD detection;
    contextual ancestor, not used in the single-cell modularity workflow.
  repo: https://github.com/yinxc/structural-information-minimisation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:ec2180be78c043ee973673e922fb6f9fd33b7bf7b7b2982203a28672273dab3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - deDoc
  - deDoc2
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# network-modularity-calculation

## Summary

Quantifies the organization strength of deDoc2-predicted nested TAD-like domains (TLDs) in single-cell Hi-C by computing modularity Q = (1/2m)Σ(M_ij − k_i k_j/2m)δ(σ_i,σ_j) on each nested TLD sub-matrix, then averaging over all nested TLDs per cell.

## When to use

Use when single-cell Hi-C contact matrices have been partitioned into nested TLDs (e.g., by deDoc2.w/deDoc2.s) and you need a per-cell scalar (modularity Q) to track how TLD hierarchy changes across cell-cycle stages (G1, early/mid/late S, G2) or other groupings.

## When NOT to use

- Insufficient TLDs can be identified for the sample (as with the '2n DNA' stage, which the article excluded from subsequent analysis) — modularity over too few/empty partitions is not meaningful.
- The goal is TLD boundary detection itself — use deDoc2 prediction rather than the downstream modularity scoring step.
- Data require imputation or normalization before TLD calling — deDoc2 'needs no normalization process, raw Hi-C data is suitable', and this scoring workflow runs on raw contact matrices with no imputation.

## Inputs

- single-cell Hi-C intra-chromosome contact matrices (n × n tab-split dense or sparse edge format, e.g. 'from_bin to_bin edge_weight' with bin count on first line) from cell-cycle-resolved scHi-C stages (G1, early S, middle S, late S, G2) of the Nagano mouse ES cell dataset
- nested TLD partitions predicted by deDoc2 (files suffixed .TAD for deDoc2.w and .window.TAD for deDoc2.s, each line a TLD of space-split node ids numbered from 1)

## Outputs

- per-cell TLD modularity Q averaged over all nested TLDs
- weighted similarity WS(T,K) values between TLD sets of matched nested pairs across stage groups
- summary table cell_cycle_tld_metrics.csv with per-stage aggregated modularity and WS

## How to apply

Run deDoc2 on each single-cell contact matrix without imputation to obtain nested TLD partitions (deDoc2.w for higher-level TLDs, deDoc2.s for lower-level TLDs). For each cell, treat every nested TLD as an independent sub-matrix, replace zero diagonal elements, log2-transform the remaining elements, and compute modularity Q = (1/2m)Σ(M_ij − k_i k_j/2m)δ(σ_i,σ_j), where M_ij is the (log2) contact intensity, k_i is node degree, 2m the total edge weight, and σ_i the partition assignment. TLD modularity is the average modularity over all nested TLDs in the cell. Optionally pair this with the weighted similarity WS(T,K) = Σ_j S_K(T_j)*K_j / Σ_j K_j between TLD sets of stage groups, excluding intermediate windows, and average WS over subTLDs within matched nested TLD pairs. Aggregate per-cell values by stage (e.g., G1, early S, middle S, late S, G2) and save as cell_cycle_tld_metrics.csv.

## Related tools

- **deDoc2** (Predicts the nested TLD partitions whose sub-matrices are scored for modularity; deDoc2.w for higher-level and deDoc2.s for lower-level TLDs, default settings, no RWR imputation (-rwr omitted).) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Predecessor structural-entropy-minimisation tool for bulk Hi-C TAD detection; contextual ancestor, not used in the single-cell modularity workflow.) — https://github.com/yinxc/structural-information-minimisation

## Examples

```
java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40
```

## Evaluation signals

- Per-stage mean modularity and WS of nested TLDs should reproduce the reported gradual decrease from G1 into early and middle S stages (validated against cell_cycle_tld_metrics.csv).
- Output .TAD / .window.TAD files must contain one TLD per line as space-split node ids numbered from 1; single-bin lines should be removed as non-TAD regions before scoring.
- Modularity is computed after zero-diagonal replacement and log2 transformation of the remaining elements of each nested TLD sub-matrix — verify the transform was applied, since raw intensities change Q.
- Intermediate windows are excluded from WS(T,K) computation, and WS is averaged over subTLDs within matched nested TLD pairs — check the exclusion filter before aggregating.
- Cells lacking a sufficient number of identified TLDs should be dropped from aggregation (mirroring the article's exclusion of the '2n DNA' stage).

## Limitations

- deDoc2 needs no imputation, but extreme sparsity can still prevent sufficient TLD identification; such cells must be excluded (as the paper did for the '2n DNA' stage).
- No version numbers are stated for deDoc2 or compared tools in the article, complicating reproducibility pinning.
- The modularity score is a whole-partition summary; it does not resolve which nested TLD drives the change, so it should be paired with WS-based pairwise TLD similarity comparisons.

## Evidence

- [discussion] deDoc2 decodes TLD from single-cell Hi-C data with no need of data imputation: "deDoc2 decodes TLD from single‐cell Hi‐C data with no need of data imputation, while most state‐of‐the‐art TLD predictors do"
- [results] We tested the structural entropy and modularity of TLDs on 16 GM12878 cells from Tan's dataset: "We tested the structural entropy and modularity of TLDs on 16 GM12878 cells from Tan's dataset"
- [abstract] the hierarchy of TLDs in single cells is highly dynamic during cell cycle: "the hierarchy of TLDs in single cells is highly dynamic during cell cycle, as well as among human brain cortex cells"
- [readme] The output files contains partitions as TLD results.: "The output files contains partitions as TLD results. Each line represents a TLD, containg the node id of this TLD, space split, numbering from 1."
