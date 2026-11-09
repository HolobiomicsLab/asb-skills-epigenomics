---
name: overlap-rate-variance-estimation
description: Use when quantifying significance of an observed SNP–peak (or range–range)
  overlap against block-bootstrapped null ranges, i.e. when per-bootstrap overlap
  rates, their variance, and a z-score are needed.
when_to_use_negative:
- The null sets come from conventional shuffling or permutation rather than block
  bootstrapping, since those schemes may produce overly narrow null distributions
  and over-estimated significance.
- The goal is threshold optimization or DEG–peak overlap counts (macrophage.R with
  generalized spline fitting) rather than SNP–peak overlap-rate significance testing.
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: bootRanges
  role: Generates the block-bootstrapped null genomic ranges whose per-iteration overlap
    rates form the null distribution used for the variance and z-score
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package hosting bootRanges; used with plyranges to load and
    intersect GRanges objects when computing overlap rates
  repo: https://bioconductor.org/packages/nullranges
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:e24aae9e0c523d6ed0a9056b3c621c8c0c19764473b04e45cc56b8496d786f6c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bootRanges
  - nullranges
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# overlap-rate-variance-estimation

## Summary

Computes the proportion of SNPs with a peak overlap within a 10 kb window (the overlap rate), the variance of this rate across block-bootstrapped null sets, and a z-score standardizing the observed rate against the bootRanges null distribution.

## When to use

Use when quantifying significance of an observed SNP–peak (or range–range) overlap against block-bootstrapped null ranges, i.e. when per-bootstrap overlap rates, their variance, and a z-score are needed.

## When NOT to use

- The null sets come from conventional shuffling or permutation rather than block bootstrapping, since those schemes may produce overly narrow null distributions and over-estimated significance.
- The goal is threshold optimization or DEG–peak overlap counts (macrophage.R with generalized spline fitting) rather than SNP–peak overlap-rate significance testing.

## Inputs

- Block-bootstrapped null range sets from segBootranges.R (bootRanges output) as GRanges objects
- Observed caQTL / total-cholesterol-SNP ranges as GRanges objects
- Peak ranges as GRanges objects

## Outputs

- Per-iteration overlap rates (proportion of SNPs with a peak within 10 kb)
- Across-replicate variance of the overlap rate
- Z-score of the observed overlap rate against the bootstrap null
- overlap_rate_zscore.tsv

## How to apply

Load the block-bootstrapped null range sets produced by segBootranges.R together with the observed caQTL/total-cholesterol-SNP and peak ranges as GRanges objects using nullranges/plyranges. For each bootstrap iteration b, count SNPs whose range overlaps a peak within a 10 kb window and divide by the total number of SNPs to obtain the per-iteration overlap rate. Pool the Lb per-iteration overlap rates and compute their variance to characterize the spread of the bootstrap null distribution. Standardize by computing the z-score as (observed rate minus mean of the Lb null rates) divided by the standard deviation of the null rates, then write the per-iteration rates, the across-replicate variance, and the z-score to overlap_rate_zscore.tsv. Rationale: the block bootstrap preserves local genomic correlation structure, giving more reliable null distributions than shuffling or permutation, which can over-estimate significance.

## Related tools

- **bootRanges** (Generates the block-bootstrapped null genomic ranges whose per-iteration overlap rates form the null distribution used for the variance and z-score) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package hosting bootRanges; used with plyranges to load and intersect GRanges objects when computing overlap rates) — https://bioconductor.org/packages/nullranges

## Examples

```
Rscript var.R  # compute per-bootstrap overlap rates, their variance, and z-score; writes overlap_rate_zscore.tsv
```

## Evaluation signals

- overlap_rate_zscore.tsv exists and contains per-iteration overlap rates, the across-replicate variance, and the z-score columns
- Each per-iteration overlap rate is a proportion (0–1) equal to (# SNPs overlapping a peak within 10 kb) / (total # SNPs)
- The z-score equals (observed rate − mean of the Lb null rates) / SD of the null rates; recomputing it from the written columns reproduces the reported value
- Variance is computed across the Lb pooled bootstrap iterations, not across SNPs within an iteration
- The pipeline order matches GRanges_construct.R -> segBootranges.R -> var.R -> fit_splines.R -> plot.R as in the liver script README

## Limitations

- Conventional shuffling/permutation nulls (bootstrap.R) may yield overly narrow test statistic null distributions and over-estimated statistical significance, so results are not comparable to a properly block-bootstrapped null.
- The 10 kb window and the SNP–peak overlap-rate definition are specific to the liver caQTL / total-cholesterol analysis; other studies may need different distance thresholds.

## Evidence

- [intro] The statistic compared via the z score is the overlap rate, which is defined as the proportion of SNPs that had peaks within 10 kb; per-bootstrap values of this rate and their variance across block lengths are computed in the var.R step.: [evidence span withheld — anchor status: not_found]
- [readme] *var.R*(Variance of the rate of overlaps and $z$ score): "*var.R*(Variance of the rate of overlaps and $z$ score)"
- [abstract] while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions.: "while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions."
- [abstract] We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance: "We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance"
