---
name: regulon-recovery-benchmarking
description: Use when a candidate regulon (ChIP, knockout, or database-derived) plus
  a matching expression time series (e.g. GSE6865, GSE44415, GDS38) is available and
  you need to quantify recovery rate versus a random-profile baseline.
when_to_use_negative:
- 'Regulators known to act in complexes (e.g. MBP1, SWI4, SWI6 in yeast): Genexpi
  gave its worst, random-indistinguishable results there because complex regulation
  breaks its single-regulator model'
- The regulon gene list is unrelated to any time-course expression data, so smoothing
  and ODE fitting cannot be performed
- 'A complete regulatory network is required: time-series expression data cannot in
  principle provide complete interaction information, so Genexpi is best used as one
  of multiple sources of insight'
edam_operation: http://edamontology.org/operation_2437
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: Genexpi
  role: ODE-based regulon inference via simulated annealing (128 runs) with regularization
    and 0.8 fit-quality threshold, run through R notebook or OpenCL core
  repo: github.com/cas-bioinf/genexpi
- name: ARACNE
  role: TD-Aracne comparison method run on the same smoothed time-course data for
    each regulon variant
- name: R
  role: Hosts the evaluation.Rmd / evaluation_sacharomyces.Rmd notebooks used for
    smoothing, filtering, fitting, and computing recovery statistics
  repo: github.com/cas-bioinf/genexpi
- name: CyGenexpi
  role: Cytoscape plugin wrapping Genexpi for interactive regulon discovery workflows
  repo: github.com/cas-bioinf/genexpi
provenance:
  source_task_ids:
  - task_002
  - task_001
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:cc02936302e13d4d948e623acefa1ffc343ff1d0eb21076c03d6879454355267
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ARACNE
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
yes_rate: 0.9
n_claims: 10
---

# regulon-recovery-benchmarking

## Summary

Benchmark a time-course gene-network inference method (Genexpi vs TD-Aracne) by measuring how much of a literature-derived regulon (e.g. SigB from Subtiwiki, SigR from ChIP-chip/knockout studies, yeast TFs from YEASTRACT) it recovers, and how often it falsely accepts random regulator profiles.

## When to use

Use when a candidate regulon (ChIP, knockout, or database-derived) plus a matching expression time series (e.g. GSE6865, GSE44415, GDS38) is available and you need to quantify recovery rate versus a random-profile baseline.

## When NOT to use

- Regulators known to act in complexes (e.g. MBP1, SWI4, SWI6 in yeast): Genexpi gave its worst, random-indistinguishable results there because complex regulation breaks its single-regulator model
- The regulon gene list is unrelated to any time-course expression data, so smoothing and ODE fitting cannot be performed
- A complete regulatory network is required: time-series expression data cannot in principle provide complete interaction information, so Genexpi is best used as one of multiple sources of insight

## Inputs

- Expression time series (GEO accessions GSE6865, GSE44415, or GDS38)
- Literature-derived regulon gene lists (Subtiwiki, YEASTRACT, ChIP-chip of Kallifidas et al., knockout of Kim et al.)
- evaluation.Rmd and evaluation_sacharomyces.Rmd R Markdown notebooks (Additional file 1 of the Genexpi paper)
- Random regulator profiles for the false-positive baseline

## Outputs

- Per-method, per-regulon-variant table of recovered targets (e.g. sigr_regulon_recovery_comparison.csv)
- Per-TF proportion table (Genexpi, TD-ARACNE, regulator, random; e.g. yeast_regulator_proportions.csv)
- Rendered evaluation report (e.g. sigb_evaluation_report.html)

## How to apply

Smooth the expression time series by linear regression on a B-spline basis with degrees of freedom approximately half the number of measurement points (e.g. the 4-DF spline for GSE6865). Exclude genes fitting the no-change model and the constant-synthesis model (both fitted via simulated annealing in OpenCL), then fit the Genexpi ODE regulation model with 128 annealing runs per gene–regulator set, applying the regularization term r(beta) with c ≈ one tenth of the number of time points, and keep fits with fit quality above the default 0.8 threshold. Run TD-Aracne on the same smoothed data as the comparison method, and add a random-regulator baseline. Count recovered targets against the published regulon gene lists (Subtiwiki SigB as of January 2017, Kallifidas ChIP-chip and Kim knockout SigR versions, YEASTRACT regulons with 30 randomly selected targets per TF) and assemble method-per-regulon comparison tables.

## Related tools

- **Genexpi** (ODE-based regulon inference via simulated annealing (128 runs) with regularization and 0.8 fit-quality threshold, run through R notebook or OpenCL core) — github.com/cas-bioinf/genexpi
- **ARACNE** (TD-Aracne comparison method run on the same smoothed time-course data for each regulon variant)
- **R** (Hosts the evaluation.Rmd / evaluation_sacharomyces.Rmd notebooks used for smoothing, filtering, fitting, and computing recovery statistics) — github.com/cas-bioinf/genexpi
- **CyGenexpi** (Cytoscape plugin wrapping Genexpi for interactive regulon discovery workflows) — github.com/cas-bioinf/genexpi

## Examples

```
Rscript -e "rmarkdown::render('evaluation.Rmd', output_file='sigb_evaluation_report.html')"
```

## Evaluation signals

- SigB (GSE6865, 4-DF spline): Genexpi recovers ~90% of the Subtiwiki literature regulon and predicts roughly one third of random profiles as regulating SigB targets
- SigR (GSE44415): Genexpi recovers substantially more targets from literature-derived regulons than from random regulons, while TD-Aracne's recovery is barely distinguishable between literature and random baselines
- Yeast (GDS38/YEASTRACT): per-TF proportions match reported values (FKH1 0.30, ACE2 0.40, SWI6 0.10, random 0.20)
- All kept regulations have fit quality above the default 0.8 threshold
- Genes fitted by the no-change and constant-synthesis models are excluded from the target set before recovery counting

## Limitations

- Genexpi was designed for bacterial regulons; eukaryotic applicability requires caution and is not fully validated
- Genexpi masks uncertainty in the data and uses multiple hard thresholds
- For complex-forming TFs (MBP1, SWI4, SWI6) results were random-indistinguishable, breaking Genexpi's expected model
- No software version numbers were stated for Genexpi, R, RStudio, or Cytoscape
- Time-series expression data cannot in principle provide complete regulatory interaction information

## Evidence

- [methods] B-spline smoothing and DF rule: "We have had good results with linear regression of B-spline basis with degrees of freedom equal to approximately half the number of"
- [methods] Simulated annealing runs: "For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values."
- [methods] Fit-quality threshold: "A fit is considered good if fit quality is above a given threshold (the default value is 0.8)."
- [results] Yeast 30-target subsampling: "After excluding constant and constant synthesis targets (steps 3&4 of the workflow), we selected 30 targets for each gene at random to reduce computational burden."
- [results] Datastes and regulon sources: "the SigB regulon of B. subtilis from Subtiwiki [14] as of January 2017"
- [discussion] Eukaryotic caution: "Genexpi was designed for bacterial regulons, but can be – with caution – employed also for eukaryotic data."
- [discussion] Reproduction notebooks: "evaluation_sacharomyces.Rmd – R Markdown notebook to reproduce the evaluation on Sacharomyces data."
- [readme] README repo pointer: "Gene Network Inferece from Expression Time Series.

Detailed information can be found on the project's wiki: https://github.com/cas-bioinf/genexpi/wiki"
