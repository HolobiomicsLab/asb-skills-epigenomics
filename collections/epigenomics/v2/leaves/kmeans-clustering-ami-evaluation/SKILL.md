---
name: kmeans-clustering-ami-evaluation
description: Use when single-cell TLD or chromatin-domain feature matrices (e.g.,
  TF-IDF/SVD embeddings of deDoc2 TLD predictions) need to be clustered and quantitatively
  scored against known cell-type labels via adjusted mutual information.
when_to_use_negative:
- Cells for which an insufficient number of TLDs could be identified (e.g., the '2n
  DNA' stage in the article) — such data were excluded from downstream clustering
- Data lacking any ground-truth cell-type labels, since AMI cannot be computed without
  reference labels
- Raw single-cell Hi-C contact matrices that have not been imputed (RWR, convergence
  ||Qt − Qt−1||2 ≤ 10−2) and normalized (sqrtVC), as sparse raw cells yield too few
  TLDs to embed meaningfully
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: deDoc2
  role: Produces the hierarchical TLD predictions (deDoc2.w and deDoc2.s) per cell
    that form the TF-IDF documents used as clustering features
  repo: https://github.com/zengguangjie/deDoc2
- name: scikit-learn
  role: Provides TruncatedSVD, KMeans, and sklearn.metrics.adjusted_mutual_info_score
    for dimensionality reduction, clustering, and AMI evaluation
- name: MulticoreT-SNE
  role: Generates the 2D embedding map used to qualitatively verify that k-means clusters
    align with cell-type labels
- name: harmonypy
  role: Performs batch correction of the embedding for Lee's dataset prior to 2D visualization
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:50588afa59aa4babdc51090034d4df11c899cad4ef864b0434e5e5e28a884e7c
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# kmeans-clustering-ami-evaluation

## Summary

Clusters single-cell TLD feature vectors (k-means on the first 15 TruncatedSVD dimensions of TF-IDF values from deDoc2 TLD predictions) and quantifies concordance with known cell-type labels using sklearn's adjusted mutual information score, as done on 400 randomly sampled Lee human prefrontal cortex cells.

## When to use

Use when single-cell TLD or chromatin-domain feature matrices (e.g., TF-IDF/SVD embeddings of deDoc2 TLD predictions) need to be clustered and quantitatively scored against known cell-type labels via adjusted mutual information.

## When NOT to use

- Cells for which an insufficient number of TLDs could be identified (e.g., the '2n DNA' stage in the article) — such data were excluded from downstream clustering
- Data lacking any ground-truth cell-type labels, since AMI cannot be computed without reference labels
- Raw single-cell Hi-C contact matrices that have not been imputed (RWR, convergence ||Qt − Qt−1||2 ≤ 10−2) and normalized (sqrtVC), as sparse raw cells yield too few TLDs to embed meaningfully

## Inputs

- Per-cell TLD predictions from deDoc2.w and deDoc2.s (e.g., .TAD and .window.TAD partition files)
- TF-IDF matrix of TLD boundary features per single cell
- First 15 TruncatedSVD dimensions of the TF-IDF matrix
- Ground-truth cell-type labels (e.g., 14 cell types for the 4238 Lee PFC scHi-C cells)

## Outputs

- k-means cluster assignments per cell
- Adjusted mutual information score computed via sklearn.metrics.adjusted_mutual_info_score
- 2D t-SNE embedding coordinates with cell labels (saved as CSV)

## How to apply

First decode hierarchical TLDs per cell (deDoc2.w global partition and deDoc2.s sliding-window partition with default 10 Mb window on RWR-imputed, sqrtVC-normalized contact matrices), treat each cell's TLD set as a text document, compute TF-IDF, and reduce dimensionality with TruncatedSVD retaining 15 dimensions. Fit k-means on the first 15 SVD dimensions (the article uses 14 clusters matching Lee's 14 reported PFC cell types) and compare predicted clusters to known cell-type labels with sklearn.metrics.adjusted_mutual_info_score, evaluated on 400 randomly selected Lee cells. Visualize the clustering structure with a 2D t-SNE embedding (MulticoreT-SNE), applying harmonypy batch correction for the Lee dataset. The AMI score should be interpreted qualitatively against the t-SNE clusters aligning with neural vs non-neural groupings; a score near 0 indicates chance-level agreement and near 1 indicates perfect agreement.

## Related tools

- **deDoc2** (Produces the hierarchical TLD predictions (deDoc2.w and deDoc2.s) per cell that form the TF-IDF documents used as clustering features) — https://github.com/zengguangjie/deDoc2
- **scikit-learn** (Provides TruncatedSVD, KMeans, and sklearn.metrics.adjusted_mutual_info_score for dimensionality reduction, clustering, and AMI evaluation)
- **MulticoreT-SNE** (Generates the 2D embedding map used to qualitatively verify that k-means clusters align with cell-type labels)
- **harmonypy** (Performs batch correction of the embedding for Lee's dataset prior to 2D visualization)

## Examples

```
from sklearn.cluster import KMeans; from sklearn.metrics import adjusted_mutual_info_score; pred = KMeans(n_clusters=14).fit_predict(X_svd15[idx400]); ami = adjusted_mutual_info_score(labels_true[idx400], pred)
```

## Evaluation signals

- Output CSV files exist and are parseable, containing 2D embedding coordinates with cell labels and the AMI score
- The adjusted mutual information score falls within a plausible range (0–1) via sklearn.metrics.adjusted_mutual_info_score
- t-SNE clusters qualitatively align with the 14 reported Lee cell-type labels (at least separating neural vs non-neural categories)
- k-means is run on exactly the first 15 TruncatedSVD dimensions, and AMI is evaluated on 400 randomly selected Lee cells as specified
- Cells failing to yield a sufficient number of TLDs (e.g., '2n DNA' stage) are excluded before clustering

## Limitations

- AMI was evaluated on only 400 randomly selected Lee cells, so the score is a sample-based estimate subject to sampling variability
- Clustering quality depends on upstream RWR imputation and sqrtVC normalization; cells with insufficient TLDs (e.g., '2n DNA' stage) cannot be clustered
- AMI is close to 0 for label assignments that are independent and near 1 only for perfect agreement; intermediate scores require qualitative t-SNE verification against cell-type labels
- No version numbers for deDoc2 or compared tools are stated, and no benchmark metric values for the clustering appear in the discussion section

## Evidence

- [other] SVD + k-means + AMI workflow: [evidence span withheld — anchor status: not_found]
- [results] Dimensionality reduction step: "performed dimensionality reduction on their term‐frequency‐inverse document frequency values using TrunctedSVD"
- [results] Cells excluded for insufficient TLDs: "As we failed to identify a sufficient number of TLDs from “2n DNA” stage, we excluded those data from subsequent analysis"
- [methods] t-SNE embedding on Lee dataset: "2D embedding was performed using TLD predictions as text‐document input on Lee's dataset."
- [methods] RWR imputation convergence criterion: [evidence span withheld — anchor status: not_found]
