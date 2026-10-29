---
name: runtime-memory-measurement-in-r
description: Use when benchmarking runtime and memory footprint of an R model-fitting
  routine (e.g. genogam() GAM fits) at increasing parameter counts, holding hyperparameters
  fixed and forcing single-worker execution.
when_to_use_negative:
- The goal is to evaluate smoothing-parameter estimation rather than coefficient estimation
  — fixed lambda = 4601 and theta = 4.51 bypass hyperparameter optimization, so timings
  exclude cross-validation
- The question is whole-genome wall-clock performance on many cores; this protocol
  deliberately restricts to a single worker and a 100 kb yeast test dataset
- The target software is not R/Bioconductor-based or does not expose a single model-fitting
  entry point whose timing and peak memory can be isolated
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GenoGAM 2.0
  role: R package whose genogam() fit call (Newton-Raphson fitting) is the object
    being timed and memory-profiled
  repo: https://github.com/gstricker/fastGenoGAM
- name: SuperLU
  role: Sparse direct solver used internally for parameter fitting; its scaling behavior
    is part of the measurement
- name: HDF5
  role: Out-of-core backend (hdf5 = TRUE) that keeps the memory footprint low during
    benchmarked fits
- name: BiocParallel
  role: Backend registration (SnowParam) used to pin execution to a single worker
    for single-core timing
- name: R
  role: Language in which the benchmark loop, timing, and memory recording are implemented
- name: mgcv
  role: Underlying GAM framework for the model design ~ s(x) + s(x, by = genotype)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:41cc682a66685e15c16455b59c5a2d8ae214d34f5d144a057d009171750e9825
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - BiocParallel
  - GenoGAM 2.0
  - HDF5
  - R
  - SuperLU
  - mgcv
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# runtime-memory-measurement-in-r

## Summary

A benchmarking protocol for measuring single-core coefficient-estimation runtime and peak memory of R/Bioconductor model-fitting code (here GenoGAM 2.0 GAM fits via genogam()) as the number of model parameters grows, to verify linear scaling and quantify speedup over a previous version.

## When to use

Use when benchmarking runtime and memory footprint of an R model-fitting routine (e.g. genogam() GAM fits) at increasing parameter counts, holding hyperparameters fixed and forcing single-worker execution.

## When NOT to use

- The goal is to evaluate smoothing-parameter estimation rather than coefficient estimation — fixed lambda = 4601 and theta = 4.51 bypass hyperparameter optimization, so timings exclude cross-validation
- The question is whole-genome wall-clock performance on many cores; this protocol deliberately restricts to a single worker and a 100 kb yeast test dataset
- The target software is not R/Bioconductor-based or does not expose a single model-fitting entry point whose timing and peak memory can be isolated

## Inputs

- R GenoGAMDataSet object built from the packaged yeast example (BAM files via experimentDesign.txt) with chunkSize = 50000, overhangSize = 1000, hdf5 = TRUE
- Fixed hyperparameters lambda = 4601 and theta = 4.51
- A single-worker BiocParallel backend (e.g. BiocParallel::SnowParam(worker = 1)) for single-core timing
- Reference measurements from the previous version (1104 parameters: 1 h 37 min runtime, 8 GB memory)

## Outputs

- Table of elapsed coefficient-estimation time versus number of parameters per tile
- Table of peak memory usage versus number of parameters per tile
- Speedup ratio versus the previous implementation (expected in the 2-3 orders of magnitude range)
- Fitted GenoGAM object per benchmark point (fits(result), se(result))

## How to apply

Set up the packaged yeast test dataset (first 100 kb of chrI) with GenoGAMDataSet() using chunkSize = 50000, overhangSize = 1000, HDF5 backend, and design ~ s(x) + s(x, by = genotype); compute size factors with computeSizeFactors(). Register a BiocParallel backend with a single worker (SnowParam) so timing reflects single-core performance. Fit repeatedly with genogam() at increasing model parameter counts, holding hyperparameters fixed (lambda = 4601, theta = 4.51) so only coefficient estimation, not hyperparameter optimization, is timed; record elapsed fitting time and peak memory per fit, including the SuperLU sparse solver's behavior. The rationale is that GenoGAM 2.0 exploits model sparsity via the SuperLU direct solver and HDF5 out-of-core storage, so runtime and memory should scale approximately linearly with parameters per tile; compare against the v1.0 reference point (1104 parameters: 1 h 37 min and 8 GB) where GenoGAM 2.0's Newton-Raphson method took 1 s and 52 MB.

## Related tools

- **GenoGAM 2.0** (R package whose genogam() fit call (Newton-Raphson fitting) is the object being timed and memory-profiled) — https://github.com/gstricker/fastGenoGAM
- **SuperLU** (Sparse direct solver used internally for parameter fitting; its scaling behavior is part of the measurement)
- **HDF5** (Out-of-core backend (hdf5 = TRUE) that keeps the memory footprint low during benchmarked fits)
- **BiocParallel** (Backend registration (SnowParam) used to pin execution to a single worker for single-core timing)
- **R** (Language in which the benchmark loop, timing, and memory recording are implemented)
- **mgcv** (Underlying GAM framework for the model design ~ s(x) + s(x, by = genotype))

## Examples

```
result <- genogam(ggd, lambda = 4601, theta = 4.51)
```

## Evaluation signals

- Runtime at the last comparable point (1104 parameters) should be ~1 s for GenoGAM 2.0 vs ~1 h 37 min for GenoGAM 1.0
- Peak memory at the comparable point should be ~52 MB for GenoGAM 2.0 vs ~8 GB for GenoGAM 1.0
- Plotted runtime and memory versus parameter count per tile should be approximately linear
- Observed speedup over v1.0 must fall in the 2-3 orders of magnitude range reported in the paper
- Fits should succeed with fixed hyperparameters (lambda = 4601, theta = 4.51) and return inspectable fits(result) and se(result)

## Limitations

- Performance claims reference hardware ('a typical lab server', 'a standard PC') without exact specifications, so absolute timings may not reproduce
- The packaged example is only the first 100 kb of yeast chrI, which limits the achievable parameter-count range compared to genome-scale benchmarks
- Large-matrix errors can occur if RcppArmadillo uses 32-bit matrices; ARMA_64BIT_WORD must be enabled during compilation
- Requires R 3.4.1 or higher; the fastGenoGAM repository version is deprecated in favor of the current GenoGAM Bioconductor release

## Evidence

- [abstract] SuperLU sparse solver role: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
- [discussion] Linear scaling claim: "We showed its runtime and memory footprint to scale linearly with the number of parameters per tiles"
- [abstract] Speedup magnitude: "a scalable and efficient implementation that is 2 to 3 orders of magnitude faster than the previous version"
- [readme] Benchmark fit call: "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [full_text] v1.0 vs 2.0 benchmark result: [evidence span withheld — anchor status: not_found]
