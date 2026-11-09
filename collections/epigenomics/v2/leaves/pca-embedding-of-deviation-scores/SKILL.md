---
name: pca-embedding-of-deviation-scores
description: Use when you need a low-dimensional cell embedding from scATAC-seq accessibility
  counts for clustering and want to compare it against alternatives such as SnapATAC.
when_to_use_negative:
- The goal is the best-possible clustering or embedding of scATAC-seq cells — SnapATAC
  outperformed all chromVAR variants, including k-mer + PCA, on the evaluated clustering
  tasks.
- Input is already a dimensionality-reduced feature matrix or deviation-scores table
  rather than raw per-cell peak counts requiring filterSamples/filterPeaks preprocessing.
- You need TF motif usage annotation per cell or cluster rather than a clustering
  embedding — chromVAR motif deviations (matchMotifs) may be complementary for that
  purpose.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: chromVAR
  role: Computes bias-corrected k-mer deviation scores whose PCA yields the embedding;
    provides filterSamples, filterPeaks, matchKmers, and computeDeviations
  repo: https://github.com/GreenleafLab/chromVAR
- name: motifmatchr
  role: Companion package providing fast motif/k-mer matching used by chromVAR
  repo: https://github.com/GreenleafLab/motifmatchr
- name: SummarizedExperiment
  role: Data container holding the per-cell peak count matrix, colData (e.g., Cell_Type),
    and rowData bias annotations
- name: BiocParallel
  role: Registers the parallel backend (MulticoreParam/SnowParam/SerialParam) for
    chromVAR computations
- name: SnapATAC
  role: Baseline method run on the same filtered cells for the clustering comparison;
    outperforms chromVAR k-mer + PCA in the benchmark
  repo: https://github.com/r3fang/SnapATAC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nmeth.4401
    title: 'chromVAR: inferring transcription-factor-associated accessibility from
      single-cell epigenomic data'
  source_content_hash: sha256:fdd765fd496ad226f6ffae661306e12570f5a947f3c819bcb0ef20a8aad2d043
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GreenleafLab/chromVAR
  tools:
  - BiocParallel
  - R
  - SnapATAC
  - SummarizedExperiment
  - chromVAR
  - motifmatchr
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# pca-embedding-of-deviation-scores

## Summary

Embed single cells from sparse chromatin accessibility data by running PCA on the k-mer deviation-scores matrix produced by chromVAR's matchKmers(6)/computeDeviations pipeline; this k-mer + PCA variant is reported as the best chromVAR variant for clustering.

## When to use

Use when you need a low-dimensional cell embedding from scATAC-seq accessibility counts for clustering and want to compare it against alternatives such as SnapATAC.

## When NOT to use

- The goal is the best-possible clustering or embedding of scATAC-seq cells — SnapATAC outperformed all chromVAR variants, including k-mer + PCA, on the evaluated clustering tasks.
- Input is already a dimensionality-reduced feature matrix or deviation-scores table rather than raw per-cell peak counts requiring filterSamples/filterPeaks preprocessing.
- You need TF motif usage annotation per cell or cluster rather than a clustering embedding — chromVAR motif deviations (matchMotifs) may be complementary for that purpose.

## Inputs

- chromVAR SummarizedExperiment of fragment counts per cell over peaks (e.g., example_counts from the chromVAR package)
- Reference genome for k-mer matching (BSgenome.Hsapiens.UCSC.hg19)
- Known cell-type labels (e.g., Cell_Type in colData) for benchmarking cluster assignments

## Outputs

- PCA embedding of cells computed from the k-mer deviation-scores matrix
- Cluster assignments per cell (e.g., from k-means on the embedding)
- ARI benchmark table (e.g., clustering_ari_benchmark.csv) comparing chromVAR k-mer + PCA against SnapATAC

## How to apply

Load the chromVAR example_counts (or your own fragment counts) and preprocess with addGCBias, filterSamples(min_depth = 1500, min_in_peaks = 0.15), and filterPeaks(non_overlapping = TRUE) to remove low-quality cells and reduce peaks to non-overlapping ones. Build a 6-mer annotation matrix with matchKmers(6, counts_filtered, genome = BSgenome.Hsapiens.UCSC.hg19) and compute bias-corrected deviations with computeDeviations(counts_filtered, kmer_ix). Run PCA on the resulting deviation-scores matrix to obtain the k-mer + PCA embedding, which the comparative evaluation identified as the best chromVAR variant for clustering. Cluster the embedded cells (e.g., k-means) and, per the paper's benchmark, expect that newer methods such as SnapATAC may outperform this embedding on the same clustering tasks, so use it for clustering baselines or TF-motif annotation rather than assuming state-of-the-art clustering.

## Related tools

- **chromVAR** (Computes bias-corrected k-mer deviation scores whose PCA yields the embedding; provides filterSamples, filterPeaks, matchKmers, and computeDeviations) — https://github.com/GreenleafLab/chromVAR
- **motifmatchr** (Companion package providing fast motif/k-mer matching used by chromVAR) — https://github.com/GreenleafLab/motifmatchr
- **SummarizedExperiment** (Data container holding the per-cell peak count matrix, colData (e.g., Cell_Type), and rowData bias annotations)
- **BiocParallel** (Registers the parallel backend (MulticoreParam/SnowParam/SerialParam) for chromVAR computations)
- **SnapATAC** (Baseline method run on the same filtered cells for the clustering comparison; outperforms chromVAR k-mer + PCA in the benchmark) — https://github.com/r3fang/SnapATAC

## Examples

```
counts_filtered <- filterPeaks(filterSamples(example_counts, min_depth = 1500, min_in_peaks = 0.15)); kmer_ix <- matchKmers(6, counts_filtered, genome = BSgenome.Hsapiens.UCSC.hg19); dev <- computeDeviations(counts_filtered, kmer_ix)
```

## Evaluation signals

- The clustering benchmark table contains an ARI value for both chromVAR (k-mer + PCA) and SnapATAC, each within [-1, 1].
- The recorded ARI scores reproduce the reported finding that SnapATAC outperforms chromVAR for the evaluated clustering tasks.
- Preprocessing applied the specified thresholds: filterSamples with min_depth = 1500 and min_in_peaks = 0.15, followed by filterPeaks with non_overlapping = TRUE.
- The embedding is derived from the deviation-scores matrix of the 6-mer annotation matrix built with matchKmers(6, ...) on BSgenome.Hsapiens.UCSC.hg19.
- Cluster assignments can be compared against the known cell-type labels (Cell_Type in colData) via adjusted Rand index.

## Limitations

- SnapATAC outperforms chromVAR for the clustering tasks evaluated in the paper; chromVAR k-mer + PCA is only the best chromVAR variant, not the best overall method.
- chromVAR may be complementary rather than primary — the README suggests it as a way of annotating TF motif usage in cells and clusters rather than cluster identification or embedding.
- Installation and runtime caveats: a dependency requires the gsl system library, and BiocParallel backends differ by OS (MulticoreParam for Unix, SnowParam for Windows).
- Windows users have reported S4Vector dependency failures on R 3.3.3, and C++ compilation issues on Mac.

## Evidence
