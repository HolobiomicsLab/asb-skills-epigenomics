---
name: harmony-batch-effect-correction
description: Use when a scATAC-seq (or scRNA-seq) dataset combines cells from multiple
  samples/preparations and sample-to-sample technical variability could dominate the
  low-dimensional embedding, driving clustering to separate batches rather than cell
  types.
when_to_use_negative:
- The dataset comes from a single sample/library with no batch variable to correct
  — correction would remove genuine biological signal.
- Cells have not yet been embedded in low-dimensional space; Harmony should be applied
  to the embedding, not to the raw cell-by-bin accessibility matrix.
- Differences between sample groups are biological (e.g. distinct conditions you want
  to compare) rather than technical; correcting on that variable would erase the contrast
  of interest.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3611
tools:
- name: Harmony
  role: Batch-effect correction of the low-dimensional embedding between samples introduced
    by technical variability, applied via runHarmony before Leiden clustering
- name: SnapATAC
  role: Host pipeline providing runHarmony, runNormJaccard, runDimReduct, runCluster
    and runViz/plotViz on the snap object
  repo: https://github.com/r3fang/SnapATAC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:b209ea94d2f4d21237c51e12457280aa81c585d5c6d87ff7069c7342dcbc03e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - Harmony
  - SnapATAC
  - SnapTools
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3611
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# harmony-batch-effect-correction

## Summary

Corrects batch effects between samples introduced by technical variability in the low-dimensional embedding prior to clustering, as used within the SnapATAC pipeline on the mouse secondary motor cortex snATAC-seq atlas (55,592 nuclei).

## When to use

Use when a scATAC-seq (or scRNA-seq) dataset combines cells from multiple samples/preparations and sample-to-sample technical variability could dominate the low-dimensional embedding, driving clustering to separate batches rather than cell types.

## When NOT to use

- The dataset comes from a single sample/library with no batch variable to correct — correction would remove genuine biological signal.
- Cells have not yet been embedded in low-dimensional space; Harmony should be applied to the embedding, not to the raw cell-by-bin accessibility matrix.
- Differences between sample groups are biological (e.g. distinct conditions you want to compare) rather than technical; correcting on that variable would erase the contrast of interest.

## Inputs

- SnapATAC snap object with a low-dimensional embedding from runDimReduct (e.g. after runNormJaccard + ensemble Nyström dimensionality reduction)
- Sample/batch metadata table (meta_data) mapping each of the 55,592 nuclei to its source sample
- Name of the batch variable column (vars_use)

## Outputs

- Batch-corrected low-dimensional embedding stored in the snap object (eigs.dim dimensions)
- Visualization embedding (runViz/plotViz) showing cells mixed across samples within clusters
- Per-nucleus cluster labels from runCluster that are batch-informed rather than batch-driven, exported to cell_cluster_annotations.csv

## How to apply

After dimensionality reduction of the normalized Jaccard similarity matrix (runNormJaccard followed by runDimReduct with the ensemble Nyström method), apply runHarmony on the snap object with eigs.dim set to the number of embedding dimensions to retain, and pass the sample metadata (meta_data) and the batch variable column (vars_use) so Harmony mixes cells across samples while preserving cell-type structure. The rationale is that batch effects between samples are introduced by technical variability (library prep, sequencing runs) and can confound downstream Leiden clustering (runCluster with resolution and seed.use), so correction must happen on the embedding before graph construction. Inspect the runViz/plotViz embedding colored by sample before and after correction: cells from different samples should intermingle within cell-type clusters rather than form sample-separated lobes.

## Related tools

- **Harmony** (Batch-effect correction of the low-dimensional embedding between samples introduced by technical variability, applied via runHarmony before Leiden clustering)
- **SnapATAC** (Host pipeline providing runHarmony, runNormJaccard, runDimReduct, runCluster and runViz/plotViz on the snap object) — https://github.com/r3fang/SnapATAC

## Examples

```
runHarmony(obj, eigs.dim, meta_data, vars_use)
```

## Evaluation signals

- Post-correction runViz embedding shows cells from different samples intermixed within each of the 20 major clusters instead of sample-separated regions.
- Number of detected clusters (Leiden, fixed resolution and seed) reflects cell-type heterogeneity rather than splitting by sample.
- The embedding dimensionality passed via eigs.dim is preserved after correction, and Harmony converges on the supplied meta_data/vars_use without dropping cells.
- Marker-gene accessibility checks (plotGene) confirm cell-type annotations are consistent across samples within merged clusters.
- Final cell_cluster_annotations.csv covers the QC/doublet-filtered set of 55,592 nuclei with no cells lost to the correction step.

## Limitations

- Batch correction requires accurate sample metadata; mis-specified vars_use variables will merge biological signal across the wrong axes.
- Harmony is applied within SnapATAC to address technical variability between samples; it does not by itself guarantee that underrepresented rare cell populations are recovered after correction.
- Runs in the legacy SnapATAC environment (R >= 3.4.0 & < 3.6.0, since R 3.6 does not work for the rhdf5 package); SnapATAC2 users would need the successor package's own integration workflow.

## Evidence

- [intro] Harmony usage in SnapATAC pipeline: "SnapATAC uses Harmony24 to remove potential batch effect between samples introduced by technical variability (Methods section)."
- [methods] Batch effect correction as an enabled workflow step: "SnapATAC enables batch effect correction"
- [readme] Harmony batch correction tutorial in the snATAC example: "SnapATAC enables batch effect correction](https://github.com/r3fang/SnapATAC/blob/master/examples/10X_snATAC/README.md)"
- [discussion] Peak-based methods hinder multi-sample integrative analysis context: "peak-based methods hinder multi-sample integrative analysis where each sample has its own unique peak reference."
