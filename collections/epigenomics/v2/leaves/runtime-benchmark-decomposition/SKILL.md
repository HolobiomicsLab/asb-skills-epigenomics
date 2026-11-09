---
name: runtime-benchmark-decomposition
description: Use when benchmarking a GenoGAM 2.0 genome-wide GAM fit and the goal
  is a per-stage runtime breakdown (preprocessing vs cross-validation vs fitting)
  on a specified core count and backend (in-memory vs HDF5).
when_to_use_negative:
- When the goal is biological fit quality (fits, standard errors, log p-values) rather
  than performance measurement — use computeSignificance()/plotGenoGAM instead
- When benchmarking out-of-core HDF5-backed processing or a multi-core server configuration
  (e.g. 60-core human-genome fits) — the ~13 min figure applies only to the in-memory
  8-core desktop yeast setup
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GenoGAM 2.0
  role: R package whose genome-wide fit pipeline (GenoGAMDataSet, computeSizeFactors,
    genogam) is being benchmarked stage by stage
  repo: https://github.com/gstricker/fastGenoGAM
- name: BiocParallel
  role: Registers the 8-worker SnowParam parallel backend under which the runtime
    is measured
- name: SuperLU
  role: Sparse direct solver used in the parameter-fitting stage whose ~3.5 min share
    is being quantified
- name: R
  role: Implementation language; elapsed-time instrumentation is added around each
    pipeline step
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:198c5e5722c1f3554ab6d741b0902ca787e71786aedabaae5a96c89c62e038f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - BiocParallel
  - Bioconductor
  - GenoGAM 2.0
  - R
  - SuperLU
  - mgcv
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# runtime-benchmark-decomposition

## Summary

Decomposes the end-to-end wall-clock runtime of a GenoGAM 2.0 genome-wide model fit into its stages — data preprocessing (size factor computation), cross-validation of smoothing hyperparameters, and sparse-solver parameter fitting — to quantify where compute time is spent and validate performance claims.

## When to use

Use when benchmarking a GenoGAM 2.0 genome-wide GAM fit and the goal is a per-stage runtime breakdown (preprocessing vs cross-validation vs fitting) on a specified core count and backend (in-memory vs HDF5).

## When NOT to use

- When the goal is biological fit quality (fits, standard errors, log p-values) rather than performance measurement — use computeSignificance()/plotGenoGAM instead
- When benchmarking out-of-core HDF5-backed processing or a multi-core server configuration (e.g. 60-core human-genome fits) — the ~13 min figure applies only to the in-memory 8-core desktop yeast setup

## Inputs

- Yeast ChIP-seq BAM files for two conditions with two replicates each (experiment design file with design ~ s(x) + s(x, by = genotype))
- GenoGAMSettings/hdf5Control configuration (hdf5 = FALSE for in-memory backend)
- 8-core desktop machine with SnowParam parallel backend registered

## Outputs

- Per-stage wall-clock runtime report table (preprocessing ~40 s, cross-validation ~9 min, fitting ~3.5 min)
- Total end-to-end runtime (~13 min) for the genome-wide yeast fit

## How to apply

Load the ChIP-seq read counts with GenoGAMDataSet() using chunkSize = 50000 and overhangSize = 1000, setting hdf5 = FALSE so all data stay in memory (no hard drive I/O), and register an 8-worker parallel backend via BiocParallel::register(SnowParam(workers = 8)). Instrument each stage separately with wall-clock timers: computeSizeFactors() (~40 s preprocessing), then genogam() with lambda and theta estimated by cross-validation (~9 min CV plus ~3.5 min fitting via the SuperLU sparse direct solver). Record preprocessing, cross-validation, and fitting times as rows in a runtime report table and sum them for the total (~13 min for the yeast genome-wide fit on 8 cores). Judge correctness by whether the total falls within tolerance of the reported value (roughly 8–20 min) and whether all three stages are present in the report. The rationale is that GenoGAM 2.0's speedup comes from exploiting model sparsity with SuperLU and out-of-core/in-memory data handling, so decomposing the runtime shows which stage benefits from each optimization.

## Related tools

- **GenoGAM 2.0** (R package whose genome-wide fit pipeline (GenoGAMDataSet, computeSizeFactors, genogam) is being benchmarked stage by stage) — https://github.com/gstricker/fastGenoGAM
- **BiocParallel** (Registers the 8-worker SnowParam parallel backend under which the runtime is measured)
- **SuperLU** (Sparse direct solver used in the parameter-fitting stage whose ~3.5 min share is being quantified)
- **R** (Implementation language; elapsed-time instrumentation is added around each pipeline step)

## Examples

```
system.time({ggd <- GenoGAMDataSet(expDesign, directory = folder, chunkSize = 50000, overhangSize = 1000, design = design, hdf5 = FALSE); ggd <- computeSizeFactors(ggd)}); result <- genogam(ggd)
```

## Evaluation signals

- Total end-to-end runtime falls within tolerance of the reported ~13 min on the 8-core in-memory configuration (accept roughly 8–20 min)
- Runtime report contains rows for all three stages: preprocessing, cross-validation, and fitting
- Stage decomposition sums to the reported total: ~40 s + ~9 min + ~3.5 min ≈ 13 min
- Settings confirm chunkSize = 50000, overhangSize = 1000, and hdf5 = FALSE (data processed in memory, no hard drive I/O)
- Parallel backend registered with 8 workers (SnowParam(workers = 8))

## Limitations

- The ~13 min breakdown applies to the yeast genome with in-memory processing (hdf5 = FALSE); other genomes, backends, or core counts will differ
- Performance claims reference hardware ('a typical lab server', 'a standard PC') without exact specifications, making exact reproduction hardware-dependent
- With fixed hyperparameters (e.g. lambda = 4601, theta = 4.51) the cross-validation stage is skipped, so the CV time will not appear in the decomposition
- Out-of-bound range warnings during genome tiling cannot be suppressed in the parallel Snow environment, which may clutter timing logs

## Evidence

- [abstract] This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
- [methods] Reading in data through GenoGAMDataSet(): "Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object."
- [methods] Computing size factors with computeSizeFactors(): "Computing size factors with `computeSizeFactors()`"
- [methods] Compute the model with genogam(): "Compute the model with `genogam()` to get the result `GenoGAM` object"
- [readme] README: register parallel backend with SnowParam: "BiocParallel::register(BiocParallel::SnowParam(worker = 2))"
