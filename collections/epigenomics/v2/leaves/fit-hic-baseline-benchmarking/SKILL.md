---
name: fit-hic-baseline-benchmarking
description: Use when evaluating a Hi-C/HiChIP normalization-and-significance method
  against an established baseline (Fit-Hi-C) on replicate data, using reproducible-contact
  overlap and ROC AUC (~0.85 expected) as acceptance criteria.
when_to_use_negative:
- The data are promoter-capture Hi-C PO-interactions, where the pcHi-C-specific thresholds
  (promoter side 200, other side 50, 15 kb–2 Mb range, do_shuffle=FALSE) apply instead
  of the default Hi-C settings
- The goal is contact-map reproducibility scoring (HiC-spector score) or compartment
  A/B correlation rather than significant-interaction calling benchmarking
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: covNorm
  role: 'Test method: negative binomial coverage normalization, Weibull distance-background
    removal, and p-value/FDR-based significant interaction calling'
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: Fit-Hi-C
  role: Baseline method providing q-value significance calls at 40 kb resolution with
    2 Mb distance parameter for comparison
- name: R
  role: Runtime environment for covNorm and supporting packages MASS, propagate, FAdist,
    p.adjust
  repo: https://github.com/kaistcbfg/covNormRpkg
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:b0beb658067f8af971fc065bd86fc266204cf2df5f742cd441ec06e5c8ba90fa
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kaistcbfg/covNormRpkg
  repo_url: https://github.com/kaistcbfg/covNormRpkg
  tools:
  - Fit-Hi-C
  - R
  - covNorm
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fit-hic-baseline-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks covNorm's significant long-range chromatin contact calls against Fit-Hi-C as a baseline on Hi-C biological replicates, using replicate-overlap fractions and ROC AUC of significance scores as the comparison metrics.

## When to use

Use when evaluating a Hi-C/HiChIP normalization-and-significance method against an established baseline (Fit-Hi-C) on replicate data, using reproducible-contact overlap and ROC AUC (~0.85 expected) as acceptance criteria.

## When NOT to use

- The data are promoter-capture Hi-C PO-interactions, where the pcHi-C-specific thresholds (promoter side 200, other side 50, 15 kb–2 Mb range, do_shuffle=FALSE) apply instead of the default Hi-C settings
- The goal is contact-map reproducibility scoring (HiC-spector score) or compartment A/B correlation rather than significant-interaction calling benchmarking

## Inputs

- GM19204.chr17.cis.feature.gz (Hi-C replicate 1, 783588 rows) and GM19240.chr17.cis.feature.gz (Hi-C replicate 2, 1136435 rows) tab-delimited feature tables with columns frag1, frag2, cov_frag1, cov_frag2, freq, dist
- Fit-Hi-C q-value output at 40 kb resolution with 2 Mb maximum distance
- covNorm per-contact p-values / FDR-adjusted significance calls

## Outputs

- Per-method lists of significant long-range chromatin contacts for each replicate
- Replicate-overlap fraction of reproducible significant interactions (>65% expected for both methods)
- ROC AUC of each method's significance scores for discriminating reproducible interactions (~0.85, methods differing by <1%)

## How to apply

Load the GM19204 and GM19240 chromosome 17 .feature.gz sample data (columns frag1, frag2, cov_frag1, cov_frag2, freq, dist), remove zero-valued DNA fragment pairs, and filter fragments with coverage below the default cut-off of 200 at 40 kb resolution, shuffling 50% of coverage values to prevent sorted-order bias. Normalize with covNorm via a negative binomial GLM (MASS::glm.nb) for coverage, remove the distance-dependent background with a three-parameter Weibull fit (propagate::fitDistr) within 2 Mb, and compute per-contact p-values with FAdist::pweibull3 followed by FDR correction (p.adjust, 'fdr'). Run Fit-Hi-C as the baseline using 40 kb resolution and 2 Mb distance parameters, taking its q-value as the significance threshold. Label interactions reproducibly called across the two biological replicates and compute the ROC AUC of each method's significance scores for discriminating reproducible interactions; accept results if both methods show >65% reproducible significant long-range contacts and ROC AUCs consistent with the paper's reported value of approximately 0.85, differing by less than 1% between methods.

## Related tools

- **covNorm** (Test method: negative binomial coverage normalization, Weibull distance-background removal, and p-value/FDR-based significant interaction calling) — https://github.com/kaistcbfg/covNormRpkg
- **Fit-Hi-C** (Baseline method providing q-value significance calls at 40 kb resolution with 2 Mb distance parameter for comparison)
- **R** (Runtime environment for covNorm and supporting packages MASS, propagate, FAdist, p.adjust) — https://github.com/kaistcbfg/covNormRpkg

## Examples

```
dist_result <- covNormRpkg::normDistance(cov_df, max_dist=2000000); final_df <- covNormRpkg::contactPval(dist_df, 'fit.pdf')
```

## Evaluation signals

- Over 65% of significant long-range chromatin contacts are reproducible between the GM19204 and GM19240 Hi-C replicates for both methods
- ROC AUC of each method's significance scores is approximately 0.85, with covNorm vs Fit-Hi-C AUCs differing by less than 1%
- covNorm pipeline completes all steps: coverage normalization coefficients produced, distance normalization at max_dist=2000000, and final_df containing per-contact p-values
- FDR correction applied with method='fdr' yields a valid q-value column alongside pweibull3-derived p-values

## Limitations

- No version number is stated for covNorm, so exact reproducibility of the package release may be ambiguous
- 50% of fragment coverage values are shuffled before normalization to prevent sorted-order bias, so results depend on this randomized step
- Coverage cut-off of 200 at 40 kb resolution is a default that users may redefine, affecting comparability of significance calls

## Evidence
