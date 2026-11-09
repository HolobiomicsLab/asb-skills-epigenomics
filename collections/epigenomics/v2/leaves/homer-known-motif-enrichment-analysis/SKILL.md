---
name: homer-known-motif-enrichment-analysis
description: Use when scATAC-seq clusters (e.g., Leiden assignments from SCALE on
  a Breast_Tumor.h5ad dataset) need functional interpretation through motif enrichment
  in cluster-specific open chromatin peaks.
when_to_use_negative:
- The input is single-cell RNA-seq data rather than scATAC-seq peak accessibility
  — SCALE is designed for scATAC-seq sparsity and close-to-binary peak counts
- You need batch-effect removal as a primary goal — SCALE is not specifically designed
  to identify and remove batch effects from input data
- Peaks have not been filtered/assigned to clusters — running Homer on unclustered
  genome-wide peaks will not recover cell-type-specific motifs
edam_topics: []
tools:
- name: SCALE
  role: clusters scATAC-seq cells and yields the cluster assignments used to derive
    cluster-specific peak sets for motif enrichment
  repo: https://github.com/jsxlei/SCALE
- name: Homer
  role: performs known-motif enrichment (findMotifsGenome.pl) on each cluster-specific
    peak set
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:94a492516ef6ab94a02c70d1c7745bb4c3ee464d769597038f5080475d8a899d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - Homer
  - SCALE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# homer-known-motif-enrichment-analysis

## Summary

Runs Homer known-motif enrichment (findMotifsGenome.pl) on cell-type-specific peak sets derived from SCALE clustering of scATAC-seq data (e.g., top 1000 peaks per cluster by specificity score) to recover transcription factor motifs characteristic of each cluster.

## When to use

Use when scATAC-seq clusters (e.g., Leiden assignments from SCALE on a Breast_Tumor.h5ad dataset) need functional interpretation through motif enrichment in cluster-specific open chromatin peaks.

## When NOT to use

- The input is single-cell RNA-seq data rather than scATAC-seq peak accessibility — SCALE is designed for scATAC-seq sparsity and close-to-binary peak counts
- You need batch-effect removal as a primary goal — SCALE is not specifically designed to identify and remove batch effects from input data
- Peaks have not been filtered/assigned to clusters — running Homer on unclustered genome-wide peaks will not recover cell-type-specific motifs

## Inputs

- h5ad file with SCALE Leiden cluster assignments (Breast_Tumor.h5ad)
- per-cluster peak lists: top 1000 peaks by specificity score, chr_start_end format
- reference genome for Homer background matching

## Outputs

- Homer known-motif enrichment results per cluster (findMotifsGenome.pl output directories)
- compiled motif table: homer_known_motif_results.tsv
- interpretable cell-type-specific TF motif sets (e.g., immune vs epithelial motifs)

## How to apply

First cluster cells with SCALE (VAE with Gaussian Mixture Model prior) on the count matrix, saving the annotated AnnData with Leiden cluster assignments (adata.h5ad). Then compute per-cluster peak specificity scores and select the top 1000 most specific peaks for each cell-type cluster (e.g., CD45+ vs Epcam+). Run Homer's findMotifsGenome.pl separately on each cluster-specific peak set against the appropriate genome background to obtain known-motif enrichment results. Compile enriched motifs per cluster into a single table (homer_known_motif_results.tsv); validation checks that recovered motif sets match published cell-type-specific motifs, e.g., immune (CD45+) enriched for Maz, Pu.1-Irf, Irf8, Runx1, Elk4, Nfy, Elf3, and SpiB, and epithelial (Epcam+) enriched for Klf14, Mitf, Ets1, Nrf2, and Nrf1.

## Related tools

- **SCALE** (clusters scATAC-seq cells and yields the cluster assignments used to derive cluster-specific peak sets for motif enrichment) — https://github.com/jsxlei/SCALE
- **Homer** (performs known-motif enrichment (findMotifsGenome.pl) on each cluster-specific peak set)

## Examples

```
SCALE.py -d Breast_Tumor.h5ad; findMotifsGenome.pl cd45_top1000_peaks.txt hg38 cd45_motifs_out/
```

## Evaluation signals

- CD45+ (immune) cluster is enriched for the published immune motifs: Maz, Pu.1-Irf, Irf8, Runx1, Elk4, Nfy, Elf3, and SpiB
- Epcam+ (epithelial) cluster is enriched for the published tumor-related motifs: Klf14, Mitf, Ets1, Nrf2, and Nrf1
- Each cluster-specific peak set contains exactly the top 1000 peaks ranked by specificity score
- Scale outputs a valid adata.h5ad containing Leiden cluster assignments, latent feature matrix, and UMAP results before motif analysis
- Homer outputs are compiled into a complete homer_known_motif_results.tsv with enriched motifs per cluster

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data; explicit modeling of batch effects is deferred to future work
- Prior scATAC-seq methods suffer caveats such as missingness (a diploid cell has only one or two capture opportunities per site, yielding only a few thousand distinct reads per cell), so peak specificity estimates depend on adequate clustering quality
- Motif enrichment interpretation requires combining SCALE cluster-specific peak discovery with external tools like Homer or chromVAR

## Evidence

- [discussion] cluster-specific peak discovery enables motif analysis: "we can improve the discovery of cluster-specific peaks, and regulatory motifs as well, when combined with tools like Homer38 or chromVAR13."
- [readme] SCALE clustering outputs used for downstream analysis: "**adata.h5ad**:  saved data including Leiden cluster assignment, latent feature matrix and UMAP results."
- [abstract] SCALE rationale for scATAC-seq sparsity: "SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data."
- [methods] filtering options available in SCALE: "[--min_peaks]: filter low quality cells by valid peaks number, default 100"
