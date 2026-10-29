---
name: scatac-accessibility-matrix-processing
description: 'Use when a scATAC-seq accessibility matrix clusters primarily by copy
  number state and you need to test whether CNVs confound clustering: drop CNV-chromosome
  peaks, recluster, and compare assignments via AMI.'
when_to_use_negative:
- Clustering is already based on motif activity scores (e.g., ChromVAR), where CNV-driven
  effects are less pronounced
- Tumors driven primarily by single nucleotide variants with no or rare CNVs, where
  Copy-scAT cannot infer meaningful CNVs
- You need a definitive correction for CNV effects rather than a diagnostic of confounding
  — the optimal correction approach is undetermined
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Copy-scAT
  role: CNV inference and clustering workflow (identifyCNVClusters) applied to the
    scATAC accessibility matrix
  repo: github.com/spcdot/CopyscAT
- name: CopyscAT
  role: R package providing the scDataSamp/scDataSampClusters datasets and CNV-calling
    functions (annotateCNV3/4/4B, clusterCNV, filterCells)
  repo: github.com/spcdot/CopyscAT
- name: fastcluster
  role: Hierarchical clustering dependency used for reclustering the CNV-free peak
    matrix
- name: ChromVAR
  role: Alternative motif-activity-based clustering route that reduces CNV-driven
    clustering effects
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:b2c147931612ac2e17f806111834fc379021e5cfe3e4904813de794abab5f20d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChromVAR
  - Copy-scAT
  - CopyscAT
  - R
  - fastcluster
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# scatac-accessibility-matrix-processing

## Summary

A Copy-scAT (CopyscAT R package) workflow that removes peaks mapping to CNV-bearing chromosomes from a single-cell ATAC-seq accessibility matrix, reclusters cells on the CNV-free peak matrix (identifyCNVClusters + fastcluster hierarchical clustering), and quantifies how much clustering was CNV-driven by computing adjusted mutual information (AMI) against the original assignments.

## When to use

Use when a scATAC-seq accessibility matrix clusters primarily by copy number state and you need to test whether CNVs confound clustering: drop CNV-chromosome peaks, recluster, and compare assignments via AMI.

## When NOT to use

- Clustering is already based on motif activity scores (e.g., ChromVAR), where CNV-driven effects are less pronounced
- Tumors driven primarily by single nucleotide variants with no or rare CNVs, where Copy-scAT cannot infer meaningful CNVs
- You need a definitive correction for CNV effects rather than a diagnostic of confounding — the optimal correction approach is undetermined

## Inputs

- Single-cell ATAC-seq accessibility matrix (peak x cell) — e.g. scDataSamp from CopyscAT
- Original cell cluster assignments — e.g. scDataSampClusters from CopyscAT
- Per-chromosome CNV calls / predicted CNV-bearing chromosome list from Copy-scAT
- Fragment matrix (optional upstream input, generated with process_fragment_file.py)

## Outputs

- CNV-free accessibility matrix (peaks on CNV-bearing chromosomes removed)
- Post-removal cluster assignments from identifyCNVClusters/fastcluster reclustering
- Adjusted mutual information (AMI) between original and post-removal clusters (reference: 0.634, Fig. 4I)
- CSV of AMI value, reference value, and cluster labels (e.g. ami_cnv_confounder.csv)

## How to apply

Load the scDataSamp accessibility matrix and scDataSampClusters original cluster labels from the CopyscAT package, then obtain per-chromosome CNV calls from the Copy-scAT CNV inference (annotateCNV3/annotateCNV4/annotateCNV4B, clusterCNV with medianIQR/maxClust/minDiff controls). Delete all peaks mapping to chromosomes predicted to harbor CNVs so only CNV-free peaks remain, then rerun the Copy-scAT clustering workflow (identifyCNVClusters with default parameters, hierarchical clustering via fastcluster) on the filtered matrix. Finally, compute adjusted mutual information between the original and post-removal cluster assignments over the shared set of cells; the reference analysis reported AMI = 0.634, indicating moderately concordant but more granular clusters, so clustering in scATAC is only partly driven by CNV signal. Rationale: cells sharing a CNV tend to cluster together, so removing CNV-bearing peaks tests how much of the accessibility structure reflects copy number rather than other epigenetic differences.

## Related tools

- **Copy-scAT** (CNV inference and clustering workflow (identifyCNVClusters) applied to the scATAC accessibility matrix) — github.com/spcdot/CopyscAT
- **CopyscAT** (R package providing the scDataSamp/scDataSampClusters datasets and CNV-calling functions (annotateCNV3/4/4B, clusterCNV, filterCells)) — github.com/spcdot/CopyscAT
- **fastcluster** (Hierarchical clustering dependency used for reclustering the CNV-free peak matrix)
- **ChromVAR** (Alternative motif-activity-based clustering route that reduces CNV-driven clustering effects)

## Examples

```
Rscript -e 'library(CopyscAT); data(scDataSamp); data(scDataSampClusters); cnv <- identifyCNVClusters(scDataSamp); clusters <- cutree(hclust(fastcluster::hclust.dist(cnvmat)), k=maxClust)'; write.csv(ami_cnv_confounder.csv)
```

## Evaluation signals

- Computed AMI between original and post-removal cluster assignments reproduces the reported 0.634 (Fig. 4I) within reasonable tolerance
- Post-removal clustering is more granular than the original while remaining broadly stable across the shared cell set
- The filtered matrix contains no peaks assigned to chromosomes predicted to harbor CNVs by the Copy-scAT CNV calls
- AMI is computed on the same shared set of cells in both clusterings, not on disjoint subsets

## Limitations

- Copy-scAT cannot be used with malignancies driven primarily by single nucleotide variants and having no or rare CNVs
- Further studies are needed to identify the optimal way to address the effects of CNVs in downstream analyses of scATAC datasets
- Whether CNVs prime cells to acquire specific chromatin accessibility states or stabilize existing states of genetic subclones remains undetermined
- CopyscAT is early beta; documentation and testing are incomplete, and development requires an R 4.0.0 environment

## Evidence

- [intro] intro: "cells that share a given CNV tend to cluster together in scATAC experiments, suggesting that genetics may impart information that results in the emergence of specific epigenetic profiles"
- [methods] methods: "clusterCNV(initialResultList, medianIQR, maxClust, minDiff)"
- [discussion] discussion: "Further studies are needed to identify the optimal way to address the effects of CNVs in downstream analyses of scATAC datasets"
- [readme] readme: "CopyscAT is designed to identify large-scale and local alterations in chromosomes without need for a control file."
