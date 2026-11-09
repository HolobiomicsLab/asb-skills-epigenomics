---
name: weibull-distribution-fitting
description: Use when significant long-range chromatin contacts must be called from
  distance-normalized Hi-C/capture Hi-C contact tables, requiring a fitted null distribution
  for p-value/FDR computation.
when_to_use_negative:
- Input has not yet been coverage- and distance-normalized (covNormRpkg::normCoverage
  and normDistance must run first); fitting raw frequencies violates the background
  model
- Contacts beyond 2 Mb are the target — the covNorm Hi-C workflow uses max_dist=2000000,
  and pcHi-C considers only 15 kb–2 Mb interactions
- The goal is a reproducibility score of contact maps (HiC-spector) rather than significance
  calling of individual contacts
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: covNormRpkg
  role: R package whose contactPval function performs the 3P-Weibull fitting, p-value,
    and FDR computation on distance-normalized contacts
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: R
  role: Runtime environment; uses propagate::fitDistr for 3P-Weibull fitting and FAdist::pweibull3
    plus p.adjust('fdr') for per-contact p-values and FDR
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:f8fe8ef9ba650ee5844b8fcedd8bf1254c3fd4563e1d4e9d6b0f791340e4cbc2
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
  - covNormRpkg
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# weibull-distribution-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits a three-parameter Weibull distribution (location, shape, scale) to the distance-normalized Hi-C background set within 2-fold of expected values, then computes per-contact p-values and Benjamini-Hochberg FDR for significant interaction calling, as implemented in covNorm's contactPval step.

## When to use

Use when significant long-range chromatin contacts must be called from distance-normalized Hi-C/capture Hi-C contact tables, requiring a fitted null distribution for p-value/FDR computation.

## When NOT to use

- Input has not yet been coverage- and distance-normalized (covNormRpkg::normCoverage and normDistance must run first); fitting raw frequencies violates the background model
- Contacts beyond 2 Mb are the target — the covNorm Hi-C workflow uses max_dist=2000000, and pcHi-C considers only 15 kb–2 Mb interactions
- The goal is a reproducibility score of contact maps (HiC-spector) rather than significance calling of individual contacts

## Inputs

- covNorm distance-normalized contact data.frame (from covNormRpkg::normDistance with max_dist=2000000, columns incl. frag1, frag2, cov_frag1, cov_frag2, freq, dist and normalized frequency)
- Fitted 3P-Weibull parameters (location, shape, scale) from propagate::fitDistr

## Outputs

- Annotated contact table with fragment coordinates, normalized frequencies, p_result_dist (per-contact p-values), and FDR_dist_res (Benjamini-Hochberg adjusted q-values)
- Diagnostic fit plot (e.g. 'fit.pdf') showing the Weibull fit to the background
- Significant interaction calls used for downstream ROC AUC / replicate-overlap evaluation

## How to apply

After covNorm's normCoverage and normDistance steps, restrict the background set to normalized values within 2-fold of the expected values (following the two-step spline strategy adapted from Fit-Hi-C) so extraordinarily high interaction frequencies do not distort the null. Fit the three-parameter Weibull distribution (location, shape, scale) to this background using R 'propagate::fitDistr'. Compute the p-value of every chromatin contact with R 'FAdist::pweibull3' using the fitted parameters, and adjust p-values for multiple testing with R 'p.adjust' using the 'fdr' method. In the covNormRpkg workflow this is done by the contactPval function, which takes the distance-normalized data.frame (columns frag1, frag2, cov_frag1, cov_frag2, freq, dist plus normalization columns) and outputs annotated contacts with p_result_dist and FDR_dist_res columns.

## Related tools

- **covNormRpkg** (R package whose contactPval function performs the 3P-Weibull fitting, p-value, and FDR computation on distance-normalized contacts) — https://github.com/kaistcbfg/covNormRpkg
- **R** (Runtime environment; uses propagate::fitDistr for 3P-Weibull fitting and FAdist::pweibull3 plus p.adjust('fdr') for per-contact p-values and FDR)

## Examples

```
final_df <- covNormRpkg::contactPval(dist_df, 'fit.pdf'); write.table(final_df, file=gzfile("outFileName2"), row.names=FALSE, col.names=TRUE, sep="\t", quote=FALSE)
```

## Evaluation signals

- Output table contains a p_result_dist entry for every input contact
- All FDR_dist_res values are bounded in [0, 1]
- The 'fdr' (Benjamini-Hochberg) adjustment via R p.adjust was applied to produce FDR_dist_res
- The p-value distribution of the fitted background is approximately uniform-like, with a proper FDR profile per contact
- Reproduced significance scores yield ROC AUC consistent with the paper's reported ~0.85 for discriminating reproducible interactions

## Limitations

- The Weibull background must be fitted only on normalized values within 2-fold of expectation; including extraordinarily high interaction frequencies distorts the null distribution
- Significant-interaction calling is sequential to coverage and distance normalization; skipping or altering upstream filters (e.g. coverage cut-off 200 at 40 kb, zero-valued fragment-pair removal) invalidates the result
- No changelog or explicit version/limitations discussion is available for the covNorm repository

## Evidence
