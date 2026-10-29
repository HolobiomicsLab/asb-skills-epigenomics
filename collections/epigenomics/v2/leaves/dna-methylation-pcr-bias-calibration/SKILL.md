---
name: dna-methylation-pcr-bias-calibration
description: Use when pyrosequenced or MiSeq-amplified methylation levels in a targeted
  CGI/DMR need PCR-bias correction, and standard controls with known methylation percentages
  (AMPs) are available alongside case and control samples.
when_to_use_negative:
- Input is epigenome-wide array data or whole-genome methylation rather than targeted
  bisulfite amplicon sequencing of a CGI/DMR
- No standard controls with known methylation percentages (AMPs) exist, so calibration
  curves cannot be derived
- Only average methylation per region is available, since the method explicitly avoids
  'one-at-a-time' CpG calibration but requires CpG-resolved levels
tools:
- name: MethylCal
  role: R package implementing the Bayesian calibration models (M1-M4), outlier refitting
    (MethylCalCalibrationOutlierPlot) and correction (MethylCalCorrection) with Formatting()/TableFormatting()
    utilities
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Approximate Bayesian inference engine underlying the latent Gaussian calibration
    model; must be installed manually before MethylCal
- name: R
  role: Language environment in which MethylCal is distributed and the calibration/correction
    workflow is executed
- name: Moskalev's CPR
  role: Alternative PCR-bias correction method used as the baseline against which
    MethylCal-corrected levels are compared
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:9e311902cab7d16aeb1d06172f9d4e6a464cf4689338b63db950746ae2135f49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lb664/MethylCal
  tools:
  - INLA
  - MethylCal
  - Moskalev's CPR
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# dna-methylation-pcr-bias-calibration

## Summary

Bayesian PCR-bias calibration of bisulfite amplicon sequencing methylation levels using MethylCal, which fits joint calibration curves across all CpGs of a CGI or DMR via INLA and corrects observed methylation degrees in case/control samples.

## When to use

Use when pyrosequenced or MiSeq-amplified methylation levels in a targeted CGI/DMR need PCR-bias correction, and standard controls with known methylation percentages (AMPs) are available alongside case and control samples.

## When NOT to use

- Input is epigenome-wide array data or whole-genome methylation rather than targeted bisulfite amplicon sequencing of a CGI/DMR
- No standard controls with known methylation percentages (AMPs) exist, so calibration curves cannot be derived
- Only average methylation per region is available, since the method explicitly avoids 'one-at-a-time' CpG calibration but requires CpG-resolved levels

## Inputs

- Standard-control methylation data at known AMPs (e.g. BWS_data with 0/25/50/75/100% controls)
- Case/control pyrosequenced methylation levels for a targeted region (e.g. Celiac_data: 13 controls, 17 celiac patients at diagnosis and after gluten-free treatment, in NFKBIA, RELA, TNFAIP3)
- n_Control and n_Case sample counts and the AMP vector for the assay

## Outputs

- Calibration curves per CpG-AMP combination with goodness-of-fit measures (mlik, DIC, RSS)
- Corrected methylation degrees per individual, CpG and region (values in the 0-100% range), e.g. written to corrected_methylation_celiac.csv via TableFormatting()
- Differential methylation test flags (hyper-/hypo-methylated cases) at a specified confidence level
- Outlier diagnostics from the posterior predictive interval [Q1 - 1.5*IQR, Q3 + 1.5*IQR]

## How to apply

Load the MethylCal R package and restructure the assay data with Formatting(), supplying the actual methylation percentages (e.g. AMP = c(0, 25, 50, 75, 100) for the BWS standard controls, or AMP = c(0, 12.5, 25, 37.5, 50, 62.5, 87.5, 100) for the celiac cohort) plus n_Control and n_Case. Explore the standard-control levels with ExploratoryPlot, then fit the MethylCal Bayesian regression models (M1-M4 random-effects family, Normal-Wishart priors for CpG-specific intercepts/slopes, RW1 latent field over CpG distance) with INLA, selecting the best model by DIC. Refit after outlier removal using MethylCalCalibrationOutlierPlot, whose posterior predictive interval is defined by l = Q1 - 1.5*IQR and u = Q3 + 1.5*IQR. Correct observed methylation levels of controls and cases with MethylCalCorrection (which also runs a differential methylation test flagging hyper- and hypo-methylated cases), and convert output with TableFormatting() back to the uploaded tabular format.

## Related tools

- **MethylCal** (R package implementing the Bayesian calibration models (M1-M4), outlier refitting (MethylCalCalibrationOutlierPlot) and correction (MethylCalCorrection) with Formatting()/TableFormatting() utilities) — https://github.com/lb664/MethylCal
- **INLA** (Approximate Bayesian inference engine underlying the latent Gaussian calibration model; must be installed manually before MethylCal)
- **R** (Language environment in which MethylCal is distributed and the calibration/correction workflow is executed)
- **Moskalev's CPR** (Alternative PCR-bias correction method used as the baseline against which MethylCal-corrected levels are compared)

## Examples

```
data(BWS_data); AMP = c(0,25,50,75,100); data = Formatting(BWS_data, AMP=AMP, n_Control=15, n_Case=18); corr_data = MethylCalCorrection(data, Target="KCNQ1OT1", n_Control=15, n_Case=18)
```

## Evaluation signals

- Corrected output CSV exists and contains one corrected methylation degree per individual and CpG/region, all within the 0-100% range
- Corrected control/case distributions are consistent with the corrected levels displayed in the paper's celiac disease figures (e.g. patient 09D average corrected methylation of 11.34 vs Moskalev's 5.24 in NFKBIA)
- Posterior mean predictions of the calibration curve agree with the reference MethylCal package fits within tolerance on mlik, DIC and RSS
- Outliers flagged correspond to points outside [Q1 - 1.5*IQR, Q3 + 1.5*IQR] of the posterior predictive interval, and models are refitted without them
- Model selection by DIC picks the best-fitting member of the M1-M4 random-effects family for each assay (e.g. KCNQ1OT1, H19/IGF2, NFKBIA)

## Limitations

- No software version numbers for MethylCal, R or Moskalev's CPR are stated in the article, and no accession/availability info is given for the BWS or celiac case datasets
- R-INLA must be installed manually (install.packages with the inla.r-inla-download.org repo) before MethylCal can be used
- Some CpGs may need manual removal based on visual inspection (e.g. CpGs 21-23 removed from the PLAGL1 assay)
- Borderline gain/loss-of-methylation classification depends on the 3SD confidence interval applied to corrected controls

## Evidence

- [abstract] Here, we present a Bayesian calibration tool, MethylCal, which can analyse jointly all CpGs within a CpG island (CGI) or a Differentially Methylated Region (DMR): "Here, we present a Bayesian calibration tool, MethylCal, which can analyse jointly all CpGs within a CpG island (CGI) or a Differentially Methylated Region (DMR)"
- [results] posterior predictive interval for outliers: "the posterior predictive interval [l, u] for outliers detection with l = Q1 − 1.5IQR and u = Q3 + 1.5IQR"
- [methods] outlier refitting function: "The function `MethylCalCalibrationOutlierPlot` includes the detection of outliers, where the models are refitted without the detected outliers."
- [methods] correction functions: "The main functions for the correction of the methylation levels in case and control samples are `MoskalevCorrection` and `MethycalCorrection`."
- [readme] README package description: "This package allows the calibration of methylation levels using **MethylCal**, a Bayesian calibration tool based on INLA"
- [readme] README example workflow: [evidence span withheld — anchor status: external_doc]
