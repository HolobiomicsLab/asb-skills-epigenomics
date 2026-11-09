---
name: random-effects-model-specification-methylation
description: Decision support for choosing among MethylCal's four random-effects model
  specifications (equations 2-5) for a methylation calibration assay when standard-control
  data at known methylation percentages are available and goodness-of-fit comparison
  via DIC is required.
when_to_use_negative:
- The goal is calibration of case/control methylation levels using an already-selected
  model — use MethylCalCorrection/MethycalCorrection rather than re-running model
  selection
- The data are single-CpG or average-methylation calibration problems, which MethylCal
  explicitly avoids ('one-at-a-time' CpG calibration); this skill requires joint analysis
  across all CpGs of a CGI/DMR
- Comparison against alternative calibration methods (Warnecke 1997, Moskalev 2011)
  is the goal — those use WarneckeCalibrationPlot or MoskalevCalibrationPlot, not
  the M1-M4 random-effects specifications
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: MethylCal
  role: Provides Formatting() for data restructuring and MethylCalCalibrationPlot
    to fit the four random-effects model specifications jointly across all CpGs of
    a CGI/DMR
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Bayesian inference engine used by MethylCalCalibrationPlot to fit the latent
    Gaussian random-effects models and return DIC/mlik goodness-of-fit measures
- name: R
  role: Environment in which the MethylCal package is run (library(MethylCal))
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:0d88d481677999260c23c74fa452ef6b30f5f76c373496a081443e3c1da01e0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lb664/MethylCal
  tools:
  - INLA
  - MethylCal
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# random-effects-model-specification-methylation

## Summary

Selects the best-fitting MethylCal random-effects regression specification (AMP effects, crossed CpG effects, RW1 latent Gaussian field, or random CpG intercepts/slopes) for each bisulfite amplicon-sequencing assay by comparing Deviance Information Criterion values across the four candidate models fitted jointly to all CpGs of a CGI/DMR.

## When to use

Decision support for choosing among MethylCal's four random-effects model specifications (equations 2-5) for a methylation calibration assay when standard-control data at known methylation percentages are available and goodness-of-fit comparison via DIC is required.

## When NOT to use

- The goal is calibration of case/control methylation levels using an already-selected model — use MethylCalCorrection/MethycalCorrection rather than re-running model selection
- The data are single-CpG or average-methylation calibration problems, which MethylCal explicitly avoids ('one-at-a-time' CpG calibration); this skill requires joint analysis across all CpGs of a CGI/DMR
- Comparison against alternative calibration methods (Warnecke 1997, Moskalev 2011) is the goal — those use WarneckeCalibrationPlot or MoskalevCalibrationPlot, not the M1-M4 random-effects specifications

## Inputs

- BWS_data standard-control methylation dataset (Beckwith-Wiedemann syndrome, 15 healthy controls / 18 potential cases) packaged in the MethylCal R package
- Celiac_data standard-control methylation dataset (eight NFkB-related genes, celiac case/control study) packaged in the MethylCal R package
- Vector of actual methylation percentages for the standard controls (e.g. c(0, 25, 50, 75, 100) for BWS assays)

## Outputs

- Per-assay goodness-of-fit measures (mlik, DIC, RSS) for each of the four random-effects specifications M1-M4
- A mapping of each of the eight independent assays (two CGIs on the SDHC promoter and six imprinted DMRs) to its lowest-DIC preferred model, e.g. M4 for KCNQ1OT1 and M3 for H19/IGF2

## How to apply

Restructure the packaged BWS_data or Celiac_data standard-control data with the MethylCal Formatting() function at the actual methylation percentages (0/25/50/75/100% for BWS; 0/12.5/25/37.5/50/62.5/87.5/100% for Celiac). For each independent assay, fit the four candidate random-effects specifications — AMP effects, crossed CpG effects, RW1 latent Gaussian field over AMP, and random CpG intercepts/slopes — jointly across all CpGs in the targeted CGI or DMR using MethylCalCalibrationPlot, which performs Bayesian inference with INLA. Record the goodness-of-fit measures returned for each fitted model: mlik (marginal likelihood), DIC (Deviance Information Criterion), and RSS (Residual Sum of Squares). Select the model with the lowest DIC as the preferred calibration model for that assay. Validate the selection against the preferred models reported in Ochoa et al. (2019), e.g. model M4 for KCNQ1OT1 and model M3 for H19/IGF2.

## Related tools

- **MethylCal** (Provides Formatting() for data restructuring and MethylCalCalibrationPlot to fit the four random-effects model specifications jointly across all CpGs of a CGI/DMR) — https://github.com/lb664/MethylCal
- **INLA** (Bayesian inference engine used by MethylCalCalibrationPlot to fit the latent Gaussian random-effects models and return DIC/mlik goodness-of-fit measures)
- **R** (Environment in which the MethylCal package is run (library(MethylCal)))

## Examples

```
library(MethylCal); AMP = c(0, 25, 50, 75, 100); data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18)
```

## Evaluation signals

- The DIC-selected model per assay reproduces the preferred model reported in Ochoa et al. (2019): M4 for KCNQ1OT1 and M3 for H19/IGF2
- The assay-to-model mapping contains exactly eight assay rows (two CGIs and six imprinting DMRs) with a non-empty model assignment each
- Each fitted model returns all three goodness-of-fit measures (mlik, DIC, RSS) rather than a subset
- The selected model per assay is the one with the strictly lowest DIC among M1-M4 for that assay
- Standard-control methylation percentages match the designed controls (five standards 0-100% for BWS; eight AMPs including 12.5% and 87.5% for the celiac/HGControls data)

## Limitations

- No software version numbers for MethylCal, R-INLA, or R are stated in the article, so exact reproducibility of DIC values may vary across INLA versions
- Some CpGs may need removal before fitting (e.g. CpGs 21-23 were removed from the PLAGL1 assay by visual inspection), which affects model fit and DIC comparison
- R-INLA must be installed manually (install.packages('INLA', repos='https://inla.r-inla-download.org/R/stable')) and installation may take a couple of minutes
- No accession or availability information is provided for the BWS case data set or the celiac case/control data set beyond the packaged standard-control versions

## Evidence

- [methods] MethylCal as presented in Ochoa et. (2019) is implemented in the function `MethylCalCalibrationPlot`.: "MethylCal as presented in Ochoa et. (2019) is implemented in the function `MethylCalCalibrationPlot`."
- [methods] a Bayesian calibration tool based on INLA: "This package allows the calibration of methylation levels using **MethylCal**, a Bayesian calibration tool based on INLA"
- [intro] eight independent assays in two CGIs located on SDHC gene promoter and six imprinted DMRs: "we designed eight independent assays in two CGIs located on SDHC gene promoter and six imprinted DMRs"
- [methods] The package also contains two utility functions `Formatting` and `TableFormatting`.: "The package also contains two utility functions `Formatting` and `TableFormatting`."
- [readme] data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18): "AMP = c(0, 25, 50, 75, 100)
    data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18)"
