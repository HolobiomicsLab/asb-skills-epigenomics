---
name: motif-center-proximity-statistics
description: Use when ChIP-Seq peaks (called with base-level significance) must be
  validated against known transcription-factor binding motifs, e.g. measuring how
  many significant peaks center within 30 bp of the motif center to compare summit-inference
  accuracy across methods or versions.
when_to_use_negative:
- When no reference motif center annotations exist for the factor — the 30 bp proximity
  metric cannot be computed
- For differential occupancy testing rather than summit-positioning accuracy benchmarking
  — GenoGAM's design ~ s(x) + s(x, by = genotype) serves that goal instead
- When the goal is runtime/memory benchmarking of GenoGAM 2.0 rather than peak quality
  comparison
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_0203
tools:
- name: GenoGAM 2.0
  role: Reads data via GenoGAMDataSet, fits the genome-wide GAM and calls significant
    peaks whose summit positions are scored against motif centers
  repo: https://github.com/gstricker/fastGenoGAM
- name: R
  role: Host language of the GenoGAM package and the bootstrap/summary scripting for
    the benchmark
- name: Bioconductor
  role: Distribution infrastructure for the GenoGAM release used in the pipeline
- name: HDF5
  role: Out-of-core storage backend enabling genome-scale dataset processing with
    low memory footprint
- name: mgcv
  role: General-purpose R library underlying the GAM fitting used by GenoGAM
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:a091d63c6163208f7b64f5a498730059462c64d5d590cbe657d96a2050b5bfcb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - Bioconductor
  - GenoGAM 2.0
  - HDF5
  - R
  - mgcv
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# motif-center-proximity-statistics

## Summary

Benchmark ChIP-Seq peak-summit inference by computing, per dataset, the proportion of GenoGAM-called significant peaks that fall within 30 bp of a known motif center, with 95% bootstrap confidence intervals, as used in the GenoGAM 2.0 Fig. 5c replication of the GenoGAM 1.0 summit-positioning benchmark.

## When to use

Use when ChIP-Seq peaks (called with base-level significance) must be validated against known transcription-factor binding motifs, e.g. measuring how many significant peaks center within 30 bp of the motif center to compare summit-inference accuracy across methods or versions.

## When NOT to use

- When no reference motif center annotations exist for the factor — the 30 bp proximity metric cannot be computed
- For differential occupancy testing rather than summit-positioning accuracy benchmarking — GenoGAM's design ~ s(x) + s(x, by = genotype) serves that goal instead
- When the goal is runtime/memory benchmarking of GenoGAM 2.0 rather than peak quality comparison

## Inputs

- ChIP-seq BAM files with an experimentDesign.txt (e.g. yeast TFIIB extdata/Set1; chr22 CEBPB, CTCF, USF1, MAX, PAX5, YY1 ENCODE experiments)
- Known motif center positions for each transcription factor
- GenoGAM parameters: chunkSize = 50000, overhangSize = 1000, design ~ s(x) + s(x, by = genotype), fixed lambda = 4601 and theta = 4.51
- Proximity window threshold (30 bp)

## Outputs

- Per-dataset proportion of significant peaks within 30 bp of the motif center
- 95% bootstrap confidence intervals for each proportion
- Seven-dataset summary table written to motif_proximity_results.csv

## How to apply

Process each ChIP-seq dataset (yeast TFIIB from package extdata/Set1, plus chromosome 22 CEBPB, CTCF, USF1, MAX, PAX5 and YY1) through the GenoGAM pipeline: build a GenoGAMDataSet with HDF5 backend, chunkSize = 50000, overhangSize = 1000 and design ~ s(x) + s(x, by = genotype); normalize with computeSizeFactors(); fit with genogam() using fixed hyperparameters lambda = 4601 and theta = 4.51 to skip cross-validated smoothing estimation; then compute position-wise log p-values with computeSignificance() and extract significant regions with callPeaks(). For each transcription factor, annotate which significant peaks lie within 30 bp of the corresponding motif center, compute the per-dataset proportion of motif-proximal peaks, and obtain a 95% bootstrap confidence interval. Correct application is judged by whether the resulting seven-dataset proportions and intervals match (are on par with) the equivalence values reported for GenoGAM 1.0.

## Related tools

- **GenoGAM 2.0** (Reads data via GenoGAMDataSet, fits the genome-wide GAM and calls significant peaks whose summit positions are scored against motif centers) — https://github.com/gstricker/fastGenoGAM
- **R** (Host language of the GenoGAM package and the bootstrap/summary scripting for the benchmark)
- **Bioconductor** (Distribution infrastructure for the GenoGAM release used in the pipeline)
- **HDF5** (Out-of-core storage backend enabling genome-scale dataset processing with low memory footprint)
- **mgcv** (General-purpose R library underlying the GAM fitting used by GenoGAM)

## Examples

```
ggd <- GenoGAMDataSet(expDesign, directory = folder, chunkSize = 50000, overhangSize = 1000, design = ~ s(x) + s(x, by = genotype), hdf5 = TRUE); ggd <- computeSizeFactors(ggd); result <- genogam(ggd, lambda = 4601, theta = 4.51); computeSignificance(result)
```

## Evaluation signals

- Per-dataset proportion of motif-proximal significant peaks (within 30 bp of motif center) reproduces the GenoGAM 1.0 equivalence values
- 95% bootstrap confidence intervals for the proportions overlap/match the previously reported intervals
- motif_proximity_results.csv contains rows for all seven datasets (6 ENCODE TFs on chr22 + yeast TFIIB)
- callPeaks() returns significant regions only after computeSignificance() and genogam() completed with the fixed hyperparameters (lambda = 4601, theta = 4.51)
- GenoGAM summit inference is on par with alternative methods as in the original benchmark

## Limitations

- Performance depends on hardware; whole-genome human fits (~300 million parameters) took less than 9 hours on a 60-core server, so smaller setups may be slower
- RcppArmadillo must use ARMA_64BIT_WORD to avoid matrix-dimension errors on large fits
- Out-of-bound range warnings during genome tiling (GenoGAMDataSet construction) are expected and can be ignored
- Benchmark restricted to six ENCODE transcription factors on chromosome 22 and a small yeast TFIIB example (first 100kb of yeast chromosome I in the package dataset)

## Evidence

- [methods] Fixed hyperparameters skip CV: "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [methods] Workflow steps: "compute position-wise log p-values with `computeSignificance()`"
- [readme] README quick guide: "Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object. This is done with the HDF5 backend."
- [abstract] Speed of new implementation: "Here we present GenoGAM 2.0, a scalable and efficient implementation that is 2 to 3 orders of magnitude faster than the previous version."
- [results] Benchmark finding: [evidence span withheld — anchor status: not_found]
