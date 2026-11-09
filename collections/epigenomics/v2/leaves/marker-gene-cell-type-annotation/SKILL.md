---
name: marker-gene-cell-type-annotation
description: Use when an scATAC-seq/snap object has been clustered (e.g. Leiden clusters
  in SnapATAC) and each numeric cluster needs a cell-type label based on marker-gene
  accessibility or motif activity.
when_to_use_negative:
- Clusters have not yet been defined — annotation requires runCluster output first;
  do not annotate unclustered barcodes
- No prior biological knowledge of marker genes or reference labels exists for the
  tissue — the method cannot assign de novo identities
- Data are already a peak/gene feature table with annotations (e.g. an annotated h5ad)
  — re-deriving labels from accessibility is unnecessary
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3170
tools:
- name: SnapATAC
  role: plotGene/plotViz visualize marker-gene accessibility per cluster and export
    per-nucleus cell-type annotations from the snap object
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: maintains the snap file (GM/PM sessions) that supplies the cell-by-gene matrix
    used for marker inspection
  repo: https://github.com/r3fang/SnapTools
- name: chromVAR
  role: computes motif activity deviations to infer candidate cell-type specific transcriptional
    regulators for annotation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:cc6aef08ace739f37d2922a1322c6ac064ae79775cc8858960cddc5587a5fee0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - SnapATAC
  - SnapTools
  - chromVAR
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# marker-gene-cell-type-annotation

## Summary

Annotates post-clustering single-nucleus ATAC-seq clusters to biological cell types by inspecting accessibility at marker-gene loci (plotGene) and, optionally, motif activity via chromVAR, then exporting per-nucleus cell-type labels.

## When to use

Use when an scATAC-seq/snap object has been clustered (e.g. Leiden clusters in SnapATAC) and each numeric cluster needs a cell-type label based on marker-gene accessibility or motif activity.

## When NOT to use

- Clusters have not yet been defined — annotation requires runCluster output first; do not annotate unclustered barcodes
- No prior biological knowledge of marker genes or reference labels exists for the tissue — the method cannot assign de novo identities
- Data are already a peak/gene feature table with annotations (e.g. an annotated h5ad) — re-deriving labels from accessibility is unnecessary

## Inputs

- Clustered snap object (snap file v4 with BD, AM, GM sessions) with cluster assignments from runCluster
- Cell-by-gene count matrix (GM session) or cell-by-peak matrix (PM) for marker accessibility
- Reference genome annotation (e.g. mm10) for locating marker-gene loci
- Optional scRNA-seq reference labels for cross-modality validation

## Outputs

- Cluster-to-cell-type annotation table (e.g. cell_cluster_annotations.csv)
- Per-cluster marker-gene accessibility plots (plotGene / plotViz output)
- chromVAR motif deviation scores supporting regulator assignments

## How to apply

After runDimReduct and runCluster have produced clusters on the snap object (e.g. 20–31 clusters in the mouse secondary motor cortex atlas), visualize the embedding with runViz/plotViz and query marker-gene accessibility per cluster using plotGene on the cell-by-gene matrix (GM) stored in the snap file. Assign each cluster a cell-type identity by checking that canonical marker loci show accessible chromatin in that cluster relative to others; use chromVAR motif deviation scores as complementary evidence since SnapATAC enables motif analysis using chromVAR. Cross-reference cluster annotations with scRNA-seq labels, as SnapATAC integrates scRNA and scATAC data. Finally export the per-nucleus cluster labels and cell-type annotations to a table such as cell_cluster_annotations.csv.

## Related tools

- **SnapATAC** (plotGene/plotViz visualize marker-gene accessibility per cluster and export per-nucleus cell-type annotations from the snap object) — https://github.com/r3fang/SnapATAC
- **SnapTools** (maintains the snap file (GM/PM sessions) that supplies the cell-by-gene matrix used for marker inspection) — https://github.com/r3fang/SnapTools
- **chromVAR** (computes motif activity deviations to infer candidate cell-type specific transcriptional regulators for annotation)

## Examples

```
plotGene(obj, gene.name="Slc17a7"); write.csv(obj@cluster, "cell_cluster_annotations.csv")
```

## Evaluation signals

- Each annotated cell type shows enriched accessibility at its canonical marker loci in plotGene profiles relative to other clusters
- Number of annotated cell types matches the Leiden cluster count (e.g. 20 major clusters in the mouse secondary motor cortex atlas; up to 31 distinct cell populations)
- Exported cell_cluster_annotations.csv has one annotation per nucleus, with labels consistent across nuclei of the same cluster
- chromVAR motif deviations point to transcriptional regulators consistent with the assigned cell types (candidate cell-type specific transcriptional regulators inferred)
- Cross-modality check: cluster labels agree with integrated scRNA-seq annotations

## Limitations

- Cluster annotation depends on marker-gene availability and known biology; rare populations may lack canonical markers and remain ambiguous
- Iterative clustering can underrepresent peaks in rare cell populations, which weakens marker-based annotation for those clusters
- Functional validation of additional open chromatin regions identified in rare cell types is still needed; authors state further experiments are required

## Evidence

- [methods] Annotate the clusters by marker-gene accessibility with plotGene: [evidence span withheld — anchor status: not_found]
- [methods] SnapATAC enables motif analysis using chromVAR: "SnapATAC enables motif analysis using chromVAR"
- [methods] SnapATAC integrates scRNA and scATAC: "SnapATAC integrates scRNA and scATAC"
- [abstract] analysis reveals ~370,000 candidate regulatory elements in 31 distinct cell populations: "The analysis reveals ~370,000 candidate regulatory elements in 31 distinct cell populations in this brain region"
