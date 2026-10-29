---
name: scatac-seq-motif-activity-analysis
description: Use when sparse scATAC-seq peak-by-cell data (h5ad) needs transcription
  factor motif activity scoring, especially to test whether SCALE-imputed data recovers
  more cluster-specific motifs than the raw matrix.
when_to_use_negative:
- The goal is individual-peak resolution rather than grouped motif deviation scores
  — the article notes chromVAR 'only analyzes peaks in groups and lacks the resolution
  of individual peaks'
- Data are scRNA-seq rather than scATAC-seq chromatin accessibility peaks
- No imputed/denoised matrix is available and the analysis requires single-peak (not
  grouped) motif resolution
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0749
tools:
- name: SCALE
  role: Produces the imputed/denoised peak matrix (adata.obsm['imputed']) via --impute
    that improves chromVAR motif detection on sparse scATAC-seq data
  repo: https://github.com/jsxlei/SCALE
- name: chromVAR
  role: Computes per-cell motif deviation scores on groups of peaks sharing the same
    motifs, on both raw and SCALE-imputed matrices
- name: scanpy
  role: Loads the h5ad AnnData, provides highly_variable_genes peak filtering, and
    accesses adata.obsm['imputed']
  repo: https://scanpy.readthedocs.io/en/stable/index.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:655ac2da2056d147352711b2e1b884efa4db890528e1e32c7bf6100e7b942a14
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
yes_rate: 0.7142857142857143
n_claims: 7
---

# scatac-seq-motif-activity-analysis

## Summary

Run chromVAR motif deviation scoring on scATAC-seq peak-count matrices (raw and SCALE-imputed) to quantify per-cell transcription factor motif activity and identify significantly enriched motifs, e.g. recovering TF enrichments (Mafb, Hoxd9, Dlx2, Lhx8, Arx, Neurog1) that are missed on sparse raw data.

## When to use

Use when sparse scATAC-seq peak-by-cell data (h5ad) needs transcription factor motif activity scoring, especially to test whether SCALE-imputed data recovers more cluster-specific motifs than the raw matrix.

## When NOT to use

- The goal is individual-peak resolution rather than grouped motif deviation scores — the article notes chromVAR 'only analyzes peaks in groups and lacks the resolution of individual peaks'
- Data are scRNA-seq rather than scATAC-seq chromatin accessibility peaks
- No imputed/denoised matrix is available and the analysis requires single-peak (not grouped) motif resolution

## Inputs

- Forebrain.h5ad scATAC-seq peak-count AnnData (k=8, 2088 cells) from http://zhanglab.net/SCALE_SOURCE_DATA/Forebrain.h5ad
- SCALE-imputed numerical matrix stored in adata.obsm['imputed'] of adata.h5ad (from SCALE.py --impute)
- Differentially accessible peak set (e.g. 4100 sites) defining the motif-analysis peak set
- chromVAR motif deviation scoring framework with a motif PWM/annotation database

## Outputs

- chromvar_motif_activity_results.csv containing motif deviation scores and a raw-vs-imputed verification summary
- Count of significantly enriched motifs per condition (e.g. 52 raw vs 105 imputed)
- List of enriched transcription factor motifs per cluster (e.g. Mafb, Hoxd9 in the MG cluster only on imputed data)

## How to apply

Download the scATAC-seq dataset (e.g. Forebrain.h5ad, k=8 clusters, 2088 cells) and load the raw peak-count matrix plus the SCALE-imputed matrix stored in adata.obsm['imputed'] (produced by SCALE.py with --impute) using scanpy/anndata. Subset both matrices to the differentially accessible peak set (e.g. 4100 sites) that serves as input for motif analysis. Run chromVAR identically on raw and imputed matrices to compute per-cell motif deviation scores, then compare the number of significantly enriched motifs and associated TFs. Validation: accept the result if the significant-motif count increases (e.g. 52 on raw vs 105 on imputed) and key TF motifs (Mafb, Hoxd9, Dlx2, Lhx8, Arx, Neurog1) appear among the recovered enrichments. The rationale is that SCALE's VAE+Gaussian Mixture Model denoising reduces scATAC-seq 'missingness', improving downstream discovery of cluster-specific peaks and regulatory motifs when combined with chromVAR.

## Related tools

- **SCALE** (Produces the imputed/denoised peak matrix (adata.obsm['imputed']) via --impute that improves chromVAR motif detection on sparse scATAC-seq data) — https://github.com/jsxlei/SCALE
- **chromVAR** (Computes per-cell motif deviation scores on groups of peaks sharing the same motifs, on both raw and SCALE-imputed matrices)
- **scanpy** (Loads the h5ad AnnData, provides highly_variable_genes peak filtering, and accesses adata.obsm['imputed']) — https://scanpy.readthedocs.io/en/stable/index.html

## Examples

```
SCALE.py -d Forebrain.h5ad -k 8 --impute  # then run chromVAR on raw X and adata.obsm['imputed'] matrices subset to the 4100 differential peaks
```

## Evaluation signals

- Significant-motif count increases from 52 (raw) to 105 (SCALE-imputed) chromVAR motifs
- Mafb, Hoxd9, Dlx2, Lhx8, Arx, and Neurog1 motif enrichments are all recovered on the imputed data
- Mafb and Hoxd9 are enriched specifically in the MG cluster only on imputed data
- chromVAR was run identically on both matrices subset to the same 4100 differentially accessible peak set
- Output file chromvar_motif_activity_results.csv contains motif deviation scores plus the verification summary

## Limitations

- chromVAR only analyzes peaks in groups and lacks the resolution of individual peaks
- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data; explicit batch-effect modeling is deferred to future work
- Results depend on the imputation option: use --impute for numerical imputed data (adata.obsm['imputed']); --binary stores a binarized version instead

## Evidence

- [intro] chromVAR evaluates grouped motif peaks: "A recent method specifically developed for scATAC-seq data analysis, chromVAR13, evaluates groups of peaks that share the same motifs"
- [intro] chromVAR caveat on peak resolution: "each method suffers notable caveats: chromVAR only analyzes peaks in groups and lacks the resolution of individual peaks"
- [discussion] motif discovery improved with chromVAR: "we can improve the discovery of cluster-specific peaks, and regulatory motifs as well, when combined with tools like Homer38 or chromVAR13"
- [readme] SCALE --impute option: "get numerical imputed data in adata.h5ad file using scanpy **adata.obsm['imputed']** with option **--impute**"
- [discussion] SCALE covers denoising and imputation: "SCALE is thus a powerful tool for scATAC-seq data analysis, including data visualization, clustering, and denoising and imputation."
