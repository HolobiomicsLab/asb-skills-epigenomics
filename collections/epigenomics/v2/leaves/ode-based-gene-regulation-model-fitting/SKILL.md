---
name: ode-based-gene-regulation-model-fitting
description: Use when you have a time-course expression dataset (e.g. GSE6865 B. subtilis)
  plus a candidate regulon list and need to infer direct regulator–target interactions
  with an ODE model rather than correlation or co-expression methods.
when_to_use_negative:
- The dataset is a single-sample or steady-state expression profile with no time dimension,
  since the ODE model requires a continuous expression time series
- You need a fully validated method for eukaryotic regulatory networks without caution
  — the paper states Genexpi was designed for bacterial regulons and eukaryotic use
  requires care
- Genes whose expression does not change over the time course or are explainable by
  the constant-synthesis model — these must be filtered out before model fitting,
  not fitted as targets
edam_operation: http://edamontology.org/operation_2437
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: Genexpi
  role: Core package that implements the ODE regulation model, B-spline smoothing,
    simulated-annealing fitting via OpenCL, and fit-quality scoring; available as
    command-line tool and R package
  repo: https://github.com/cas-bioinf/genexpi
- name: CyGenexpi
  role: Cytoscape plugin wrapping Genexpi for interactive regulon discovery on time-course
    data
  repo: https://github.com/cas-bioinf/genexpi
- name: R
  role: Language interface used to run Genexpi evaluations via R notebooks (e.g. evaluation.Rmd,
    evaluation_sacharomyces.Rmd)
  repo: https://github.com/cas-bioinf/genexpi
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:bdb76573c7e1cbf76d7b303e3f548b10334c8642aa38c9b0d5294e6e9ddddaee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cas-bioinf/genexpi
  tools:
  - CyGenexpi
  - Genexpi
  - R
  edam_operation: http://edamontology.org/operation_2437
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ode-based-gene-regulation-model-fitting

## Summary

Fit Genexpi's ordinary-differential-equation regulation model to smoothed time-course expression profiles to infer which candidate regulators (e.g. sigma factors) control each gene, using simulated annealing parameter search in an OpenCL core.

## When to use

Use when you have a time-course expression dataset (e.g. GSE6865 B. subtilis) plus a candidate regulon list and need to infer direct regulator–target interactions with an ODE model rather than correlation or co-expression methods.

## When NOT to use

- The dataset is a single-sample or steady-state expression profile with no time dimension, since the ODE model requires a continuous expression time series
- You need a fully validated method for eukaryotic regulatory networks without caution — the paper states Genexpi was designed for bacterial regulons and eukaryotic use requires care
- Genes whose expression does not change over the time course or are explainable by the constant-synthesis model — these must be filtered out before model fitting, not fitted as targets

## Inputs

- Time-course gene expression matrix (e.g. GEO GSE6865 B. subtilis, GSE44415 S. coelicolor, GDS38 S. cerevisiae)
- Candidate regulator/regulon gene list (e.g. SigA or SigB regulon from Subtiwiki, SigR regulon from ChIP-chip or knockouts, YEASTRACT regulons)
- User-set error margin and good-fit threshold (default fit quality 0.8)

## Outputs

- Per gene–regulator-set fitted ODE parameters (w_j, rho, b, k1) from the best annealing run
- Fit-quality scores per target gene and list of genes passing the 0.8 threshold
- Predicted regulon membership (e.g. 215 predicted novel SigA targets written to siga_predictions.csv)

## How to apply

First smooth the expression profiles by linear regression on a B-spline basis with degrees of freedom equal to approximately half the number of measurement points. Exclude genes matching the no-change model and the constant-synthesis model (both fitted via simulated annealing in OpenCL) from the target set. For each remaining target, fit the Genexpi ODE regulation model against the candidate regulator set by running 128 simulated-annealing runs with different initial parameter values, with regularization bounds max|w_j*y_j| < 10, min|rho| < 0.5, and k1 below maximal expression; set the regularization constant c to roughly one tenth of the number of smoothed time points. Integrate the ODE by subsampling and the cheap Euler method, then score each fit by 'fit quality' — the proportion of time points where the fitted profile falls within the user-set error margin of the measured profile — and retain fits above the default threshold of 0.8.

## Related tools

- **Genexpi** (Core package that implements the ODE regulation model, B-spline smoothing, simulated-annealing fitting via OpenCL, and fit-quality scoring; available as command-line tool and R package) — https://github.com/cas-bioinf/genexpi
- **CyGenexpi** (Cytoscape plugin wrapping Genexpi for interactive regulon discovery on time-course data) — https://github.com/cas-bioinf/genexpi
- **R** (Language interface used to run Genexpi evaluations via R notebooks (e.g. evaluation.Rmd, evaluation_sacharomyces.Rmd)) — https://github.com/cas-bioinf/genexpi

## Examples

```
Rscript -e "genexpi::run(genexpi::get_data("sigb"), regulon="sigb_regulon.csv", spline_df=4, annealing_runs=128, fit_quality_threshold=0.8)"
```

## Evaluation signals

- Recovered literature regulon fraction matches published values, e.g. ~90% recovery of the Subtiwiki SigB regulon with the 4-DF spline on GSE6865
- Random-regulator false-positive rate matches reported level: roughly one third of random regulator profiles predicted to regulate SigB targets
- Every retained target fit has fit quality (proportion of time points within the error margin) above the 0.8 threshold
- Genes fitted by the no-change or constant-synthesis models are absent from the target set
- Number of predicted novel targets reproduces the paper's counts (e.g. 215 predicted SigA targets; 5 of 10 tested genes confirmed SigA-dependent in vitro)

## Limitations

- Genexpi masks uncertainty in the data and uses multiple hard thresholds
- Results are incomplete on their own; the tool must be used alongside other sources of insight because time-series data cannot in principle provide complete information about regulatory interactions
- Applicability outside bacterial regulons (eukaryotic data) is not fully validated and requires caution
- No software version numbers are stated for Genexpi, R, RStudio, or Cytoscape, complicating exact reproduction

## Evidence

- [methods] 128 annealing runs with different initial parameter values: "Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values."
- [methods] B-spline smoothing with DF = half the time points: "Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom equal to approximately half the number of"
- [methods] Fit quality threshold 0.8: "Fit quality is then the proportion of time points where the fitted profile is within the error margin of the measured profile. A fit is considered good if fit quality is above a given threshold"
- [results] R notebook reproduces results: "Complete code to reproduce all of the results for this and the following section is attached as an R notebook in Additional file 1."
- [results] Hard thresholds limitation: "Most notably, Genexpi masks uncertainty in the data and uses multiple hard thresholds."
- [readme] Wiki documentation location: "Detailed information can be found on the project's wiki: https://github.com/cas-bioinf/genexpi/wiki"
