---
name: empirical-density-estimation
description: Use when tumour RRBS samples contain normal tissue contamination and
  per-locus epiallele distributions are available for tumour regions and matched normal
  tissue, so that purity can be estimated from the empirical density of xi.
when_to_use_negative:
- Reads cannot be phased between disjoint loci, so do not apply this to infer purity
  or heterogeneity across phasing assumptions between separate loci
- Do not use on loci failing the study filters (median read depth < 100, fewer than
  six CpGs, > 25% missing data, or chromosomes X/Y)
- The model was validated on a single patient (one tumour with seven regions plus
  one matched normal); do not assume multi-patient generalisation without re-estimating
  rho per sample
tools:
- name: bed
  role: R package providing bed_fit(Y) to infer epialleles and bed_marginal(Y, X,
    W) to marginalise over w and obtain the per-locus epiallele distributions used
    for the xi statistic
  repo: https://github.com/james-e-barrett/bed
- name: devtools
  role: Used to install the bed package from GitHub via install_github
  repo: https://github.com/james-e-barrett/bed
- name: R
  role: Language in which the epiallele fitting, xi computation, density estimation,
    and decontamination are performed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:5e2776de718d446ad31eb849f71f22201491961b324a4edf22159d026b6004fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/james-e-barrett/bed
  tools:
  - R
  - bed
  - devtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# empirical-density-estimation

## Summary

Estimate tumour sample purity from Bayesian epiallele detections by computing the per-locus epiallele divergence xi = 0.5 * sum_q |phi_q - n_q| and reading the rightmost maximum of the empirical density of xi as the purity rho (rho = 0.535 for CRUK0062 tumour region 2).

## When to use

Use when tumour RRBS samples contain normal tissue contamination and per-locus epiallele distributions are available for tumour regions and matched normal tissue, so that purity can be estimated from the empirical density of xi.

## When NOT to use

- Reads cannot be phased between disjoint loci, so do not apply this to infer purity or heterogeneity across phasing assumptions between separate loci
- Do not use on loci failing the study filters (median read depth < 100, fewer than six CpGs, > 25% missing data, or chromosomes X/Y)
- The model was validated on a single patient (one tumour with seven regions plus one matched normal); do not assume multi-patient generalisation without re-estimating rho per sample

## Inputs

- Per-locus epiallele distributions phi_s for tumour region 2 obtained from bed_marginal(Y, X, W) after bed_fit(Y) on pooled CRUK0062 RRBS reads
- Matched normal epiallele distribution n (marginalised over w)
- CRUK0062 RRBS read data from seven tumour regions plus one matched normal sample, restricted to loci with median read depth >= 100, six or more CpGs, <= 25% missing data, and no chromosomes X/Y

## Outputs

- Empirical density of xi values across eligible loci
- Estimated purity rho (expected ~0.535 for region 2)
- Clipped decontaminated epiallele profiles t_q in [0,1] per locus
- CSV of xi values, estimated rho, and decontaminated profiles (region2_decontamination_results.csv)

## How to apply

Fit the Bayesian epiallele detection model with bed_fit(Y) on reads pooled from all tissue samples, then obtain per-locus epiallele distributions phi_s for tumour region 2 and the matched normal distribution n via bed_marginal(Y, X, W). At each eligible locus compute xi = 0.5 * sum_q |phi_q - n_q| over inferred epialleles, first discarding any epialleles that accounted for less than 5% of observed reads before computing phi_s. Plot the empirical density of xi across all eligible loci and estimate the purity rho from the rightmost maximum of the density; the paper reports rho = 0.535 (rightmost maximum at xi = 0.53 in Fig. 2) for region 2. Decontaminate each locus using t_q = (phi_q - (1-rho)*n_q)/rho, then clip any decontaminated proportions below 0 to 0 and above 1 to 1, since t_q outside [0,1] is inadmissible.

## Related tools

- **bed** (R package providing bed_fit(Y) to infer epialleles and bed_marginal(Y, X, W) to marginalise over w and obtain the per-locus epiallele distributions used for the xi statistic) — https://github.com/james-e-barrett/bed
- **devtools** (Used to install the bed package from GitHub via install_github) — https://github.com/james-e-barrett/bed
- **R** (Language in which the epiallele fitting, xi computation, density estimation, and decontamination are performed)

## Examples

```
library(devtools); install_github("james-e-barrett/bed")
```

## Evaluation signals

- Estimated purity rho for region 2 matches the reported value of approximately 0.535 (rightmost maximum of the xi density at xi = 0.53, as in Fig. 2)
- All decontaminated epiallele proportions lie within [0,1] after clipping
- xi values were computed only after discarding epialleles accounting for less than 5% of observed reads
- Only loci with median read depth >= 100, >= 6 CpGs, <= 25% missing data, and excluding chromosomes X/Y contributed xi values
- Decontamination follows t_q = (phi_q - (1-rho)*n_q)/rho exactly, then clips to [0,1]

## Limitations

- Reads cannot be phased between disjoint loci, so the number of epialleles and the entropy may be correlated only between close loci
- Study is limited to a single patient (one tumour with seven regions plus one matched normal); larger multi-patient studies are deferred to future work
- Any decontaminated proportions falling below 0 or above 1 are clipped, meaning the linear decontamination formula can be inadmissible for extreme mixtures

## Evidence

- [methods] Estimation of sample purity: "If we plot the empirical density of ξ values the parameter ρ can be estimated from the maximum value of ξ."
- [methods] Decontamination formula and clipping: [evidence span withheld — anchor status: not_found]
- [methods] Epiallele filter before computing phi_s: "any epialleles that accounted for less than 5% of observed reads were discarded prior to the computation of ϕ s"
- [readme] bed package installation: "To install this R package use the devtools install_github command: library(devtools) install_github("james-e-barrett/bed")"
- [intro] Purity/decontamination protocol objective: "We have developed a protocol for estimating the tumour sample purity and consequently decontaminating the inferred epiallele profiles."
