---
name: single-cell-tld-hierarchical-decoding
description: Use when input is sparse single-cell Hi-C contact matrices (e.g., Lee
  human prefrontal cortex cells) and the goal is to detect hierarchical TAD-like domains
  per cell and test whether nested TLD features distinguish cell types.
when_to_use_negative:
- Input is bulk Hi-C or dense population contact data where standard TAD callers suffice
  — deDoc2's single-cell hierarchy decoding targets sparse per-cell matrices
- The goal is allele-specific or haplotype-resolved domain calling, which this structural-entropy
  workflow does not address
- The cell yields too few contacts to identify a sufficient number of TLDs (as with
  '2n DNA' stage cells, which the article excluded from subsequent analysis)
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3169
tools:
- name: deDoc2
  role: 'Primary predictor: dynamic-programming minimization of 2D structural entropy
    to decode hierarchical TLDs per cell via deDoc2.w (whole matrix) and deDoc2.s
    (10 Mb sliding windows); optional -rwr RWR imputation'
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Predecessor tool for bulk Hi-C TAD prediction based on structural entropy
    minimisation; context for the 1D/2D-nSE metrics used by the family of tools
  repo: https://github.com/yinxc/structural-information-minimisation
- name: scikit-learn
  role: TF-IDF/TruncatedSVD alternative implementation target and k-means clustering
    plus adjusted_mutual_info_score evaluation of TLD-derived cell clusters
- name: MulticoreT-SNE
  role: 2D embedding of the 15-dimension TruncatedSVD-reduced TF-IDF matrix of per-cell
    TLD predictions
- name: harmonypy
  role: Batch correction of the embedding for Lee's dataset prior to visualization/clustering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:da4d74b2441aea0681c224e34c1fe03123843596db89917381a3e7b0e41b072b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - MulticoreT-SNE
  - TruncatedSVD
  - deDoc
  - deDoc2
  - harmonypy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# single-cell-tld-hierarchical-decoding

## Summary

Decode the hierarchy of TAD-like domains (TLDs) in individual single-cell Hi-C contact matrices using deDoc2 (deDoc2.w global partition and deDoc2.s sliding-window partition), then convert each cell's TLD set into a feature vector for cell-type clustering. This skill exists because single-cell Hi-C is too sparse for standard TAD callers and because TLD hierarchy carries cell identity information.

## When to use

Use when input is sparse single-cell Hi-C contact matrices (e.g., Lee human prefrontal cortex cells) and the goal is to detect hierarchical TAD-like domains per cell and test whether nested TLD features distinguish cell types.

## When NOT to use

- Input is bulk Hi-C or dense population contact data where standard TAD callers suffice — deDoc2's single-cell hierarchy decoding targets sparse per-cell matrices
- The goal is allele-specific or haplotype-resolved domain calling, which this structural-entropy workflow does not address
- The cell yields too few contacts to identify a sufficient number of TLDs (as with '2n DNA' stage cells, which the article excluded from subsequent analysis)

## Inputs

- Per-cell intra-chromosomal single-cell Hi-C contact matrix (n × n tab-split contact matrix or sparse from_bin to_bin edge_weight format with bin count header, binsize e.g. 40 kb)
- Cell-type labels for cells (e.g., 14 reported cell types from Lee's human prefrontal cortex dataset, 4238 cells)
- Single-cell RNA-seq expression data for representative cell types (e.g., Astro, MG, L2/3, Vip from the Allen brain human multiple cortical areas Smart-seq resource)

## Outputs

- Per-cell TLD partitions: deDoc2.w .TAD file (higher-level TLDs) and .window.TAD file (lower-level TLDs from deDoc2.s), each line listing node ids of one TLD, numbered from 1
- 2D t-SNE embedding coordinates with cell labels and k-means cluster assignments (CSV)
- Adjusted mutual information (AMI) score quantifying agreement between TLD-feature clusters and cell-type labels
- Cell-type-specific highly expressed gene tables and cell-type-specific nested TLD boundary lists (CSV)

## How to apply

First normalize each cell's contact matrix with sqrtVC normalization and impute it with random walk with restart (RWR), iterating until convergence at ||Qt − Qt−1||2 ≤ 10−2 with restart probability α. Run deDoc2 per cell: deDoc2.w on the whole contact matrix to predict higher-level, larger TLDs and deDoc2.s with the default 10 Mb sliding window to predict lower-level, smaller TLDs; treat each cell's TLD predictions as a text-document input and compute TF-IDF values. Reduce the TF-IDF matrix with TruncatedSVD (retain the first 15 dimensions), optionally batch-correct with harmonypy, embed with MulticoreT-SNE, and cluster the 15 SVD dimensions with k-means. Assess whether clusters recover known cell-type labels by computing the adjusted mutual information score (sklearn.metrics.adjusted_mutual_info_score) against reported labels; the article reports this on 400 randomly selected Lee cells. To link hierarchy to cell function, compute occurrence frequencies of nested TLD boundaries per cell type and call a boundary cell-type-specific when its frequency exceeds the mean plus three standard deviations and no high-frequency boundary occurs in the three bins before/after that position in other cell types.

## Related tools

- **deDoc2** (Primary predictor: dynamic-programming minimization of 2D structural entropy to decode hierarchical TLDs per cell via deDoc2.w (whole matrix) and deDoc2.s (10 Mb sliding windows); optional -rwr RWR imputation) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Predecessor tool for bulk Hi-C TAD prediction based on structural entropy minimisation; context for the 1D/2D-nSE metrics used by the family of tools) — https://github.com/yinxc/structural-information-minimisation
- **scikit-learn** (TF-IDF/TruncatedSVD alternative implementation target and k-means clustering plus adjusted_mutual_info_score evaluation of TLD-derived cell clusters)
- **MulticoreT-SNE** (2D embedding of the 15-dimension TruncatedSVD-reduced TF-IDF matrix of per-cell TLD predictions)
- **harmonypy** (Batch correction of the embedding for Lee's dataset prior to visualization/clustering)

## Examples

```
java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40 -rwr -rp 0.5
```

## Evaluation signals

- Output CSV files (embedding coordinates with labels, AMI score, gene tables, boundary lists) exist and are parseable
- Computed AMI score (sklearn.metrics.adjusted_mutual_info_score) falls in a plausible 0–1 range
- t-SNE clusters qualitatively align with the 14 reported cell-type labels; neural vs. non-neural categories (e.g., L2/3/Vip vs. Astro/MG) separate in the embedding
- Each output TLD line contains space-split node ids numbered from 1; single-bin lines are removed as non-TAD regions
- RWR imputation converges: ||Qt − Qt−1||2 ≤ 10−2 before TLD decoding

## Limitations

- deDoc2 output requires post-processing: single bins occupying one line must be removed as non-TAD regions
- The tool requires no input normalization for raw Hi-C, but RWR imputation needs sqrtVC normalization to converge when applied
- Cells for which an insufficient number of TLDs can be identified (e.g., '2n DNA' stage) must be excluded from downstream analysis
- No version numbers, installation instructions, or data accession IDs are provided in the article/README; supporting benchmark tables are in supplementary material not included here
- Compared tools differ in resources: SuperTAD requires several orders of magnitude higher CPU time and memory to solve high-dimensional structural entropy

## Evidence

- [readme] deDoc2.w minimizes the structural entropy in the whole Hi-C contact map, while the deDoc2.s minimizes the structural entropy in the matrices of sliding windows along the genome.: "deDoc2.w minimizes the structural entropy in the whole Hi-C contact map, while the deDoc2.s minimizes the structural entropy in the matrices of sliding windows along the genome."
- [readme] The output files contains partitions as TLD results. Each line represents a TLD, containg the node id of this TLD, space split, numbering from 1.: "The output files contains partitions as TLD results. Each line represents a TLD, containg the node id of this TLD, space split, numbering from 1."
- [methods] The random walk stops when ∥Qt − Qt−1∥2 ≤ 10−2, which means that the process has converged.: [evidence span withheld — anchor status: not_found]
- [results] performed dimensionality reduction on their term‐frequency‐inverse document frequency values using TrunctedSVD: "performed dimensionality reduction on their term‐frequency‐inverse document frequency values using TrunctedSVD"
- [methods] 2D embedding was performed using TLD predictions as text‐document input on Lee's dataset.: "2D embedding was performed using TLD predictions as text‐document input on Lee's dataset."
- [abstract] the hierarchy of TLDs in single cells is highly dynamic during cell cycle, as well as among human brain cortex cells: "the hierarchy of TLDs in single cells is highly dynamic during cell cycle, as well as among human brain cortex cells"
- [results] As we failed to identify a sufficient number of TLDs from “2n DNA” stage, we excluded those data from subsequent analysis: "As we failed to identify a sufficient number of TLDs from “2n DNA” stage, we excluded those data from subsequent analysis"
