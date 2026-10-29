---
name: sigma-factor-regulon-inference
description: Use when you have a candidate regulon list for a transcription factor
  (from ChIP or literature) plus time-course expression data, and need to predict
  which candidate genes are genuinely regulated.
when_to_use_negative:
- Genes whose expression does not change significantly throughout the whole time-course,
  or genes that can be modelled by the constant synthesis model — these are excluded
  as targets and regulators
- Eukaryotic regulatory inference where full validation is required — Genexpi was
  designed for bacterial regulons and eukaryotic use requires caution
- Analyses needing complete regulatory interaction maps — time-series expression data
  cannot in principle provide complete information and results are incomplete on their
  own
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Genexpi
  role: Core software performing B-spline smoothing, simulated-annealing ODE fitting,
    filtering, and regulon target prediction; available as standalone command-line
    tool and R package
  repo: http://github.com/cas-bioinf/genexpi/wiki/
- name: CyGenexpi
  role: Cytoscape plugin integrating Genexpi for interactive regulon discovery from
    time-course expression data
- name: CyDataseries
  role: Cytoscape plugin for importing and handling the time series data consumed
    by CyGenexpi
- name: Cytoscape
  role: Platform hosting CyGenexpi and CyDataseries for network visualization and
    analysis
- name: R
  role: Language interface to Genexpi's core functionality; reproducible analyses
    via R Markdown notebooks (e.g. evaluation_sacharomyces.Rmd)
  repo: http://github.com/cas-bioinf/genexpi/wiki/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:017c135dd1d7ec279d5338e601db28c9d335256cd6728f1d029b63ce7fa6baea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CyDataseries
  - CyGenexpi
  - Cytoscape
  - Genexpi
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# sigma-factor-regulon-inference

## Summary

A method for inferring sigma-factor regulons by combining ChIP- or literature-derived candidate regulation lists with time-course gene expression data, fitting an ODE regulation model with simulated annealing and filtering through no-change, constant-synthesis, and fit-quality thresholds (e.g. GSE6865 Bacillus subtilis SigA regulon prediction with Genexpi).

## When to use

Use when you have a candidate regulon list for a transcription factor (from ChIP or literature) plus time-course expression data, and need to predict which candidate genes are genuinely regulated.

## When NOT to use

- Genes whose expression does not change significantly throughout the whole time-course, or genes that can be modelled by the constant synthesis model — these are excluded as targets and regulators
- Eukaryotic regulatory inference where full validation is required — Genexpi was designed for bacterial regulons and eukaryotic use requires caution
- Analyses needing complete regulatory interaction maps — time-series expression data cannot in principle provide complete information and results are incomplete on their own

## Inputs

- Time-course gene expression dataset (e.g. GEO series GSE6865 for B. subtilis, GSE44415 for S. coelicolor, GDS38 for S. cerevisiae)
- Candidate regulon list from ChIP experiments or literature/database mining (e.g. SigA regulon of Bacillus subtilis, SigB regulon from Subtiwiki, SigR regulons of Streptomyces coelicolor)
- Model parameters: number of annealing runs, regularization constant, fit-quality threshold

## Outputs

- List of predicted regulon targets passing all filter steps (e.g. siga_predictions.csv with 215 predicted SigA genes)
- Fit-quality statistics for each gene/regulator model
- Optionally, labelled confirmation status for a validated subset of predictions

## How to apply

Smooth the expression time series by linear regression on a B-spline basis with degrees of freedom equal to approximately half the number of measurement points. Exclude genes not changing significantly over the time course and genes explainable by the constant synthesis model, each fitted via simulated annealing in OpenCL. For each remaining gene and candidate regulator set, run 128 simulated-annealing fits of the ODE regulation model (with regularization constant c set to roughly one tenth of the number of smoothed time points), and retain genes whose fits exceed the fit-quality threshold (default 0.8, i.e. the proportion of time points within the error margin of the measured profile). Verify the resulting target list against reported counts (e.g. 215 novel SigA targets on GSE6865) and optionally record downstream in-vitro confirmation status as labelled predictions rather than new experiments.

## Related tools

- **Genexpi** (Core software performing B-spline smoothing, simulated-annealing ODE fitting, filtering, and regulon target prediction; available as standalone command-line tool and R package) — http://github.com/cas-bioinf/genexpi/wiki/
- **CyGenexpi** (Cytoscape plugin integrating Genexpi for interactive regulon discovery from time-course expression data)
- **CyDataseries** (Cytoscape plugin for importing and handling the time series data consumed by CyGenexpi)
- **Cytoscape** (Platform hosting CyGenexpi and CyDataseries for network visualization and analysis)
- **R** (Language interface to Genexpi's core functionality; reproducible analyses via R Markdown notebooks (e.g. evaluation_sacharomyces.Rmd)) — http://github.com/cas-bioinf/genexpi/wiki/

## Examples

```
git clone https://github.com/cas-bioinf/genexpi
```

## Evaluation signals

- Predicted target list count matches the reported value (215 novel SigA targets on the GSE6865 dataset)
- Every retained gene's fit quality exceeds the default threshold of 0.8 (proportion of time points within the error margin)
- Genes fitted by the no-change model and constant synthesis model are absent from the final target list
- For each gene and candidate regulator set, 128 annealing runs were executed with different initial parameter values
- The in-vitro confirmation subset is labelled per the paper's reported results (5 of 10 tested genes SigA-dependent; for the remaining five regulation could not be excluded)

## Limitations

- Genexpi masks uncertainty in the data and uses multiple hard thresholds
- Time-series expression data cannot in principle provide complete information about the regulatory interactions; results are best used as one of multiple sources of insight
- Designed for bacterial regulons; applicability to eukaryotic data is not fully validated and requires caution
- No software version numbers are stated for Genexpi, R, RStudio, or Cytoscape in the article

## Evidence

- [abstract] Genexpi as a sigma-factor identification tool combining candidate and time-series inputs: "We introduce Genexpi: a tool to identify sigma factors by combining candidates obtained from ChIP experiments or literature mining with time-course gene expression data."
- [methods] Simulated annealing fitting protocol with 128 runs per gene/regulator set: "Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values."
- [methods] Fit-quality definition underlying the good-fit threshold: "Fit quality is then the proportion of time points where the fitted profile is within the error margin of the measured profile."
- [results] SigA regulon prediction grounded in the GSE6865 B. subtilis time series: "the SigA regulon of Bacillus subtilis combined with the expression time series from GSE6865"
- [intro] R language interface for reproducible regulon inference: "CyGenexpi is built on top of the Genexpi software package that provides the core functionality also as a command-line tool and an interface to the R language."
- [readme] Repository documentation and wiki for build and usage details: "Detailed information can be found on the project's wiki: https://github.com/cas-bioinf/genexpi/wiki"
