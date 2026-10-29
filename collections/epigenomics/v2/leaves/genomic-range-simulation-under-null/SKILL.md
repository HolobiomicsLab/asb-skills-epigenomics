---
name: genomic-range-simulation-under-null
description: Use when testing overlap/enrichment between two genomic range sets (e.g.
  caQTLs vs GWAS SNPs, CREs vs genes) and a properly calibrated null is needed, especially
  when local genomic correlation structure must be preserved.
when_to_use_negative:
- When the analysis goal is differential expression threshold optimization via overlap
  counts of DEGs and differential accessibility peaks (macrophage.R workflow) rather
  than null range generation
- When the goal is benchmarking runtime of null-range methods against genome-scale
  statistics (bootranges_vs_GSC.R / GSC comparison), not statistical calibration
- When range sets lack genomic correlation structure such that a simple permutation
  would already be calibrated (the article's advantage of block bootstrap is preserving
  local genomic correlation)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0622
tools:
- name: bootRanges
  role: Generates block-bootstrapped null sets of genomic ranges representing the
    null hypothesis in enrichment analysis, preserving local genomic correlation structure
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package hosting the bootRanges functions used to create the
    null range sets
  repo: https://bioconductor.org/packages/nullranges
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:ca2c9b88ec1f15b71e4e1e1cf180a74a0fb5af22238769e11b888ab19e4a6cf6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bootRanges
  - nullranges
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# genomic-range-simulation-under-null

## Summary

Generates null sets of genomic ranges via block bootstrap (bootRanges from the R/Bioconductor nullranges package) to test enrichment between range sets, and contrasts them with conventional shuffling to show shuffling yields overly narrow null distributions and over-estimated significance.

## When to use

Use when testing overlap/enrichment between two genomic range sets (e.g. caQTLs vs GWAS SNPs, CREs vs genes) and a properly calibrated null is needed, especially when local genomic correlation structure must be preserved.

## When NOT to use

- When the analysis goal is differential expression threshold optimization via overlap counts of DEGs and differential accessibility peaks (macrophage.R workflow) rather than null range generation
- When the goal is benchmarking runtime of null-range methods against genome-scale statistics (bootranges_vs_GSC.R / GSC comparison), not statistical calibration
- When range sets lack genomic correlation structure such that a simple permutation would already be calibrated (the article's advantage of block bootstrap is preserving local genomic correlation)

## Inputs

- Genomic range sets (GRanges objects) for query and subject features, e.g. caQTL SNPs associated with total cholesterol from human liver tissue
- Genomic feature segments (e.g. segmental duplications) used as blocking covariates for the block bootstrap
- Simulation specification for range sets with no true association (Supplementary Section S2.1)

## Outputs

- Block-bootstrapped null sets of genomic ranges per iteration
- Overlap rates between null query and subject ranges with variance and z score statistics (var.R output)
- FPR comparison table for bootRanges vs shuffling (e.g. fpr_simulation_results.csv)
- Conditional density plots and figures (fit_splines.R, plot.R)

## How to apply

Preprocess inputs into GRanges objects (GRanges_construct.R), then generate block-bootstrapped null range sets for the query ranges with bootRanges (segBootranges.R), preserving local genomic correlation structure of features such as segmental duplications. For each null iteration, compute the rate of overlaps between the null query set and the subject set, then estimate the variance of the overlap rate and derive a z score statistic (var.R). Optionally run conventional shuffling (bootstrap.R) as a comparator to demonstrate that its null distribution has a similar mean but much lower variance, producing an inflated false positive rate at nominal alpha. Assess calibration by repeating over simulation replicates and checking that the bootRanges FPR is near the nominal significance threshold alpha while the shuffling FPR is clearly elevated.

## Related tools

- **bootRanges** (Generates block-bootstrapped null sets of genomic ranges representing the null hypothesis in enrichment analysis, preserving local genomic correlation structure) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package hosting the bootRanges functions used to create the null range sets) — https://bioconductor.org/packages/nullranges

## Examples

```
GRanges_construct.R -> segBootranges.R -> var.R -> fit_splines.R -> plot.R; bootstrap.R for conventional shuffling
```

## Evaluation signals

- bootRanges FPR is close to the nominal significance threshold alpha (correct calibration) in the no-association simulation
- Shuffling-based FPR is clearly elevated above alpha, reflecting over-estimation of statistical significance
- Shuffling null distribution has a similar mean to bootRanges but much lower variance
- Block-bootstrapped range sets preserve local genomic correlation structure of the original features
- Workflow outputs (overlap rates, variances, z scores) are reproduced via the var.R computation on each null iteration

## Limitations

- Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance, so comparator results must be interpreted as anti-conservative
- The block bootstrap requires appropriate blocking covariates/features (e.g. segmental duplications) to reflect genomic correlation; results depend on this choice

## Evidence

- [abstract] bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis.: "bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis."
- [abstract] We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance: "We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance"
- [abstract] while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions.: "while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions."
- [abstract] bootRanges is freely available in the R/Bioconductor package nullranges hosted at https://bioconductor.org/packages/nullranges.: "bootRanges is freely available in the R/Bioconductor package nullranges hosted at https://bioconductor.org/packages/nullranges."
- [readme] *var.R*(Variance of the rate of overlaps and $z$ score): "*var.R*(Variance of the rate of overlaps and $z$ score)"
- [readme] *bootstrap.R*: perform conventional shuffling.: "*bootstrap.R*: perform conventional shuffling."
