---
name: scatac-scrna-integration-label-transfer
description: Use when scATAC-seq cells lack annotations and a matched or related scRNA-seq
  reference with known cell types is available, e.g. predicting PBMC cell types from
  a ~10k-cell scRNA reference.
when_to_use_negative:
- The scATAC and scRNA data come from unrelated tissues/species with no shared features,
  so CCA anchors cannot be meaningfully established
- The goal is de novo scATAC clustering without any annotated reference — use SnapATAC's
  own runDimReduct/runCluster workflow instead
- Working in R >= 3.6.0, where the rhdf5 dependency breaks the original SnapATAC (R
  must be >= 3.4.0 & < 3.6.0); consider SnapATAC2 in Python instead
edam_operation: http://edamontology.org/operation_3731
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2229
tools:
- name: SnapATAC
  role: Provides the scATAC dimensionality reduction (runDimReduct on jaccard matrix),
    leiden clustering (runCluster), and the snapToSeurat conversion that exports the
    embedding into Seurat
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: Preprocesses reads and creates/annotates the snap file (snap-pre, snap-add-bmat)
    that is the input object for the SnapATAC R workflow
  repo: https://github.com/r3fang/SnapTools
- name: Seurat
  role: Receives the snapToSeurat object and performs CCA-based anchor transfer of
    cell-type labels from the scRNA-seq reference to the scATAC cells
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:aace4320bd4b8420b2c27bc68949d689ba9508819b7a912a16613ff94c914a52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - R
  - Seurat
  - SnapATAC
  - SnapTools
  edam_operation: http://edamontology.org/operation_3731
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# scATAC-scRNA-integration-label-transfer

## Summary

Transfers cell-type labels from a scRNA-seq reference onto a scATAC-seq dataset by exporting a SnapATAC embedding into Seurat (snapToSeurat) and applying Seurat's CCA-based anchor label transfer, as demonstrated on the 10X PBMC scATAC/scRNA pair.

## When to use

Use when scATAC-seq cells lack annotations and a matched or related scRNA-seq reference with known cell types is available, e.g. predicting PBMC cell types from a ~10k-cell scRNA reference.

## When NOT to use

- The scATAC and scRNA data come from unrelated tissues/species with no shared features, so CCA anchors cannot be meaningfully established
- The goal is de novo scATAC clustering without any annotated reference — use SnapATAC's own runDimReduct/runCluster workflow instead
- Working in R >= 3.6.0, where the rhdf5 dependency breaks the original SnapATAC (R must be >= 3.4.0 & < 3.6.0); consider SnapATAC2 in Python instead

## Inputs

- SnapATAC snap file (e.g. 10X_PBMC_15K snap) with cell-by-bin accessibility matrix (AM session)
- SnapATAC R object after runDimReduct and runCluster
- Processed scRNA-seq reference Seurat object with cell-type annotations (~10k-cell PBMC reference)

## Outputs

- Seurat object containing the scATAC cells with the SnapATAC embedding (via snapToSeurat)
- Per-cell RNA-derived cell-type label predictions on the scATAC cells
- Joint t-SNE visualization with transferred labels (e.g. PBMC_ATAC_RNA.png produced by plotViz)

## How to apply

Load the snap file (e.g. 10X_PBMC_15K) into SnapATAC, compute the cell-by-bin matrix with addBmatToSnap and binarize with makeBinary, then reduce dimensionality with runDimReduct on the jaccard matrix and cluster with runCluster (leiden). Convert the SnapATAC object to a Seurat object with snapToSeurat so the SnapATAC embedding is preserved as the dimensionality reduction in Seurat. Run Seurat's CCA-based anchor-finding transfer from the scRNA-seq reference (here the ~10k-cell PBMC reference) to assign RNA-derived cell-type labels to each scATAC cell. Project the transferred labels onto the scATAC t-SNE produced by runViz and save the joint visualization (e.g. PBMC_ATAC_RNA.png) with plotViz; the transferred labels can then be used to score embedding quality such as connectivity index.

## Related tools

- **SnapATAC** (Provides the scATAC dimensionality reduction (runDimReduct on jaccard matrix), leiden clustering (runCluster), and the snapToSeurat conversion that exports the embedding into Seurat) — https://github.com/r3fang/SnapATAC
- **SnapTools** (Preprocesses reads and creates/annotates the snap file (snap-pre, snap-add-bmat) that is the input object for the SnapATAC R workflow) — https://github.com/r3fang/SnapTools
- **Seurat** (Receives the snapToSeurat object and performs CCA-based anchor transfer of cell-type labels from the scRNA-seq reference to the scATAC cells)

## Examples

```
seurat.obj <- snapToSeurat(snap.obj, tmp.folder, eigs.dims, eigs.comps); transfer.anchors <- FindTransferAnchors(reference=rna, query=seurat.obj, features=VariableFeatures(rna), reduction="cca"); predicted.labels <- TransferData(anchorset=transfer.anchors, refdata=rna$celltype)
```

## Evaluation signals

- Transferred RNA-derived labels, when plotted on the SnapATAC t-SNE via plotViz, form spatially coherent clusters matching the leiden clusters
- On the 4,792-cell PBMC subset, the SnapATAC embedding achieves a lower connectivity index (greater cluster separation) than LSA for the predicted cell populations
- snapToSeurat output preserves the SnapATAC embedding so label transfer runs on ATAC-derived low-dimensional space, not raw peaks
- Joint visualization (PBMC_ATAC_RNA.png) faithfully reproduces the published PBMC_ATAC_RNA panel
- Sanity check that environment matches R (>= 3.4.0 & < 3.6.0) since 3.6 breaks rhdf5

## Limitations

- R version constraint: R (>= 3.4.0 & < 3.6.0) is required because R 3.6 does not work with the rhdf5 package
- Label transfer quality depends on the scRNA reference covering the cell types present in the scATAC data; rare populations may be underrepresented
- The original SnapATAC is superseded by SnapATAC2 (kaizhang/SnapATAC2, scverse), so new projects may prefer the Python/Rust implementation

## Evidence

- [methods] integration of scRNA and scATAC data: "SnapATAC integrates scRNA and scATAC"
- [other] Export to Seurat and CCA label transfer: "SnapATAC links distal elements to putative target genes"
- [readme] R version constraint: "R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
- [other] Cluster separation finding on PBMC subset: [evidence span withheld — anchor status: not_found]
