---
name: tsne-dimensionality-reduction-visualization
description: Use when single-cell TLD predictions (or other per-cell text-like feature
  sets) must be visualized or clustered in 2D to test whether domain hierarchy carries
  cell-identity information.
when_to_use_negative:
- When cells yield too few TLDs to build a meaningful feature document (e.g., the
  '2n DNA' stage cells excluded in the article)
- When the goal is bulk Hi-C TAD calling rather than single-cell feature embedding
  — deDoc2.w/deDoc2.s TLD detection alone suffices
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: deDoc2
  role: Generates the hierarchical TLD predictions per cell that are encoded as TF-IDF
    documents prior to SVD and t-SNE
  repo: https://github.com/zengguangjie/deDoc2
- name: TruncatedSVD
  role: Reduces the TF-IDF matrix of TLD features to its first 15 dimensions before
    t-SNE
- name: MulticoreT-SNE
  role: Python module used to compute the 2D embedding map from the SVD-reduced matrix
- name: harmonypy
  role: Batch correction applied to Lee's dataset embedding
- name: scikit-learn
  role: Provides TF-IDF, TruncatedSVD, k-means, and adjusted_mutual_info_score for
    embedding, clustering, and evaluation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:d18dc38f28acc8c05d663c916ea64e1f721a08af2a0c4b0008fe1a33ee7c087e
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
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# tsne-dimensionality-reduction-visualization

## Summary

Two-dimensional embedding of single-cell chromatin-domain features via TF-IDF encoding, TruncatedSVD, and t-SNE, used in the deDoc2 paper to show that hierarchical TLD features distinguish 14 cell types in the human prefrontal cortex (Lee dataset).

## When to use

Use when single-cell TLD predictions (or other per-cell text-like feature sets) must be visualized or clustered in 2D to test whether domain hierarchy carries cell-identity information.

## When NOT to use

- When cells yield too few TLDs to build a meaningful feature document (e.g., the '2n DNA' stage cells excluded in the article)
- When the goal is bulk Hi-C TAD calling rather than single-cell feature embedding — deDoc2.w/deDoc2.s TLD detection alone suffices

## Inputs

- Per-cell TLD boundary/partition predictions from deDoc2.w and deDoc2.s (node-id lists per TLD, bin size 40 kb)
- Cell-type label annotations (14 cell types for the 4238 Lee PFC cells)
- Optional batch identifiers for harmonypy correction

## Outputs

- 2D t-SNE embedding coordinates with cell labels (CSV)
- TruncatedSVD-reduced feature matrix (first 15 dimensions)
- Adjusted mutual information (AMI) score from k-means clustering vs. cell-type labels

## How to apply

Treat each cell's deDoc2 TLD predictions (from deDoc2.w global partition and deDoc2.s 10 Mb sliding-window partition) as a text document and compute TF-IDF values. Reduce dimensionality with TruncatedSVD, retaining the first 15 dimensions. For batched datasets such as Lee's, apply harmonypy batch correction before or during embedding. Run t-SNE (MulticoreT-SNE Python module) on the reduced matrix to obtain a 2D embedding map. Cluster the 15 SVD dimensions with k-means and quantify cluster-vs-label agreement with sklearn.metrics.adjusted_mutual_info_score (evaluated on 400 randomly selected Lee cells) to judge whether the embedding captures cell identity.

## Related tools

- **deDoc2** (Generates the hierarchical TLD predictions per cell that are encoded as TF-IDF documents prior to SVD and t-SNE) — https://github.com/zengguangjie/deDoc2
- **TruncatedSVD** (Reduces the TF-IDF matrix of TLD features to its first 15 dimensions before t-SNE)
- **MulticoreT-SNE** (Python module used to compute the 2D embedding map from the SVD-reduced matrix)
- **harmonypy** (Batch correction applied to Lee's dataset embedding)
- **scikit-learn** (Provides TF-IDF, TruncatedSVD, k-means, and adjusted_mutual_info_score for embedding, clustering, and evaluation)

## Examples

```
from sklearn.feature_extraction.text import TfidfVectorizer; X15 = TruncatedSVD(n_components=15).fit_transform(TfidfVectorizer().fit_transform(tld_docs)); emb = MulticoreTSNE(n_components=2).fit_transform(X15)
```

## Evaluation signals

- Output CSV of 2D embedding coordinates exists and is parseable, with one row per input cell and cell labels attached
- AMI score computed via sklearn.metrics.adjusted_mutual_info_score falls in a plausible 0–1 range on the 400 randomly selected Lee cells
- t-SNE clusters qualitatively align with the 14 reported cell-type labels (e.g., neural vs. non-neural separation as in Figure 5e)
- SVD reduction retains exactly 15 dimensions and the TF-IDF matrix was built from per-cell TLD boundary sets

## Limitations

- Cells with insufficient identified TLDs must be excluded from the embedding (as done for the '2n DNA' stage)
- Batch correction via harmonypy is needed for batched datasets (Lee's), otherwise the embedding may reflect batch rather than biology
- AMI is evaluated on a 400-cell random subsample, so cluster-label agreement estimates carry sampling variability

## Evidence

- [results] Dimensionality reduction of TF-IDF values using TruncatedSVD followed by t-SNE visualization: "performed dimensionality reduction on their term‐frequency‐inverse document frequency values using TrunctedSVD"
- [methods] 2D embedding performed on Lee's dataset using TLD predictions as text-document input: "2D embedding was performed using TLD predictions as text‐document input on Lee's dataset."
- [results] Single cells classified into neural and non-neural categories via hierarchical clustering on TLD modularity and TLD-adjR2: [evidence span withheld — anchor status: not_found]
- [readme] deDoc2 provides two predictors, deDoc2.w and deDoc2.s, for higher- and lower-level TLDs: "The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher level, larger and lower level, smaller TLDs, respectively."
