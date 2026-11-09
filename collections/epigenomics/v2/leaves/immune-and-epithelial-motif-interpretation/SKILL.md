---
name: immune-and-epithelial-motif-interpretation
description: Use when you have cluster assignments from scATAC-seq data (e.g. SCALE
  Leiden clusters on an h5ad like Breast_Tumor.h5ad) and need to assign biological
  identity to clusters via enriched TF binding motifs in type-specific peaks.
when_to_use_negative:
- Input is already a motif enrichment table or feature matrix rather than raw peak-by-cell
  scATAC-seq counts needing clustering
- The goal is single-batch scRNA-seq expression analysis, since SCALE/Homer motif
  interpretation requires open-chromatin peak data
- You need batch-effect removal as the primary objective — the article states SCALE
  'is not specifically designed to identify and remove these artifacts from the input
  data'
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0749
tools:
- name: SCALE
  role: Clusters scATAC-seq cells (VAE + Gaussian Mixture Model prior) to define cell-type
    clusters whose specific peaks feed motif enrichment
  repo: https://github.com/jsxlei/SCALE
- name: Homer
  role: Runs known-motif enrichment (findMotifsGenome.pl) on each cluster-specific
    peak set to recover TF binding motifs
- name: chromVAR
  role: Alternative/complementary tool for regulatory motif activity on groups of
    peaks sharing motifs, mentioned alongside Homer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:63962bf58388262762c23d9a078465bf2c069d6cedfebedb274aaa3a59da539d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - Homer
  - SCALE
  - chromVAR
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# immune-and-epithelial-motif-interpretation

## Summary

Interprets cell-type identity from scATAC-seq clusters by running Homer known-motif enrichment on cluster-specific peaks, recovering immune-specific motifs (Maz, Pu.1-Irf, Irf8, Runx1, Elk4, Nfy, Elf3, SpiB) in CD45+ cells and tumor-related motifs (Klf14, Mitf, Ets1, Nrf2, Nrf1) in Epcam+ cells.

## When to use

Use when you have cluster assignments from scATAC-seq data (e.g. SCALE Leiden clusters on an h5ad like Breast_Tumor.h5ad) and need to assign biological identity to clusters via enriched TF binding motifs in type-specific peaks.

## When NOT to use

- Input is already a motif enrichment table or feature matrix rather than raw peak-by-cell scATAC-seq counts needing clustering
- The goal is single-batch scRNA-seq expression analysis, since SCALE/Homer motif interpretation requires open-chromatin peak data
- You need batch-effect removal as the primary objective — the article states SCALE 'is not specifically designed to identify and remove these artifacts from the input data'

## Inputs

- SCALE output adata.h5ad containing count matrix, Leiden cluster assignment, latent feature matrix and UMAP results (e.g. from Breast_Tumor.h5ad)
- Genome reference appropriate to the sample for Homer background (findMotifsGenome.pl)
- Per-cluster peak coordinates (chr_start_end format) ranked by specificity score

## Outputs

- Cluster-specific peak sets (top 1000 peaks by specificity score per cluster)
- Homer known-motif enrichment results per cluster
- Compiled motif table (homer_known_motif_results.tsv) listing enriched motifs for CD45+ and Epcam+ clusters

## How to apply

Run SCALE (VAE with a Gaussian Mixture Model prior) on the scATAC-seq count matrix — e.g. Breast_Tumor.h5ad downloaded from the SCALE source-data site — and save the annotated AnnData with Leiden cluster assignments (adata.h5ad output). Compute per-cluster peak specificity scores and select the top 1000 most specific peaks for each cell-type cluster (here CD45+ immune and Epcam+ epithelial clusters). Run Homer known-motif enrichment (findMotifsGenome.pl) separately on each cluster-specific peak set against the appropriate genome background. Compile enriched motifs per cluster into a single table (homer_known_motif_results.tsv) and interpret immune vs. epithelial identity from the recovered motif sets; the article notes cluster-specific peak discovery improves 'when combined with tools like Homer or chromVAR'.

## Related tools

- **SCALE** (Clusters scATAC-seq cells (VAE + Gaussian Mixture Model prior) to define cell-type clusters whose specific peaks feed motif enrichment) — https://github.com/jsxlei/SCALE
- **Homer** (Runs known-motif enrichment (findMotifsGenome.pl) on each cluster-specific peak set to recover TF binding motifs)
- **chromVAR** (Alternative/complementary tool for regulatory motif activity on groups of peaks sharing motifs, mentioned alongside Homer)

## Examples

```
SCALE.py -d Breast_Tumor.h5ad  # then: findMotifsGenome.pl cluster_top1000_peaks.txt <genome> -size given
```

## Evaluation signals

- CD45+ (immune) cluster motif set matches the published enrichment: Maz, Pu.1-Irf, Irf8, Runx1, Elk4, Nfy, Elf3, SpiB
- Epcam+ (epithelial/tumor) cluster motif set matches: Klf14, Mitf, Ets1, Nrf2, Nrf1
- Each cluster's peak set contains exactly the top 1000 peaks ranked by specificity score
- adata.h5ad output contains Leiden cluster assignment, latent feature matrix and UMAP results as specified by the SCALE README
- homer_known_motif_results.tsv contains enrichment entries for both clusters compiled into one table

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data; explicit batch modeling is deferred to future work
- Existing scATAC-seq methods have caveats the motif step inherits: chromVAR only analyzes peaks in groups and lacks individual peak resolution, whereas scABC depends on landmark samples with high sequencing depths
- scATAC-seq data suffers from 'missingness' and high dimensionality (only a few thousand distinct reads per cell), so direct application of scRNA-seq methods to the peak matrix is unsuitable

## Evidence

- [discussion] Homer on cluster-specific peaks: "we can improve the discovery of cluster-specific peaks, and regulatory motifs as well, when combined with tools like Homer38 or chromVAR13."
- [abstract] SCALE GMM prior: "SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data."
- [methods] Breast_Tumor dataset: "[Breast_Tumor](http://zhanglab.net/SCALE_SOURCE_DATA/Breast_Tumor.h5ad)"
- [readme] SCALE output adata.h5ad: "**adata.h5ad**:  saved data including Leiden cluster assignment, latent feature matrix and UMAP results."
- [readme] peak file format: "**peak file**: 1-column of peaks **chr_start_end**, filename contains key word **"peak"**"
