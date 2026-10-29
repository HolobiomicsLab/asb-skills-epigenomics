---
name: genomic-distance-background-modeling
description: Use when distance-normalized Hi-C or pcHi-C contact tables (covNorm normDistance
  output) need per-contact significance scores (p-value and FDR) so that reproducible
  significant long-range interactions can be detected.
when_to_use_negative:
- Input is raw or coverage-normalized contact data that has not yet passed covNormRpkg::normDistance,
  since the Weibull null must be fit on distance-normalized values
- Inter-chromosomal (trans) interactions, since covNorm's significance calling here
  focuses on cis-interactions only
- Contacts already carrying significance calls from a different null model (e.g. Fit-Hi-C
  or Mango output) that would be mixed with p_result_dist/FDR_dist_res columns
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: covNorm (covNormRpkg)
  role: R package whose contactPval function performs background restriction, 3P-Weibull
    fitting, p-value and FDR assignment on distance-normalized contacts
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: R
  role: Implementation language; propagate::fitDistr fits the three-parameter Weibull
    and FAdist::pweibull3 computes contact p-values, with p.adjust 'fdr' for multiple
    testing
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:56e34b611b8c3eff07107f70208375a34fccbaed279c5f562cc5ae14a6bfe7d9
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
  - covNorm (covNormRpkg)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# genomic-distance-background-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Call statistically significant chromatin contacts from distance-normalized Hi-C/capture Hi-C data by fitting a three-parameter Weibull null distribution to the genomic-distance-dependent background and computing per-contact p-values and Benjamini-Hochberg FDR, as implemented in covNorm's contactPval step.

## When to use

Use when distance-normalized Hi-C or pcHi-C contact tables (covNorm normDistance output) need per-contact significance scores (p-value and FDR) so that reproducible significant long-range interactions can be detected.

## When NOT to use

- Input is raw or coverage-normalized contact data that has not yet passed covNormRpkg::normDistance, since the Weibull null must be fit on distance-normalized values
- Inter-chromosomal (trans) interactions, since covNorm's significance calling here focuses on cis-interactions only
- Contacts already carrying significance calls from a different null model (e.g. Fit-Hi-C or Mango output) that would be mixed with p_result_dist/FDR_dist_res columns

## Inputs

- covNorm distance-normalized data.frame with columns frag1, frag2, cov_frag1, cov_frag2, freq, dist (tab-separated, e.g. GM19204.chr17.cis.feature at 40 kb resolution)
- Maximum genomic distance parameter (e.g., max_dist=2000000 for Hi-C; 15 kb-2 Mb range for pcHi-C PO-interactions)

## Outputs

- Annotated contact table with fragment coordinates, normalized frequencies, p_result_dist (3P-Weibull p-values), and FDR_dist_res (Benjamini-Hochberg adjusted values), e.g. chr17_contact_pvalues.tsv
- Fit diagnostic plot (e.g. fit.pdf) showing the fitted three-parameter Weibull background distribution

## How to apply

Start from the covNorm distance-normalized data.frame (e.g., chr17 GM19204/GM19240 Hi-C at 40 kb resolution, max_dist=2000000) and restrict the background set to normalized values within 2-fold of the expected contact frequencies, a two-step spline strategy adapted from Fit-Hi-C that prevents extraordinarily high interaction frequencies from distorting the null distribution. Fit the three-parameter Weibull distribution (location, shape, scale) to this background with R 'propagate::fitDistr'. Compute the p-value of every chromatin contact using R 'FAdist::pweibull3' with the fitted parameters, storing it as the p_result_dist column, then adjust p-values for multiple testing with R 'p.adjust' using the 'fdr' method and store as FDR_dist_res. In practice, covNormRpkg::contactPval(dist_df, 'fit.pdf') performs the 3P-Weibull fitting and p-value/FDR assignment, yielding a uniform-like p-value distribution and a proper FDR profile per contact; the annotated contact table is then written out (optionally split per chromosome with saveEachChr).

## Related tools

- **covNorm (covNormRpkg)** (R package whose contactPval function performs background restriction, 3P-Weibull fitting, p-value and FDR assignment on distance-normalized contacts) — https://github.com/kaistcbfg/covNormRpkg
- **R** (Implementation language; propagate::fitDistr fits the three-parameter Weibull and FAdist::pweibull3 computes contact p-values, with p.adjust 'fdr' for multiple testing)

## Examples

```
final_df <- covNormRpkg::contactPval(dist_df, 'fit.pdf')
```

## Evaluation signals

- Every input contact has a non-NA p_result_dist entry in the output table
- All FDR_dist_res values are bounded in [0, 1]
- The 'fdr' (Benjamini-Hochberg) adjustment via p.adjust was applied, and FDR_dist_res >= p_result_dist for each contact
- The p-value distribution is uniform-like after fitting the 3P-Weibull background, and the FDR profile is proper per contact
- The background was restricted to normalized values within 2-fold of expected values before fitting (two-step spline strategy adapted from Fit-Hi-C)

## Limitations

- If the background set is not restricted to normalized values within 2-fold of expectation, extraordinarily high interaction frequencies can distort the null distribution and the fitted Weibull
- For pcHi-C PO-interaction normalization, only promoter-centered long-range interactions over 15 kb and within 2 Mb are considered
- Zero-valued DNA fragment pairs must be removed from the input before normalization; the registration covers copyright only, with no license terms or changelog stated

## Evidence
