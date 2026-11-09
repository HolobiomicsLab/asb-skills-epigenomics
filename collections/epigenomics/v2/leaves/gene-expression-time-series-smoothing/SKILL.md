---
name: gene-expression-time-series-smoothing
description: Use when you have a measured gene-expression time series (e.g. GSE6865,
  GDS38, GSE44415) that must be converted into continuous smooth curves prior to ODE
  model fitting or regulon scoring.
when_to_use_negative:
- Data are not a time series (Genexpi/CyDataseries is specific to time-course expression
  data; Cytoscape does not natively support time series).
- The downstream analysis is a eukaryotic regulator-complex problem such as MBP1/SWI4/SWI6
  in yeast, where the sigmoidal single-regulator model gave random-indistinguishable
  results.
- You need statistical uncertainty quantification — the smoothing-plus-thresholds
  approach masks uncertainty in the data.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Genexpi
  role: Provides the smoothing-by-B-spline-regression preprocessing and the OpenCL
    core that consumes the smoothed curves for ODE fitting
  repo: https://github.com/cas-bioinf/genexpi
- name: R
  role: Interface and notebook environment (evaluation.Rmd) in which the B-spline
    smoothing of the expression profiles is executed
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:a1a3d6e54386ce3c2746bb6ec232053512865805fcb53ad7c8cbc87891ae8392
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cas-bioinf/genexpi
  tools:
  - ARACNE
  - Genexpi
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gene-expression-time-series-smoothing

## Summary

Smooth noisy gene-expression time-course profiles by linear regression on a B-spline basis before ODE-based regulator inference with Genexpi, so that fitted regulation models can be integrated cheaply with the Euler method on continuous curves.

## When to use

Use when you have a measured gene-expression time series (e.g. GSE6865, GDS38, GSE44415) that must be converted into continuous smooth curves prior to ODE model fitting or regulon scoring.

## When NOT to use

- Data are not a time series (Genexpi/CyDataseries is specific to time-course expression data; Cytoscape does not natively support time series).
- The downstream analysis is a eukaryotic regulator-complex problem such as MBP1/SWI4/SWI6 in yeast, where the sigmoidal single-regulator model gave random-indistinguishable results.
- You need statistical uncertainty quantification — the smoothing-plus-thresholds approach masks uncertainty in the data.

## Inputs

- Measured gene-expression time-course matrix (e.g. GSE6865 B. subtilis, GDS38 S. cerevisiae cell cycle, GSE44415 S. coelicolor series)
- Number of measurement points per gene (determines spline degrees of freedom)
- Genexpi R package / evaluation.Rmd or evaluation_sacharomyces.Rmd notebook

## Outputs

- Smoothed, subsampled expression curves suitable for Euler integration of the Genexpi ODE model
- Downstream inputs to fit-quality scoring (proportion of time points within error margin, threshold 0.8)

## How to apply

Fit each measured expression profile by linear regression on a B-spline basis with degrees of freedom equal to approximately half the number of measurement points. Subsample the fitted smooth curves so the Genexpi ODE (dz/dt = k1*f(rho) - k2*z, logistic f) can be integrated with the cheap Euler method, enabling fast error-function evaluation in the OpenCL core. This smoothing step precedes exclusion of no-change and constant-synthesis targets and the simulated-annealing parameter fitting (128 annealing runs per gene/regulator set). The degrees-of-freedom choice is itself a sensitivity axis: on the SigB/GSE6865 evaluation, the 4-DF spline variant recovered ~90% of the Subtiwiki SigB regulon but falsely accepted about one third of random regulator profiles.

## Related tools

- **Genexpi** (Provides the smoothing-by-B-spline-regression preprocessing and the OpenCL core that consumes the smoothed curves for ODE fitting) — https://github.com/cas-bioinf/genexpi
- **R** (Interface and notebook environment (evaluation.Rmd) in which the B-spline smoothing of the expression profiles is executed)

## Examples

```
Rscript -e "rmarkdown::render('evaluation.Rmd', output_file='sigb_evaluation_report.html')"
```

## Evaluation signals

- Spline degrees of freedom equal approximately half the number of measurement points (e.g. 4 DF for the SigB evaluation).
- On the GSE6865 SigB evaluation, the smoothed-profile pipeline with a 4-DF spline should reproduce ~90% recovery of the Subtiwiki SigB regulon.
- The random-regulator false-positive rate should match the reported roughly one third of random profiles predicted as regulating SigB targets.
- Subsequent Euler integration and simulated annealing (128 runs per gene/regulator set) should converge to stable, replicable parameter values with fit quality above the default 0.8 threshold.
- Constant-expression genes and constant-synthesis genes should be excluded as targets before downstream scoring.

## Limitations

- Genexpi masks uncertainty in the data and uses multiple hard thresholds, so smoothing artifacts are not propagated as uncertainty.
- Over-smoothing or the 4-DF variant falsely accepts about one third of random regulator profiles.
- Time-series expression data cannot in principle provide complete information about regulatory interactions; the tool is best used as one of multiple sources of insight.

## Evidence

- [methods] B-spline smoothing recommendation: "Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom equal to approximately half the number of"
- [methods] Euler integration rationale: "The subsampling then allows us to integrate (1) accurately with the computationally cheap Euler method, making evaluation of the error function fast and easy to implement in OpenCL."
- [results] 4-DF spline evaluation result: [evidence span withheld — anchor status: not_found]
- [results] Notebook availability: "Complete code to reproduce all of the results for this and the following section is attached as an R notebook in Additional file 1."
- [readme] Repository README pointer: "Detailed information can be found on the project's wiki: https://github.com/cas-bioinf/genexpi/wiki"
