---
name: cca-based-cross-modality-annotation
description: Use when a scATAC-seq clustering lacks cell-type identities and a matched
  or same-tissue scRNA-seq reference with curated labels is available; triggered by
  the need to color scATAC clusters (e.g. on a t-SNE embedding) with RNA-derived cell
  types.
when_to_use_negative:
- No matched scRNA-seq reference with trusted cell-type labels exists for the tissue,
  leaving nothing for the CCA anchor transfer to align against
- The scATAC-seq data has not yet been dimensionality-reduced and clustered in SnapATAC
  — snapToSeurat expects the SnapATAC embedding and cluster structure to already exist
- Working in R >= 3.6.0, where the rhdf5 dependency breaks and SnapATAC (v1) cannot
  load snap files
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2229
tools:
- name: SnapATAC
  role: Clusters the scATAC-seq cells and exposes the snapToSeurat function that exports
    the SnapATAC embedding into a Seurat object for label transfer
  repo: https://github.com/r3fang/SnapATAC
- name: Seurat
  role: Performs the CCA-based anchor transfer of cell-type labels from the scRNA-seq
    reference onto the scATAC-seq cells
- name: SnapTools
  role: Preprocessing module that builds the snap file (cell-by-bin accessibility
    matrix) consumed by SnapATAC before annotation
  repo: https://github.com/r3fang/SnapTools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:3247b3d3ee690e0d95e55590b0eb39cd3b6965d2cf04b9d625bf532a8d5506d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - R
  - Seurat
  - SnapATAC
  - SnapTools
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cca-based-cross-modality-annotation

## Summary

Transfers cell-type labels from a scRNA-seq reference onto a scATAC-seq dataset by exporting the SnapATAC embedding to Seurat and using CCA-based anchor integration, as demonstrated on the 4,792-cell PBMC scATAC subset annotated from the ~10k-cell PBMC scRNA-seq reference.

## When to use

Use when a scATAC-seq clustering lacks cell-type identities and a matched or same-tissue scRNA-seq reference with curated labels is available; triggered by the need to color scATAC clusters (e.g. on a t-SNE embedding) with RNA-derived cell types.

## When NOT to use

- No matched scRNA-seq reference with trusted cell-type labels exists for the tissue, leaving nothing for the CCA anchor transfer to align against
- The scATAC-seq data has not yet been dimensionality-reduced and clustered in SnapATAC — snapToSeurat expects the SnapATAC embedding and cluster structure to already exist
- Working in R >= 3.6.0, where the rhdf5 dependency breaks and SnapATAC (v1) cannot load snap files

## Inputs

- Clustered SnapATAC snap object (cell-by-bin matrix, jaccard-normalized embedding, leiden clusters) for the scATAC-seq cells (e.g. 4,792-cell 10X_PBMC_15K subset)
- Processed scRNA-seq reference Seurat object with curated cell-type labels (e.g. ~10k-cell PBMC scRNA-seq reference)
- R (>= 3.4.0 & < 3.6.0) environment with SnapATAC and Seurat installed

## Outputs

- Per-cell predicted cell-type labels for each scATAC-seq cell/barcode
- Joint scATAC/scRNA visualization (e.g. PBMC_ATAC_RNA.png) showing RNA-derived labels projected onto the scATAC t-SNE embedding via plotViz
- Labeled Seurat object containing the SnapATAC embedding and transferred identities

## How to apply

First cluster the scATAC-seq data in SnapATAC (runDimReduct on the jaccard matrix, runCluster with the leiden algorithm), then convert the snap object to a Seurat object with snapToSeurat so the SnapATAC embedding is carried into R's Seurat framework. Next, run Seurat's CCA-based anchor transfer to project cell-type labels from the scRNA-seq reference (e.g. the ~10k-cell PBMC scRNA-seq reference) onto the scATAC cells, since CCA anchors align gene activity/accessibility features across modalities despite their sparsity. Finally, plot the transferred RNA-derived labels on the scATAC t-SNE embedding produced by runViz (e.g. saving PBMC_ATAC_RNA.png with plotViz) so label quality can be visually checked against cluster boundaries. The rationale is that scATAC-seq profiles have only 5–15% of peaks detected per cell, making direct annotation unreliable, whereas the scRNA reference provides interpretable cell-type definitions.

## Related tools

- **SnapATAC** (Clusters the scATAC-seq cells and exposes the snapToSeurat function that exports the SnapATAC embedding into a Seurat object for label transfer) — https://github.com/r3fang/SnapATAC
- **Seurat** (Performs the CCA-based anchor transfer of cell-type labels from the scRNA-seq reference onto the scATAC-seq cells)
- **SnapTools** (Preprocessing module that builds the snap file (cell-by-bin accessibility matrix) consumed by SnapATAC before annotation) — https://github.com/r3fang/SnapTools

## Examples

```
seurat_obj <- snapToSeurat(snap_obj, tmp.folder, eigs.dims, eigs.std); anchors <- FindTransferAnchors(rna_ref, seurat_obj, dims = 1:30); labels <- TransferData(anchorset = anchors, refdata = rna_ref$celltype)
```

## Evaluation signals

- Transferred labels show concordant, spatially coherent populations on the runViz t-SNE embedding — each leiden cluster receives a consistent predicted cell type
- The joint visualization (e.g. PBMC_ATAC_RNA.png) faithfully reproduces the published PBMC_ATAC_RNA panel, per the expert_review validation of this task
- Downstream comparison is meaningful: labeled populations can be separated in the SnapATAC embedding (lower connectivity index) versus LSA, confirming the annotation supports cluster-separation assessment
- snapToSeurat output retains the SnapATAC embedding dimensions, so Seurat's UMAP/t-SNE coordinates match the original runViz projection

## Limitations

- Requires R (>= 3.4.0 & < 3.6.0); R 3.6 does not work for the rhdf5 package that SnapATAC depends on
- SnapATAC (v1) is superseded by SnapATAC2 (scverse), so this R/Seurat-based transfer path applies to the legacy Python-2.7-compatible SnapTools / R 3.4–3.5 stack
- Transferred labels are predictions from RNA integration ('cell types predicted via integration'), not ground truth; label confidence depends on reference quality and cross-modality alignment
- Sparse scATAC-seq signal (5–15% of peaks detected per cell) limits the feature space available for cross-modality anchoring

## Evidence

- [methods] SnapATAC integrates scRNA and scATAC: "SnapATAC integrates scRNA and scATAC"
- [readme] Export embedding to Seurat and transfer labels: "SnapATAC links distal elements to putative target genes"
- [readme] R version constraint: "R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
- [intro] Sparsity motivation for scATAC clustering methods: "The growing volume of scATAC-seq datasets coupled with the sparsity of signals in each individual profile due to low detection efficiency (5–15% of peaks detected per cell)7 present a unique"
- [intro] SnapATAC does not require prior peak annotation: "we introduce a software package, Single Nucleus Analysis Pipeline for ATAC-seq—SnapATAC (https://github.com/r3fang/SnapATAC)—that does not require population-level peak annotation prior to clustering"
