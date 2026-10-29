---
name: chromvar-motif-deviation-scoring
description: Use when analyzing scATAC-seq peak-count data (e.g., h5ad from SCALE)
  to quantify motif activity per cell and compare motif detection between raw and
  imputed/denoised matrices.
when_to_use_negative:
- Input is single-cell RNA-seq data rather than scATAC-seq peak accessibility (direct
  applications of scRNA-seq methods to scATAC-seq may not be suitable due to the close-to-binary,
  sparse nature of the data)
- Per-peak (individual) resolution is required, since chromVAR only analyzes peaks
  in groups sharing the same motifs and lacks the resolution of individual peaks
- The peak set has not been restricted to informative (e.g., differentially accessible)
  sites, so motif deviation scores would be diluted by background peaks
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0749
tools:
- name: SCALE
  role: Denoises and imputes the sparse scATAC-seq count matrix (--impute) to produce
    the adata.obsm['imputed'] matrix used as chromVAR input
  repo: https://github.com/jsxlei/SCALE
- name: chromVAR
  role: Computes per-cell motif deviation scores over groups of peaks sharing the
    same motifs to identify significantly enriched TF motifs
- name: scanpy
  role: Used with anndata to load the h5ad files, access adata.obsm['imputed'], and
    filter peaks via highly_variable_genes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:2a589a769066d22078f26115f978111eb66c4a266907118da40646ea5d35b392
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
yes_rate: 0.6
n_claims: 10
---

# chromvar-motif-deviation-scoring

## Summary

Run chromVAR on scATAC-seq peak-count matrices to compute per-cell motif deviation scores and identify significantly enriched transcription factor motifs, using SCALE-imputed data to recover additional motifs beyond what sparse raw data yields.

## When to use

Use when analyzing scATAC-seq peak-count data (e.g., h5ad from SCALE) to quantify motif activity per cell and compare motif detection between raw and imputed/denoised matrices.

## When NOT to use

- Input is single-cell RNA-seq data rather than scATAC-seq peak accessibility (direct applications of scRNA-seq methods to scATAC-seq may not be suitable due to the close-to-binary, sparse nature of the data)
- Per-peak (individual) resolution is required, since chromVAR only analyzes peaks in groups sharing the same motifs and lacks the resolution of individual peaks
- The peak set has not been restricted to informative (e.g., differentially accessible) sites, so motif deviation scores would be diluted by background peaks

## Inputs

- Raw scATAC-seq peak-count matrix (h5ad, e.g., Forebrain.h5ad from http://zhanglab.net/SCALE_SOURCE_DATA/Forebrain.h5ad)
- SCALE-imputed matrix stored as adata.obsm['imputed'] in adata.h5ad (from SCALE.py --impute)
- Peak set of differentially accessible sites (e.g., 4100 peaks) as chromVAR input

## Outputs

- Per-cell chromVAR motif deviation scores for raw and imputed matrices
- Count of significantly enriched motifs per condition (e.g., 52 raw vs 105 imputed)
- List of significant TF motifs (e.g., Mafb, Hoxd9, Dlx2, Lhx8, Arx, Neurog1)
- chromvar_motif_activity_results.csv with deviation scores and verification summary

## How to apply

Load the raw peak-count matrix and the SCALE-imputed matrix (stored as adata.obsm['imputed'] in adata.h5ad, produced with SCALE.py --impute) with scanpy/anndata. Subset both matrices to the differentially accessible sites serving as the peak set (e.g., 4100 sites in the Forebrain dataset with k=8 clusters, 2088 cells). Run chromVAR identically on the raw and imputed matrices to compute per-cell motif deviation scores and record the number of significantly enriched motifs with their associated transcription factors. The rationale is that SCALE's denoising and imputation addresses the 'missingness' of scATAC-seq data, so motifs like Mafb and Hoxd9 that are enriched only in specific clusters (e.g., the MG cluster) become detectable on imputed data. Compare raw vs. imputed results by counting significant motifs and verifying that cluster-specific TF motif enrichments are recovered. Write motif deviation scores and the verification summary to an output CSV (e.g., chromvar_motif_activity_results.csv).

## Related tools

- **SCALE** (Denoises and imputes the sparse scATAC-seq count matrix (--impute) to produce the adata.obsm['imputed'] matrix used as chromVAR input) — https://github.com/jsxlei/SCALE
- **chromVAR** (Computes per-cell motif deviation scores over groups of peaks sharing the same motifs to identify significantly enriched TF motifs)
- **scanpy** (Used with anndata to load the h5ad files, access adata.obsm['imputed'], and filter peaks via highly_variable_genes)

## Examples

```
SCALE.py -d Forebrain.h5ad -k 8 --impute; # then in Python: adata = sc.read_h5ad('adata.h5ad'); X = adata.obsm['imputed']
```

## Evaluation signals

- Significant-motif count increases from 52 (raw Forebrain) to 105 (SCALE-imputed Forebrain)
- All six TF motifs — Mafb, Hoxd9, Dlx2, Lhx8, Arx, Neurog1 — appear among the recovered enrichments on imputed data
- Mafb and Hoxd9 are enriched specifically in the MG cluster only on the imputed data
- Output CSV contains motif deviation score matrix plus a verification summary and is accepted only if both count-increase and motif-recovery criteria hold
- Raw and imputed chromVAR runs used the identical peak set (4100 differentially accessible sites), so any score differences are attributable to imputation

## Limitations

- chromVAR only analyzes peaks in groups and lacks the resolution of individual peaks
- SCALE's imputed values must be accessed as adata.obsm['imputed'] via the --impute option; the --binary option produces binarized imputation values instead, which is not what this workflow uses
- Results depend on prior peak filtering (--min_cells default 3, --min_peaks default 100, or highly_variable_genes selection via --n_feature), which can change the motif signal
- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data

## Evidence

- [intro] chromVAR evaluates groups of peaks sharing motifs: "A recent method specifically developed for scATAC-seq data analysis, chromVAR13, evaluates groups of peaks that share the same motifs or functional annotations together."
- [discussion] SCALE improves cluster-specific peak and motif discovery: "we can improve the discovery of cluster-specific peaks, and regulatory motifs as well, when combined with tools like Homer38 or chromVAR13."
- [intro] chromVAR caveat: group-level analysis only: "chromVAR only analyzes peaks in groups and lacks the resolution of individual peaks"
- [readme] SCALE imputation option stores data in adata.obsm: "get numerical imputed data in adata.h5ad file using scanpy **adata.obsm['imputed']** with option **--impute**"
- [methods] scanpy-based peak filtering: "Introduce the highly_variable_genes from scanpy to filter peaks"
