---
name: r-markdown-reproducible-analysis
description: Use when reproducing or re-running the Genexpi evaluation pipeline on
  time-course expression data (e.g. GSE6865 SigB or GDS38 yeast datasets), since the
  complete parameterized workflow is shipped as R notebooks in the paper's Additional
  file 1.
when_to_use_negative:
- If the goal is a point-and-click network-inference session in Cytoscape — use the
  CyGenexpi plugin instead of re-running the evaluation notebooks
- If the dataset has no literature/ChIP-derived candidate regulon to validate against
  — the notebooks are evaluation harnesses, not standalone inference tools
- If the data are static (single-sample) rather than time-course expression profiles
  — the spline smoothing and ODE model assume time series
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Genexpi
  role: Core inference engine invoked from the notebooks (also usable as R package
    / CLI) to fit the ODE regulation model and score fit quality
  repo: https://github.com/cas-bioinf/genexpi
- name: R
  role: Runtime environment in which the R Markdown evaluation notebooks are rendered
    to reproduce reported results
- name: ARACNE
  role: TD-ARACNE baseline computed in parallel within the yeast evaluation notebook
    for comparison against Genexpi
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:557becd7ea456ca26ccf4e4a0a2172e5d6df88c9422b0e14fe721f301115f3cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cas-bioinf/genexpi
  tools:
  - ARACNE
  - Genexpi
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# R Markdown reproducible analysis

## Summary

Reproduce the Genexpi paper's regulon-recovery evaluations by rendering the authors' R Markdown notebooks (evaluation.Rmd, evaluation_sacharomyces.Rmd) with R, which encode the full pipeline from spline smoothing and simulated-annealing model fitting to validation against reported recovery rates.

## When to use

Use when reproducing or re-running the Genexpi evaluation pipeline on time-course expression data (e.g. GSE6865 SigB or GDS38 yeast datasets), since the complete parameterized workflow is shipped as R notebooks in the paper's Additional file 1.

## When NOT to use

- If the goal is a point-and-click network-inference session in Cytoscape — use the CyGenexpi plugin instead of re-running the evaluation notebooks
- If the dataset has no literature/ChIP-derived candidate regulon to validate against — the notebooks are evaluation harnesses, not standalone inference tools
- If the data are static (single-sample) rather than time-course expression profiles — the spline smoothing and ODE model assume time series

## Inputs

- Time-course gene expression data (e.g. GEO GSE6865 for B. subtilis, GDS38 for S. cerevisiae cell cycle)
- Literature-derived regulon gene lists (Subtiwiki SigB regulon, YEASTRACT regulons for eight yeast transcription factors)
- R Markdown notebook from Additional file 1 of the Genexpi paper (evaluation.Rmd, evaluation_sacharomyces.Rmd)
- Genexpi R package / OpenCL core from github.com/cas-bioinf/genexpi

## Outputs

- Rendered HTML report (e.g. sigb_evaluation_report.html)
- Per-TF recovery-proportion table (yeast_regulator_proportions.csv with Genexpi, TD-ARACNE, regulator, random columns)
- SigB regulon recovery fraction and random-regulator false-positive rate

## How to apply

Load the supplied R Markdown notebook (evaluation.Rmd for B. subtilis SigB, evaluation_sacharomyces.Rmd for the S. cerevisiae GDS38 cell-cycle comparison) together with the time-course expression data and the literature-derived regulon lists (Subtiwiki as of January 2017, or YEASTRACT as of 2018-02-09). The notebooks implement B-spline smoothing (degrees of freedom ≈ half the number of measurement points, e.g. 4 DF for GSE6865), exclusion of no-change and constant-synthesis genes via simulated-annealing fits, and Genexpi ODE fitting with 128 annealing runs per gene-regulator set using the OpenCL core and regularization bounds (max|w_j*y_j| < 10, min|rho| < 0.5, k1 below maximal expression). Renders can be validated by diffing computed proportions against paper values: SigB recovery ≈ 90% with a 4-DF spline, and per-TF yeast proportions (FKH1 0.30, ACE2 0.40, SWI6 0.10, random 0.20). Genexpi is also available as an R package and command-line tool from github.com/cas-bioinf/genexpi, so notebooks can be re-run against a locally built copy.

## Related tools

- **Genexpi** (Core inference engine invoked from the notebooks (also usable as R package / CLI) to fit the ODE regulation model and score fit quality) — https://github.com/cas-bioinf/genexpi
- **R** (Runtime environment in which the R Markdown evaluation notebooks are rendered to reproduce reported results)
- **ARACNE** (TD-ARACNE baseline computed in parallel within the yeast evaluation notebook for comparison against Genexpi)

## Examples

```
rmarkdown::render("evaluation.Rmd", output_file = "sigb_evaluation_report.html")
```

## Evaluation signals

- SigB regulon recovery with the 4-DF spline model matches the reported ~90%
- Random-regulator positive rate matches the reported roughly one third for SigB targets
- Yeast per-TF proportions match the published values: FKH1 0.30, ACE2 0.40, SWI6 0.10, random 0.20
- Good fits pass the default fit-quality threshold of 0.8 before regulon proportions are counted
- Rendered notebook executes without errors using 128 annealing runs per gene and regulator set

## Limitations

- Genexpi masks uncertainty in the data and uses multiple hard thresholds, which limits the robustness of evaluation outcomes
- No software version numbers are stated for Genexpi, R, RStudio, or Cytoscape, complicating exact environment reproduction
- No accession ID or repository is provided for the Sacharomyces evaluation dataset beyond evaluation_sacharomyces.Rmd
- Genexpi was designed for bacterial regulons; eukaryotic results (e.g. MBP1/SWI4/SWI6 complexes) degrade to random-indistinguishable levels

## Evidence

- [results] Complete code to reproduce all of the results for this and the following section is attached as an R notebook in Additional file 1.: "Complete code to reproduce all of the results for this and the following section is attached as an R notebook in Additional file 1."
- [discussion] evaluation_sacharomyces.Rmd notebook reference: "evaluation_sacharomyces.Rmd – R Markdown notebook to reproduce the evaluation on Sacharomyces data."
- [abstract] Genexpi R package availability: "Genexpi is also available as a standalone command line tool and an R package."
- [methods] Smoothing degrees of freedom: "Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom equal to approximately half the number of"
- [methods] Simulated annealing runs: "Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values."
- [readme] README wiki pointer: "Detailed information can be found on the project's wiki: https://github.com/cas-bioinf/genexpi/wiki"
