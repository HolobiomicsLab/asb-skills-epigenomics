---
name: b-spline-expression-profile-smoothing
description: Use when preparing noisy time-course expression data (e.g. GEO series
  such as GSE6865) for Genexpi ODE regulation modeling and a continuous smoothed profile
  is needed before fitting.
when_to_use_negative:
- Data are single-snapshot (non-time-series) expression measurements; Cytoscape and
  Genexpi require time series and the smoothing basis has no meaning without repeated
  time points.
- Genes whose expression does not change significantly over the whole time course
  — these should be excluded via the no-change model rather than smoothed and fitted
  further.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: Genexpi
  role: Toolset whose preprocessing pipeline performs the B-spline smoothing prior
    to ODE model fitting; available as command-line tool and R package
  repo: https://github.com/cas-bioinf/genexpi/wiki/
- name: R
  role: Interface and package language in which Genexpi's smoothing and downstream
    fitting can be scripted for reproducibility (e.g. via the provided R notebook)
  repo: https://github.com/cas-bioinf/genexpi/wiki/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:bd8505d87bdac120f88ff2a8c2d3baa8d47cd876c553256a15976ba0bc1cbcbd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Genexpi
  - R
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# b-spline-expression-profile-smoothing

## Summary

Preprocess time-course gene expression data by fitting linear regression on a B-spline basis before ODE-based regulon inference with Genexpi, providing continuous curves that stabilize downstream simulated-annealing parameter fitting.

## When to use

Use when preparing noisy time-course expression data (e.g. GEO series such as GSE6865) for Genexpi ODE regulation modeling and a continuous smoothed profile is needed before fitting.

## When NOT to use

- Data are single-snapshot (non-time-series) expression measurements; Cytoscape and Genexpi require time series and the smoothing basis has no meaning without repeated time points.
- Genes whose expression does not change significantly over the whole time course — these should be excluded via the no-change model rather than smoothed and fitted further.

## Inputs

- Time-course gene expression matrix (e.g. GEO GSE6865 B. subtilis expression time series)
- Number of measurement time points per gene (used to set B-spline degrees of freedom)
- Candidate regulon/gene list to which the smoothed profiles are matched

## Outputs

- Smoothed, continuous expression profiles per gene (B-spline fitted curves)
- Smoothed profiles consumed by downstream ODE regulation-model fitting and fit-quality evaluation

## How to apply

Load the expression time series (e.g. the GSE6865 Bacillus subtilis dataset) and regress each gene's profile on a B-spline basis using linear regression, setting degrees of freedom to approximately half the number of measurement points, as recommended in the Genexpi methods. Apply this smoothing before the no-change and constant-synthesis model tests, and before the ODE model fits that use 128 simulated-annealing runs per gene with a regularization constant of about one tenth of the number of smoothed time points. Smoothing also supports fast Euler integration of the ODE model after subsampling, since evaluating the error function on a continuous curve is cheap in OpenCL. Verify that the smoothed profiles feed the fit-quality metric (proportion of time points within error margin), where good fits are those with fit quality above the default 0.8 threshold.

## Related tools

- **Genexpi** (Toolset whose preprocessing pipeline performs the B-spline smoothing prior to ODE model fitting; available as command-line tool and R package) — https://github.com/cas-bioinf/genexpi/wiki/
- **R** (Interface and package language in which Genexpi's smoothing and downstream fitting can be scripted for reproducibility (e.g. via the provided R notebook)) — https://github.com/cas-bioinf/genexpi/wiki/

## Examples

```
Rscript -e "library(genexpi); smoothed <- genexpi::smoothData(expressionTimeSeries, splineDf = ncol(expressionTimeSeries)/2)"
```

## Evaluation signals

- Degrees of freedom of the B-spline basis is approximately half the number of raw measurement points per gene.
- Smoothed profiles are continuous curves suitable for the subsequent ODE Euler integration on subsampled time grids.
- Downstream fit-quality scores (proportion of time points within the error margin) are computable on the smoothed curves, with the default good-fit threshold at 0.8.
- Genes that remain flat after smoothing can still be flagged and removed by the no-change model test before ODE fitting.
- Reproducing the GSE6865 SigA workflow with this smoothing step recovers the reported 215 predicted novel SigA-regulated targets.

## Limitations

- Smoothing masks uncertainty in the data, and Genexpi applies multiple hard thresholds downstream — noise in sparse or irregular time series can distort the spline fit.
- The degree-of-freedom rule (about half the number of measurement points) is heuristic; very short time courses leave few effective degrees of freedom.
- Smoothed profiles alone cannot confirm regulation; the article notes time-series data provide incomplete information and Genexpi should be one of multiple sources of insight.

## Evidence

- [methods] B-spline smoothing recommendation: "Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom equal to approximately half the number of"
- [methods] Downstream ODE fitting uses simulated annealing: "Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values."
- [methods] Good-fit threshold on smoothed profiles: "Fit quality is then the proportion of time points where the fitted profile is within the error margin of the measured profile. A fit is considered good if fit quality is above a given threshold"
- [methods] Euler integration after subsampling: "The subsampling then allows us to integrate (1) accurately with the computationally cheap Euler method, making evaluation of the error function fast and easy to implement in OpenCL."
- [results] Limitation: uncertainty masking and hard thresholds: "Most notably, Genexpi masks uncertainty in the data and uses multiple hard thresholds."
