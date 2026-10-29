---
name: negative-binomial-coverage-bias-correction
description: Use when raw Hi-C/pcHi-C interaction tables (frag1, frag2, coverage values,
  freq, dist) show coverage-driven bias and you need implicit bias correction before
  distance normalization or significant interaction calling.
when_to_use_negative:
- Input is already normalized contact maps or .hic files — covNorm expects raw fragment-level
  interaction tables with coverage columns
- You only need matrix balancing (ICE/SCN) of 40 kb contact maps — the article benchmarks
  those with iced/HiC-spector as alternative baselines, not the coverage model
- Interactions beyond 2 Mb or inter-chromosomal contacts in pcHi-C mode — the PO-normalization
  workflow restricts to promoter-centered interactions over 15 kb and within 2 Mb
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_0622
tools:
- name: covNorm (covNormRpkg)
  role: R package whose normCoverage function performs the negative binomial model-based
    coverage normalization
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: BEDTools coverage
  role: Computes the cov_frag1/cov_frag2 bin coverage values consumed by the normalization
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:8543b102817839bf1b97bfed5994ffd1a49c96ce2ec0b2ec77dfab12187a0488
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
  - BEDTools coverage
  - CHiCAGO
  - R
  - covNorm
  - covNorm (covNormRpkg)
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# negative-binomial-coverage-bias-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

covNorm's implicit normalization step that models experimental and intrinsic 'C'-technology biases as DNA fragment coverage and removes them by fitting expected ligation frequencies with a negative binomial GLM, implemented in the covNormRpkg R package.

## When to use

Use when raw Hi-C/pcHi-C interaction tables (frag1, frag2, coverage values, freq, dist) show coverage-driven bias and you need implicit bias correction before distance normalization or significant interaction calling.

## When NOT to use

- Input is already normalized contact maps or .hic files — covNorm expects raw fragment-level interaction tables with coverage columns
- You only need matrix balancing (ICE/SCN) of 40 kb contact maps — the article benchmarks those with iced/HiC-spector as alternative baselines, not the coverage model
- Interactions beyond 2 Mb or inter-chromosomal contacts in pcHi-C mode — the PO-normalization workflow restricts to promoter-centered interactions over 15 kb and within 2 Mb

## Inputs

- Preprocessed Hi-C/pcHi-C cis interaction table (.feature.gz) with columns frag1, frag2, cov_frag1, cov_frag2, freq, dist
- For pcHi-C PO-interaction normalization: promoter-other interaction tables such as GM12878.po.no0.feature.gz and GM19240.po.no0.feature.gz (frag1 = promoter site)

## Outputs

- Coverage-normalized data.frame (result_df) with adjusted interaction frequencies
- GLM coefficients coeff_cov1 and coeff_cov2
- QC plots (QCplot_coverage_PCC.pdf, QCplot_coverage_heatmap.pdf)

## How to apply

Load the preprocessed interaction table as a data.frame with columns frag1, frag2, cov_frag1, cov_frag2, freq, and dist (computed with BEDTools coverage at 40 kb resolution; zero-valued fragment pairs already removed). Run covNormRpkg::filterInputDF, then covNormRpkg::normCoverage, which shuffles 50% of the fragment 1 and 2 coverage values to prevent order bias and fits a GLM to estimate expected ligation frequencies at given coverages; for pcHi-C promoter-other interactions use do_shuffle=FALSE with cov1_thresh=200 (promoter side) and cov2_thresh=50 (other side). The hypothesis is that experimental and intrinsic biases (restriction sites, digestion efficiency, ligation efficiency, mappability) can be combined in the form of coverage, so adjusting ligated read numbers by the fitted expectation removes them. Inspect normCoverage's coeff_cov1/coeff_cov2 and QC plots via checkFreqCovPCC and plotCovNormRes before passing result_df to normDistance and contactPval. The normalized result_df feeds subsequent distance-dependent background removal and Weibull-based significant interaction calling.

## Related tools

- **covNorm (covNormRpkg)** (R package whose normCoverage function performs the negative binomial model-based coverage normalization) — https://github.com/kaistcbfg/covNormRpkg
- **BEDTools coverage** (Computes the cov_frag1/cov_frag2 bin coverage values consumed by the normalization) — https://github.com/arq5x/bedtools2

## Examples

```
cov_result <- covNormRpkg::normCoverage(raw_data_filter, do_shuffle=FALSE, cov1_thresh=200, cov2_thresh=50); cov_df <- cov_result$result_df
```

## Evaluation signals

- normCoverage returns fitted coefficients coeff_cov1 and coeff_cov2; verify they converge and are reported for both fragment sides
- QC plot from checkFreqCovPCC should show improved Pearson correlation between frequency and coverage after normalization
- Downstream HiC-spector reproducibility scores on normalized maps should be enhanced or similar to raw data (paired t-test vs raw was significant, p = 0.019)
- The shuffled-coverage design means results should be robust to input row ordering; a run with do_shuffle=TRUE should give comparable coefficients
- Normalized result_df must retain the input schema (frag1, frag2, freq, dist) so normDistance(max_dist=2000000) and contactPval can proceed

## Limitations

- Coverage threshold choices are user-defined (default 200 at 40 kb resolution; pcHi-C example used 200 for promoter side and 50 for the 'other' side), and mis-setting them changes filtering behavior
- 50% of coverage values are shuffled by default to avoid sorted-order bias, which the pcHi-C PO workflow disables (do_shuffle=FALSE)
- No software version number or changelog is stated for covNorm in the paper; no license terms beyond copyright registration at the Korea Copyright Commission

## Evidence

- [intro] we hypothesized that the experimental and intrinsic biases can be combined in the form of coverage: [evidence span withheld — anchor status: not_found]
