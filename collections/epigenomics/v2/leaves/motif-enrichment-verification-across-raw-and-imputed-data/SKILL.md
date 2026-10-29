---
name: motif-enrichment-verification-across-raw-and-imputed-data
description: Use when a scATAC-seq imputation result (e.g. SCALE --impute output)
  must be validated by showing downstream chromVAR motif detection improves relative
  to the raw peak-count matrix.
when_to_use_negative:
- Input is already a motif activity or deviation-score table — this skill operates
  on peak-count matrices, not downstream enrichment results
- The analysis goal is clustering or visualization of scATAC-seq cells rather than
  verifying imputation quality via motif recovery (use SCALE's UMAP/Leiden outputs
  instead)
- chromVAR group-level motif analysis is insufficient and individual-peak resolution
  is required — the article notes chromVAR 'only analyzes peaks in groups and lacks
  the resolution of individual peaks'
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0749
tools:
- name: SCALE
  role: Produces the denoised/imputed matrix (adata.obsm['imputed'] via --impute)
    that is compared against the raw peak-count matrix
  repo: https://github.com/jsxlei/SCALE
- name: chromVAR
  role: Computes per-cell motif deviation scores on group-of-peaks sharing motifs,
    run identically on raw and imputed matrices
- name: scanpy
  role: Loads raw and imputed h5ad matrices (anndata) and provides highly_variable_genes
    peak filtering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:2d707f175d96e3ab5ce8d51728108e2e4c4dce23428d719684f96167b96f6acd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - SCALE
  - chromVAR
  - scanpy
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# motif-enrichment-verification-across-raw-and-imputed-data

## Summary

A verification skill that runs chromVAR motif deviation analysis identically on raw and SCALE-imputed scATAC-seq matrices and compares the counts and identities of significantly enriched motifs to confirm that imputation recovers additional TF motifs.

## When to use

Use when a scATAC-seq imputation result (e.g. SCALE --impute output) must be validated by showing downstream chromVAR motif detection improves relative to the raw peak-count matrix.

## When NOT to use

- Input is already a motif activity or deviation-score table — this skill operates on peak-count matrices, not downstream enrichment results
- The analysis goal is clustering or visualization of scATAC-seq cells rather than verifying imputation quality via motif recovery (use SCALE's UMAP/Leiden outputs instead)
- chromVAR group-level motif analysis is insufficient and individual-peak resolution is required — the article notes chromVAR 'only analyzes peaks in groups and lacks the resolution of individual peaks'

## Inputs

- Raw scATAC-seq peak-count matrix (h5ad, e.g. Forebrain.h5ad from http://zhanglab.net/SCALE_SOURCE_DATA/Forebrain.h5ad)
- SCALE-imputed matrix stored as adata.obsm['imputed'] in adata.h5ad (produced via SCALE.py --impute)
- Differentially accessible peak set defining the motif-analysis peak set (e.g. 4100 sites)

## Outputs

- chromvar_motif_activity_results.csv containing per-cell motif deviation scores for raw and imputed runs
- Significant-motif counts per condition (52 raw vs 105 imputed) and list of recovered TF motifs (Mafb, Hoxd9, Dlx2, Lhx8, Arx, Neurog1)
- Verification summary documenting raw-vs-imputed motif enrichment comparison

## How to apply

Download the scATAC-seq dataset as an h5ad file (e.g. Forebrain.h5ad, 2088 cells, k=8 clusters) and obtain the SCALE-imputed matrix from adata.obsm['imputed'] produced with the --impute option. Load both the raw peak-count matrix and the imputed matrix with scanpy/anndata, and subset both to the same differentially accessible peak set (e.g. 4100 sites) so the comparison is on identical peaks. Run chromVAR on the raw matrix to compute per-cell motif deviation scores and record the number of significantly enriched motifs and their TFs, then run chromVAR identically on the imputed matrix. Compare results: the validation threshold in the source task is an increase from 52 significant motifs (raw) to 105 (imputed), with recovery of cluster-specific motifs such as Mafb and Hoxd9 enriched in the MG cluster only on imputed data, alongside Dlx2, Lhx8, Arx, and Neurog1. Write motif deviation scores and a verification summary (e.g. chromvar_motif_activity_results.csv) documenting the counts and the recovered TF list. Rationale: SCALE's denoising and imputation addresses scATAC-seq 'missingness', which otherwise suppresses motif signal in sparse peak-by-cell matrices.

## Related tools

- **SCALE** (Produces the denoised/imputed matrix (adata.obsm['imputed'] via --impute) that is compared against the raw peak-count matrix) — https://github.com/jsxlei/SCALE
- **chromVAR** (Computes per-cell motif deviation scores on group-of-peaks sharing motifs, run identically on raw and imputed matrices)
- **scanpy** (Loads raw and imputed h5ad matrices (anndata) and provides highly_variable_genes peak filtering)

## Examples

```
SCALE.py -d Forebrain.h5ad -k 8 --impute; then run chromVAR on X and adata.obsm['imputed'] subset to the 4100 differential peaks and write chromvar_motif_activity_results.csv
```

## Evaluation signals

- Significant-motif count on the imputed matrix exceeds the raw count (validation threshold: 52 raw -> 105 imputed on Forebrain)
- All six TF motifs (Mafb, Hoxd9, Dlx2, Lhx8, Arx, Neurog1) appear among the recovered enrichments on imputed data
- Mafb and Hoxd9 are enriched specifically in the MG cluster only on imputed data
- Both raw and imputed runs used the identical peak set (the 4100 differentially accessible sites) and identical chromVAR settings
- Output CSV contains per-cell motif deviation scores and passes schema/summary checks for both conditions

## Limitations

- chromVAR only analyzes peaks in groups and lacks the resolution of individual peaks
- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data; explicit batch-effect modeling is deferred to future work
- Motif recovery gains depend on the quality of the imputation; the binary imputation option (adata.obsm['binary'] via --binary) is recommended for storage but this verification uses the numerical imputed values

## Evidence

- [intro] chromVAR evaluates motif deviations on grouped peaks: "A recent method specifically developed for scATAC-seq data analysis, chromVAR13, evaluates groups of peaks that share the same motifs or functional annotations together."
- [discussion] SCALE improves cluster-specific peak and motif discovery with Homer/chromVAR: "we can improve the discovery of cluster-specific peaks, and regulatory motifs as well, when combined with tools like Homer38 or chromVAR13."
- [readme] Imputed data access via adata.obsm['imputed']: "or get numerical imputed data in adata.h5ad file using scanpy **adata.obsm['imputed']** with option **--impute**"
- [readme] Forebrain dataset and parameters: "**[Tutorial Forebrain](https://github.com/jsxlei/SCALE/wiki/Forebrain)**   Run SCALE on dense matrix **Forebrain** dataset (k=8, 2088 cells)"
- [intro] scATAC-seq missingness motivates imputation: "The analysis of scATAC-seq data hence suffers from the curse of “missingness” in addition to high dimensionality3."
