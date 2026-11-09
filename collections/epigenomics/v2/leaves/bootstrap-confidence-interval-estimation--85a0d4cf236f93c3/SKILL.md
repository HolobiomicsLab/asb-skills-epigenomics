---
name: bootstrap-confidence-interval-estimation
description: Use when a benchmark metric is a proportion derived from a finite set
  of significant peaks (e.g., motif-proximal peak fraction) and its sampling uncertainty
  must be reported as a 95% bootstrap confidence interval for method comparison.
when_to_use_negative:
- When the comparison target is a per-position quantity (fits, standard errors, log
  p-values) rather than a summary statistic over a finite peak set — use plotGenoGAM()/pval(result)
  inspection instead
- When no motif center annotations are available for the factor, since the 30 bp motif-proximity
  proportion cannot be defined
- When the goal is runtime or memory benchmarking of GenoGAM 2.0 rather than peak-summit
  positioning quality
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
tools:
- name: GenoGAM 2.0
  role: R/Bioconductor package used to fit the genome-wide GAM, compute position-wise
    log p-values and call significant peaks whose motif proximity is bootstrapped
  repo: https://github.com/gstricker/fastGenoGAM
- name: R
  role: Language in which the bootstrap resampling and proportion/CI computation are
    implemented
- name: HDF5
  role: Backend used to store the tiled GenoGAMDataSet on disk while fitting, keeping
    memory footprint low
- name: mgcv
  role: Underlying generalized additive model fitting library used by genogam()
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:f760fd7c4f3172e2e0f0b90bd716303ac24d5b9803c60c25a6992b53f654b0fe
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# bootstrap-confidence-interval-estimation

## Summary

Quantifies uncertainty in derived ChIP-Seq peak statistics by resampling significant peaks and computing 95% bootstrap confidence intervals, as used in the GenoGAM 2.0 peak-summit positioning benchmark (Fig. 5c) to report, per transcription factor, the proportion of significant peaks within 30 bp of the motif center.

## When to use

Use when a benchmark metric is a proportion derived from a finite set of significant peaks (e.g., motif-proximal peak fraction) and its sampling uncertainty must be reported as a 95% bootstrap confidence interval for method comparison.

## When NOT to use

- When the comparison target is a per-position quantity (fits, standard errors, log p-values) rather than a summary statistic over a finite peak set — use plotGenoGAM()/pval(result) inspection instead
- When no motif center annotations are available for the factor, since the 30 bp motif-proximity proportion cannot be defined
- When the goal is runtime or memory benchmarking of GenoGAM 2.0 rather than peak-summit positioning quality

## Inputs

- Significant peak regions (GRanges) from callPeaks() on a fitted GenoGAM object
- Motif center coordinates for each transcription factor (CEBPB, CTCF, USF1, MAX, PAX5, YY1, yeast TFIIB)
- BAM files referenced by the experiment design (yeast TFIIB from extdata/Set1; chromosome 22 ENCODE ChIP-Seq experiments)

## Outputs

- Per-dataset proportion of significant peaks within 30 bp of the motif center
- 95% bootstrap confidence interval for each proportion
- Seven-dataset summary table written to motif_proximity_results.csv

## How to apply

First obtain the set of significant regions for each dataset by fitting GenoGAM on a GenoGAMDataSet built with chunkSize = 50000, overhangSize = 1000 and design ~ s(x) + s(x, by = genotype), computing size factors with computeSizeFactors(), fitting with genogam() using fixed hyperparameters lambda = 4601 and theta = 4.51, then running computeSignificance() and callPeaks(). For each transcription factor (CEBPB, CTCF, USF1, MAX, PAX5, YY1 on chromosome 22, plus the yeast TFIIB dataset from package extdata/Set1), annotate which significant peaks fall within 30 bp of the corresponding motif center. Compute the proportion of motif-proximal significant peaks per dataset, then generate bootstrap resamples of the significant peaks to derive a 95% confidence interval around that proportion. The interval allows an equivalence-style comparison against the values reported for GenoGAM 1.0, establishing that summit-inference performance matched between versions.

## Related tools

- **GenoGAM 2.0** (R/Bioconductor package used to fit the genome-wide GAM, compute position-wise log p-values and call significant peaks whose motif proximity is bootstrapped) — https://github.com/gstricker/fastGenoGAM
- **R** (Language in which the bootstrap resampling and proportion/CI computation are implemented)
- **HDF5** (Backend used to store the tiled GenoGAMDataSet on disk while fitting, keeping memory footprint low)
- **mgcv** (Underlying generalized additive model fitting library used by genogam())

## Examples

```
ggd <- GenoGAMDataSet(expDesign, directory = folder, chunkSize = 50000, overhangSize = 1000, design = ~ s(x) + s(x, by = genotype), settings = settings, hdf5 = TRUE); ggd <- computeSizeFactors(ggd); result <- genogam(ggd, lambda = 4601, theta = 4.51)
```

## Evaluation signals

- The seven-row summary table (six ENCODE TFs on chromosome 22 plus yeast TFIIB) is written to motif_proximity_results.csv with proportion and CI columns populated
- Every reported 95% bootstrap interval contains the point estimate of the motif-proximal peak proportion for that dataset
- Proportions and intervals fall within the equivalence values reported for GenoGAM 1.0, demonstrating matched peak summit inference performance
- Peak calling upstream is reproducible: computeSignificance() yields position-wise log p-values and callPeaks() returns significant regions consistent with the fitted model
- All significant peaks evaluated lie within 30 bp or beyond the motif center with a documented annotation, so the proportion denominator equals the number of significant peaks called

## Limitations

- Requires R 3.4.1 or higher, constraining compatible environments
- Fixed hyperparameters (lambda = 4601, theta = 4.51) skip cross-validated smoothing-parameter estimation, so results depend on these vignette values rather than data-driven estimation
- The package's example dataset is restricted to the first 100kb of the first yeast chromosome, so the yeast TFIIB arm of the benchmark must use appropriate data; software is licensed under GPL-2 and no Zenodo archival record is provided

## Evidence

- [other] Benchmark metric definition: [evidence span withheld — anchor status: not_found]
- [other] Benchmark replication finding: [evidence span withheld — anchor status: not_found]
- [readme] Peak-calling workflow upstream of the bootstrap: "compute position-wise log p-values with `computeSignificance()`"
- [readme] Fixed hyperparameters used before computing the benchmark proportions: "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [abstract] Rationale for HDF5-backed data handling during fitting: "the HDF5 library is employed to store data efficiently on hard drive, reducing memory footprint while keeping I/O low."
