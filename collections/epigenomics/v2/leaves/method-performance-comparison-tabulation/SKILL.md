---
name: method-performance-comparison-tabulation
description: Use when a newly proposed computational method's identification performance
  must be quantified against established baselines under controlled perturbations
  like added noise or spurious-correlation confounders.
when_to_use_negative:
- The goal is causal validation of TR-drug relationships — the article notes TR knockout/activation
  experiments combined with drug response examination are required for causal validation,
  which simulation benchmarking cannot provide
- Reliable ChIP-seq target information is unavailable or targets are too few — RePhine
  identification is restricted to cases with reliable target information and would
  be misleading in such data
- The analysis goal is drug clustering by mechanism of action or biomarker discovery
  on real data (e.g., CCLE), rather than controlled head-to-head method performance
  evaluation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3517
tools:
- name: RePhine
  role: Method under test — infers TR-drug response associations from simulated pharmacogenomic
    and ChIP-seq/RP data
  repo: https://github.com/coexps/RePhine
- name: R
  role: Language used to run the simulation scripts (simulation_code.R), the RePhine
    package, and the baseline methods
  repo: https://github.com/coexps/RePhine
- name: Python
  role: Modified Python scripts (RP_calculation_modified.py) compute regulatory potential
    scores used in the simulated data
  repo: https://github.com/coexps/RePhine
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:ebf03f007f04f2fbcb049c072d2da2b15c07975dc4f59634bef382ecd55a940b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - Python
  - R
  - RePhine
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Method performance comparison tabulation

## Summary

Benchmark a new method (e.g., RePhine) against standard baselines (Pearson correlation, logistic regression, GSEA) on controlled simulated data with added noises or confounders, then tabulate per-scenario, per-method identification performance into a comparison table such as simulation_benchmark.csv.

## When to use

Use when a newly proposed computational method's identification performance must be quantified against established baselines under controlled perturbations like added noise or spurious-correlation confounders.

## When NOT to use

- The goal is causal validation of TR-drug relationships — the article notes TR knockout/activation experiments combined with drug response examination are required for causal validation, which simulation benchmarking cannot provide
- Reliable ChIP-seq target information is unavailable or targets are too few — RePhine identification is restricted to cases with reliable target information and would be misleading in such data
- The analysis goal is drug clustering by mechanism of action or biomarker discovery on real data (e.g., CCLE), rather than controlled head-to-head method performance evaluation

## Inputs

- Simulation R scripts from the method repository (simulation_code.R)
- Supplementary file defining simulation scenario parameters (File S1)
- Simulated pharmacogenomic data: TR and target gene expression, drug response, noise data, RP scores
- ChIP-seq-derived target/regulatory potential data for target inference
- Baseline method implementations (Pearson correlation, logistic regression, GSEA)

## Outputs

- Per-scenario, per-method performance comparison table (simulation_benchmark.csv)
- Identification performance metrics per method: true TR-drug relationship detection, false positives/negatives

## How to apply

Obtain the method's package and its simulation scripts from the official repository (e.g., github.com/coexps/RePhine: simulation_code.R) plus the supplementary parameter definitions (File S1 scenario parameters). Execute the simulation R code to generate synthetic pharmacogenomic data (drug response plus TR/target gene expression and RP scores) under an added-noise scenario, then re-run it under an added-confounders scenario where spurious correlations not mediated by the transcriptional regulator are introduced. Apply the new method and each baseline (Pearson correlation analysis, logistic regression model, gene set enrichment analysis) to the identical simulated inputs, scoring detection of the true simulated TR-drug relationship versus false positives and negatives. Assemble per-scenario, per-method performance values into a comparison table and validate that the new method outperforms all baselines in both scenarios.

## Related tools

- **RePhine** (Method under test — infers TR-drug response associations from simulated pharmacogenomic and ChIP-seq/RP data) — https://github.com/coexps/RePhine
- **R** (Language used to run the simulation scripts (simulation_code.R), the RePhine package, and the baseline methods) — https://github.com/coexps/RePhine
- **Python** (Modified Python scripts (RP_calculation_modified.py) compute regulatory potential scores used in the simulated data) — https://github.com/coexps/RePhine

## Examples

```
Rscript simulation_code.R  # run under File S1 noise and confounder parameters; apply RePhine, Pearson, logistic regression, GSEA; write simulation_benchmark.csv
```

## Evaluation signals

- The benchmark table must show RePhine achieving higher identification performance than Pearson correlation, logistic regression, and GSEA under the added-noise scenario
- The benchmark table must show RePhine outperforming all three baselines under the added-confounders scenario, where spurious correlations not mediated by the TR are introduced
- Each method is run on identical simulated inputs for a given scenario, so per-scenario values are directly comparable
- Output must be a per-scenario, per-method table (e.g., simulation_benchmark.csv) covering both scenarios and all four methods
- Simulated data must reproduce the scenario parameters from File S1, including simulated TR/target expression, noise data, and RP scores

## Limitations

- Benchmark success in simulation does not establish causality — upstream regulators or kinases may influence drug response while regulating downstream TRs
- RePhine-based evaluation requires reliable target information; simulated RP scores may not capture real ChIP-seq variability
- Robustness conclusions are limited to the tested noise and confounder scenarios; other perturbations (e.g., CNVs, confounding kinase/tumor suppressor mutations) are additional challenges noted for real data

## Evidence

- [abstract] RePhine outperformed Pearson correlation analysis, logistic regression model, and gene set enrichment analysis on simulated data with noises/confounders and on pharmacogenomic data: "RePhine demonstrated an improved performance in comparison with three commonly used methods (including Pearson correlation analysis, logistic regression model, and gene set enrichment analysis)"
- [readme] simulation_code.R generates synthetic TR/target expression, noise, and RP scores: [evidence span withheld — anchor status: external_doc]
- [discussion] RePhine measures correlation patterns by adjusting confounding factors not under TR regulation: "an accurate measurement of correlation patterns by adjusting all potential CFs that are not under the impact of TR regulation"
- [discussion] Identification robustness comes from integrative analysis on ChIP-seq targets: "an integrative analysis on ChIP-seq targets to produce TR identification robust toward noise and complicated protein modifications"
- [discussion] Causal validation requires TR knockout/activation experiments with drug response examination: "TR target analyses and TR knockout or activation experiments combined with drug response examination are also required to validate the causal relationship"
