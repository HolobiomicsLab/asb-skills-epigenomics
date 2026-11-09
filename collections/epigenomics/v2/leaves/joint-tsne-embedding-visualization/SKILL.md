---
name: joint-tsne-embedding-visualization
description: Use when a scATAC-seq embedding (e.g., SnapATAC t-SNE) must be overlaid
  with cell-type identities predicted by integrating a scRNA-seq reference, to visualize
  cluster separation or validate label transfer.
when_to_use_negative:
- The dataset has no matched scRNA-seq reference available for anchor-based label
  transfer, so no labels exist to overlay on the ATAC embedding
- The goal is quantitative comparison of embedding quality (e.g., connectivity-index
  benchmarking against LSA/cisTopic) rather than visual presentation — the t-SNE panel
  is a qualitative output
- The analysis requires scaling to ~a million cells, where t-SNE visualization becomes
  impractical even though SnapATAC itself scales via the ensemble Nyström method
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3308
tools:
- name: SnapATAC
  role: Provides runViz for the scATAC t-SNE embedding and plotViz for rendering and
    saving the joint RNA/ATAC panel; snapToSeurat exports the embedding to Seurat
    for label transfer
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: Preprocesses reads and builds the snap file (snap-pre, snap-add-bmat) that
    supplies the cell-by-bin matrix underlying the embedding
  repo: https://github.com/r3fang/SnapTools
- name: R
  role: Runtime environment for SnapATAC and Seurat CCA anchor transfer; must be >=
    3.4.0 and < 3.6.0
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:730d452733fead3858b46515f17fef7408bfe096334dd7d6befa3f61cd418379
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - R
  - SnapATAC
  - SnapTools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Joint t-SNE embedding visualization

## Summary

Projects cell-type labels transferred from a scRNA-seq reference onto the scATAC-seq t-SNE embedding computed by SnapATAC's runViz, producing a joint RNA/ATAC visualization panel (e.g., PBMC_ATAC_RNA.png). It demonstrates that genome-wide accessibility profiles resolve cell populations and enables visual assessment of label-transfer concordance.

## When to use

Use when a scATAC-seq embedding (e.g., SnapATAC t-SNE) must be overlaid with cell-type identities predicted by integrating a scRNA-seq reference, to visualize cluster separation or validate label transfer.

## When NOT to use

- The dataset has no matched scRNA-seq reference available for anchor-based label transfer, so no labels exist to overlay on the ATAC embedding
- The goal is quantitative comparison of embedding quality (e.g., connectivity-index benchmarking against LSA/cisTopic) rather than visual presentation — the t-SNE panel is a qualitative output
- The analysis requires scaling to ~a million cells, where t-SNE visualization becomes impractical even though SnapATAC itself scales via the ensemble Nyström method

## Inputs

- SnapATAC snap file (e.g., 10X_PBMC_15K) with cell-by-bin accessibility matrix added via addBmatToSnap
- 10x PBMC scRNA-seq reference dataset (~10k cells) for CCA-based anchor label transfer
- Seurat object created from the SnapATAC embedding via snapToSeurat

## Outputs

- t-SNE embedding of the scATAC-seq cells produced by runViz
- Joint RNA/ATAC visualization image (PBMC_ATAC_RNA.png) with transferred cell-type labels overlaid on the scATAC t-SNE

## How to apply

Run the SnapATAC pipeline (R >= 3.4.0 & < 3.6.0, since R 3.6 breaks the rhdf5 package) on the snap file — here the 10X_PBMC_15K dataset reduced to a 4,792-cell PBMC subset — computing the binary matrix with addBmatToSnap and makeBinary, then runDimReduct on the jaccard matrix and runCluster with the leiden algorithm. Export the SnapATAC object to Seurat with snapToSeurat and transfer cell-type labels from the ~10k-cell 10x PBMC scRNA-seq reference using Seurat's CCA-based anchor transfer. Compute the scATAC t-SNE embedding with runViz and color cells by the transferred RNA-derived labels. Save the joint panel as PBMC_ATAC_RNA.png with plotViz; the rationale is that SnapATAC resolves cellular heterogeneity by directly comparing genome-wide accessibility profiles between cells, so RNA-derived labels should segregate coherently on the ATAC embedding.

## Related tools

- **SnapATAC** (Provides runViz for the scATAC t-SNE embedding and plotViz for rendering and saving the joint RNA/ATAC panel; snapToSeurat exports the embedding to Seurat for label transfer) — https://github.com/r3fang/SnapATAC
- **SnapTools** (Preprocesses reads and builds the snap file (snap-pre, snap-add-bmat) that supplies the cell-by-bin matrix underlying the embedding) — https://github.com/r3fang/SnapTools
- **R** (Runtime environment for SnapATAC and Seurat CCA anchor transfer; must be >= 3.4.0 and < 3.6.0)

## Examples

```
R> plotViz(obj = pbmc.snap, method = "tsne", cell.groups = pbmc@metaData$predicted.celltype, prefix = "PBMC_ATAC_RNA")
```

## Evaluation signals

- The joint t-SNE figure (PBMC_ATAC_RNA.png) faithfully reproduces the published PBMC_ATAC_RNA panel from the 10X_PBMC_15K example
- Transferred RNA-derived labels form spatially coherent groups on the 4,792-cell scATAC t-SNE rather than scattering across clusters
- Expert review confirms the visualization matches the published panel for the 4,792-cell PBMC subset
- Cluster membership on the t-SNE is consistent with the leiden clustering run via runCluster on the jaccard-normalized embedding

## Limitations

- R 3.6 breaks the rhdf5 dependency, so the workflow must run under R >= 3.4.0 & < 3.6.0
- SnapTools is recommended for Python 2.7 (>= 2.7 & < 3.0) for the original r3fang SnapATAC; SnapATAC2 is the successor but is a separate Python/Rust package
- The t-SNE panel is qualitative; quantitative separation claims require a metric such as the connectivity index comparing SnapATAC vs LSA/cisTopic embeddings

## Evidence

- [methods] integration of scRNA and scATAC data: "SnapATAC integrates scRNA and scATAC"
- [intro] Genome-wide similarity basis for the embedding: "it resolves cellular heterogeneity by directly comparing the similarity in genome-wide accessibility profiles between cells."
- [readme] PBMC 15K example provides the scRNA/scATAC integration vignette: "[SnapATAC integrates scRNA and scATAC](https://github.com/r3fang/SnapATAC/blob/master/examples/10X_PBMC_15K/README.md)"
