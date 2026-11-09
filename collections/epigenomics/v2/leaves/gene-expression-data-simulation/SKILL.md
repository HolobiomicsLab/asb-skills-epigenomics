---
name: gene-expression-data-simulation
description: 'Use when you need to benchmark a statistical mediation test or prioritize
  candidate driver genes: simulate gene expression with known mediated vs. independent
  cis-trans pairs before running the test.'
when_to_use_negative:
- When the goal is to detect mediators in real multi-omics data (e.g. TCGA SCNA/gene
  expression) — use the iEDGE run_iEDGE pipeline on construct_iEDGE objects instead
  of simulation
- When evaluating methods other than mediation testing that require different ground-truth
  structures (e.g. pathway enrichment benchmarking via run_hyperEnrichment)
- When true mediation effect sizes are known to be weak and sample size is small —
  the Sobel test's validity depends on adequate sample size and strong correlation
  effects
tools:
- name: iEDGE
  role: R package whose calc_sobel(x, y, z, y.names, z.names) function is used to
    test mediation on each simulated cis-trans pair; installed via install_github('montilab/iEDGE')
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Language in which the simulation loop over the (correlationXY, N) grid and
    the metric computation are executed (rmarkdown notebook workflow)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:3b36566fc91bb981281e468f020d7c00cc5fb3be149c9d970926799bede97192
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
yes_rate: 0.6363636363636364
n_claims: 11
---

# gene-expression-data-simulation

## Summary

Simulates paired cis-trans gene expression data with known ground-truth mediation (true-positive and true-negative pairs) so that mediation tests such as the Sobel test implemented in iEDGE's calc_sobel can be benchmarked across a grid of sample size N and correlation between X and Y, yielding AUC/sensitivity/specificity performance estimates.

## When to use

Use when you need to benchmark a statistical mediation test or prioritize candidate driver genes: simulate gene expression with known mediated vs. independent cis-trans pairs before running the test.

## When NOT to use

- When the goal is to detect mediators in real multi-omics data (e.g. TCGA SCNA/gene expression) — use the iEDGE run_iEDGE pipeline on construct_iEDGE objects instead of simulation
- When evaluating methods other than mediation testing that require different ground-truth structures (e.g. pathway enrichment benchmarking via run_hyperEnrichment)
- When true mediation effect sizes are known to be weak and sample size is small — the Sobel test's validity depends on adequate sample size and strong correlation effects

## Inputs

- Parameter grid over correlationXY (converted from Gaussian noise level sigma) and sample size N
- Effect size settings for simulated mediation (beta1=beta2=0.7)
- Number of true-positive and true-negative pairs per cell (1000 each)
- Random seed

## Outputs

- Simulated gene expression matrices for cis and trans genes with known mediation labels
- Sobel test p-values per simulated cis-trans pair
- sobelf_performance grid of AUC, sensitivity, specificity per (correlationXY, N) cell (sobel_performance_grid.csv)

## How to apply

Set a random seed and, for each cell of a parameter grid over correlationXY (Pearson correlation between independent variable and mediator, derived from the noise level sigma) and sample size N, simulate 1000 true-positive cis-trans gene pairs under total mediation with effect sizes beta1=beta2=0.7 plus Gaussian noise, and 1000 true-negative independent gene pairs. Run the Sobel test of mediation on each simulated pair using calc_sobel(x, y, z, y.names, z.names) from the iEDGE R package, then classify pairs as mediated/non-mediated from the Sobel p-values. Compute per-cell performance metrics (AUC, sensitivity, specificity) and assemble them into a grid over correlationXY and N, written out as sobel_performance_grid.csv. The rationale is that the iEDGE workflow ranks cis genes by how much they mediate trans gene expression, so simulation is the article's stated method to validate that the Sobel test reliably uncovers true mediation effects before applying it to TCGA data.

## Related tools

- **iEDGE** (R package whose calc_sobel(x, y, z, y.names, z.names) function is used to test mediation on each simulated cis-trans pair; installed via install_github('montilab/iEDGE')) — https://github.com/montilab/iEDGE
- **R** (Language in which the simulation loop over the (correlationXY, N) grid and the metric computation are executed (rmarkdown notebook workflow))

## Examples

```
library(iEDGE); sobel_p <- calc_sobel(x, y, z, y.names, z.names)  # run per simulated (correlationXY, N) grid cell on 1000 true-positive + 1000 true-negative pairs
```

## Evaluation signals

- Sobel test AUC increases monotonically with sample size N and with correlationXY, confirming high AUC under adequate sample size and strong correlation
- Specificity remains high across all tested (correlationXY, N) parameter ranges
- Sensitivity drops only under low-correlation and low-sample-size conditions, matching the reported finding
- The output grid contains one performance value per (correlationXY, N) cell and serializes to sobel_performance_grid.csv with the paper's performance-evaluation layout
- Each grid cell is based on exactly 1000 true-positive and 1000 true-negative simulated pairs

## Limitations

- The Sobel test only achieves high AUC under adequate sample size and strong correlation effects; conditions 'which were met in most of the TCGA datasets used in this study' but not all
- Simulation assumes total mediation with fixed effect sizes (beta1=beta2=0.7) and Gaussian noise, which may not reflect partial mediation in real cancer genomics data
- The assumption that the number of mediated trans genes proxies driver importance 'may not be an appropriate assumption for specific use cases'

## Evidence

- [methods] Simulate mediation pairs and benchmark Sobel test: "we will simulate the input gene expression data, the cis and trans genes with known mediations"
- [discussion] Simulations validate Sobel test performance: "simulations were performed to validate the performance of the Sobel test of mediation"
- [discussion] Sobel achieves high AUC under adequate sample size and strong correlation: "the Sobel test of mediation achieved high AUC for uncovering true mediation effects"
- [readme] Install iEDGE for calc_sobel: "install_github("montilab/iEDGE")"
- [readme] Mediation module prunes cis/trans connections: "pruning of cis/trans epi-DNA to gene expression connections using a statistical mediation model"
