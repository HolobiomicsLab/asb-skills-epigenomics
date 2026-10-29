---
name: imputed-accessibility-matrix-handling
description: Use when a raw, sparse scATAC-seq peak matrix has been denoised with
  SCALE and the task requires the numerical imputed matrix (adata.obsm['imputed'])
  as input to downstream tools like chromVAR.
when_to_use_negative:
- Only a binarized imputation is needed for storage-constrained workflows — the README
  recommends adata.obsm['binary'] with --binary for saving storage instead
- The analysis requires individual-peak resolution or explicit batch-effect removal
  — the article notes chromVAR 'only analyzes peaks in groups and lacks the resolution
  of individual peaks' and SCALE 'is not specifically designed to identify and remove
  these artifacts from the input data'
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0769
tools:
- name: SCALE
  role: Generates the numerical imputed matrix via the --impute option, stored as
    adata.obsm['imputed'] in the output adata.h5ad
  repo: https://github.com/jsxlei/SCALE
- name: chromVAR
  role: Consumes the raw and imputed matrices to compute per-cell motif deviation
    scores and count significantly enriched motifs
- name: scanpy
  role: Loads the adata.h5ad file and provides access to adata.obsm['imputed']; scanpy's
    highly_variable_genes is also used to filter peaks
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:1c7480da0bf129a6cbe2135b6c7dccf446483bdaf691caded427f2291a8c5062
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - SCALE
  - chromVAR
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# imputed-accessibility-matrix-handling

## Summary

Handles the numerical imputed scATAC-seq peak-count matrix produced by SCALE's --impute option (stored as adata.obsm['imputed'] in adata.h5ad) so it can be used for downstream analyses such as chromVAR motif deviation scoring instead of the sparse raw matrix.

## When to use

Use when a raw, sparse scATAC-seq peak matrix has been denoised with SCALE and the task requires the numerical imputed matrix (adata.obsm['imputed']) as input to downstream tools like chromVAR.

## When NOT to use

- Only a binarized imputation is needed for storage-constrained workflows — the README recommends adata.obsm['binary'] with --binary for saving storage instead
- The analysis requires individual-peak resolution or explicit batch-effect removal — the article notes chromVAR 'only analyzes peaks in groups and lacks the resolution of individual peaks' and SCALE 'is not specifically designed to identify and remove these artifacts from the input data'

## Inputs

- Raw scATAC-seq peak-count matrix as h5ad file (e.g. Forebrain.h5ad from http://zhanglab.net/SCALE_SOURCE_DATA/Forebrain.h5ad, 2088 cells, k=8 clusters)
- SCALE output adata.h5ad containing adata.obsm['imputed'] numerical imputation matrix
- Peak set for downstream motif analysis (e.g. 4100 differentially accessible sites)

## Outputs

- Numerical imputed accessibility matrix (adata.obsm['imputed'] in adata.h5ad)
- Per-cell chromVAR motif deviation scores for raw and imputed matrices
- Comparison summary of significant motif counts (e.g. chromvar_motif_activity_results.csv: 52 raw vs 105 imputed)

## How to apply

Run SCALE on the input h5ad (e.g. Forebrain.h5ad, k=8 clusters, 2088 cells) with the --impute option so SCALE writes numerical imputed data into the saved adata.h5ad as adata.obsm['imputed'] (use --binary instead only if a binarized matrix is desired). Load the output with scanpy/anndata, extract adata.obsm['imputed'], and subset both raw and imputed matrices to the same peak set (e.g. the 4100 differentially accessible sites used as the chromVAR peak set). Run chromVAR identically on raw and imputed matrices so per-cell motif deviation scores are directly comparable, then record the number of significantly enriched motifs for each. Rationale: SCALE's VAE+GMM latent-feature imputation addresses scATAC-seq 'missingness', recovering signal (e.g. Mafb, Hoxd9, Dlx2, Lhx8, Arx, Neurog1 motif enrichments) that is absent in the raw data.

## Related tools

- **SCALE** (Generates the numerical imputed matrix via the --impute option, stored as adata.obsm['imputed'] in the output adata.h5ad) — https://github.com/jsxlei/SCALE
- **chromVAR** (Consumes the raw and imputed matrices to compute per-cell motif deviation scores and count significantly enriched motifs)
- **scanpy** (Loads the adata.h5ad file and provides access to adata.obsm['imputed']; scanpy's highly_variable_genes is also used to filter peaks)

## Examples

```
SCALE.py -d Forebrain.h5ad --impute -k 8
```

## Evaluation signals

- adata.obsm['imputed'] exists in the SCALE output adata.h5ad and matches the raw matrix dimensions (2088 cells x peaks for Forebrain)
- Raw and imputed matrices are subset to the identical peak set (4100 differentially accessible sites) before chromVAR runs
- Significant-motif count on the imputed matrix (105) exceeds the raw-matrix count (52)
- All six TF motifs (Mafb, Hoxd9, Dlx2, Lhx8, Arx, Neurog1) appear among the enrichments recovered on imputed data, including Mafb and Hoxd9 specific to the MG cluster only on imputed data

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data
- Numerical imputation consumes more storage than the binarized alternative; the README recommends --binary (adata.obsm['binary']) when saving storage is a priority
- chromVAR analyzes peaks in groups only and lacks individual peak resolution, so imputed per-peak values cannot be interpreted peak-by-peak through chromVAR alone

## Evidence

- [readme] Numerical imputation via --impute: "or get numerical imputed data in adata.h5ad file using scanpy **adata.obsm['imputed']** with option **--impute**"
- [discussion] Downstream motif use of imputed data: "we can improve the discovery of cluster-specific peaks, and regulatory motifs as well, when combined with tools like Homer38 or chromVAR13"
- [methods] Forebrain benchmark dataset: "[Forebrain](http://zhanglab.net/SCALE_SOURCE_DATA/Forebrain.h5ad)"
- [intro] chromVAR group-based caveat: "chromVAR only analyzes peaks in groups and lacks the resolution of individual peaks"
