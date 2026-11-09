---
name: tfidf-text-document-embedding-of-tld-profiles
description: Use when you have per-cell TLD/TAD partitions from single-cell Hi-C and
  want a low-dimensional embedding that separates cell types based on nested domain
  boundaries rather than gene expression.
when_to_use_negative:
- Input is already a gene-expression feature table — use standard scRNA-seq clustering
  instead; this skill operates on chromatin TLD boundary profiles.
- Cells yield too few TLDs to profile (e.g. the '2n DNA' stage cells the authors excluded)
  — the TF-IDF document representation is uninformative.
- Bulk Hi-C data where per-cell domain variation is absent — treat as a single document
  and the embedding collapses.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3673
tools:
- name: deDoc2
  role: Predicts the per-cell hierarchical TLD partitions (deDoc2.w global, deDoc2.s
    10 Mb sliding window) that become the text-document inputs for TF-IDF
  repo: https://github.com/zengguangjie/deDoc2
- name: scikit-learn
  role: Provides TruncatedSVD for 15-dimension reduction, k-means clustering, and
    adjusted_mutual_info_score evaluation
- name: MulticoreT-SNE
  role: Produces the 2D embedding map of the dimensionality-reduced TF-IDF/SVD matrix
- name: harmonypy
  role: Batch correction of the embedding for Lee's dataset
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:6a94802e69337f65a7a44a92500914c260222e9f2da34a836501d03338cc3cdd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - MulticoreT-SNE
  - TruncatedSVD
  - deDoc2
  - harmonypy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# TF-IDF text-document embedding of TLD profiles for single-cell clustering

## Summary

Treats each single cell's predicted hierarchical TAD-like domain (TLD) set as a text document, computes TF-IDF values, reduces them with TruncatedSVD (15 dimensions), and embeds with t-SNE to classify single cells by chromatin-domain identity, e.g. distinguishing 14 cell types in the human prefrontal cortex.

## When to use

Use when you have per-cell TLD/TAD partitions from single-cell Hi-C and want a low-dimensional embedding that separates cell types based on nested domain boundaries rather than gene expression.

## When NOT to use

- Input is already a gene-expression feature table — use standard scRNA-seq clustering instead; this skill operates on chromatin TLD boundary profiles.
- Cells yield too few TLDs to profile (e.g. the '2n DNA' stage cells the authors excluded) — the TF-IDF document representation is uninformative.
- Bulk Hi-C data where per-cell domain variation is absent — treat as a single document and the embedding collapses.

## Inputs

- Single-cell Hi-C contact matrices (e.g. 4238 Lee human prefrontal cortex cells; sqrtVC-normalized, RWR-imputed with restart probability α)
- Per-cell deDoc2 TLD predictions (deDoc2.w and deDoc2.s .TAD / .window.TAD partition files, one line per TLD, space-split bin ids numbering from 1)
- Reference cell-type labels (e.g. 14 reported Lee PFC cell types)
- Optional matched single-cell RNA-seq data (Allen brain Smart-seq human multiple cortical areas) for cell-type-specific highly expressed genes

## Outputs

- 15-dimensional TruncatedSVD feature matrix derived from per-cell TF-IDF values of TLD profiles
- 2D t-SNE embedding coordinates with cell labels
- k-means clusters and adjusted mutual information (AMI) score versus known labels
- Cell-type-specific highly expressed gene tables and cell-type-specific high-frequency nested TLD boundary lists (CSV)

## How to apply

First decode hierarchical TLDs for each cell (e.g. 4238 Lee PFC cells) using deDoc2.w (global partition) and deDoc2.s (default 10 Mb sliding window) on RWR-imputed contact matrices after sqrtVC normalization, with RWR convergence at ||Qt − Qt−1||2 ≤ 10−2. Then treat each cell's TLD predictions as a text-document input, compute TF-IDF values, and reduce dimensionality with TruncatedSVD, retaining the first 15 dimensions. Apply t-SNE (MulticoreT-SNE; harmonypy batch correction for Lee's dataset) to obtain a 2D embedding, and cluster the 15 SVD dimensions with k-means. Assess clustering against known cell-type labels with sklearn.metrics.adjusted_mutual_info_score; hierarchical clustering on TLD modularity and TLD-adjR2 roughly separates neural from non-neural cells. The rationale is that cell-type-specific high-frequency nested TLD boundaries carry cell identity information, complementing expression-based labels.

## Related tools

- **deDoc2** (Predicts the per-cell hierarchical TLD partitions (deDoc2.w global, deDoc2.s 10 Mb sliding window) that become the text-document inputs for TF-IDF) — https://github.com/zengguangjie/deDoc2
- **scikit-learn** (Provides TruncatedSVD for 15-dimension reduction, k-means clustering, and adjusted_mutual_info_score evaluation)
- **MulticoreT-SNE** (Produces the 2D embedding map of the dimensionality-reduced TF-IDF/SVD matrix)
- **harmonypy** (Batch correction of the embedding for Lee's dataset)

## Examples

```
python -c "from sklearn.feature_extraction.text import TfidfVectorizer; from sklearn.decomposition import TruncatedSVD; X=TruncatedSVD(n_components=15).fit_transform(TfidfVectorizer().fit_transform(tld_docs))"
```

## Evaluation signals

- Output CSVs (embedding coordinates with labels, AMI score, cell-type-specific gene tables, boundary lists) exist and are parseable.
- Computed adjusted_mutual_info_score falls within the plausible 0–1 range.
- t-SNE clusters qualitatively align with the 14 reported Lee PFC cell-type labels.
- Hierarchical clustering on TLD modularity and TLD-adjR2 roughly separates neural from non-neural cells (Figure 5e).
- Cell-type-specific boundaries are those whose per-cell-type frequency exceeds mean + 3 SD with no high-frequency boundary in the three bins before/after in other cell types.

## Limitations

- Cells with insufficient identified TLDs (e.g. '2n DNA' stage) must be excluded from downstream analysis.
- deDoc2 typically requires RWR imputation (sqrtVC-normalized input, restart probability α) for reliable single-cell TLDs even though it can run without imputation.
- AMI was evaluated on only 400 randomly selected Lee cells, so the score is a sample-based estimate.

## Evidence

- [results] performed dimensionality reduction on their term‐frequency‐inverse document frequency values using TrunctedSVD: "performed dimensionality reduction on their term‐frequency‐inverse document frequency values using TrunctedSVD"
- [methods] 2D embedding was performed using TLD predictions as text‐document input on Lee's dataset.: "2D embedding was performed using TLD predictions as text‐document input on Lee's dataset."
- [results] 4238 single cells are from the human prefrontal cortex (PFC) and were previously identified as having 14 cell types: "4238 single cells are from the human prefrontal cortex (PFC) and were previously identified as having 14 cell types"
- [results] As we failed to identify a sufficient number of TLDs from “2n DNA” stage, we excluded those data from subsequent analysis: "As we failed to identify a sufficient number of TLDs from “2n DNA” stage, we excluded those data from subsequent analysis"
- [readme] The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher level, larger and lower level, smaller TLDs, respectively.: "The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher level, larger and lower level, smaller TLDs, respectively."
