---
name: tumour-purity-estimation-from-epialleles
description: Use when RRBS reads from one or more tumour regions plus a matched normal
  sample have been fit with bed_fit and you need to quantify normal-tissue contamination
  and produce cleaned (decontaminated) epiallele profiles per locus.
when_to_use_negative:
- No matched normal sample is available, since xi and rho are computed by comparing
  tumour phi_s against the normal distribution n
- Loci fail the study filters (median read depth <100, fewer than six CpGs, >25% missing
  data, or chromosomes X/Y), since the paper only considered eligible loci
- The goal is phylogenetic or entropy analysis on raw contaminated profiles rather
  than purity estimation/decontamination — apply decontamination first
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: R
  role: Language in which the bed package implements epiallele inference, purity estimation,
    and decontamination
  repo: github.com/james-e-barrett
- name: bed
  role: bed_fit(Y) fits the Bayesian epiallele model on pooled reads; bed_marginal(Y,
    X, W) marginalises over w to give per-locus epiallele distributions phi_s used
    for xi and rho
  repo: github.com/james-e-barrett/bed
- name: devtools
  role: install_github("james-e-barrett/bed") installs the bed package
  repo: github.com/james-e-barrett/bed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:cc49e732faa14531684955cf7dffe860da4de5c694a365ed0aa92cf1eb3ee7e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/github.com/james-e-barrett
  tools:
  - R
  - bed
  - devtools
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# tumour-purity-estimation-from-epialleles

## Summary

Estimates tumour sample purity (rho), the proportion of tumour tissue versus contaminating normal tissue, from epiallele distributions inferred at CpG loci, then decontaminates the tumour epiallele profiles by removing the normal-tissue fraction. Implemented with the bed R package on RRBS data.

## When to use

Use when RRBS reads from one or more tumour regions plus a matched normal sample have been fit with bed_fit and you need to quantify normal-tissue contamination and produce cleaned (decontaminated) epiallele profiles per locus.

## When NOT to use

- No matched normal sample is available, since xi and rho are computed by comparing tumour phi_s against the normal distribution n
- Loci fail the study filters (median read depth <100, fewer than six CpGs, >25% missing data, or chromosomes X/Y), since the paper only considered eligible loci
- The goal is phylogenetic or entropy analysis on raw contaminated profiles rather than purity estimation/decontamination — apply decontamination first

## Inputs

- Per-locus epiallele distributions phi_s for a tumour region, from bed_marginal(Y, X, W) after bed_fit(Y) on pooled RRBS reads
- Matched normal tissue epiallele distribution n
- Epiallele posterior weights from the bed R package (github.com/james-e-barrett/bed)

## Outputs

- Purity estimate rho (e.g. ~0.535 for CRUK0062 region 2) read from the rightmost maximum of the xi density
- Empirical density plot of xi values across loci
- Clipped decontaminated epiallele proportions t_q per locus, all in [0,1]
- Results table (e.g. region2_decontamination_results.csv) with xi values, rho, and decontaminated profiles

## How to apply

Fit the Bayesian epiallele detection model on pooled CRUK0062 reads with bed_fit(Y), then obtain per-locus epiallele distributions phi_s for the tumour region and the matched normal distribution n via bed_marginal(Y, X, W) (marginalising over w). At each locus compute xi = 0.5 * sum_q abs(phi_q - n_q), summing over inferred epialleles and discarding epialleles accounting for less than 5% of observed reads before computing phi_s. Plot the empirical density of xi across eligible loci and take the rightmost maximum as the purity rho (the paper reports rho = 0.535 for tumour region 2). Decontaminate each locus with t_q = (phi_q - (1-rho)*n_q)/rho, then clip values below 0 to 0 and above 1 to 1.

## Related tools

- **R** (Language in which the bed package implements epiallele inference, purity estimation, and decontamination) — github.com/james-e-barrett
- **bed** (bed_fit(Y) fits the Bayesian epiallele model on pooled reads; bed_marginal(Y, X, W) marginalises over w to give per-locus epiallele distributions phi_s used for xi and rho) — github.com/james-e-barrett/bed
- **devtools** (install_github("james-e-barrett/bed") installs the bed package) — github.com/james-e-barrett/bed

## Examples

```
library(devtools); install_github("james-e-barrett/bed")  # then ?bed_fit, bed_fit(Y) and bed_marginal(Y, X, W) to obtain phi_s and n before computing xi and rho
```

## Evaluation signals

- Estimated purity for tumour region 2 matches the reported value of approximately rho = 0.535 (rightmost maximum at xi = 0.53 in Fig. 2)
- All decontaminated epiallele proportions t_q lie in [0,1] after clipping
- xi = 0.5 * sum_q abs(phi_q - n_q) is computed only over epialleles contributing at least 5% of observed reads
- The xi empirical density shows a rightmost maximum from which rho is read off
- Loci meet the eligibility filters: median read depth >= 100, six or more CpGs, no more than 25% missing data, chromosomes X and Y excluded

## Limitations

- Study limited to a single patient (one tumour with seven regions plus one matched normal); larger multi-patient studies deferred to future work
- Decontaminated values t_q outside [0,1] must be clipped, indicating the linear decontamination formula can produce out-of-range proportions
- Epialleles accounting for less than 5% of observed reads are discarded prior to computing phi_s, which affects xi and rho

## Evidence

- [methods] If we plot the empirical density of ξ values the parameter ρ can be estimated from the maximum value of ξ.: "If we plot the empirical density of ξ values the parameter ρ can be estimated from the maximum value of ξ."
- [methods] once estimates of ρ have been obtained we can calculate the 'decontaminated' tumour epiallele profiles at each locus: [evidence span withheld — anchor status: not_found]
- [methods] any epialleles that accounted for less than 5% of observed reads were discarded prior to the computation of ϕs: [evidence span withheld — anchor status: not_found]
- [methods] Any cases where t^μ<0 are set to 0 and any cases where t^μ>1 are set to 1.: [evidence span withheld — anchor status: not_found]
- [readme] To install this R package use the devtools install_github command: "To install this R package use the devtools install_github command"
