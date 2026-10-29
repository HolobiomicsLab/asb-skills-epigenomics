---
name: jaccard-similarity-dimensionality-reduction
description: 'Use when clustering sparse single-cell ATAC-seq profiles without population-level
  peak calls: raw cell-by-bin (5kb/10kb) accessibility matrices or .snap AM matrices
  need a cell-level embedding before Leiden clustering.'
when_to_use_negative:
- Input is already a cell-by-peak feature matrix from population-level peak calling
  — SnapATAC's Jaccard approach is designed to avoid predefined peak references, which
  bias clustering toward abundant cell types and hinder multi-sample integration
- Data are scRNA-seq rather than single-cell ATAC-seq accessibility profiles — use
  standard expression-based embeddings instead
- Dataset is so large that even the Nyström-approximated pairwise similarity is impractical
  and SnapATAC2 (scverse) is available instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: SnapATAC
  role: Computes the normalized Jaccard similarity matrix (runNormJaccard) and the
    low-dimensional embedding (runDimReduct, ensemble Nyström) from the binarized
    genome-wide bin matrix
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: Pre-processes reads and populates the .snap file's cell-by-bin count matrix
    (snap-add-bmat) that serves as the Jaccard input
  repo: https://github.com/r3fang/SnapTools
- name: Harmony
  role: Corrects batch effects between samples in the SnapATAC embedding (runHarmony)
    before clustering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:e3ecd684f1fe0e9eeeb63b63f2350db0f938860d8aa83564196162a0c3d2c4f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - Harmony
  - SnapATAC
  - SnapTools
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6190476190476191
n_claims: 21
---

# jaccard-similarity-dimensionality-reduction

## Summary

SnapATAC's core representation step: compute a normalized Jaccard similarity matrix between cells directly on genome-wide binary accessibility bins (no peak annotation needed), then embed cells in low-dimensional space via spectral/diffusion embedding accelerated with the ensemble Nyström method.

## When to use

Use when clustering sparse single-cell ATAC-seq profiles without population-level peak calls: raw cell-by-bin (5kb/10kb) accessibility matrices or .snap AM matrices need a cell-level embedding before Leiden clustering.

## When NOT to use

- Input is already a cell-by-peak feature matrix from population-level peak calling — SnapATAC's Jaccard approach is designed to avoid predefined peak references, which bias clustering toward abundant cell types and hinder multi-sample integration
- Data are scRNA-seq rather than single-cell ATAC-seq accessibility profiles — use standard expression-based embeddings instead
- Dataset is so large that even the Nyström-approximated pairwise similarity is impractical and SnapATAC2 (scverse) is available instead

## Inputs

- Snap object (.snap HDF5, version 4) containing the cell-by-bin matrix (AM session) at e.g. 5kb/10kb resolution
- Cell-by-bin accessibility count matrix added by addBmatToSnap/snaptools snap-add-bmat
- QC/doublet-filtered barcode set from filterCells (subset.names, low.thresholds, high.thresholds)

## Outputs

- Cell-by-cell normalized Jaccard similarity matrix
- Low-dimensional embedding of cells (spectral/diffusion components) suitable for runCluster (Leiden) and runViz visualization
- Batch-corrected embedding if runHarmony is applied downstream

## How to apply

First binarize genome-wide accessibility with makeBinary on the bin count matrix added via addBmatToSnap (e.g., bin.size 5000 or 10000), then compute the normalized Jaccard similarity between all cell pairs with runNormJaccard (row.center=TRUE, row.scale=TRUE, chunked over ncell.chunk cells with a fixed seed.use to keep memory bounded). Because the pairwise Jaccard matrix scales quadratically, use the ensemble Nyström method in runDimReduct to approximate the spectral embedding, which SnapATAC reports scales to ~1 million cells. On multi-sample datasets, follow the embedding with batch-effect correction (runHarmony) before Leiden clustering (runCluster), since technical variability between samples otherwise dominates low-dimensional structure. The rationale is that comparing genome-wide accessibility similarity between cells avoids the bias of peak-based methods toward abundant cell types, which underrepresent rare populations' cis-regulatory elements.

## Related tools

- **SnapATAC** (Computes the normalized Jaccard similarity matrix (runNormJaccard) and the low-dimensional embedding (runDimReduct, ensemble Nyström) from the binarized genome-wide bin matrix) — https://github.com/r3fang/SnapATAC
- **SnapTools** (Pre-processes reads and populates the .snap file's cell-by-bin count matrix (snap-add-bmat) that serves as the Jaccard input) — https://github.com/r3fang/SnapTools
- **Harmony** (Corrects batch effects between samples in the SnapATAC embedding (runHarmony) before clustering)

## Examples

```
obj <- runNormJaccard(obj, tmp.folder=tmp.dir, row.center=TRUE, row.scale=TRUE, ncell.chunk=1000, num.cores=8, seed.use=10); obj <- runDimReduct(obj, eigs.dims=2:50, method="ood")
```

## Evaluation signals

- The .snap QC report shows usable unique fragments (UQ) after MAPQ>30 / proper-pair / max-flen 1000bp filtering, so Jaccard is computed on clean cell barcodes
- The Jaccard matrix is computed on binarized bins (makeBinary outlier.filter applied), not raw counts, and row.center/row.scale normalization was enabled
- The resulting embedding yields clusters resolved by Leiden that can be annotated by marker-gene accessibility (plotGene); in the mouse secondary motor cortex atlas this recovered the expected major cell-type structure (20 major clusters; 31 distinct cell populations with ~370,000 candidate regulatory elements in the full analysis)
- Chunked computation with a fixed seed.use reproduces the same embedding across runs
- After runHarmony on multi-sample data, samples no longer segregate by batch in the embedding

## Limitations

- Iterative peak-based re-clustering remains problematic: crude clusters represent dominant cell types so rare populations' peaks may be underrepresented — prefer the direct genome-wide Jaccard representation for rare cell types
- Pairwise Jaccard computation is memory- and compute-intensive for very large datasets, requiring the ensemble Nyström approximation and ncell.chunk chunking to tmp.folder
- Environment constraints from the original implementation: SnapTools with Python 2.7 (SnapATAC2 supersedes this) and R >= 3.4.0 & < 3.6.0 (R 3.6 breaks the rhdf5 package)

## Evidence

- [methods] runNormJaccard(obj, tmp.folder, method, row.center, row.scale, low.threshold, high.threshold, do.par, ncell.chunk, num.cores, seed.use): "runNormJaccard(obj, tmp.folder, method, row.center, row.scale, low.threshold, high.threshold, do.par, ncell.chunk, num.cores, seed.use)"
- [intro] it resolves cellular heterogeneity by directly comparing the similarity in genome-wide accessibility profiles between cells. We also adopt a technique, ensemble Nyström method: "it resolves cellular heterogeneity by directly comparing the similarity in genome-wide accessibility profiles between cells. We also adopt a technique, ensemble Nyström method"
- [abstract] Using the Nyström method, SnapATAC can process data from up to a million cells.: "Using the Nyström method, SnapATAC can process data from up to a million cells."
- [intro] that does not require population-level peak annotation prior to clustering: "that does not require population-level peak annotation prior to clustering"
- [readme] The cell-by-bin matrices will be added to `demo.snap` without creating another file.: [evidence span withheld — anchor status: external_doc]
