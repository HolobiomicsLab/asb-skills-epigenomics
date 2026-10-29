---
name: methylation-level-correction-evaluation
description: Use when observed bisulfite amplicon/pyrosequencing methylation percentages
  in case and control samples need PCR-bias correction, and the stability or concordance
  of the resulting hyper/hypomethylation classifications must be assessed.
when_to_use_negative:
- Input data are already PCR-bias-corrected or produced by controlled mixtures spanning
  the CpG range — recalibration would double-correct
- The goal is calibration-curve fitting alone on standard controls without case/control
  samples, where no differential methylation test is needed (use MethylCalCalibrationPlot
  directly)
- Methylation data come from a platform other than bisulfite amplicon sequencing/pyrosequencing
  whose PCR behaviour the calibration curves were not derived for
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: MethylCal
  role: Bayesian calibration and correction of observed methylation levels; includes
    MethylCalCorrection, MethylCalCalibrationOutlierPlot, Formatting and TableFormatting
    functions
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Backend used by MethylCal for Bayesian inference of the calibration curves;
    must be installed manually before the package
  repo: http://www.r-inla.org/
- name: Moskalev's CPR
  role: Alternative correction method used as a comparator to assess threshold sensitivity
    of the corrected methylation calls
- name: R
  role: Runtime environment in which the MethylCal package, corrections, classification
    and CSV export are performed
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:11c13f409ea8dd9d6cc2b7ff58154e603acbb376cd0f2adf86c91658b8a8549d
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
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# methylation-level-correction-evaluation

## Summary

Apply MethylCal's Bayesian calibration (R/INLA) to correct PCR-bias-distorted pyrosequenced methylation levels in case/control cohorts, then evaluate whether downstream loss/gain-of-methylation calls (e.g. ±3SD vs 1% thresholds) are stable under the corrected levels.

## When to use

Use when observed bisulfite amplicon/pyrosequencing methylation percentages in case and control samples need PCR-bias correction, and the stability or concordance of the resulting hyper/hypomethylation classifications must be assessed.

## When NOT to use

- Input data are already PCR-bias-corrected or produced by controlled mixtures spanning the CpG range — recalibration would double-correct
- The goal is calibration-curve fitting alone on standard controls without case/control samples, where no differential methylation test is needed (use MethylCalCalibrationPlot directly)
- Methylation data come from a platform other than bisulfite amplicon sequencing/pyrosequencing whose PCR behaviour the calibration curves were not derived for

## Inputs

- Pyrosequenced/amplicon methylation level table (e.g. packaged BWS_data or Celiac_data cohorts) with CpG-level percentages for controls and cases
- Vector of actual methylation percentages AMP used for standard controls (e.g. c(0, 12.5, 25, 37.5, 50, 62.5, 87.5, 100))
- Cohort sizes n_Control and n_Case (e.g. 13 controls / 34 celiac cases; 15 controls / 18 BWS patients)
- Target region or DMR name (e.g. KCNQ1OT1, H19/IGF2, NFKBIA)

## Outputs

- Corrected methylation levels per sample and CpG/region, in the 0–100% range (e.g. corrected_methylation_celiac.csv)
- Differential methylation calls (hyper-/hypo-methylated vs normal) per case per target
- Threshold-sensitivity table comparing classifications under ±3SD vs 1% decision thresholds

## How to apply

Load the MethylCal package (with its R-INLA dependency) in R and reformat the raw methylation table with Formatting(), supplying the actual methylation percentages used for calibration (e.g. AMP = c(0, 12.5, 25, 37.5, 50, 62.5, 87.5, 100) for the celiac data or c(0, 25, 50, 75, 100) for BWS_data) plus n_Control and n_Case. Fit the Bayesian calibration on standard controls with MethylCalCalibrationOutlierPlot, which refits models after detecting outliers via the posterior predictive interval l = Q1 − 1.5*IQR, u = Q3 + 1.5*IQR. Correct observed control/case levels per target region or DMR (e.g. NFKBIA, RELA, TNFAIP3 for celiac; KCNQ1OT1 and H19/IGF2 for BWS) with MethylCalCorrection, which also runs a differential methylation test flagging hyper- and hypo-methylated cases. Convert the corrected object back to the original tabular format with TableFormatting(). Evaluate call stability by reclassifying patients at the conservative ±3SD control interval (α = 0.0027) and at a less conservative 1% threshold, confirming the calls match MethylCalCorrection's differential methylation test output.

## Related tools

- **MethylCal** (Bayesian calibration and correction of observed methylation levels; includes MethylCalCorrection, MethylCalCalibrationOutlierPlot, Formatting and TableFormatting functions) — https://github.com/lb664/MethylCal
- **INLA** (Backend used by MethylCal for Bayesian inference of the calibration curves; must be installed manually before the package) — http://www.r-inla.org/
- **Moskalev's CPR** (Alternative correction method used as a comparator to assess threshold sensitivity of the corrected methylation calls)
- **R** (Runtime environment in which the MethylCal package, corrections, classification and CSV export are performed)

## Examples

```
data(BWS_data); AMP = c(0, 25, 50, 75, 100); data = Formatting(BWS_data, AMP=AMP, n_Control=15, n_Case=18); corr_data = MethylCalCorrection(data, Target="KCNQ1OT1", n_Control=15, n_Case=18)
```

## Evaluation signals

- Corrected methylation values fall within the 0–100% range for every individual and CpG/region
- Corrected control/case distributions are consistent with the corrected levels displayed in the paper's celiac disease and BWS figures (e.g. patient 09D ≈ 11.34 average corrected methylation, not Moskalev's 5.24)
- Hyper-/hypomethylation calls are identical between the ±3SD and 1% thresholds and match the differential methylation test output printed by MethylCalCorrection
- Output CSV from TableFormatting has one corrected methylation degree per individual and CpG/region in the same tabular format as the uploaded dataset
- Refitted calibration models exclude exactly the outliers detected by the l = Q1 − 1.5*IQR, u = Q3 + 1.5*IQR posterior predictive interval

## Limitations

- MethylCal's BWS patient classifications are robust to the ±3SD vs 1% threshold choice, whereas Moskalev's CPR calls are threshold-sensitive — comparisons across methods must state which correction was used
- Outliers are removed and models refitted automatically, so CpGs flagged by the posterior predictive interval no longer inform the calibration
- No software version numbers for MethylCal or Moskalev's CPR, nor accession/availability information for the BWS and celiac case data sets, are stated in the article

## Evidence

- [readme] differential methylation test: "If cases are provided, MethylCal performs a differential methylation test to detect hyper- and hypo-methylated samples."
- [methods] correction functions: "The main functions for the correction of the methylation levels in case and control samples are `MoskalevCorrection` and `MethycalCorrection`."
- [results] outlier detection: "the posterior predictive interval [l, u] for outliers detection with l = Q1 − 1.5IQR and u = Q3 + 1.5IQR"
- [results] BWS correction: "The observed methylation levels of 15 healthy controls and 18 potential BWS patients were corrected using the calibration curves obtained by MethylCal"
- [discussion] R package availability: "Written in R and available as an R package on https://github.com/lb664/MethylCal."
- [abstract] borderline-case benefit: "The calibration of the methylation levels obtained by MethylCal allows a clearer identification of patients undergoing loss or gain of methylation in borderline cases"
