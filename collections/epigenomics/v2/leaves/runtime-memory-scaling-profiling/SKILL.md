---
name: runtime-memory-scaling-profiling
description: Use when comparing runtime and memory scaling of two alternative implementations
  of the same computation (e.g. variance/standard-error computation in GAM fits) across
  increasing problem sizes on a single core.
when_to_use_negative:
- The analysis goal is model fitting speed rather than standard-error/variance computation
  — use GenoGAM's SuperLU-based fitting benchmarks instead
- The comparison targets multi-core or distributed performance; this benchmark is
  explicitly single-core
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: GenoGAM 2.0 (fastGenoGAM)
  role: Provides the GenoGAMDataSet construction, genogam() fitting at fixed hyperparameters,
    and the sparse inverse subset variance computation
  repo: https://github.com/gstricker/fastGenoGAM
- name: mgcv
  role: Supplies Simon Wood's indirect method for computing position-wise variances,
    the slower comparison baseline
- name: sparseinv
  role: Sparse inverse subset algorithm used to compute position-wise standard errors
    efficiently
- name: BiocParallel
  role: Registers a single-core parallel backend for reproducible single-core timing
    of both methods
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:f3e6104277621a0c82ad037001b1c0b819de158c197dc8c7ccf9a08fbf6677cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - BiocParallel
  - GenoGAM 2.0
  - GenoGAM 2.0 (fastGenoGAM)
  - R
  - Rcpp
  - mgcv
  - sparseinv
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# runtime-memory-scaling-profiling

## Summary

Benchmarks two algorithms for computing position-wise standard errors in GenoGAM 2.0 — the sparse inverse subset algorithm (sparseinv) and Simon Wood's indirect method (mgcv) — recording elapsed time and peak memory as the number of parameters per tile grows, then fitting log-log scaling slopes to compare runtime and memory scaling.

## When to use

Use when comparing runtime and memory scaling of two alternative implementations of the same computation (e.g. variance/standard-error computation in GAM fits) across increasing problem sizes on a single core.

## When NOT to use

- The analysis goal is model fitting speed rather than standard-error/variance computation — use GenoGAM's SuperLU-based fitting benchmarks instead
- The comparison targets multi-core or distributed performance; this benchmark is explicitly single-core

## Inputs

- GenoGAMDataSet object built from the packaged yeast example design (extdata/Set1, 100kb of chromosome I)
- Fitted GenoGAM object from genogam(ggd, lambda = 4601, theta = 4.51)
- Vector of parameter counts per tile to sweep (increasing basis dimensions per tile)

## Outputs

- Benchmark table (variance_benchmark_results.csv) with runtime and peak memory per method at each parameter count
- Log-log scaling slopes for the memory curves of both methods
- Runtime ratio between the indirect and sparse methods at matched parameter counts

## How to apply

Load fastGenoGAM in R and build a GenoGAMDataSet from the packaged yeast example (first 100kb of chromosome I, design ~ s(x) + s(x, by = genotype), chunkSize 50000, overhangSize 1000), then fit with genogam() at fixed hyperparameters (lambda = 4601, theta = 4.51) so both methods operate on identical fitted objects. Compute position-wise variances with the sparse inverse subset algorithm (sparseinv) on a single-core BiocParallel backend, recording elapsed time and peak memory at each parameter count; repeat with Simon Wood's indirect method (mgcv) on the same fitted objects and parameter counts. Tabulate runtime and memory footprint versus number of parameters per tile for both methods, fit log-log scaling slopes to the memory curves, and compute the runtime ratio at matched parameter counts. Rationale: the sparse inverse subset algorithm exploits model sparsity, giving linear runtime versus quadratic for the indirect approach.

## Related tools

- **GenoGAM 2.0 (fastGenoGAM)** (Provides the GenoGAMDataSet construction, genogam() fitting at fixed hyperparameters, and the sparse inverse subset variance computation) — https://github.com/gstricker/fastGenoGAM
- **mgcv** (Supplies Simon Wood's indirect method for computing position-wise variances, the slower comparison baseline)
- **sparseinv** (Sparse inverse subset algorithm used to compute position-wise standard errors efficiently)
- **BiocParallel** (Registers a single-core parallel backend for reproducible single-core timing of both methods)

## Examples

```
result <- genogam(ggd, lambda = 4601, theta = 4.51); se_sparse <- sparseinv::covsubset(...)  # then time and profile vs mgcv indirect method
```

## Evaluation signals

- Runtime ratio between the indirect method and the sparse inverse subset method at matched parameter counts falls in the ~2 orders of magnitude regime reported in the paper
- Sparse inverse subset runtime scales linearly with parameters per tile, while the indirect method scales quadratically
- Memory-slope ratio (indirect / sparse) from log-log fits of the memory curves is approximately 4x, with both methods linear in memory
- The script runs to completion and variance_benchmark_results.csv contains rows for both methods at every tested parameter count
- Both methods are applied to identical fitted GenoGAM objects (same lambda = 4601, theta = 4.51) so comparisons are confounded only by the variance method

## Limitations

- Requires R 3.4.1 or higher, constraining compatible environments
- Performance claims reference hardware ('a typical lab server', 'a standard PC') without specifying exact specifications
- Warnings about out-of-bound ranges during tile shifting in GenoGAMDataSet() cannot be easily suppressed in the parallel Snow environment
- 32bit Armadillo matrices can cause errors on large matrices unless ARMA_64BIT_WORD is enabled during compilation

## Evidence

- [results] sparse inverse subset vs indirect Wood method runtime finding: [evidence span withheld — anchor status: not_found]
- [results] memory scaling finding: [evidence span withheld — anchor status: not_found]
- [readme] fitting call with fixed hyperparameters: "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [discussion] linear scaling claim: "We showed its runtime and memory footprint to scale linearly with the number of parameters per tiles"
- [abstract] sparsity exploitation rationale: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
