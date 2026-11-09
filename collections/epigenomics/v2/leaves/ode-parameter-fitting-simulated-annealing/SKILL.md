---
name: ode-parameter-fitting-simulated-annealing
description: Use when time-course gene expression profiles must be fitted with an
  ODE regulatory model to decide which candidate gene–regulator interactions are real,
  as in Genexpi/CyGenexpi regulon inference.
when_to_use_negative:
- The gene's expression does not change significantly over the whole time range —
  the Genexpi model contains no information for such genes and they are excluded as
  regulators and targets.
- The gene is better explained by the constant synthesis model — such genes are excluded
  as targets because any regulator can fit them with w = 0 and large b.
- Eukaryotic data involving transcription factors that regulate in complexes (e.g.
  MBP1, SWI4, SWI6), which breaks Genexpi's expected single-regulator model and yields
  random-indistinguishable results.
edam_operation: http://edamontology.org/operation_3893
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2258
tools:
- name: Genexpi
  role: Core tool that minimizes the ODE error function by simulated annealing (128
    runs per gene–regulator set) via OpenCL; available as command-line tool, R package,
    and CyGenexpi plugin
  repo: https://github.com/cas-bioinf/genexpi
- name: CyGenexpi
  role: Cytoscape plugin wrapping Genexpi's annealing-based fitting for interactive
    regulon discovery
  repo: https://github.com/cas-bioinf/genexpi
- name: R
  role: R interface/package used for preprocessing (B-spline smoothing, filtering)
    and to run the fitting in notebooks such as evaluation_sacharomyces.Rmd
  repo: https://github.com/cas-bioinf/genexpi
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:e5d19ca4360eb7d5057195c27d863cc70ad538c51d9c9195e747e2584b31bcba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cas-bioinf/genexpi
  tools:
  - ARACNE
  - CyGenexpi
  - Genexpi
  - R
  edam_operation: http://edamontology.org/operation_3893
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# ode-parameter-fitting-simulated-annealing

## Summary

Fits Genexpi's ordinary differential equation model of gene regulation to smoothed time-course expression data by minimizing the error function with simulated annealing in OpenCL, running 128 annealing runs per gene–regulator set with regularization r(beta) and a fit-quality threshold of 0.8.

## When to use

Use when time-course gene expression profiles must be fitted with an ODE regulatory model to decide which candidate gene–regulator interactions are real, as in Genexpi/CyGenexpi regulon inference.

## When NOT to use

- The gene's expression does not change significantly over the whole time range — the Genexpi model contains no information for such genes and they are excluded as regulators and targets.
- The gene is better explained by the constant synthesis model — such genes are excluded as targets because any regulator can fit them with w = 0 and large b.
- Eukaryotic data involving transcription factors that regulate in complexes (e.g. MBP1, SWI4, SWI6), which breaks Genexpi's expected single-regulator model and yields random-indistinguishable results.

## Inputs

- Smoothed time-course expression profiles (B-spline linear regression, df ≈ number of measurement points / 2), e.g. GSE6865, GSE44415, or GDS38 series
- Candidate regulator–target sets from ChIP experiments, knockouts, or database mining (e.g. Subtiwiki, YEASTRACT, Kallifidas et al. ChIP-chip)
- Number of measurement points per profile (sets degrees of freedom and regularization constant c)

## Outputs

- Fitted ODE parameters (w, beta) per gene–regulator pair from 128 annealing runs
- Fit-quality scores per regulation; fitted regulations retained above the 0.8 threshold
- Inferred regulons of recovered targets, e.g. comparison tables such as sigr_regulon_recovery_comparison.csv or yeast_regulator_proportions.csv

## How to apply

First smooth each expression profile by linear regression on a B-spline basis with degrees of freedom approximately equal to half the number of measurement points, then subsample so the ODE can be integrated accurately with the cheap Euler method in OpenCL. For each gene and candidate regulator set, execute 128 simulated-annealing runs with different initial parameter values and apply the regularization term r(beta) — with c set to about one tenth of the number of time points — to force parameters into biologically interpretable ranges and discriminate between models. Judge each fit by fit quality, defined as the proportion of time points where the fitted profile lies within the error margin of the measured profile; keep regulations with fit quality above the default 0.8 threshold. Before fitting, exclude genes that do not change significantly over the time range and genes better explained by the constant synthesis model, since those could be fitted by any regulator via w = 0 and large b.

## Related tools

- **Genexpi** (Core tool that minimizes the ODE error function by simulated annealing (128 runs per gene–regulator set) via OpenCL; available as command-line tool, R package, and CyGenexpi plugin) — https://github.com/cas-bioinf/genexpi
- **CyGenexpi** (Cytoscape plugin wrapping Genexpi's annealing-based fitting for interactive regulon discovery) — https://github.com/cas-bioinf/genexpi
- **R** (R interface/package used for preprocessing (B-spline smoothing, filtering) and to run the fitting in notebooks such as evaluation_sacharomyces.Rmd) — https://github.com/cas-bioinf/genexpi

## Examples

```
Rscript -e "genexpi::fit(experimentData, regulatorSets, chainLength=10000, chainsPerChainSet=128, tempReductionRate=0.9, warmupThreshold=0.8, compareWithChainLength=1000, saveChain=TRUE)"
```

## Evaluation signals

- Fit quality (proportion of time points within the error margin of the measured profile) exceeds the default 0.8 threshold for retained regulations.
- Each gene–regulator set was fitted with all 128 annealing runs and the regularization constant c was set to approximately one tenth of the number of time points.
- Genes excluded by the no-change and constant-synthesis pre-filters match the expected set before any ODE fit is attempted.
- On the S. cerevisiae GDS38 benchmark, recovered YEASTRACT proportions reach the reported values (FKH1 0.30, ACE2 0.40, random baseline 0.20).
- On GSE44415 SigR regulons, literature-derived regulons (Kallifidas ChIP-chip and Kim knockout variants) recover substantially more targets than random regulator profiles.

## Limitations

- Genexpi masks uncertainty in the data and uses multiple hard thresholds.
- Genexpi was designed for bacterial regulons; eukaryotic use requires caution, and complex-regulating TFs (MBP1, SWI4, SWI6) yield random-indistinguishable results.
- Time-series expression data cannot in principle provide complete information about regulatory interactions; results are best used as one of multiple sources of insight.
- No software version numbers are stated for Genexpi, R, RStudio, or Cytoscape in the article.

## Evidence

- [methods] 128 annealing runs with regularization and 0.8 threshold: "Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values."
- [methods] fit quality definition and threshold: "Fit quality is then the proportion of time points where the fitted profile is within the error margin of the measured profile. A fit is considered good if fit quality is above a given threshold"
- [methods] Euler integration enabling OpenCL evaluation: "The subsampling then allows us to integrate (1) accurately with the computationally cheap Euler method, making evaluation of the error function fast and easy to implement in OpenCL."
- [methods] B-spline smoothing with df ≈ half measurement points: "Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom equal to approximately half the number of"
- [methods] regularization rationale: "To discriminate between those models and to force the parameters into biologically interpretable ranges, we introduce the regularization term r(β)."
- [readme] README build context: "Gene Network Inferece from Expression Time Series. Detailed information can be found on the project's wiki"
