---
name: permutation-shuffling-null-generation
description: 'Use when benchmarking null-range generation for genomic overlap testing:
  you need a shuffling/permutation baseline null distribution of overlap rates to
  compare against block bootstrapping (bootRanges) for FPR calibration.'
when_to_use_negative:
- When the goal is a calibrated null that preserves local genomic correlation structure
  — use the block bootstrap (bootRanges) instead, since shuffling produces overly
  narrow null distributions and over-estimated significance
- When the analysis target is not genomic range overlap testing (e.g. differential
  expression or correlation analyses handled by macrophage.R or multiomicsCor.R scripts)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0749
tools:
- name: bootRanges
  role: Companion block-bootstrapping method whose FPR calibration is contrasted with
    the shuffling null in the simulation comparison
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package hosting bootRanges used for the benchmarking comparison
  repo: https://bioconductor.org/packages/nullranges
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:af91b3ba862772356a02049ac6ef5e07efbf59570f23051c78c39582c65380a7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bootRanges
  - nullranges
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# permutation-shuffling-null-generation

## Summary

Generates null sets of genomic ranges by conventional shuffling/permutation (bootstrap.R-style) to serve as a baseline null hypothesis generator in genomic overlap enrichment analysis, and to contrast against block-bootstrap nulls whose overly narrow test statistic null distributions inflate significance.

## When to use

Use when benchmarking null-range generation for genomic overlap testing: you need a shuffling/permutation baseline null distribution of overlap rates to compare against block bootstrapping (bootRanges) for FPR calibration.

## When NOT to use

- When the goal is a calibrated null that preserves local genomic correlation structure — use the block bootstrap (bootRanges) instead, since shuffling produces overly narrow null distributions and over-estimated significance
- When the analysis target is not genomic range overlap testing (e.g. differential expression or correlation analyses handled by macrophage.R or multiomicsCor.R scripts)

## Inputs

- Genomic range sets x and y as GRanges objects (e.g. simulated with no true association per Supplementary Section S2.1)
- Number of null/permutation iterations
- Nominal significance threshold alpha

## Outputs

- Null range sets produced by conventional shuffling (bootstrap.R scheme)
- Per-iteration overlap rates between null x and y
- Variance of the rate of overlaps and z score statistics (var.R-style)
- Estimated FPR table comparing shuffling vs bootRanges, saved as fpr_simulation_results.csv

## How to apply

Start from two genomic range sets x and y simulated or observed to have no true association (Supplementary Section S2.1 spec). Generate null range sets for x by conventional shuffling/permutation as implemented in bootstrap.R, ignoring local genomic correlation structure. For each null iteration compute the overlap rate between null x and y, then compute the variance of the rate of overlaps and the z score as in var.R. Aggregate over many simulation replicates and estimate the false positive rate at the nominal significance threshold alpha, comparing the shuffled null to a bootRanges block-bootstrapped null. The expected diagnostic outcome is that shuffling yields a null distribution with a similar mean but much lower variance than block bootstrap, producing an elevated FPR and over-estimation of statistical significance.

## Related tools

- **bootRanges** (Companion block-bootstrapping method whose FPR calibration is contrasted with the shuffling null in the simulation comparison) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package hosting bootRanges used for the benchmarking comparison) — https://bioconductor.org/packages/nullranges

## Examples

```
Rscript bootstrap.R  # perform conventional shuffling, then Rscript var.R to compute variance of the rate of overlaps and z score
```

## Evaluation signals

- Shuffling null distribution of overlap rates has a mean similar to the block-bootstrap null but much lower variance
- Shuffling-based FPR at threshold alpha is clearly elevated above alpha, while bootRanges FPR stays close to alpha
- Z-score statistics computed with var.R-style variance of the rate of overlaps are reproducible across simulation replicates
- Results tabulated in fpr_simulation_results.csv include both bootRanges and shuffling FPRs for direct comparison

## Limitations

- Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance
- Conventional shuffling does not preserve local genomic correlation structure, unlike the block bootstrap

## Evidence

- [abstract] Shuffling produces a narrow null and high FPR: "We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance"
- [abstract] Block bootstrap preserves local correlation: "while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions."
- [readme] Workflow position of the shuffling script: "*bootstrap.R*: perform conventional shuffling."
- [readme] Z-score computation step: "*var.R*(Variance of the rate of overlaps and $z$ score)"
