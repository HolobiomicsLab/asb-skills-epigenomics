---
name: differentially-accessible-site-subsetting
description: 'Use when comparing motif activity between raw and imputed scATAC-seq
  matrices: the differentially accessible sites (e.g. 4100 peaks) must define the
  peak set for chromVAR on both matrices.'
when_to_use_negative:
- When the raw and imputed matrices would be analyzed on different peak sets — the
  comparison of significant motif counts is only valid if both are subset to the same
  sites
- When the goal is SCALE clustering or UMAP embedding on all peaks rather than motif
  scoring on a differential peak set
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0769
tools:
- name: SCALE
  role: Produces the imputed matrix (adata.obsm['imputed'] via --impute) that is subset
    alongside the raw matrix to the differential peak set
  repo: https://github.com/jsxlei/SCALE
- name: scanpy
  role: Loads the h5ad files and provides highly_variable_genes-based peak filtering
    used to define features
- name: chromVAR
  role: Consumes the subset raw and imputed matrices to compute per-cell motif deviation
    scores
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:e7f8a98be37f67dee31a7e8051701481b2bdb4ae3e6c327d16690216e19bfa2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - SCALE
  - chromVAR
  - scanpy
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# differentially-accessible-site-subsetting

## Summary

Subsets both the raw peak-count matrix and the SCALE-imputed matrix (adata.obsm['imputed']) to a defined set of differentially accessible sites so that downstream chromVAR motif deviation scoring operates on a shared, comparable peak set.

## When to use

Use when comparing motif activity between raw and imputed scATAC-seq matrices: the differentially accessible sites (e.g. 4100 peaks) must define the peak set for chromVAR on both matrices.

## When NOT to use

- When the raw and imputed matrices would be analyzed on different peak sets — the comparison of significant motif counts is only valid if both are subset to the same sites
- When the goal is SCALE clustering or UMAP embedding on all peaks rather than motif scoring on a differential peak set

## Inputs

- Forebrain.h5ad raw scATAC-seq peak-count matrix (2088 cells, k=8 clusters) from http://zhanglab.net/SCALE_SOURCE_DATA/Forebrain.h5ad
- SCALE-imputed matrix stored as adata.obsm['imputed'] in adata.h5ad (from SCALE.py --impute)
- Set of 4100 differentially accessible sites serving as the peak set

## Outputs

- Raw matrix subset to the differentially accessible peak set (input to chromVAR)
- Imputed matrix subset to the same differentially accessible peak set (input to chromVAR)
- chromvar_motif_activity_results.csv with motif deviation scores and verification summary

## How to apply

Load the Forebrain.h5ad dataset (k=8 clusters, 2088 cells) with scanpy/anndata and access the SCALE-imputed values from adata.obsm['imputed'], produced via the SCALE --impute option. Define or obtain the set of differentially accessible sites (here 4100 sites) that serve as the peak set for motif analysis. Subset both the raw peak-count matrix and the imputed matrix to exactly these sites, keeping the peak identifiers (chr_start_end) and cell barcodes aligned across the two matrices. Run chromVAR identically on the raw and imputed subsets so any difference in significant motif counts (52 raw vs 105 imputed) reflects imputation rather than peak-set differences. Write motif deviation scores and the verification summary to chromvar_motif_activity_results.csv.

## Related tools

- **SCALE** (Produces the imputed matrix (adata.obsm['imputed'] via --impute) that is subset alongside the raw matrix to the differential peak set) — https://github.com/jsxlei/SCALE
- **scanpy** (Loads the h5ad files and provides highly_variable_genes-based peak filtering used to define features)
- **chromVAR** (Consumes the subset raw and imputed matrices to compute per-cell motif deviation scores)

## Examples

```
SCALE.py -d Forebrain.h5ad --impute -k 8; # then in Python: import scanpy as sc; adata = sc.read_h5ad('adata.h5ad'); raw_sub = adata.X[peak_mask]; imp_sub = adata.obsm['imputed'][peak_mask]
```

## Evaluation signals

- The raw and imputed subsets contain the same 4100 differentially accessible sites with aligned peak identifiers
- chromVAR on the raw subset yields 52 significant motifs while the imputed subset yields 105
- The Mafb, Hoxd9, Dlx2, Lhx8, Arx, and Neurog1 motif enrichments are recovered on the imputed data
- Mafb and Hoxd9 are enriched in the MG cluster only on the imputed data
- chromvar_motif_activity_results.csv contains per-cell motif deviation scores for both conditions

## Limitations

- chromVAR only analyzes peaks in groups and lacks the resolution of individual peaks
- The imputed values depend on SCALE's denoising step; a different --impute run or n_feature/min_cells filtering settings change which sites are available for subsetting

## Evidence

- [other] Subset both the raw and imputed matrices to the 4100 differentially accessible sites that serve as the peak set for the motif analysis.: [evidence span withheld — anchor status: not_found]
- [readme] SCALE.py --impute produces imputed data in adata.h5ad as adata.obsm['imputed']: "or get numerical imputed data in adata.h5ad file using scanpy **adata.obsm['imputed']** with option **--impute**"
- [discussion] chromVAR combined with SCALE improves discovery of cluster-specific peaks and regulatory motifs: "we can improve the discovery of cluster-specific peaks, and regulatory motifs as well, when combined with tools like Homer38 or chromVAR13."
- [methods] Introduce the highly_variable_genes from scanpy to filter peaks: "Introduce the highly_variable_genes from scanpy to filter peaks"
