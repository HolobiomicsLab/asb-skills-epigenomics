---
name: regulon-dataset-preparation
description: Use when a candidate regulon definition (ChIP-derived, knockout-derived,
  or literature/database-mined) must be paired with a time-course expression series
  and preprocessed before network inference.
when_to_use_negative:
- The data are not time-course measurements — Cytoscape does not natively support
  time series, and the Genexpi ODE model requires expression over time
- The candidate regulon has no source (no ChIP, knockout, or database/literature-mined
  candidates) to combine with the expression data
- The target organism's regulons are eukaryotic and no caution is applied — the article
  states Genexpi was designed for bacterial regulons and eukaryotic use requires validation
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Genexpi
  role: 'Core tool whose workflow defines the dataset preparation: smoothing, no-change
    and constant-synthesis filtering; available as command-line tool and R package'
  repo: https://github.com/cas-bioinf/genexpi
- name: R
  role: Interface for running Genexpi core functionality and reproducing preparation
    steps via the attached R notebook
  repo: https://github.com/cas-bioinf/genexpi
- name: CyGenexpi
  role: Cytoscape plugin integrating the Genexpi workflow for users preferring a point&click
    interface
  repo: https://github.com/cas-bioinf/genexpi
- name: CyDataseries
  role: Cytoscape plugin for importing and handling time series data, since Cytoscape
    lacks native time-series support
  repo: https://github.com/cas-bioinf/genexpi
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:9bfb615cfbf1924fce502401e32ac5ea764571ba7b798e2c04b05c98a2a9b225
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cas-bioinf/genexpi
  tools:
  - ARACNE
  - CyDataseries
  - CyGenexpi
  - Genexpi
  - Java
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# regulon-dataset-preparation

## Summary

Prepares paired inputs for Genexpi-style regulon inference: a network of putative regulatory relationships (from databases like Subtiwiki, YEASTRACT, or ChIP-chip/knockout experiments) combined with time-course gene expression data, smoothed via B-spline linear regression and cleaned of non-informative genes.

## When to use

Use when a candidate regulon definition (ChIP-derived, knockout-derived, or literature/database-mined) must be paired with a time-course expression series and preprocessed before network inference.

## When NOT to use

- The data are not time-course measurements — Cytoscape does not natively support time series, and the Genexpi ODE model requires expression over time
- The candidate regulon has no source (no ChIP, knockout, or database/literature-mined candidates) to combine with the expression data
- The target organism's regulons are eukaryotic and no caution is applied — the article states Genexpi was designed for bacterial regulons and eukaryotic use requires validation

## Inputs

- Candidate regulon definitions (e.g. Subtiwiki SigB regulon, SigR regulon from ChIP-chip (Kallifidas et al.) or knockouts (Kim et al.), YEASTRACT regulons)
- Time-course gene expression datasets (e.g. GSE6865, GSE44415, GDS38)
- Number of time points in the expression series (to set B-spline degrees of freedom)

## Outputs

- Smoothed expression profiles (continuous curves from B-spline linear regression)
- Filtered regulon/gene sets excluding no-change and constant-synthesis genes
- Paired regulon + expression dataset ready for Genexpi parameter fitting

## How to apply

Start with a network of putative regulations from database mining (e.g. Subtiwiki for the B. subtilis SigB regulon, YEASTRACT for yeast) or experiments (e.g. ChIP-chip or knockout-derived SigR regulons of Streptomyces coelicolor), and import the matching expression time series (e.g. GSE44415 for SigR, GSE6865 for SigA). Smooth the expression profiles with a B-spline linear regression, using degrees of freedom equal to approximately half the number of time points, to provide a continuous curve for fitting. Then remove genes whose expression does not change significantly over the whole time range (no-change model) and genes that can be fitted by the constant synthesis model, since either class carries no regulatory information and the latter can be fitted by any regulator with w=0 and large b. Optionally inspect the results of these two removal steps and override the algorithm's decisions; for large eukaryotic datasets, randomly subsample targets (e.g. 30 per transcription factor) to reduce computational burden.

## Related tools

- **Genexpi** (Core tool whose workflow defines the dataset preparation: smoothing, no-change and constant-synthesis filtering; available as command-line tool and R package) — https://github.com/cas-bioinf/genexpi
- **R** (Interface for running Genexpi core functionality and reproducing preparation steps via the attached R notebook) — https://github.com/cas-bioinf/genexpi
- **CyGenexpi** (Cytoscape plugin integrating the Genexpi workflow for users preferring a point&click interface) — https://github.com/cas-bioinf/genexpi
- **CyDataseries** (Cytoscape plugin for importing and handling time series data, since Cytoscape lacks native time-series support) — https://github.com/cas-bioinf/genexpi

## Examples

```
Rscript -e "library(genexpi); smoothed <- smoothData(expressionTimeSeries, dfs = ncol(expressionTimeSeries)/2)"
```

## Evaluation signals

- Genes excluded by the no-change model are absent from both the regulator and target sets
- Genes fitted by the constant synthesis model are excluded as targets (otherwise any regulator could fit them with w=0 and large b)
- Smoothed profiles are continuous curves produced by B-spline linear regression with degrees of freedom ≈ half the number of time points
- The prepared dataset pairs each candidate regulon (e.g. SigB from Subtiwiki, SigR ChIP-chip vs knockout) with the correct expression series (GSE6865 for SigA/SigB context, GSE44415 for both SigR versions)
- Reproduction code in the attached R notebook regenerates the same filtered dataset

## Limitations

- For large eukaryote datasets (e.g. yeast), targets must be subsampled at random (30 per transcription factor) to reduce computational burden, which introduces sampling variability
- Human inspection of the no-change and constant-synthesis filtering steps is optional but results may need manual overriding of the algorithm's decisions
- No accession ID or repository was provided for the Sacharomyces evaluation dataset, complicating exact reproduction of that preparation

## Evidence

- [methods] B-spline smoothing with degrees of freedom ≈ half the time points: "Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom equal to approximately half the number of"
- [results] Start from putative regulatory network: "Start with a network of putative regulations either obtained from database mining or experiments."
- [results] No-change filtering step: "Remove genes whose expression does not change significantly throughout the whole time-course."
- [methods] Constant-synthesis filtering rationale: "Since genes with constant synthesis could be fitted by any regulator by simply putting w = 0, and large b, those genes are excluded as targets."
- [readme] Repository with documentation: "Detailed information can be found on the project's wiki: https://github.com/cas-bioinf/genexpi/wiki"
