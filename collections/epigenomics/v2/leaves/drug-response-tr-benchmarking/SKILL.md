---
name: drug-response-tr-benchmarking
description: Use when comparing methods for identifying drug response-related TRs
  on synthetic pharmacogenomic data (gene expression + drug response + ChIP-seq targets),
  especially under injected noise or spurious-correlation confounders.
when_to_use_negative:
- No reliable target information (ChIP-seq data missing or with too few targets) exists
  — RePhine identification would be misled
- The goal is detecting acquired/secondary resistance from post-treatment data — CCLE
  lacks post-treatment data so secondary resistance (e.g., acquired STAT3 activation
  for erlotinib) cannot be detected
- The task is establishing causal TR-drug relationships rather than association-based
  benchmarking — TR knockout/activation experiments with drug response examination
  are required for causal validation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0204
tools:
- name: RePhine
  role: 'Method under benchmarking: regression-based pharmacogenomic and ChIP-seq
    data integration method identifying TRs associated with drug response via target
    correlation patterns'
  repo: https://github.com/coexps/RePhine
- name: R
  role: Runtime environment and language for the RePhine package, simulation_code.R,
    and baseline model implementations (packages glmnet, MASS, lmtest, ppcor, psych,
    ROCR)
  repo: https://github.com/coexps/RePhine
- name: Python
  role: Runs RP_calculation_modified.py to compute modified RP scores (distance +
    peak strength) from ChIP-seq peak BED files used in simulations
  repo: https://github.com/coexps/RePhine
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:deaa349ebd987a8918528b90fe1653f6272266927aa0a58e4e3431ee76e44755
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
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# drug-response-tr-benchmarking

## Summary

Benchmark RePhine, a regression-based pharmacogenomic and ChIP-seq data integration method, against Pearson correlation analysis, logistic regression, and GSEA on simulated pharmacogenomic data with added noises or confounders to quantify its superiority in identifying drug response-related transcriptional regulators (TRs).

## When to use

Use when comparing methods for identifying drug response-related TRs on synthetic pharmacogenomic data (gene expression + drug response + ChIP-seq targets), especially under injected noise or spurious-correlation confounders.

## When NOT to use

- No reliable target information (ChIP-seq data missing or with too few targets) exists — RePhine identification would be misled
- The goal is detecting acquired/secondary resistance from post-treatment data — CCLE lacks post-treatment data so secondary resistance (e.g., acquired STAT3 activation for erlotinib) cannot be detected
- The task is establishing causal TR-drug relationships rather than association-based benchmarking — TR knockout/activation experiments with drug response examination are required for causal validation

## Inputs

- Simulated pharmacogenomic data: synthetic TR and target gene expression, drug response (pharmacological profiles), noise data
- Simulated/ChIP-seq-derived regulatory potential (RP) scores computed from peak BED files via RP_calculation_modified.py (distance and peak strength)
- Simulation scenario parameters from File S1 (noise and confounder settings)
- RePhine R package (Rephine_1.1.0.tar.gz / .zip) and simulation_code.R

## Outputs

- Per-scenario, per-method performance values comparing RePhine, Pearson correlation, logistic regression, and GSEA
- Comparison table (e.g., simulation_benchmark.csv) of identification performance (true TR-drug detection vs false positives/negatives) under noise and confounder scenarios

## How to apply

Obtain the RePhine R package and simulation_code.R from github.com/coexps/RePhine along with the File S1 scenario parameters, then run the simulation scripts to generate synthetic TR/target expression data, noise data, and RP scores under two scenarios: added-noise and added-confounders (where spurious correlations not mediated by the TR are introduced). On each simulated dataset, apply RePhine — which computes copy-number-adjusted expression, partial correlations controlling confounding factors, a likelihood-ratio-test-based uniP significance for concordant target correlations, elastic net multiP for independent TRs, and a modified GSEA filter — and apply the three baselines (Pearson correlation, logistic regression, GSEA) with identical inputs. Score all four methods on detection of the true simulated TR-drug relationship versus false positives/negatives, assemble per-scenario, per-method performance into a comparison table (simulation_benchmark.csv), and validate that RePhine achieves higher identification performance than all three baselines under both scenarios.

## Related tools

- **RePhine** (Method under benchmarking: regression-based pharmacogenomic and ChIP-seq data integration method identifying TRs associated with drug response via target correlation patterns) — https://github.com/coexps/RePhine
- **R** (Runtime environment and language for the RePhine package, simulation_code.R, and baseline model implementations (packages glmnet, MASS, lmtest, ppcor, psych, ROCR)) — https://github.com/coexps/RePhine
- **Python** (Runs RP_calculation_modified.py to compute modified RP scores (distance + peak strength) from ChIP-seq peak BED files used in simulations) — https://github.com/coexps/RePhine

## Examples

```
Rscript simulation_code.R
```

## Evaluation signals

- The benchmark table must show RePhine achieving higher identification performance than Pearson correlation analysis, logistic regression model, and GSEA under BOTH the noise and confounder scenarios
- Simulated datasets must include both added-noise and added-confounder conditions, with confounder data introducing spurious correlations not mediated by the transcriptional regulator
- Method execution must use identical inputs for all four methods (pharmacogenomic data + ChIP-seq target/RP scores)
- Partial correlations in RePhine runs must control the confounding factors; copy-number effects must be removed from expression by linear regression before analysis
- Output must be a complete per-scenario, per-method performance matrix with true-positive and false-positive/negative rates for the simulated TR-drug relationship

## Limitations

- RePhine is restricted to cases where reliable target information is available; lack of ChIP-seq data or ChIP-seq data with too few targets would mislead TR identification
- Simulated/benchmark performance does not establish causality — some upstream regulators or kinases may influence drug response and simultaneously regulate downstream TRs; TR knockout/activation experiments are required to validate causal relationships
- No changelog found for the released code/artifacts, so version drift in simulation behavior may be hard to track

## Evidence

- [abstract] Superiority over baselines under noise/confounders: "RePhine demonstrated an improved performance in comparison with three commonly used methods (including Pearson correlation analysis, logistic regression model, and gene set enrichment analysis)"
- [readme] Simulation code availability and purpose: [evidence span withheld — anchor status: external_doc]
- [discussion] Confounding-factor adjustment rationale: "an accurate measurement of correlation patterns by adjusting all potential CFs that are not under the impact of TR regulation"
- [discussion] Target inference uses ChIP-seq signal and distance: "application of quantitative and informative target inference by considering both ChIP-seq signals and the distances from peaks to the targets"
- [discussion] Causality caveat: "TR target analyses and TR knockout or activation experiments combined with drug response examination are also required to validate the causal relationship"
