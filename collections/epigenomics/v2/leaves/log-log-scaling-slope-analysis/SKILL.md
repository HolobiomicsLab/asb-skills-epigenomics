---
name: log-log-scaling-slope-analysis
description: Use when comparing two algorithms' runtime/memory behavior across increasing
  problem sizes and you need empirical complexity estimates (log-log slopes) plus
  speedup ratios at matched sizes to validate complexity claims.
when_to_use_negative:
- You need absolute wall-clock performance on a specific production workload rather
  than empirical complexity scaling across problem sizes
- The two methods cannot be run on identical fitted objects, so ratios at matched
  parameter counts would be confounded
- The workload is parallelized across many cores — the paper's claim (2-orders-of-magnitude
  gap, slope ratio ~4x) was established on a single core
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GenoGAM 2.0 (fastGenoGAM)
  role: Provides the fitted GAM objects and the sparse inverse subset variance computation
    (via sparseinv) benchmarked against the indirect method
  repo: https://github.com/gstricker/fastGenoGAM
- name: mgcv
  role: Supplies Simon Wood's indirect method for computing standard errors, the comparison
    baseline
- name: sparseinv
  role: Sparse inverse subset algorithm used to compute position-wise variances efficiently
- name: BiocParallel
  role: Registers the single-core parallel backend under which both methods are timed
- name: R
  role: Implementation environment for the benchmark script and log-log slope fitting
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:c9865b6fbfbbd6b8a8c629d791337f58b9645ffcf87b666d4160da340c5af0cb
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# log-log-scaling-slope-analysis

## Summary

Benchmark-scaling analysis that tabulates runtime and peak memory versus problem size (e.g. number of parameters per tile), fits log-log scaling slopes to estimate empirical complexity, and compares method ratios at matched sizes — as done in GenoGAM 2.0 (Fig. 3) to show the sparse inverse subset algorithm computes standard errors ~2 orders of magnitude faster than the indirect Simon Wood method, with linear runtime (vs quadratic) and a ~4x shallower memory slope.

## When to use

Use when comparing two algorithms' runtime/memory behavior across increasing problem sizes and you need empirical complexity estimates (log-log slopes) plus speedup ratios at matched sizes to validate complexity claims.

## When NOT to use

- You need absolute wall-clock performance on a specific production workload rather than empirical complexity scaling across problem sizes
- The two methods cannot be run on identical fitted objects, so ratios at matched parameter counts would be confounded
- The workload is parallelized across many cores — the paper's claim (2-orders-of-magnitude gap, slope ratio ~4x) was established on a single core

## Inputs

- Fitted GenoGAM objects at increasing numbers of parameters per tile (from the yeast example: first 100kb of chromosome I)
- Benchmark design specifying chunkSize = 50000 and overhangSize = 1000
- Single-core BiocParallel backend configuration

## Outputs

- Benchmark table (variance_benchmark_results.csv) with runtime and peak memory versus parameters per tile for both methods
- Fitted log-log scaling slopes for the memory curves of each method
- Runtime ratio between methods at matched parameter counts

## How to apply

Construct a fitted object per problem size — e.g. build a GenoGAMDataSet from the packaged yeast example (first 100kb of chromosome I, design ~ s(x) + s(x, by = genotype), chunkSize 50000, overhangSize 1000) and fit with genogam() at fixed hyperparameters (lambda = 4601, theta = 4.51) so both methods run on identical inputs. For each tested parameter count per tile, run each method (here: sparse inverse subset via sparseinv, and Simon Wood's indirect mgcv method) on a single-core BiocParallel backend, recording elapsed time and peak memory. Tabulate runtime and memory versus parameters per tile for both methods, then fit log-log scaling slopes to the memory curves and compute the runtime ratio at matched parameter counts. Interpretation: the sparse inverse subset approach should be faster by roughly two orders of magnitude with linear runtime (vs quadratic for the indirect method), and its memory slope about four times lower (both linear in memory).

## Related tools

- **GenoGAM 2.0 (fastGenoGAM)** (Provides the fitted GAM objects and the sparse inverse subset variance computation (via sparseinv) benchmarked against the indirect method) — https://github.com/gstricker/fastGenoGAM
- **mgcv** (Supplies Simon Wood's indirect method for computing standard errors, the comparison baseline)
- **sparseinv** (Sparse inverse subset algorithm used to compute position-wise variances efficiently)
- **BiocParallel** (Registers the single-core parallel backend under which both methods are timed)
- **R** (Implementation environment for the benchmark script and log-log slope fitting)

## Examples

```
result <- genogam(ggd, lambda = 4601, theta = 4.51); se(result)
```

## Evaluation signals

- The runtime ratio at matched parameter counts falls within the ~2-orders-of-magnitude regime reported in the paper (sparse inverse subset faster than the indirect Wood method)
- The fitted log-log memory-slope ratio between the indirect and sparse methods is approximately 4x, with both methods linear in memory
- Runtime scaling is linear for the sparse inverse subset method and quadratic for the indirect method (slopes ~1 and ~2 on log-log axes)
- The script runs to completion and variance_benchmark_results.csv contains rows for both methods at every tested parameter count
- Both methods were run on the same fitted objects with identical hyperparameters (lambda = 4601, theta = 4.51)

## Limitations

- Benchmark results reference hardware ('a typical lab server', 'a standard PC') without exact specifications in the article, so absolute timings may not reproduce
- The reported scaling behavior was measured on a single core; multi-core backends may change the runtime ratio
- fastGenoGAM (the repo used for the benchmark) is deprecated in favor of the Bioconductor GenoGAM version, and no changelog is available

## Evidence

- [discussion] Linear scaling of runtime and memory with parameters per tile: "We showed its runtime and memory footprint to scale linearly with the number of parameters per tiles"
- [results] Sparse inverse subset vs indirect Wood method speedup: [evidence span withheld — anchor status: not_found]
- [readme] Fixed hyperparameter fit invocation: "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [methods] Model fitting workflow step: "Compute the model with `genogam()` to get the result `GenoGAM` object"
- [abstract] GenoGAM 2.0 performance claim: "Here we present GenoGAM 2.0, a scalable and efficient implementation that is 2 to 3 orders of magnitude faster than the previous version."
