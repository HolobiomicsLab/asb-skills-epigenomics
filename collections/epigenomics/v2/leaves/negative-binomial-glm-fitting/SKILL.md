---
name: negative-binomial-glm-fitting
description: Use when Hi-C or pcHi-C fragment-pair tables (frag1, frag2, cov_frag1,
  cov_frag2, freq, dist) contain coverage-driven ligation bias that must be estimated
  and removed before distance normalization or replicate comparison.
when_to_use_negative:
- The data are already coverage-normalized and only distance-dependent background
  removal (normDistance with max_dist=2000000) is needed
- Working with pcHi-C PO-interactions without lowering the 'other'-side coverage threshold
  (cov2_thresh=50) and disabling shuffling — the default Hi-C parameters would mis-filter
  promoter-side bins
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: covNorm
  role: Provides normCoverage, which wraps MASS::glm.nb negative binomial fitting,
    filtering, shuffling, and residual computation
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: R
  role: Runtime environment; MASS::glm.nb performs the negative binomial GLM fitting
- name: bedtools
  role: coverageBed computes the bin coverage values (cov_frag1, cov_frag2) used as
    GLM covariates
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_004
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:be423e1bfbfbf602629d317364c207a742483dec4e6d89b38a4a5842f60982c3
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
  - HiCcompare
  - R
  - bedtools
  - covNorm
  - iced
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# negative-binomial-glm-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fit a negative binomial generalized linear model, log(uij) = β0 + β1*Ci + β2*Cj, to Hi-C/capture Hi-C fragment-pair data to estimate coverage bias coefficients, which are then used to compute coverage-bias-removed residuals via covNormRpkg::normCoverage.

## When to use

Use when Hi-C or pcHi-C fragment-pair tables (frag1, frag2, cov_frag1, cov_frag2, freq, dist) contain coverage-driven ligation bias that must be estimated and removed before distance normalization or replicate comparison.

## When NOT to use

- The data are already coverage-normalized and only distance-dependent background removal (normDistance with max_dist=2000000) is needed
- Working with pcHi-C PO-interactions without lowering the 'other'-side coverage threshold (cov2_thresh=50) and disabling shuffling — the default Hi-C parameters would mis-filter promoter-side bins

## Inputs

- Tab-delimited gzipped fragment-pair table with columns frag1, frag2, cov_frag1, cov_frag2, freq, dist (e.g. GM19204.chr17.cis.feature.gz)
- Coverage profiles at 40 kb resolution computed with BEDTools coverage

## Outputs

- Coverage-normalized result data.frame whose capture_res column holds the coverage-normalized interaction frequency, plus estimated coefficients coeff_cov1 and coeff_cov2
- QC plots of coverage-frequency correlation (checkFreqCovPCC) and coverage normalization heatmap (plotCovNormRes)

## How to apply

Load the fragment-pair table in the covNorm input format and preprocess it with covNormRpkg::filterInputDF, which removes zero-valued pairs and filters fragments with coverage below the default threshold of 200 at 40 kb resolution; shuffle 50% of fragment 1 and 2 coverage values (do_shuffle default TRUE) to prevent bias from sorted input order. Fit the negative binomial GLM log(uij) = β0 + β1*Ci + β2*Cj using MASS::glm.nb via covNormRpkg::normCoverage to estimate coverage bias coefficients (retrieved as result$coeff_cov1 and result$coeff_cov2). Compute bias-removed residuals Rij = Yij / exp(β̂0 + β̂1*Ci + β̂2*Cj) and write result$result_df as the coverage-normalized table. For pcHi-C PO-interaction normalization, restrict to 15 kb–2 Mb interactions and set normCoverage(raw_data_filter, do_shuffle=FALSE, cov1_thresh=200, cov2_thresh=50).

## Related tools

- **covNorm** (Provides normCoverage, which wraps MASS::glm.nb negative binomial fitting, filtering, shuffling, and residual computation) — https://github.com/kaistcbfg/covNormRpkg
- **R** (Runtime environment; MASS::glm.nb performs the negative binomial GLM fitting)
- **bedtools** (coverageBed computes the bin coverage values (cov_frag1, cov_frag2) used as GLM covariates) — https://github.com/arq5x/bedtools2

## Examples

```
cov_result <- covNormRpkg::normCoverage(raw_data_filter); cov_df <- cov_result$result_df; write.table(cov_df, file=gzfile("outFileName1"), sep="\t", quote=FALSE)
```

## Evaluation signals

- cov_result returns non-empty coeff_cov1 and coeff_cov2 coefficient estimates from the negative binomial fit
- Normalized output table exists and its capture_res column is populated for every retained fragment pair
- Normalized frequencies equal raw frequencies divided by exp(β̂0 + β̂1*Ci + β̂2*Cj) within numerical tolerance
- checkFreqCovPCC QC plot shows reduced correlation between coverage and frequency after normalization
- Zero-valued pairs and fragments below the coverage-200 threshold are absent from the fitted data

## Limitations

- Coverage cut-off threshold of 200 at 40 kb resolution is a default the user must adjust for other resolutions or protocols
- 50% coverage-value shuffling is needed because sorted input order can bias the fit; sorted pcHi-C PO-interaction data instead disables shuffling
- No explicit version number stated for covNorm; software registered at the Korea Copyright Commission with no license terms beyond copyright, commercial use requires contacting the authors

## Evidence

- [other] Fit the negative binomial GLM log(u_ij) = b0 + b1*Ci + b2*Cj via MASS::glm.nb to remove coverage bias: [evidence span withheld — anchor status: not_found]
