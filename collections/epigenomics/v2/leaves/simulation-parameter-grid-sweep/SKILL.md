---
name: simulation-parameter-grid-sweep
description: Use when validating a statistical test (e.g., Sobel mediation) whose
  performance depends on sample size and effect strength, by simulating positive/negative
  controls over a parameter grid and computing per-cell performance metrics.
when_to_use_negative:
- The input is real experimental data with no known mediation ground truth — simulation-based
  validation requires known positive/negative pairs
- The statistical test's performance does not plausibly depend on sample size or effect
  correlation, so a parameter sweep adds no information
- The goal is driver discovery on real multi-omics data (use the full iEDGE run_iEDGE
  pipeline instead of a simulation grid)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: iEDGE
  role: R package providing calc_sobel, the Sobel test of mediation applied to each
    simulated pair in every grid cell
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Implementation environment for simulating gene pairs, running the grid sweep,
    and computing AUC/sensitivity/specificity per cell
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:35bc3f8bdf8fed46a3dae84144b15b3e68a900db6326da24cd9a2502e41aabda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - R
  - iEDGE
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# simulation-parameter-grid-sweep

## Summary

Simulates synthetic mediation ground truth (true-positive and true-negative pairs) across a two-dimensional grid of effect correlation and sample size, then evaluates the Sobel test of mediation (via iEDGE's calc_sobel) per grid cell to characterize sensitivity, specificity, and AUC as a function of those parameters.

## When to use

Use when validating a statistical test (e.g., Sobel mediation) whose performance depends on sample size and effect strength, by simulating positive/negative controls over a parameter grid and computing per-cell performance metrics.

## When NOT to use

- The input is real experimental data with no known mediation ground truth — simulation-based validation requires known positive/negative pairs
- The statistical test's performance does not plausibly depend on sample size or effect correlation, so a parameter sweep adds no information
- The goal is driver discovery on real multi-omics data (use the full iEDGE run_iEDGE pipeline instead of a simulation grid)

## Inputs

- Simulated true-positive cis-trans gene pairs under total mediation (beta1=beta2=0.7 + Gaussian noise)
- Simulated true-negative independent gene pairs
- Parameter grid definitions: Pearson correlation between X and Y (converted from noise sigma) and sample sizes N
- Random seed

## Outputs

- Per-cell performance metrics (AUC, sensitivity, specificity) for each (correlationXY, N) grid cell
- Grid table of Sobel test performance over correlationXY and sample size, e.g. sobel_performance_grid.csv

## How to apply

Set a random seed and simulate 1000 true-positive cis-trans gene pairs under total mediation with effect sizes beta1=beta2=0.7 plus Gaussian noise, plus 1000 true-negative independent gene pairs, for each grid setting. Convert each noise level sigma to the corresponding Pearson correlation between X and Y to define the correlationXY axis, and sweep sample sizes N along the other axis. For every (correlationXY, N) cell, run the Sobel test on each simulated pair with the calc_sobel(x, y, z, y.names, z.names) function from the iEDGE R package, classify pairs as mediated/non-mediated from Sobel p-values, and compute AUC, sensitivity, and specificity per cell. Assemble the per-cell values into a grid over correlationXY and N (e.g., sobel_performance_grid.csv), mirroring the paper's performance evaluation layout. Rationale: the article used such simulations to validate the Sobel test before applying mediation-based pruning of cis/trans epi-DNA to gene expression connections.

## Related tools

- **iEDGE** (R package providing calc_sobel, the Sobel test of mediation applied to each simulated pair in every grid cell) — https://github.com/montilab/iEDGE
- **R** (Implementation environment for simulating gene pairs, running the grid sweep, and computing AUC/sensitivity/specificity per cell)

## Examples

```
sobel_p <- calc_sobel(x, y, z, y.names, z.names)
```

## Evaluation signals

- Sobel test AUC increases monotonically with sample size N within each correlationXY level
- AUC increases with correlation strength (lower noise sigma) at fixed N, consistent with the reported high AUC under adequate sample size and strong correlation
- Specificity remains high across all tested (correlationXY, N) cells, while sensitivity drops at low correlation and low N
- Output grid contains a metric value for every cell of the correlationXY x N sweep, with correct orientation matching the paper's evaluation layout (e.g., sobel_performance_grid.csv)
- True-positive pairs are simulated as total mediation (beta1=beta2=0.7 plus noise) and true-negative pairs as independent, with balanced 1000/1000 class sizes per cell

## Limitations

- Sobel test validity depends on adequate sample size and strong correlation effects, conditions not met in all datasets
- Simulation assumes total mediation with fixed effect sizes (beta1=beta2=0.7); partial mediation or other effect-size regimes are not covered by this grid
- Sensitivity drops under low correlation and low sample size, so grid cells in that region yield unreliable mediation calls

## Evidence

- [discussion] simulation-based validation of Sobel test performance: "simulations were performed to validate the performance of the Sobel test of mediation"
- [discussion] reported performance finding: "the Sobel test of mediation achieved high AUC for uncovering true mediation effects"
- [methods] concrete Sobel invocation: "calc_sobel(x, y, z, y.names, z.names)"
- [readme] mediation role in iEDGE pipeline: "pruning of cis/trans epi-DNA to gene expression connections using a statistical mediation model"
