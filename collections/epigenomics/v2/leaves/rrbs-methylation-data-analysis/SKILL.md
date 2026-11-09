---
name: rrbs-methylation-data-analysis
description: Use when RRBS reads from multi-region tumour samples plus matched normal
  tissue have been fitted with bed_fit and per-locus epiallele proportions are needed,
  purity (rho) must be estimated from the xi density, or tumour profiles must be decontaminated
  before phylogeny/entropy analysis.
when_to_use_negative:
- The sample has no matched normal tissue, so n cannot be marginalised and xi cannot
  separate tumour from contaminating signal
- Loci fail the article's filters (median read depth < 100, fewer than 6 CpGs, > 25%
  missing data, or chromosomes X/Y), where phi and n are unreliable
- The goal is differential methylation testing between sample groups rather than purity
  estimation and decontamination of epiallele profiles
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_2640
tools:
- name: bed
  role: R package providing bed_fit(Y) to infer epialleles X, W and noise from pooled
    RRBS reads, and bed_marginal(Y, X, W) to obtain per-locus epiallele distributions
    used for xi, rho and decontamination
  repo: github.com/james-e-barrett/bed
- name: devtools
  role: Used to install the bed package from GitHub via install_github
  repo: github.com/james-e-barrett/bed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:0fcadc6ce1f3b01ad08d7d69aecdf28896f8f1399b5d6d921c7dc38b06b8c525
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - bed
  - devtools
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# rrbs-methylation-data-analysis

## Summary

Estimates tumour sample purity (rho) from RRBS-derived epiallele distributions by comparing tumour vs matched-normal per-locus epiallele profiles, then decontaminates profiles with t_q = (phi_q - (1-rho)*n_q)/rho. Applied with the R 'bed' Bayesian Epiallele Detection package.

## When to use

Use when RRBS reads from multi-region tumour samples plus matched normal tissue have been fitted with bed_fit and per-locus epiallele proportions are needed, purity (rho) must be estimated from the xi density, or tumour profiles must be decontaminated before phylogeny/entropy analysis.

## When NOT to use

- The sample has no matched normal tissue, so n cannot be marginalised and xi cannot separate tumour from contaminating signal
- Loci fail the article's filters (median read depth < 100, fewer than 6 CpGs, > 25% missing data, or chromosomes X/Y), where phi and n are unreliable
- The goal is differential methylation testing between sample groups rather than purity estimation and decontamination of epiallele profiles

## Inputs

- Per-sample RRBS read matrices Y of CpG methylation states at eligible loci (median read depth >= 100, >= 6 CpGs, <= 25% missing data, autosomes only)
- bed_fit output: inferred epialleles X, assignments W, and noise hyperparameter from pooled reads across all tissue samples
- Per-locus marginalised epiallele distributions phi_s for each tumour region and n for the matched normal sample

## Outputs

- Per-locus xi values (0.5 * L1 distance between phi and n over retained epialleles)
- Estimated tumour purity rho (e.g. 0.535 for CRUK0062 region 2)
- Clipped decontaminated epiallele profiles t_q per locus in [0,1], e.g. written to region2_decontamination_results.csv

## How to apply

Fit the Bayesian epiallele model on pooled CRUK0062 RRBS reads with bed_fit(Y), then obtain per-locus tumour and normal epiallele distributions phi_s and n via bed_marginal(Y, X, W), discarding epialleles accounting for less than 5% of observed reads before computing phi_s. At each locus compute xi = 0.5 * sum_q abs(phi_q - n_q) over the retained epialleles; the normal sample has xi near 0 while contaminated tumour regions push xi toward 1-rho. Plot the empirical density of xi across loci and take rho as the rightmost maximum (e.g. rho = 0.535 for tumour region 2, rightmost mode at xi ≈ 0.53). Decontaminate each locus with t_q = (phi_q - (1-rho)*n_q)/rho, then clip values below 0 to 0 and above 1 to 1, since (1-rho) is an upper-bound estimate of the contaminating normal fraction.

## Related tools

- **bed** (R package providing bed_fit(Y) to infer epialleles X, W and noise from pooled RRBS reads, and bed_marginal(Y, X, W) to obtain per-locus epiallele distributions used for xi, rho and decontamination) — github.com/james-e-barrett/bed
- **devtools** (Used to install the bed package from GitHub via install_github) — github.com/james-e-barrett/bed

## Examples

```
library(devtools); install_github("james-e-barrett/bed"); fit <- bed_fit(Y); phi <- bed_marginal(Y, fit$X, fit$W); xi <- 0.5 * abs(phi_tumour2 - phi_normal)
```

## Evaluation signals

- Estimated rho for CRUK0062 tumour region 2 should match the reported value of approximately 0.535, appearing as the rightmost maximum of the xi density (rightmost mode at xi ≈ 0.53 in Fig. 2)
- The normal sample's xi density should peak near 0, confirming the metric separates normal from contaminated tumour tissue
- All decontaminated epiallele proportions t_q must lie within [0,1] after clipping
- Epialleles contributing less than 5% of observed reads must be excluded before computing phi_s and xi
- xi values must equal 0.5 * sum_q |phi_q - n_q| using only retained epialleles at each eligible locus

## Limitations

- Study and validation are limited to a single patient (CRUK0062, seven tumour regions plus one matched normal); larger multi-patient evaluation was deferred to future work
- Reads cannot be phased between disjoint loci, so purity/decontamination operates locus-by-locus and may miss correlated epiallele structure between close loci
- Decontamination uses (1-rho) as an upper-bound estimate of the normal fraction, requiring clipping of t_q into [0,1]

## Evidence

- [abstract] The degree to which tumour samples are contaminated with normal tissue can be estimated and corrected for.: "The degree to which tumour samples are contaminated with normal tissue can be estimated and corrected for."
- [methods] purity from xi density: "If we plot the empirical density of ξ values the parameter ρ can be estimated from the maximum value of ξ."
- [methods] decontamination formula and clipping: [evidence span withheld — anchor status: not_found]
- [methods] discard low-weight epialleles before phi: "any epialleles that accounted for less than 5% of observed reads were discarded prior to the computation of ϕ s"
- [readme] bed installation: "To install this R package use the devtools install_github command:

    library(devtools)
    install_github("james-e-barrett/bed")"
