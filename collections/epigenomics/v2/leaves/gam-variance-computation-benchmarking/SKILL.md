---
name: gam-variance-computation-benchmarking
description: Use when evaluating or comparing algorithms for computing standard errors
  or position-wise variances of fitted generalized additive models (GAMs), especially
  tiled genome-wide GenoGAM fits where scalability in parameters per tile matters.
when_to_use_negative:
- When the goal is fitting the GAM itself (parameter estimation via the SuperLU direct
  solver) rather than benchmarking post-fit variance/standard-error computation
- When comparing end-to-end GenoGAM 2.0 against the previous GenoGAM version — that
  is a whole-package speed benchmark, not the sparse-vs-indirect variance method comparison
- When the fitted objects differ between methods — the benchmark requires identical
  GenoGAM fits (fixed lambda = 4601, theta = 4.51) for both variance computations
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: GenoGAM 2.0 (fastGenoGAM)
  role: R package providing GenoGAMDataSet construction, genogam() fitting at fixed
    hyperparameters, and the sparse inverse subset variance computation used as the
    fast arm of the benchmark
  repo: https://github.com/gstricker/fastGenoGAM
- name: mgcv
  role: General-purpose R GAM library supplying Simon Wood's indirect method for standard-error
    computation, used as the baseline arm of the benchmark
- name: sparseinv
  role: Provides the sparse inverse subset algorithm imported by fastGenoGAM for computing
    position-wise variances from the sparse fitted precision structure
- name: BiocParallel
  role: Registers the single-core parallel backend under which elapsed time and peak
    memory are recorded for both methods
- name: Matrix
  role: Sparse matrix infrastructure underpinning the sparse variance computations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:d8131fae4fa6b98edf070af57ef08abec0d2e9c3204756cbeb866735c514fd62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - BiocParallel
  - GenoGAM 2.0
  - GenoGAM 2.0 (fastGenoGAM)
  - Matrix
  - R
  - Rcpp
  - mgcv
  - sparseinv
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gam-variance-computation-benchmarking

## Summary

A benchmarking skill for comparing position-wise variance/standard-error computation methods for tiled genome-wide GAM fits: the sparse inverse subset algorithm (via sparseinv/fastGenoGAM) versus Simon Wood's indirect method (mgcv). It measures single-core runtime and peak memory across increasing parameter counts per tile to demonstrate that the sparse method is ~2 orders of magnitude faster with linear (not quadratic) runtime scaling and a memory slope about 4x shallower.

## When to use

Use when evaluating or comparing algorithms for computing standard errors or position-wise variances of fitted generalized additive models (GAMs), especially tiled genome-wide GenoGAM fits where scalability in parameters per tile matters.

## When NOT to use

- When the goal is fitting the GAM itself (parameter estimation via the SuperLU direct solver) rather than benchmarking post-fit variance/standard-error computation
- When comparing end-to-end GenoGAM 2.0 against the previous GenoGAM version — that is a whole-package speed benchmark, not the sparse-vs-indirect variance method comparison
- When the fitted objects differ between methods — the benchmark requires identical GenoGAM fits (fixed lambda = 4601, theta = 4.51) for both variance computations

## Inputs

- GenoGAMDataSet built from the fastGenoGAM packaged yeast example (experimentDesign.txt, BAM folder in extdata/Set1, first 100kb of chromosome I)
- Design formula ~ s(x) + s(x, by = genotype) with chunkSize 50000 and overhangSize 1000
- Fitted GenoGAM object from genogam(ggd, lambda = 4601, theta = 4.51)
- Single-core BiocParallel backend (e.g. MulticoreParam/SnowParam with worker = 1)

## Outputs

- variance_benchmark_results.csv with runtime and peak memory rows for both methods at every tested parameter count per tile
- Log-log scaling slopes of the memory curves for each method
- Runtime ratio between indirect Wood method and sparse inverse subset at matched parameter counts

## How to apply

Load the GenoGAM 2.0 package (fastGenoGAM) in R and build a GenoGAMDataSet from the packaged yeast example (first 100kb of chromosome I, design ~ s(x) + s(x, by = genotype), chunkSize 50000, overhangSize 1000) so design matrices at increasing parameter counts per tile are available. Fit the model with genogam() at fixed hyperparameters (lambda = 4601, theta = 4.51) so identical fitted objects feed both variance methods. On a single-core BiocParallel backend, compute position-wise variances first with the sparse inverse subset algorithm (sparseinv, as imported by fastGenoGAM) and then with Simon Wood's indirect method (mgcv), recording elapsed time and peak memory at each parameter count. Tabulate runtime and memory versus number of parameters per tile, fit log-log scaling slopes to the memory curves, and compute the runtime ratio between methods at matched parameter counts. The decision point is method selection: prefer the sparse inverse subset approach when runtime must scale linearly with parameters per tile and memory slope must be minimized.

## Related tools

- **GenoGAM 2.0 (fastGenoGAM)** (R package providing GenoGAMDataSet construction, genogam() fitting at fixed hyperparameters, and the sparse inverse subset variance computation used as the fast arm of the benchmark) — https://github.com/gstricker/fastGenoGAM
- **mgcv** (General-purpose R GAM library supplying Simon Wood's indirect method for standard-error computation, used as the baseline arm of the benchmark)
- **sparseinv** (Provides the sparse inverse subset algorithm imported by fastGenoGAM for computing position-wise variances from the sparse fitted precision structure)
- **BiocParallel** (Registers the single-core parallel backend under which elapsed time and peak memory are recorded for both methods)
- **Matrix** (Sparse matrix infrastructure underpinning the sparse variance computations)

## Examples

```
result <- genogam(ggd, lambda = 4601, theta = 4.51); se(result)
```

## Evaluation signals

- The runtime ratio at matched parameter counts falls within the ~2-orders-of-magnitude regime favoring the sparse inverse subset method over the indirect Simon Wood method
- Log-log scaling of runtime is linear for the sparse inverse subset method and quadratic for the indirect method
- The memory-slope ratio between the indirect and sparse methods is approximately 4x, with both methods linear in memory
- The benchmark script runs to completion and variance_benchmark_results.csv contains rows for both methods at every tested parameter count
- Both variance computations use the identical fitted GenoGAM object (fixed lambda = 4601, theta = 4.51) so results are comparable

## Limitations

- Performance claims reference hardware ('a typical lab server', 'a standard PC') without specifying exact specifications, so absolute timings may not reproduce
- The packaged example dataset is restricted to the first 100kb of the first yeast chromosome; scaling behavior must be extrapolated from parameter counts per tile rather than genome size
- fastGenoGAM is deprecated in favor of the new GenoGAM Bioconductor version, which may affect reproducibility of the benchmark environment (requires R 3.4.1 or higher; GPL-2 license)
- No DOI or archival (Zenodo) record is provided for the repository, only GitHub hosting

## Evidence

- [results] The sparse inverse subset algorithm computes standard errors orders of magnitude faster than the indirect Simon Wood method with linear scaling: [evidence span withheld — anchor status: not_found]
- [discussion] Runtime and memory scale linearly with parameters per tile: "We showed its runtime and memory footprint to scale linearly with the number of parameters per tiles"
- [readme] Fixed hyperparameter fitting command from the README: "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [readme] Standard errors are exposed on the fitted object: "## Check the fit and standard error
fits(result)
se(result)"
- [readme] Example dataset restricted to yeast chromosome I: [evidence span withheld — anchor status: external_doc]
