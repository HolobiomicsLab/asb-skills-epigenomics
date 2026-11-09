---
name: chipseq-peak-calling
description: Use when base-level ChIP-seq peak calling is needed for factorial-design
  experiments (IP vs control/genotype), especially on gigabase-scale genomes where
  memory-efficient, out-of-core GAM fitting is required.
when_to_use_negative:
- Data are not ChIP-seq read counts from a factorial design (IP vs control/genotype)
  modeled by smooth functions along chromosomes
- A quick region-level-only caller is desired without base-resolution significance
  testing or GAM smoothing-parameter control
- Environment runs R < 3.4.1 or lacks correct Armadillo/RcppArmadillo 64-bit-word
  configuration needed for large sparse matrices
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
tools:
- name: GenoGAM 2.0
  role: 'Core R/Bioconductor package: GenoGAMDataSet, computeSizeFactors, genogam,
    computeSignificance, callPeaks for genome-wide peak calling'
  repo: https://github.com/gstricker/fastGenoGAM
- name: SuperLU
  role: Direct sparse solver used for GAM parameter fitting, exploiting model sparsity
- name: HDF5
  role: Out-of-core on-disk storage of tiles and data, reducing memory footprint while
    keeping I/O low
- name: mgcv
  role: General-purpose R library underlying the GAM fitting framework
- name: R / Bioconductor
  role: Implementation and distribution platform for the GenoGAM package
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:62990e9aa763ec53a1586637eb8f6c684596663a74a6230387eb225277c069f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - Bioconductor
  - GenoGAM 2.0
  - HDF5
  - R
  - R / Bioconductor
  - SuperLU
  - mgcv
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 24
---

# chipseq-peak-calling

## Summary

Call significant ChIP-seq peaks using GenoGAM 2.0, a genome-wide generalized additive model approach that fits smooth functions along chromosomes on HDF5-backed tiles and provides base- and region-level significance testing without ad-hoc binning or windowing.

## When to use

Use when base-level ChIP-seq peak calling is needed for factorial-design experiments (IP vs control/genotype), especially on gigabase-scale genomes where memory-efficient, out-of-core GAM fitting is required.

## When NOT to use

- Data are not ChIP-seq read counts from a factorial design (IP vs control/genotype) modeled by smooth functions along chromosomes
- A quick region-level-only caller is desired without base-resolution significance testing or GAM smoothing-parameter control
- Environment runs R < 3.4.1 or lacks correct Armadillo/RcppArmadillo 64-bit-word configuration needed for large sparse matrices

## Inputs

- BAM files of ChIP-seq experiments (e.g. yeast TFIIB from package extdata/Set1, chromosome 22 CEBPB, CTCF, USF1, MAX, PAX5, YY1 ENCODE datasets)
- Experiment design file mapping samples to conditions (e.g. experimentDesign.txt with genotype columns)
- Motif center coordinates for peak-summit positioning benchmarks (optional, for summit evaluation)

## Outputs

- GenoGAM object with fitted smooths and standard errors (fits(result), se(result))
- Position-wise log p-values from computeSignificance()
- Significant peak regions from callPeaks()
- Motif-proximity summary table (e.g. motif_proximity_results.csv) with proportions and 95% bootstrap confidence intervals

## How to apply

Read each BAM-based ChIP-seq dataset into a GenoGAMDataSet with GenoGAMDataSet(), enabling the HDF5 backend, chunkSize = 50000, overhangSize = 1000, and the design ~ s(x) + s(x, by = genotype). Normalize with computeSizeFactors(), then fit the genome-wide GAM with genogam(); in the vignette-style benchmark, cross-validated smoothing-parameter estimation is skipped by fixing hyperparameters lambda = 4601 and theta = 4.51. Compute position-wise log p-values with computeSignificance() and extract significant regions with callPeaks(). Model sparsity is exploited via the SuperLU direct solver and data are kept on disk with HDF5, so whole-genome human fits (~300 million parameters) complete in under 9 hours on a 60-core server. For peak-summit benchmarks, annotate which significant peaks fall within 30 bp of the motif center and compute proportions with 95% bootstrap confidence intervals.

## Related tools

- **GenoGAM 2.0** (Core R/Bioconductor package: GenoGAMDataSet, computeSizeFactors, genogam, computeSignificance, callPeaks for genome-wide peak calling) — https://github.com/gstricker/fastGenoGAM
- **SuperLU** (Direct sparse solver used for GAM parameter fitting, exploiting model sparsity)
- **HDF5** (Out-of-core on-disk storage of tiles and data, reducing memory footprint while keeping I/O low)
- **mgcv** (General-purpose R library underlying the GAM fitting framework)
- **R / Bioconductor** (Implementation and distribution platform for the GenoGAM package)

## Examples

```
library(fastGenoGAM); ggd <- GenoGAMDataSet(expDesign, directory = folder, chunkSize = 50000, overhangSize = 1000, design = ~ s(x) + s(x, by = genotype), settings = settings, hdf5 = TRUE); ggd <- computeSizeFactors(ggd); result <- genogam(ggd, lambda = 4601, theta = 4.51)
```

## Evaluation signals

- Peak-summit benchmark: proportion of significant peaks within 30 bp of the motif center (with 95% bootstrap CI) must match GenoGAM 1.0 equivalence values on the six chr22 ENCODE TFs and yeast TFIIB
- Fitted values and standard errors retrievable via fits(result) and se(result) per chromosome/tile
- Position-wise log p-values present after computeSignificance(); pval(result) non-empty
- Runtime and memory footprint scale linearly with the number of parameters per tile; whole human-genome fits (<300M parameters) complete in under ~9 hours on a 60-core server
- Size factors computed for every dataset before fitting (computeSizeFactors runs without error)

## Limitations

- Requires R 3.4.1 or higher
- Out-of-bound range warnings during genome tiling cannot be suppressed in the parallel Snow environment
- Large-matrix errors can occur if RcppArmadillo is not built with ARMA_64BIT_WORD enabled; manually #define ARMA_64BIT_WORD 1 if needed
- fastGenoGAM repository version is deprecated in favor of the Bioconductor GenoGAM release / gstricker/GenoGAM
- Cross-validation of smoothing parameters is slow; the vignette/benchmark uses fixed lambda = 4601 and theta = 4.51 to save time

## Evidence

- [abstract] Here we present GenoGAM 2.0, a scalable and efficient implementation that is 2 to 3 orders of magnitude faster than the previous version.: "Here we present GenoGAM 2.0, a scalable and efficient implementation that is 2 to 3 orders of magnitude faster than the previous version."
- [abstract] This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
- [methods] Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object.: "Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object."
- [methods] compute position-wise log p-values with `computeSignificance()`: "compute position-wise log p-values with `computeSignificance()`"
- [methods] result <- genogam(ggd, lambda = 4601, theta = 4.51): "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [results] Whole-genome fits for human ChIP-seq datasets (ca. 300 million parameters) obtained in less than 9 hours on a standard 60-core server: [evidence span withheld — anchor status: not_found]
