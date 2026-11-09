---
name: roc-auc-performance-evaluation
description: Use when a statistical test or predictor (e.g., a mediation test) must
  be validated on simulated data with known true positives/negatives, and you need
  AUC, sensitivity, and specificity as a function of controlled parameters such as
  effect strength and sample size.
when_to_use_negative:
- The data are real observational data without known ground-truth mediation labels
  — simulation with known true/false pairs is required for this skill
- The question is applying mediation testing to discover drivers (use the iEDGE run_iEDGE
  workflow), not benchmarking the test's statistical performance
- Conditions of adequate sample size and strong correlation are already guaranteed,
  so a full performance grid adds little
tools:
- name: iEDGE
  role: R package providing calc_sobel to run the Sobel test of mediation on each
    simulated pair
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Language in which the simulation grid, Sobel calls, and AUC/sensitivity/specificity
    computation are implemented (rmarkdown notebook output)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:5822ab0a449e8b5b3112a0efd82c0b4641f4803f0c6e89cfb49f8597b384bf42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - R
  - iEDGE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# roc-auc-performance-evaluation

## Summary

Simulation-based evaluation of a binary classifier's discriminative power by sweeping a parameter grid (here, correlation between independent variable and mediator vs. sample size N) and computing AUC, sensitivity, and specificity per cell — used in iEDGE to validate the Sobel test of mediation against known true/false mediation labels.

## When to use

Use when a statistical test or predictor (e.g., a mediation test) must be validated on simulated data with known true positives/negatives, and you need AUC, sensitivity, and specificity as a function of controlled parameters such as effect strength and sample size.

## When NOT to use

- The data are real observational data without known ground-truth mediation labels — simulation with known true/false pairs is required for this skill
- The question is applying mediation testing to discover drivers (use the iEDGE run_iEDGE workflow), not benchmarking the test's statistical performance
- Conditions of adequate sample size and strong correlation are already guaranteed, so a full performance grid adds little

## Inputs

- Simulated true-positive instances (e.g., 1000 cis-trans gene pairs under total mediation, beta1=beta2=0.7 plus Gaussian noise)
- Simulated true-negative instances (e.g., 1000 independent gene pairs)
- Grid of parameter values: noise levels sigma mapped to Pearson correlationXY, and sample sizes N
- Simulated gene expression matrices for X (independent), Z (mediator), and Y (outcome) variables per pair

## Outputs

- Per-cell performance metrics table (e.g., sobel_performance_grid.csv) over correlationXY and N with AUC, sensitivity, specificity
- Mediated/non-mediated classifications per simulated pair from Sobel test p-values
- Validation assessment of monotonic AUC increase with sample size and correlation strength

## How to apply

Set a random seed and simulate matched true-positive and true-negative cases — in the iEDGE validation, 1000 true-positive cis-trans gene pairs under total mediation with effect sizes beta1=beta2=0.7 plus Gaussian noise, and 1000 true-negative independent pairs, for each grid setting. Convert each noise level sigma to the corresponding Pearson correlation between X and Y to define one grid axis, and sweep sample sizes N on the other axis. For every (correlationXY, N) cell, run the classifier under test (calc_sobel(x, y, z, y.names, z.names) from the iEDGE R package) on every simulated pair, then classify pairs as mediated/non-mediated from the Sobel test p-values. Compute per-cell AUC, sensitivity, and specificity, assemble them into a grid over correlationXY and N (e.g., sobel_performance_grid.csv), and confirm expected behavior: AUC increases with sample size and correlation strength, with high specificity across all cells but sensitivity dropping at low correlation and low N.

## Related tools

- **iEDGE** (R package providing calc_sobel to run the Sobel test of mediation on each simulated pair) — https://github.com/montilab/iEDGE
- **R** (Language in which the simulation grid, Sobel calls, and AUC/sensitivity/specificity computation are implemented (rmarkdown notebook output))

## Examples

```
p <- calc_sobel(x, y, z, y.names, z.names)
```

## Evaluation signals

- Per-cell AUC increases monotonically (or near-monotonically) with both sample size N and correlationXY, consistent with the paper's reported result
- Specificity is high across all tested (correlationXY, N) cells because true-negative pairs are simulated as independent
- Sensitivity drops sharply at cells combining low correlation and low sample size, matching the reported failure mode
- Each grid cell is computed from 1000 true-positive and 1000 true-negative simulated pairs with a fixed random seed, enabling reproducibility
- Output grid has one performance row per (correlationXY, N) combination with no missing cells

## Limitations

- The Sobel test achieves high AUC only under adequate sample size and strong correlation assumptions — conditions not met in all datasets
- The simulation assumes total mediation with fixed effect sizes (beta1=beta2=0.7); partial mediation or other effect-size regimes are not covered
- Simulation-based validation may not fully reflect performance on real multi-omics data such as TCGA

## Evidence

- [discussion] Simulation-based validation of Sobel test of mediation performance: "simulations were performed to validate the performance of the Sobel test of mediation"
- [discussion] High AUC under adequate sample size and strong correlation: "the Sobel test of mediation achieved high AUC for uncovering true mediation effects"
- [discussion] Conditions not met in all datasets: "conditions which were met in most of the TCGA datasets used in this study"
- [methods] calc_sobel invocation: "calc_sobel(x, y, z, y.names, z.names)"
- [discussion] iEDGE is an open-source R package: "iEDGE is available as an open-source R package (https://github.com/montilab/iEDGE)"
