---
name: hi-c-significant-contact-calling
description: Use when you have covNorm distance-normalized contact tables (frag1/frag2,
  cov, freq, dist) and need p-value/FDR-based significant long-range interaction calling.
when_to_use_negative:
- The input is raw ligation frequencies that have not been coverage-normalized (normCoverage)
  and distance-normalized (normDistance) by covNorm; p-values on raw counts reflect
  uncorrected coverage and distance biases
- The goal is differential or inter-chromosomal interaction analysis — the article
  focuses on cis-interactions and filters inter-chromosomal contacts, and pcHi-C P-P
  vs P-O interactions are processed separately (promoter side coverage threshold 200,
  other side 50)
- You need reproducibility scoring between biological replicates rather than per-contact
  significance — use HiC-spector's run_reproducibility script instead
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: covNorm
  role: Provides the covNormRpkg::contactPval function that fits the 3P-Weibull background,
    computes contact p-values and FDR on the distance-normalized output
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: R
  role: Implementation language; uses propagate::fitDistr for 3P-Weibull fitting,
    FAdist::pweibull3 for p-value computation, and p.adjust('fdr') for multiple-testing
    correction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:5f97a7e7dd355c2e11cd39442b2ff87b7bd3cb6e9621da1b64acb748c4f8e3f7
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
  - R
  - covNorm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# hi-c-significant-contact-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calls statistically significant chromatin interactions from covNorm distance-normalized Hi-C/pcHi-C contacts by fitting a three-parameter Weibull background (via propagate::fitDistr) and computing per-contact p-values (FAdist::pweibull3) and Benjamini-Hochberg FDR (p.adjust).

## When to use

Use when you have covNorm distance-normalized contact tables (frag1/frag2, cov, freq, dist) and need p-value/FDR-based significant long-range interaction calling.

## When NOT to use

- The input is raw ligation frequencies that have not been coverage-normalized (normCoverage) and distance-normalized (normDistance) by covNorm; p-values on raw counts reflect uncorrected coverage and distance biases
- The goal is differential or inter-chromosomal interaction analysis — the article focuses on cis-interactions and filters inter-chromosomal contacts, and pcHi-C P-P vs P-O interactions are processed separately (promoter side coverage threshold 200, other side 50)
- You need reproducibility scoring between biological replicates rather than per-contact significance — use HiC-spector's run_reproducibility script instead

## Inputs

- Distance-normalized covNorm output data.frame with columns frag1, frag2, cov_frag1, cov_frag2, freq, dist plus normalized frequency (e.g. from covNormRpkg::normDistance with max_dist=2000000)
- Sample Hi-C/pcHi-C feature tables (gzipped) such as GM19204.chr17.cis.feature.gz and GM19240.chr17.cis.feature.gz (frag1, frag2, cov_frag1, cov_frag2, freq, dist) at 40 kb resolution
- Fitted 3P-Weibull parameters (location, shape, scale) estimated by propagate::fitDistr on the ≤2-fold-of-expectation background set

## Outputs

- Contact table with fragment coordinates, normalized frequencies, per-contact p-values (p_result_dist from FAdist::pweibull3), and Benjamini-Hochberg FDR (FDR_dist_res from p.adjust), e.g. chr17_contact_pvalues.tsv
- Weibull fit QC PDF (e.g. 'fit.pdf') from covNormRpkg::contactPval
- Optional per-chromosome split outputs via covNormRpkg::saveEachChr

## How to apply

First ensure the input is a covNorm distance-normalized data.frame (normDistance output, e.g. max_dist=2000000 for 2 Mb). Restrict the background set to normalized frequencies within 2-fold of the distance-dependent expected values, following the two-step spline strategy adapted from Fit-Hi-C, so extraordinarily high interaction frequencies do not distort the null distribution. Fit the three-parameter Weibull distribution (location, shape, scale) to this background using R 'propagate::fitDistr' (the covNormRpkg contactPval function wraps this and draws a fit QC plot). Compute the p-value of every contact with 'FAdist::pweibull3' using the fitted parameters, then adjust for multiple testing with 'p.adjust' using the 'fdr' (Benjamini-Hochberg) method. The result (columns such as p_result_dist and FDR_dist_res) is appended to the contact table and written out alongside fragment coordinates and normalized frequencies.

## Related tools

- **covNorm** (Provides the covNormRpkg::contactPval function that fits the 3P-Weibull background, computes contact p-values and FDR on the distance-normalized output) — https://github.com/kaistcbfg/covNormRpkg
- **R** (Implementation language; uses propagate::fitDistr for 3P-Weibull fitting, FAdist::pweibull3 for p-value computation, and p.adjust('fdr') for multiple-testing correction)

## Examples

```
library(covNormRpkg); final_df <- covNormRpkg::contactPval(dist_df, 'fit.pdf'); write.table(final_df, file=gzfile("outFileName2"), row.names=FALSE, col.names=TRUE, sep="\t", quote=FALSE)
```

## Evaluation signals

- The output table contains a p_result_dist entry for every input contact (row count preserved from the distance-normalized input)
- All FDR_dist_res values are bounded in [0, 1], and p-values show a uniform-like distribution under the null background
- Confirm the 'fdr' (Benjamini-Hochberg) method was applied via p.adjust, not raw p-values
- The 3P-Weibull fit PDF (fit.pdf) shows a fit restricted to normalized values within 2-fold of expected frequencies, excluding extraordinarily high interaction frequencies
- A final data.frame with both distance normalization and significant-interaction columns is written (outFileName2 in the README example)

## Limitations

- The significance-calling step assumes the null background is captured by fitting on contacts within 2-fold of expected values; extraordinarily high-frequency contacts are excluded from the null fit and could bias results if included
- No changelog or explicit version constraints are documented for covNormRpkg (source registered at Korea Copyright Commission, C-2021-022800); no explicit limitations of covNorm are discussed by the authors
- For pcHi-C PO-interaction normalization, the pipeline must be restricted to 15 kb–2 Mb interactions and P-P vs P-O handled separately with asymmetric coverage thresholds (promoter 200, other 50)

## Evidence

- [results] After fitting the 3P-Weibull background on normalized values below 2-fold expectation, covNorm's procedure yields a uniform-like p-value distribution and a proper FDR profile: [evidence span withheld — anchor status: not_found]
