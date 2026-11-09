---
name: normal-wishart-hierarchical-prior-elicitation
description: Use when calibrating observed methylation levels from bisulfite amplicon
  sequencing standard controls (e.g. AMP = c(0, 25, 50, 75, 100)) jointly across all
  CpGs of a targeted DMR/CGI, and a hierarchical Bayesian random-effects formulation
  with elicited priors is needed.
when_to_use_negative:
- Only a single CpG is of interest — MethylCal's purpose is joint analysis of all
  CpGs within a CGI or DMR, avoiding 'one-at-a-time' CpG calibration
- No standard-control samples with known methylation percentages are available to
  estimate the calibration curve
- The data are average methylation levels across CpGs, since calibration of averages
  neglects variability across CpGs
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MethylCal
  role: R package implementing the four Bayesian regression models M1-M4 with the
    specified priors, model selection via DIC, and calibration/correction functions
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Approximate Bayesian inference engine used to fit the latent Gaussian model
    (Rue et al., 2009)
- name: R
  role: Language in which the model script and MethylCal package are written and executed
  repo: https://github.com/lb664/MethylCal
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:1b506dfd2a4872213882134925af8efa21f65504c071c22240e3cebdc450d3d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lb664/MethylCal
  tools:
  - INLA
  - MethylCal
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Normal-Wishart hierarchical prior elicitation

## Summary

Specifies the prior set-up of MethylCal's joint Bayesian calibration model for bisulfite amplicon sequencing: a Normal-Wishart prior on CpG-specific random intercepts and slopes, a Gamma prior on the model precision, and an inverse-Gamma prior on the observation variance, within four random-effects model variants (M1-M4) fitted with INLA.

## When to use

Use when calibrating observed methylation levels from bisulfite amplicon sequencing standard controls (e.g. AMP = c(0, 25, 50, 75, 100)) jointly across all CpGs of a targeted DMR/CGI, and a hierarchical Bayesian random-effects formulation with elicited priors is needed.

## When NOT to use

- Only a single CpG is of interest — MethylCal's purpose is joint analysis of all CpGs within a CGI or DMR, avoiding 'one-at-a-time' CpG calibration
- No standard-control samples with known methylation percentages are available to estimate the calibration curve
- The data are average methylation levels across CpGs, since calibration of averages neglects variability across CpGs

## Inputs

- Methylation data from bisulfite amplicon sequencing standard controls with known methylation percentages (packaged as BWS_data or Celiac_data)
- AMP vector of actual methylation percentages for the standard controls (e.g. c(0, 25, 50, 75, 100))
- Targeted assay definition (e.g. KCNQ1OT1, H19/IGF2, NFKBIA) with CpG positions within a DMR/CGI
- R script implementing equations (1)-(5) of the MethylCal model with the prior set-up

## Outputs

- Posterior mean predictions of the calibration curve per CpG and AMP combination
- Goodness-of-fit measures mlik (marginal likelihood), DIC and RSS for each estimated model M1-M4
- DIC-selected best model among the four random-effects specifications REij
- INLA posterior estimates of random intercepts, slopes, model precision and observation variance

## How to apply

Load the packaged datasets (BWS_data or Celiac_data) and restructure them with Formatting() using the assay's AMP vector (e.g. c(0, 25, 50, 75, 100) for BWS; c(0, 12.5, 25, 37.5, 50, 62.5, 87.5, 100) for celiac). Write a joint regression model over all CpGs in the targeted DMR/CGI with an RW1 latent Gaussian field whose precision depends on CpG distance, distinguishing models M1-M4 by how the random-effects term REij is specified. Assign the paper's prior set-up: Normal-Wishart for the CpG-specific random intercepts and slopes, a Gamma prior for the model precision, and an inverse-Gamma prior for the observation variance. Fit each assay (e.g. KCNQ1OT1, H19/IGF2, NFKBIA) with INLA's approximate Bayesian inference (Rue et al., 2009), extract mlik, DIC and RSS for each model, and select the best-fitting model via DIC. The rationale is to capture the variability of apparent methylation across CpGs that one-at-a-time calibration (e.g. Moskalev's CPR) neglects.

## Related tools

- **MethylCal** (R package implementing the four Bayesian regression models M1-M4 with the specified priors, model selection via DIC, and calibration/correction functions) — https://github.com/lb664/MethylCal
- **INLA** (Approximate Bayesian inference engine used to fit the latent Gaussian model (Rue et al., 2009))
- **R** (Language in which the model script and MethylCal package are written and executed) — https://github.com/lb664/MethylCal

## Examples

```
data(BWS_data); AMP = c(0, 25, 50, 75, 100); data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18); corr_data = MethylCalCorrection(data, Target = "KCNQ1OT1", n_Control = 15, n_Case = 18)
```

## Evaluation signals

- The R script implementing equations (1)-(5) runs without error for each targeted assay (e.g. KCNQ1OT1, H19/IGF2, NFKBIA)
- Posterior mean predictions agree with the reference MethylCal package fits within the stated numerical tolerance on mlik, DIC and RSS
- The model with the lowest DIC among M1-M4 is the one selected as best fitting the standard-control data
- Corrected methylation levels (equations (9)-(10)) show clearer separation of controls and loss/gain-of-methylation cases in borderline samples than uncorrected levels
- Goodness-of-fit comparison against Moskalev's CPR shows MethylCal captures the variability of apparent methylation at each AMP

## Limitations

- No software version numbers for MethylCal or INLA are stated in the article; reproducibility requires pinning the GitHub package state (no changelog found in the repository)
- Outliers must be handled separately: MethylCalCalibrationOutlierPlot refits models after removing outliers detected via the [l, u] interval with l = Q1 − 1.5IQR and u = Q3 + 1.5IQR, and CpGs 21-23 of PLAGL1 were removed by visual inspection
- INLA is a manual dependency that must be installed separately from https://inla.r-inla-download.org/R/stable; the package also depends on lattice and latticeExtra

## Evidence

- [other] MethylCal comprises four Bayesian regression models distinguished by how the random-effects term REij is specified, and the model that best fits the standard-control data is selected using DIC.: [evidence span withheld — anchor status: not_found]
- [abstract] Joint CpG analysis and prior structure as described in the abstract: "Here, we present a Bayesian calibration tool, MethylCal, which can analyse jointly all CpGs within a CpG island (CGI) or a Differentially Methylated Region (DMR)"
- [methods] INLA is the inference engine behind the MethylCal package: "This package allows the calibration of methylation levels using **MethylCal**, a Bayesian calibration tool based on INLA"
- [intro] The method captures variability of apparent methylation compared to existing calibration tools: "Compared to existing calibration tools (12,15), our method is able to capture with precision the variability of the apparent level of methylation observed after amplification at different actual"
- [readme] Package installation and dependencies documented in the README: [evidence span withheld — anchor status: external_doc]
