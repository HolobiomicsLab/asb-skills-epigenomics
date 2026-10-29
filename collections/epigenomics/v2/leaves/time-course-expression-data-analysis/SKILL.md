---
name: time-course-expression-data-analysis
description: Use when you need to infer direct regulons of a sigma factor from time-course
  expression series (e.g. GEO datasets like GSE6865 or GSE44415) combined with candidate
  target lists from ChIP-chip, knockout, or database mining.
when_to_use_negative:
- The organism/analysis is eukaryotic and strict validation is required — Genexpi
  was designed for bacterial regulons and must be employed with caution outside that
  scope
- You need complete regulatory-interaction information from expression data alone
  — time-series expression cannot in principle provide complete information and Genexpi
  is best used as one of multiple sources of insight
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Genexpi
  role: Core OpenCL-accelerated tool that smooths data, fits ODE regulation models
    by simulated annealing (128 runs/gene), and applies the no-change, constant-synthesis,
    and 0.8 fit-quality filters
  repo: http://github.com/cas-bioinf/genexpi/wiki/
- name: CyGenexpi
  role: Cytoscape plugin wrapping Genexpi for point-and-click regulon discovery from
    time-course data
  repo: http://github.com/cas-bioinf/genexpi/wiki/
- name: CyDataseries
  role: Cytoscape plugin for importing and handling the time-series expression input
    that Cytoscape does not natively support
- name: R
  role: R package/R notebook interface used to run the Genexpi workflow and reproduce
    results (e.g. evaluation_sacharomyces.Rmd)
  repo: http://github.com/cas-bioinf/genexpi/wiki/
- name: ARACNE
  role: TD-Aracne baseline used for computing-time comparison against Genexpi (reported
    447/718/446 s vs 26/109/150 s on GPU)
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:d26ec15ca0623d95d432246d3b9569eff5a094a91a7afb6648971bf080b057c3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ARACNE
  - CyDataseries
  - CyGenexpi
  - Genexpi
  - Java
  - R
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# time-course-expression-data-analysis

## Summary

A Genexpi-based workflow for identifying sigma-factor/transcription-factor regulons by fitting an ODE regulation model to smoothed time-course gene expression data, with candidate regulators sourced from ChIP experiments or literature mining.

## When to use

Use when you need to infer direct regulons of a sigma factor from time-course expression series (e.g. GEO datasets like GSE6865 or GSE44415) combined with candidate target lists from ChIP-chip, knockout, or database mining.

## When NOT to use

- The organism/analysis is eukaryotic and strict validation is required — Genexpi was designed for bacterial regulons and must be employed with caution outside that scope
- You need complete regulatory-interaction information from expression data alone — time-series expression cannot in principle provide complete information and Genexpi is best used as one of multiple sources of insight

## Inputs

- Time-course gene expression dataset (e.g. GSE6865 for B. subtilis SigA, GSE44415 for S. coelicolor SigR, GDS38 for S. cerevisiae cell cycle)
- Candidate regulon definition (ChIP-chip, knockout-derived, or database-mined, e.g. Subtiwiki SigB regulon or YEASTRACT regulons)
- Optional manually selected target subset for validation (e.g. 10 genes chosen for in-vitro transcription assays)

## Outputs

- List of predicted novel regulon targets passing the 0.8 fit-quality threshold (e.g. 215 SigA targets written to siga_predictions.csv)
- Fit quality values per gene/regulator pair
- Labelled confirmation status for externally validated subsets (e.g. 5-of-10 SigA-dependent from in-vitro transcription assays)

## How to apply

Smooth the expression profiles by linear regression on a B-spline basis with degrees of freedom approximately equal to half the number of measurement points. Exclude genes that show no significant change over the time course and genes explainable by the constant synthesis model, both fitted via simulated annealing in OpenCL. For each remaining gene and candidate regulator set, run 128 simulated-annealing runs fitting the ODE model (integrated with the cheap Euler method after subsampling), with the regularization constant c set to approximately one tenth of the number of smoothed time points to force parameters into biologically interpretable ranges. Retain only genes whose fits pass the goodness threshold (default fit quality 0.8, the proportion of time points where the fitted profile falls within the error margin of the measured profile), then assemble the predicted regulon list — e.g. the 215 predicted novel SigA targets on GSE6865 Bacillus subtilis.

## Related tools

- **Genexpi** (Core OpenCL-accelerated tool that smooths data, fits ODE regulation models by simulated annealing (128 runs/gene), and applies the no-change, constant-synthesis, and 0.8 fit-quality filters) — http://github.com/cas-bioinf/genexpi/wiki/
- **CyGenexpi** (Cytoscape plugin wrapping Genexpi for point-and-click regulon discovery from time-course data) — http://github.com/cas-bioinf/genexpi/wiki/
- **CyDataseries** (Cytoscape plugin for importing and handling the time-series expression input that Cytoscape does not natively support)
- **R** (R package/R notebook interface used to run the Genexpi workflow and reproduce results (e.g. evaluation_sacharomyces.Rmd)) — http://github.com/cas-bioinf/genexpi/wiki/
- **ARACNE** (TD-Aracne baseline used for computing-time comparison against Genexpi (reported 447/718/446 s vs 26/109/150 s on GPU))

## Evaluation signals

- On GSE6865 B. subtilis, the predicted novel SigA target list should reproduce the reported count of 215 genes
- Each gene/regulator fit must involve exactly 128 simulated-annealing runs, and retained genes must have fit quality above the 0.8 threshold
- Constant and constant-synthesis genes must be excluded as targets before candidate ranking (steps 3&4 of the workflow)
- Timing benchmark: Genexpi wall-clock times on a mid-tier GPU should fall near 26 s, 109 s, and 150 s for the SigB and both SigR regulons (vs TD-Aracne 447/718/446 s)
- On the 10 genes selected for in-vitro SigA testing, 5 should be confirmed SigA-dependent with regulation not excludable for the remaining five

## Limitations

- Genexpi masks uncertainty in the data and uses multiple hard thresholds (e.g. the 0.8 fit-quality cutoff)
- Time-series expression data cannot in principle provide complete information about regulatory interactions; results must be used alongside other sources of insight
- Eukaryotic applicability is not fully validated — apply with caution outside bacterial regulons
- No software version numbers are stated for Genexpi, R, RStudio, or Cytoscape in the paper

## Evidence

- [methods] Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values.: "Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values."
- [methods] Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom equal to approximately half the number of: "Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom equal to approximately half the number of"
- [methods] Fit quality is then the proportion of time points where the fitted profile is within the error margin of the measured profile.: "Fit quality is then the proportion of time points where the fitted profile is within the error margin of the measured profile."
- [results] Remove genes that could be modelled by the constant synthesis model.: "Remove genes that could be modelled by the constant synthesis model."
- [discussion] Genexpi was designed for bacterial regulons, but can be – with caution – employed also for eukaryotic data.: "Genexpi was designed for bacterial regulons, but can be – with caution – employed also for eukaryotic data."
- [readme] Detailed information can be found on the project's wiki: https://github.com/cas-bioinf/genexpi/wiki: "Detailed information can be found on the project's wiki: https://github.com/cas-bioinf/genexpi/wiki"
