---
name: normal-tissue-decontamination
description: Use when multi-region tumour RRBS data plus matched normal tissue have
  been fit with bed_fit and per-locus epiallele distributions phi_s exist, and you
  need to estimate tumour purity and remove normal-tissue contamination before phylogeny
  or entropy analyses.
when_to_use_negative:
- The sample has no matched normal tissue, so n cannot be marginalised and the (1-rho)*n_q
  subtraction cannot be performed
- Loci fail the study's quality filters (median read depth < 100, fewer than six CpGs,
  > 25% missing data, or chromosomes X/Y), making xi estimates unreliable
- The analysis is a single-sample fit with no pooled multi-region data, since the
  model borrows information across tumour regions for statistical power
tools:
- name: R
  role: Language in which the bed package implementing epiallele fitting, marginalisation,
    purity estimation and decontamination is written and run
  repo: github.com/james-e-barrett
- name: devtools
  role: Used to install the bed package from GitHub via install_github before running
    the decontamination workflow
  repo: github.com/james-e-barrett/bed
- name: bed
  role: Provides bed_fit to infer epialleles from pooled reads and bed_marginal to
    obtain the phi_s and n distributions used in the rho estimation and decontamination
    formulas
  repo: github.com/james-e-barrett/bed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:cac6f419af49d8b9c896115555368ff38ce884e3717c9222ff1070669211f842
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/github.com/james-e-barrett
  tools:
  - R
  - bed
  - devtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# normal-tissue-decontamination

## Summary

Estimates tumour sample purity (rho) from per-locus epiallele distributions inferred by Bayesian epiallele detection on RRBS data, then computes 'decontaminated' tumour epiallele profiles by subtracting the matched-normal contribution. Applied to the CRUK0062 lung tumour with the bed R package, it reproduced the reported purity of rho = 0.535 for tumour region 2.

## When to use

Use when multi-region tumour RRBS data plus matched normal tissue have been fit with bed_fit and per-locus epiallele distributions phi_s exist, and you need to estimate tumour purity and remove normal-tissue contamination before phylogeny or entropy analyses.

## When NOT to use

- The sample has no matched normal tissue, so n cannot be marginalised and the (1-rho)*n_q subtraction cannot be performed
- Loci fail the study's quality filters (median read depth < 100, fewer than six CpGs, > 25% missing data, or chromosomes X/Y), making xi estimates unreliable
- The analysis is a single-sample fit with no pooled multi-region data, since the model borrows information across tumour regions for statistical power

## Inputs

- Per-locus epiallele distributions phi_s for the tumour region (from bed_marginal(Y, X, W) after bed_fit(Y) on pooled RRBS reads)
- Matched normal tissue epiallele distribution n at the same loci
- Loci pre-filtered to median read depth >= 100, six or more CpGs, <= 25% missing data, autosomes only, present in two or more samples

## Outputs

- Per-locus xi (tumour-vs-normal epiallele distribution distance) values
- Estimated sample purity rho (rightmost maximum of the xi density; expected ~0.535 for CRUK0062 region 2)
- Clipped decontaminated epiallele profiles t_q per locus, all in [0,1] (e.g. written to region2_decontamination_results.csv)

## How to apply

After bed_fit(Y) on the pooled CRUK0062 reads, obtain per-locus epiallele distributions phi_s for the tumour region and the matched normal distribution n via bed_marginal(Y, X, W). At each locus compute the distance xi = 0.5 * sum_q abs(phi_q - n_q), discarding any epialleles that accounted for less than 5% of observed reads before computing phi_s. Plot the empirical density of xi across eligible loci and estimate purity rho from the rightmost maximum of the density (region 2 gives xi = 0.53, rho = 0.535). Then decontaminate each locus with t_q = (phi_q - (1-rho)*n_q)/rho, clipping any t values below 0 to 0 and above 1 to 1.

## Related tools

- **R** (Language in which the bed package implementing epiallele fitting, marginalisation, purity estimation and decontamination is written and run) — github.com/james-e-barrett
- **devtools** (Used to install the bed package from GitHub via install_github before running the decontamination workflow) — github.com/james-e-barrett/bed
- **bed** (Provides bed_fit to infer epialleles from pooled reads and bed_marginal to obtain the phi_s and n distributions used in the rho estimation and decontamination formulas) — github.com/james-e-barrett/bed

## Examples

```
library(devtools); install_github("james-e-barrett/bed"); phi <- bed_marginal(Y, X, W); rho <- estimate_purity(phi, normal)
```

## Evaluation signals

- The estimated purity for CRUK0062 tumour region 2 matches the reported value of approximately rho = 0.535 (rightmost density maximum at xi = 0.53, as in Fig. 2)
- All decontaminated epiallele proportions t_q lie within [0,1] after clipping
- xi values were computed only at loci passing the filters (median read depth >= 100, >= 6 CpGs, <= 25% missing, autosomal, present in >= 2 samples) and after discarding epialleles contributing < 5% of reads
- The empirical density of xi is unimodal or rightmost-maximum identifiable, allowing a well-defined rho estimate
- Decontaminated profiles reproduce phylogenetically consistent sample representations when fed downstream into tree construction

## Limitations

- The study is limited to a single patient (one tumour with seven regions plus one matched normal); larger multi-patient validation was deferred to future work
- Reads cannot be phased between disjoint loci, so purity/decontamination is necessarily locus-local
- Decontaminated values outside [0,1] must be clipped, which is an ad hoc correction when (phi_q - (1-rho)*n_q)/rho underflows or overflows

## Evidence

- [abstract] Normal tissue contamination can be estimated and corrected: "The degree to which tumour samples are contaminated with normal tissue can be estimated and corrected for."
- [methods] rho estimated from maximum of xi density: "If we plot the empirical density of ξ values the parameter ρ can be estimated from the maximum value of ξ."
- [methods] 5% epiallele discard threshold: "any epialleles that accounted for less than 5% of observed reads were discarded prior to the computation of ϕ"
- [methods] clipping decontaminated proportions to [0,1]: [evidence span withheld — anchor status: not_found]
- [readme] bed package installation via devtools: "To install this R package use the devtools install_github command"
