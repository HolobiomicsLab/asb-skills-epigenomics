---
name: cluster-specific-peak-ranking
description: Use when you have scATAC-seq cluster assignments (e.g., SCALE Leiden
  clusters saved in adata.h5ad from a dataset like Breast_Tumor.h5ad) and need to
  extract the peaks most specific to each cell-type cluster (e.g., CD45+ vs Epcam+)
  to feed into motif enrichment with Homer.
when_to_use_negative:
- Input is already a motif enrichment table or curated feature set — peak ranking
  and Homer enrichment are redundant at that stage.
- Data lacks cell-type clusters (e.g., bulk ATAC-seq or a single homogeneous population)
  — there is no contrast against which to compute cluster-specific peak specificity.
- 'The goal is explicit batch-effect correction: SCALE is not specifically designed
  to identify and remove batch effects or other artifacts from the input data.'
- You need per-peak motif activity rather than cluster-specific peak sets — chromVAR
  analyzes groups of peaks sharing motifs but lacks individual peak resolution, a
  different analysis mode.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SCALE
  role: Clusters scATAC-seq cells via VAE with Gaussian Mixture Model prior and outputs
    adata.h5ad with Leiden cluster assignment, providing the cell-type labels used
    to compute per-cluster peak specificity scores.
  repo: https://github.com/jsxlei/SCALE
- name: Homer
  role: Runs known-motif enrichment (findMotifsGenome.pl) on each cluster-specific
    top-1000 peak set against the appropriate genome background to identify cell-type-specific
    regulatory motifs.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:37bbc6495915168273b0257f22d76a3a2a377148416b27f1b2f236306f0bdcb6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - Homer
  - SCALE
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cluster-specific-peak-ranking

## Summary

Rank open-chromatin peaks by per-cluster specificity after SCALE (VAE + Gaussian Mixture Model) clustering of scATAC-seq data, and select the top 1000 most specific peaks per cell-type cluster as input for Homer known-motif enrichment. This links SCALE's cluster-specific peak discovery to regulatory motif interpretation.

## When to use

Use when you have scATAC-seq cluster assignments (e.g., SCALE Leiden clusters saved in adata.h5ad from a dataset like Breast_Tumor.h5ad) and need to extract the peaks most specific to each cell-type cluster (e.g., CD45+ vs Epcam+) to feed into motif enrichment with Homer.

## When NOT to use

- Input is already a motif enrichment table or curated feature set — peak ranking and Homer enrichment are redundant at that stage.
- Data lacks cell-type clusters (e.g., bulk ATAC-seq or a single homogeneous population) — there is no contrast against which to compute cluster-specific peak specificity.
- The goal is explicit batch-effect correction: SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data.
- You need per-peak motif activity rather than cluster-specific peak sets — chromVAR analyzes groups of peaks sharing motifs but lacks individual peak resolution, a different analysis mode.

## Inputs

- Peak-by-barcode scATAC-seq count matrix (h5ad file, txt/tsv/csv, or mtx folder containing count, peak (chr_start_end), and barcode files)
- Cluster/cell-type assignments from SCALE (adata.h5ad with Leiden cluster assignment)
- Reference genome build for Homer background (findMotifsGenome.pl)

## Outputs

- Per-cluster ranked peak lists (top 1000 specificity-scored peaks per cluster, chr_start_end format)
- Homer known-motif enrichment results per cluster
- Compiled motif enrichment table (homer_known_motif_results.tsv)

## How to apply

Run SCALE on the peak-by-barcode count matrix (h5ad, txt/tsv/csv, or mtx folder with count/peak/barcode files) to obtain Leiden cluster assignments saved in adata.h5ad. Apply SCALE's default quality filters before clustering (min_peaks=100 valid peaks per cell, min_cells=3 valid cells per peak) and optionally restrict to highly variable features (n_feature, default 100,000). For each cell-type cluster, compute a specificity score per peak (how selectively accessible that peak is in the cluster relative to others) and keep the top 1000 most specific peaks per cluster. Run Homer known-motif enrichment (findMotifsGenome.pl) separately on each cluster-specific peak set against the appropriate genome background, then compile enriched motifs per cluster into a single table (homer_known_motif_results.tsv). The rationale is that SCALE's latent-feature clustering improves discovery of cluster-specific peaks, which the paper notes becomes more informative for regulatory motifs when combined with Homer or chromVAR.

## Related tools

- **SCALE** (Clusters scATAC-seq cells via VAE with Gaussian Mixture Model prior and outputs adata.h5ad with Leiden cluster assignment, providing the cell-type labels used to compute per-cluster peak specificity scores.) — https://github.com/jsxlei/SCALE
- **Homer** (Runs known-motif enrichment (findMotifsGenome.pl) on each cluster-specific top-1000 peak set against the appropriate genome background to identify cell-type-specific regulatory motifs.)

## Examples

```
SCALE.py -d Breast_Tumor.h5ad; # then rank peaks by per-cluster specificity and run: findMotifsGenome.pl cluster1_top1000.peaks hg38 -bg background.peaks
```

## Evaluation signals

- Each cluster-specific peak set contains at most 1000 peaks, ranked by specificity score, and the union of sets is drawn from the filtered peak matrix (min_peaks=100, min_cells=3).
- Motif sets match published cell-type-specific expectations: immune CD45+ cluster enriched for Maz, Pu.1-Irf, Irf8, Runx1, Elk4, Nfy, Elf3, and SpiB; epithelial Epcam+ cluster enriched for Klf14, Mitf, Ets1, Nrf2, and Nrf1.
- adata.h5ad output exists and contains Leiden cluster assignment, latent feature matrix, and UMAP results, confirming the clustering step that feeds the ranking.
- homer_known_motif_results.tsv is populated for every cluster with non-empty enrichment results from findMotifsGenome.pl runs on the correct genome background.

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data, so confounded clusters can distort specificity scores.
- scATAC-seq data suffer from the curse of 'missingness' and high dimensionality (only a few thousand distinct reads per cell), which makes per-peak specificity estimates noisy.
- The top-1000-peaks-per-cluster cutoff is a heuristic choice; results depend on this threshold and on the Homer genome background used.
-  chromVAR, an alternative for motif analysis, only analyzes peaks in groups and lacks the resolution of individual peaks.

## Evidence

- [discussion] Cluster-specific peak discovery is the stated bridge to motif analysis: "we can improve the discovery of cluster-specific peaks, and regulatory motifs as well, when combined with tools like Homer38 or chromVAR13."
- [readme] SCALE output provides the cluster assignments used for ranking: "adata.h5ad:  saved data including Leiden cluster assignment, latent feature matrix and UMAP results."
- [methods] Default quality filters applied before clustering: "[--min_peaks]: filter low quality cells by valid peaks number, default 100"
- [methods] Feature filtering option for peak selection: "[--n_feature]: filter peaks by selecting highly variable features, default 100,000"
- [discussion] SCALE's analytical scope motivating cluster-level peak discovery: "SCALE is thus a powerful tool for scATAC-seq data analysis, including data visualization, clustering, and denoising and imputation."
