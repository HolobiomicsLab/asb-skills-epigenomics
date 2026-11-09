---
name: gene-regulatory-network-method-comparison
description: Use when validating a gene regulatory network inference method on time-course
  expression data against a curated reference regulon set, especially to compare two
  methods and a random baseline per transcription factor.
when_to_use_negative:
- The study system has no curated reference regulons (e.g. no YEASTRACT/Subtiwiki
  equivalent), since recovery proportions cannot be computed against documented regulations.
- Expression data are not time-course measurements; Genexpi's ODE model and smoothing
  assume expression profiles over time.
- The transcription factors of interest are known to regulate in complexes (e.g. MBP1/SWI4/SWI6),
  where Genexpi's single-regulator model is expected to break and yield random-indistinguishable
  results.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: Genexpi
  role: ODE-model regulon inference by simulated annealing on smoothed time-course
    data; compared against TD-ARACNE and a random baseline
  repo: github.com/cas-bioinf/genexpi
- name: TD-ARACNE
  role: Competing network inference method evaluated on the same filtered targets
    and reference regulons
- name: R
  role: Environment for the evaluation notebook, B-spline smoothing, and assembly
    of the per-TF proportion table
  repo: github.com/cas-bioinf/genexpi
- name: Cytoscape
  role: Platform hosting the CyGenexpi plugin through which the Genexpi workflow can
    be executed interactively
  repo: github.com/cas-bioinf/genexpi
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:bee2dd061fd8e74ed1a6dcd457ac011fd53088715970ed9ca5ebfc27d1892f7d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ARACNE
  - Cytoscape
  - Genexpi
  - R
  - TD-ARACNE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# gene-regulatory-network-method-comparison

## Summary

A benchmarking skill that compares inferred regulons (e.g. Genexpi vs TD-ARACNE vs a random baseline) against curated reference regulons from databases such as YEASTRACT, quantifying the per-transcription-factor proportion of documented regulations recovered at or above chance level.

## When to use

Use when validating a gene regulatory network inference method on time-course expression data against a curated reference regulon set, especially to compare two methods and a random baseline per transcription factor.

## When NOT to use

- The study system has no curated reference regulons (e.g. no YEASTRACT/Subtiwiki equivalent), since recovery proportions cannot be computed against documented regulations.
- Expression data are not time-course measurements; Genexpi's ODE model and smoothing assume expression profiles over time.
- The transcription factors of interest are known to regulate in complexes (e.g. MBP1/SWI4/SWI6), where Genexpi's single-regulator model is expected to break and yield random-indistinguishable results.

## Inputs

- Time-course gene expression matrix (e.g. GDS38 Saccharomyces cerevisiae cell-cycle series)
- Reference regulons per transcription factor (e.g. YEASTRACT regulons for eight TFs, as of 2018-02-09)
- R Markdown evaluation notebook (evaluation_sacharomyces.Rmd)
- Genexpi model parameters: fit-quality threshold 0.8, 128 annealing runs, regularization c ≈ n_timepoints/10, 30 random targets per TF

## Outputs

- Per-TF table of recovered-regulation proportions (Genexpi, TD-ARACNE, regulator, random), e.g. yeast_regulator_proportions.csv
- Per-TF comparison of each method against the random baseline

## How to apply

Load the evaluation R notebook (e.g. evaluation_sacharomyces.Rmd) together with the time-series dataset (e.g. GDS38 S. cerevisiae cell-cycle) and reference regulons (YEASTRACT for eight transcription factors). Pre-filter genes with simulated annealing fits: exclude genes that do not change significantly and genes better explained by the constant synthesis model, judged by the fit-quality threshold of 0.8. Smooth expression profiles with linear regression on a B-spline basis using degrees of freedom approximately equal to half the number of measurement points, then fit the Genexpi ODE model by simulated annealing (128 annealing runs per gene and regulator set, regularization with c set to about one tenth of the number of time points). To limit computational burden, select 30 targets at random per transcription factor after the exclusion steps. For each TF compute the proportion of YEASTRACT-documented regulations recovered at or above the chance level, in parallel for Genexpi, TD-ARACNE, and a random baseline, then assemble per-TF proportions (Genexpi, TD-ARACNE, regulator, random) into a table such as yeast_regulator_proportions.csv.

## Related tools

- **Genexpi** (ODE-model regulon inference by simulated annealing on smoothed time-course data; compared against TD-ARACNE and a random baseline) — github.com/cas-bioinf/genexpi
- **TD-ARACNE** (Competing network inference method evaluated on the same filtered targets and reference regulons)
- **R** (Environment for the evaluation notebook, B-spline smoothing, and assembly of the per-TF proportion table) — github.com/cas-bioinf/genexpi
- **Cytoscape** (Platform hosting the CyGenexpi plugin through which the Genexpi workflow can be executed interactively) — github.com/cas-bioinf/genexpi

## Examples

```
Rscript -e "rmarkdown::render('evaluation_sacharomyces.Rmd'); write.csv(yeast_regulator_proportions, 'yeast_regulator_proportions.csv')"
```

## Evaluation signals

- Computed per-TF proportions must match published values, e.g. FKH1 0.30, ACE2 0.40, SWI6 0.10, random 0.20.
- Genes must be excluded both as regulators and targets when they fail the no-change model or are better explained by the constant synthesis model at fit-quality threshold 0.8.
- Exactly 30 targets per transcription factor should remain after exclusion of constant and constant-synthesis targets, selected at random.
- The output CSV must contain four proportion columns per TF: Genexpi, TD-ARACNE, regulator, random.
- Methods performing no better than the random baseline (e.g. Genexpi on MBP1, SWI4, SWI6) should be flagged as random-indistinguishable.

## Limitations

- Genexpi gave its worst, random-indistinguishable results for MBP1, SWI4 and SWI6, transcription factors known to regulate in complexes, which breaks Genexpi's expected model.
- Genexpi was designed for bacterial regulons but can be — with caution — employed also for eukaryotic data; applicability outside bacterial regulons is not fully validated.
- Genexpi masks uncertainty in the data and uses multiple hard thresholds.
- Randomly selecting 30 targets per TF reduces computational burden but limits the comprehensiveness of the evaluation.

## Evidence

- [methods] smoothing: "Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom equal to approximately half the number of"
- [methods] annealing runs: "Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values."
- [methods] fit threshold: "A fit is considered good if fit quality is above a given threshold (the default value is 0.8)."
- [results] target subsampling: "After excluding constant and constant synthesis targets (steps 3&4 of the workflow), we selected 30 targets for each gene at random to reduce computational burden."
- [discussion] eukaryotic caveat: "Genexpi was designed for bacterial regulons, but can be – with caution – employed also for eukaryotic data."
- [discussion] notebook: "evaluation_sacharomyces.Rmd – R Markdown notebook to reproduce the evaluation on Sacharomyces data."
- [readme] README: "Gene Network Inferece from Expression Time Series."
