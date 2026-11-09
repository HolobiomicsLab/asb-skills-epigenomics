---
name: inla-approximate-bayesian-inference
description: Use when calibrating observed/apparent methylation levels from bisulfite
  amplicon sequencing of standard controls (known AMP percentages) across multiple
  CpGs in a CGI or DMR, and a joint Bayesian model with latent Gaussian field inference
  is needed.
when_to_use_negative:
- Data are ordinary case/control methylation measurements without standard controls
  of known AMP — MethylCal calibration curves require five or more standard controls
  with known methylation percentages
- The goal is one-at-a-time CpG calibration or calibration of average methylation
  — MethylCal explicitly analyses all CpGs in a CGI/DMR jointly to avoid this
- You need a different calibration method than the Bayesian one — the package separately
  provides WarneckeCalibrationPlot and MoskalevCorrection/MoskalevCalibrationPlot
edam_operation: http://edamontology.org/operation_0004
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3053
tools:
- name: MethylCal
  role: R package implementing the Bayesian calibration models (M1–M4) fitted with
    INLA; provides Formatting, MethylCalCalibrationPlot and MethylCalCorrection
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: R-INLA backend performing the approximate Bayesian inference (Rue et al.,
    2009); must be installed manually before MethylCal
  repo: http://www.r-inla.org/
- name: R
  role: Environment in which MethylCal is written and the calibration/correction workflow
    is run
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:09746360327124b9f76e20f6cb585407eba893e36e86c46c5c47617fdf36a3f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lb664/MethylCal
  tools:
  - INLA
  - MethylCal
  - R
  edam_operation: http://edamontology.org/operation_0004
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# inla-approximate-bayesian-inference

## Summary

Fit MethylCal's joint Bayesian calibration models (M1–M4) for bisulfite amplicon sequencing standard controls using INLA's approximate Bayesian inference, avoiding MCMC while yielding posterior mean calibration curves and goodness-of-fit measures (mlik, DIC, RSS) per assay.

## When to use

Use when calibrating observed/apparent methylation levels from bisulfite amplicon sequencing of standard controls (known AMP percentages) across multiple CpGs in a CGI or DMR, and a joint Bayesian model with latent Gaussian field inference is needed.

## When NOT to use

- Data are ordinary case/control methylation measurements without standard controls of known AMP — MethylCal calibration curves require five or more standard controls with known methylation percentages
- The goal is one-at-a-time CpG calibration or calibration of average methylation — MethylCal explicitly analyses all CpGs in a CGI/DMR jointly to avoid this
- You need a different calibration method than the Bayesian one — the package separately provides WarneckeCalibrationPlot and MoskalevCorrection/MoskalevCalibrationPlot

## Inputs

- BWS_data — packaged methylation data for patients potentially affected by Beckwith-Wiedemann syndrome (15 controls, 18 cases)
- Celiac_data — packaged methylation levels for eight NFkB-related genes in celiac disease patients/controls
- AMP vector of known standard-control methylation percentages (e.g. c(0, 25, 50, 75, 100))
- R-INLA package installed from https://inla.r-inla-download.org/R/stable

## Outputs

- Posterior mean calibration curve per CpG-AMP combination for each targeted assay
- Model-fit measures (mlik, DIC, RSS) for each of the four models M1–M4
- Selected best model by DIC, used to correct apparent methylation levels via equations (9) and (10)

## How to apply

Load BWS_data or Celiac_data and restructure with Formatting(), setting AMP = c(0, 25, 50, 75, 100) for BWS or AMP = c(0, 12.5, 25, 37.5, 50, 62.5, 87.5, 100) for celiac. Implement the model family M1–M4, which differ in how the CpG-specific random-effects term REij is specified: a joint calibration model over all CpGs in the targeted DMR/CGI with an RW1 latent Gaussian field whose precision depends on CpG distance. Assign the paper's priors: Normal-Wishart for the CpG-specific random intercepts and slopes, Gamma prior for the model precision, and inverse-Gamma prior for the observation variance. Fit each model per targeted assay (e.g. KCNQ1OT1, H19/IGF2, NFKBIA) with INLA (Rue et al., 2009) rather than MCMC. Select the best-fitting model with the Deviance Information Criterion (DIC), and extract posterior mean predictions of the calibration curve plus mlik, DIC and RSS for comparison against Moskalev's CPR.

## Related tools

- **MethylCal** (R package implementing the Bayesian calibration models (M1–M4) fitted with INLA; provides Formatting, MethylCalCalibrationPlot and MethylCalCorrection) — https://github.com/lb664/MethylCal
- **INLA** (R-INLA backend performing the approximate Bayesian inference (Rue et al., 2009); must be installed manually before MethylCal) — http://www.r-inla.org/
- **R** (Environment in which MethylCal is written and the calibration/correction workflow is run)

## Examples

```
data = Formatting(BWS_data, AMP = c(0, 25, 50, 75, 100), n_Control = 15, n_Case = 18); corr_data = MethylCalCorrection(data, Target = "KCNQ1OT1", n_Control = 15, n_Case = 18, printing = FALSE)
```

## Evaluation signals

- Script runs without error after Formatting() and produces fits for each targeted assay (e.g. KCNQ1OT1, H19/IGF2, NFKBIA)
- Posterior mean predictions agree with the reference MethylCal package fits within stated numerical tolerance on mlik, DIC and RSS
- The selected model is the one with the lowest DIC among M1–M4 (Deviance Information Criterion selection)
- Resulting corrected methylation levels reproduce the paper's clearer identification of loss/gain of methylation in borderline BWS cases (3SD confidence interval classification on KCNQ1OT1 and H19/IGF2)
- R-INLA is installed correctly, since MethylCal depends on it being installed manually before devtools::install_github('lb664/MethylCal')

## Limitations

- R-INLA is not auto-installed — it must be installed manually from https://inla.r-inla-download.org/R/stable, and 'it may take a couple of minutes'
- Outlier handling matters: three CpGs (21–23) were removed from the PLAGL1 assay by visual inspection, and MethylCalCalibrationOutlierPlot refits models without detected outliers
- Cross-validation showed performance varies by removal scenario (CpG-AMP combination, whole CpG, whole AMP removed)
- No software version numbers for MethylCal or R are stated in the paper

## Evidence

- [readme] INLA backend and purpose: "This package allows the calibration of methylation levels using **MethylCal**, a Bayesian calibration tool based on INLA ([Rue et al., 2009]"
- [readme] R-INLA installation requirement: [evidence span withheld — anchor status: external_doc]
- [methods] Model fitted via INLA: "This package allows the calibration of methylation levels using **MethylCal**, a Bayesian calibration tool based on INLA"
- [abstract] Joint modelling scope: "which can analyse jointly all CpGs within a CpG island (CGI) or a Differentially Methylated Region (DMR)"
