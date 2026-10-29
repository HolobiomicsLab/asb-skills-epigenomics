---
name: bayesian-latent-gaussian-model-specification
description: Use when raw apparent methylation levels from bisulfite amplicon sequencing
  standard controls (e.g. 0/25/50/75/100% AMP) must be modeled jointly per CpG to
  derive PCR-bias calibration curves.
when_to_use_negative:
- Only one CpG is of interest — the method's value comes from joint modelling of all
  CpGs in a CGI/DMR, not 'one-at-a-time' CpG calibration
- An existing correction such as Moskalev's CPR is required for direct comparison
  only — that is a separate calibration/correction path, not the Bayesian M1-M4 model
  family
- No standard-control samples with known methylation percentages are available — calibration
  curves cannot be derived
edam_topics:
- http://edamontology.org/topic_3676
- http://edamontology.org/topic_2269
tools:
- name: MethylCal
  role: R package providing Formatting/TableFormatting utilities, calibration plotting
    (MethylCalCalibrationPlot, MethylCalCalibrationOutlierPlot) and correction (MethycalCorrection)
    that wrap the Bayesian model family
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Performs approximate Bayesian inference for the latent Gaussian model instead
    of MCMC; must be installed manually from the R-INLA repository
  repo: http://www.r-inla.org/
- name: R
  role: Language in which the model script implementing equations (1)-(5) and the
    priors is written; MethylCal is distributed as an R package
  repo: https://github.com/lb664/MethylCal
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:96b2d462723470fa16ecebf38457b07253f5acf41a8ac7af156de7218a81205e
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
  - http://edamontology.org/topic_3676
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# bayesian-latent-gaussian-model-specification

## Summary

Specify and fit MethylCal's family of Bayesian hierarchical regression models (M1-M4) that jointly calibrate bisulfite-amplicon methylation levels across all CpGs of a CGI or DMR, using an RW1 latent Gaussian field and INLA's approximate Bayesian inference.

## When to use

Use when raw apparent methylation levels from bisulfite amplicon sequencing standard controls (e.g. 0/25/50/75/100% AMP) must be modeled jointly per CpG to derive PCR-bias calibration curves.

## When NOT to use

- Only one CpG is of interest — the method's value comes from joint modelling of all CpGs in a CGI/DMR, not 'one-at-a-time' CpG calibration
- An existing correction such as Moskalev's CPR is required for direct comparison only — that is a separate calibration/correction path, not the Bayesian M1-M4 model family
- No standard-control samples with known methylation percentages are available — calibration curves cannot be derived

## Inputs

- BWS_data (methylation data on 18 patients potentially affected by Beckwith-Wiedemann syndrome; 15 healthy controls)
- Celiac_data (pyrosequenced methylation levels in three NFkB-related target regions for 17 individuals with celiac disease)
- Standard-control methylation percentages per assay (AMP levels such as 0, 25, 50, 75, 100% for BWS; 0–100% in 12.5% steps for celiac)
- CpG positions/indices within the targeted CGI or DMR (distance-dependent RW1 structure)

## Outputs

- Posterior mean calibration curves per CpG for the best model among M1-M4
- Goodness-of-fit metrics per estimated model: mlik (marginal likelihood), DIC and RSS
- Outlier flags from the posterior predictive interval [l, u] = [Q1 − 1.5IQR, Q3 + 1.5IQR], with models refitted without detected outliers
- Corrected (PCR-bias-adjusted) methylation levels via equations (9) and (10)

## How to apply

Restructure the packaged datasets with Formatting() (BWS_data with AMP = c(0, 25, 50, 75, 100); Celiac_data with AMP = c(0, 12.5, 25, 37.5, 50, 62.5, 87.5, 100)), then write an R script implementing equations (1)-(5): a joint calibration model over all CpGs in the targeted DMR/CGI with an RW1 latent Gaussian field whose precision depends on CpG distance. Specify M1-M4 by varying the random-effects term REij, and assign the paper's priors: Normal-Wishart for the CpG-specific random intercepts and slopes, Gamma for the model precision, and inverse-Gamma for the observation variance. Fit each assay (e.g. KCNQ1OT1, H19/IGF2, NFKBIA) with INLA (Rue et al., 2009) rather than MCMC. Select among M1-M4 via the Deviance Information Criterion, extracting posterior mean calibration curves plus mlik, DIC and RSS per model; optionally refit after outlier removal using the posterior predictive interval [l, u] with l = Q1 − 1.5IQR and u = Q3 + 1.5IQR.

## Related tools

- **MethylCal** (R package providing Formatting/TableFormatting utilities, calibration plotting (MethylCalCalibrationPlot, MethylCalCalibrationOutlierPlot) and correction (MethycalCorrection) that wrap the Bayesian model family) — https://github.com/lb664/MethylCal
- **INLA** (Performs approximate Bayesian inference for the latent Gaussian model instead of MCMC; must be installed manually from the R-INLA repository) — http://www.r-inla.org/
- **R** (Language in which the model script implementing equations (1)-(5) and the priors is written; MethylCal is distributed as an R package) — https://github.com/lb664/MethylCal

## Examples

```
install.packages("INLA", repos = c(getOption("repos"), INLA = "https://inla.r-inla-download.org/R/stable"), dep = TRUE)
```

## Evaluation signals

- Script implementing equations (1)-(5) runs without error for each targeted assay (KCNQ1OT1, H19/IGF2, NFKBIA, etc.)
- Posterior mean predictions agree with reference MethylCal package fits within stated numerical tolerance on mlik, DIC and RSS
- Best model among M1-M4 is selected by lowest DIC, matching the paper's model-choice criterion
- RW1 latent field precision structure respects CpG distances (adjacent CpG effects correlated, distant CpGs decoupling)
- Refitting after outlier removal uses the IQR rule (l = Q1 − 1.5IQR, u = Q3 + 1.5IQR) and no model reuses previously flagged outliers

## Limitations

- No software version numbers are stated for MethylCal, R-INLA or R — reproducibility requires pinning versions yourself
- No accession or availability information is given for the BWS case data set or the celiac case/control data set
- CpGs 21–23 of the PLAGL1 assay were removed by visual inspection, so model inputs can depend on subjective data-cleaning decisions
- R-INLA is a manual-install dependency and the installation 'may take a couple of minutes'; lattice/latticeExtra are pulled in automatically

## Evidence

- [readme] This package allows the calibration of methylation levels using **MethylCal**, a Bayesian calibration tool based on INLA: "This package allows the calibration of methylation levels using
**MethylCal**, a Bayesian calibration tool based on INLA"
- [methods] The former transforms the data into a suitable data format for **MethylCal** analysis: "The former transforms the data into a suitable data format for **MethylCal** analysis"
- [intro] we designed eight independent assays in two CGIs located on SDHC gene promoter and six imprinted DMRs: "we designed eight independent assays in two CGIs located on SDHC gene promoter and six imprinted DMRs"
- [results] the posterior predictive interval [l, u] for outliers detection with l = Q1 − 1.5IQR and u = Q3 + 1.5IQR: "the posterior predictive interval [l, u] for outliers detection with l = Q1 − 1.5IQR and u = Q3 + 1.5IQR"
- [discussion] Written in R and available as an R package on https://github.com/lb664/MethylCal.: "Written in R and available as an R package on https://github.com/lb664/MethylCal."
