---
name: false-discovery-rate-correction
description: Use when the goal is to call significant chromatin contacts from covNorm
  distance-normalized interaction data, i.e. a p-value and FDR are needed for each
  pairwise contact after removal of distance-dependent background.
when_to_use_negative:
- Inter-chromosomal interactions are not covered — the article filters them out since
  only cis-interactions are analyzed
- Contacts below coverage cut-offs (default 200 at 40 kb resolution; 200 promoter
  side / 50 other side for pcHi-C PO-interactions) — low-coverage fragments should
  be filtered before significance calling
- Raw (non-normalized) contact maps — the Weibull background must be fit on distance-normalized
  values, not raw interaction frequencies
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: covNorm
  role: R package providing contactPval(), which fits the 3P-Weibull null, computes
    per-contact p-values and applies FDR adjustment for significant interaction calling
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: R
  role: Runtime environment; uses 'propagate::fitDistr' for 3P-Weibull fitting, 'FAdist::pweibull3'
    for p-values, and 'p.adjust' with the 'fdr' method for multiple-testing correction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:577b78c1fb14321d43079a879a115c22692e148124091bf7665d18a51c0acb90
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# false-discovery-rate-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns per-contact statistical significance in the covNorm pipeline by fitting a 3P-Weibull background to distance-normalized Hi-C/pcHi-C contact frequencies and converting fitted p-values (p_result_dist) into Benjamini-Hochberg FDR values (FDR_dist_res) for significant long-range chromatin interaction calling.

## When to use

Use when the goal is to call significant chromatin contacts from covNorm distance-normalized interaction data, i.e. a p-value and FDR are needed for each pairwise contact after removal of distance-dependent background.

## When NOT to use

- Inter-chromosomal interactions are not covered — the article filters them out since only cis-interactions are analyzed
- Contacts below coverage cut-offs (default 200 at 40 kb resolution; 200 promoter side / 50 other side for pcHi-C PO-interactions) — low-coverage fragments should be filtered before significance calling
- Raw (non-normalized) contact maps — the Weibull background must be fit on distance-normalized values, not raw interaction frequencies

## Inputs

- covNorm distance-normalized data.frame (output of covNormRpkg::normDistance) with fragment coordinates, normalized frequencies and genomic distances
- Hi-C/pcHi-C feature table input (frag1, frag2, cov_frag1, cov_frag2, freq, dist), e.g. GM19204.chr17.cis / GM19240.chr17.cis sample data

## Outputs

- Annotated contact table (e.g. chr17_contact_pvalues.tsv) with fragment coordinates, normalized frequencies, p_result_dist p-values and FDR_dist_res multiple-testing-adjusted values
- Fitted 3P-Weibull background distribution parameters (location, shape, scale) with diagnostic fit plot ('fit.pdf')

## How to apply

Start from the covNorm distance-normalized data.frame (e.g. chr17 GM19204/GM19240 Hi-C at 40 kb, max_dist=2000000) produced by covNormRpkg::normDistance. Restrict the background set to normalized values within 2-fold of the expected values (two-step spline strategy adapted from Fit-Hi-C) so extraordinarily high interaction frequencies do not distort the null distribution. Fit the three-parameter Weibull distribution (location, shape, scale) to this background with R 'propagate::fitDistr', then compute each contact's p-value with 'FAdist::pweibull3' and store it as the p_result_dist column. Adjust p-values for multiple testing with R 'p.adjust' using the 'fdr' (Benjamini-Hochberg) option and store the result as the FDR_dist_res column before writing the annotated contact table (e.g. chr17_contact_pvalues.tsv).

## Related tools

- **covNorm** (R package providing contactPval(), which fits the 3P-Weibull null, computes per-contact p-values and applies FDR adjustment for significant interaction calling) — https://github.com/kaistcbfg/covNormRpkg
- **R** (Runtime environment; uses 'propagate::fitDistr' for 3P-Weibull fitting, 'FAdist::pweibull3' for p-values, and 'p.adjust' with the 'fdr' method for multiple-testing correction)

## Examples

```
final_df <- covNormRpkg::contactPval(dist_df, 'fit.pdf'); write.table(final_df, file=gzfile('outFileName2'), row.names=FALSE, col.names=TRUE, sep='\t', quote=FALSE)
```

## Evaluation signals

- Every input contact has a non-NA entry in the p_result_dist column of the output table
- All FDR_dist_res values are bounded in [0, 1]
- The 'fdr' (Benjamini-Hochberg) adjustment was applied via p.adjust, and FDR values are monotonically consistent with the ranked p-values
- The fitted p-value distribution on the background set is approximately uniform, and the FDR profile is proper (FDR does not exceed p-value for small p-values)
- The 3P-Weibull fit was computed only on normalized values within 2-fold of the expected values, as visualized in the diagnostic fit plot

## Limitations

- The null model is a 3P-Weibull fit on a restricted background (within 2-fold of expectation); extraordinarily high interaction frequencies are excluded from fitting, so the background assumptions may not hold for all distance regimes
- The article's demonstrated workflow applies to cis-interactions up to 2 Mb (pcHi-C long-range interactions over 15 kb and within 2 Mb); behavior outside these scopes is not validated in the paper
- No changelog or explicit version coverage is available for the repository beyond covNormRpkg_1.1.0 referenced in the README build command

## Evidence
