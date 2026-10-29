---
name: sobel-mediation-testing
description: Use when you have paired copy-number/epi-DNA alteration states and gene
  expression data and need to test whether a cis gene mediates alteration-to-trans
  expression effects, or to rank candidate cis drivers.
when_to_use_negative:
- Sample size is small or the correlation between the alteration and the mediator/expression
  is weak — simulations showed sensitivity drops under low correlation and low N
- The goal is differential expression alone (cis/trans signatures) without needing
  to rank candidate mediators — run_iEDGE/iEDGE_DE suffices
- Paired alteration and expression data are not available from the same samples, since
  mediation testing requires the triplet alteration–cis–trans per sample
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: iEDGE
  role: R package providing calc_sobel for Sobel mediation testing and the pruning
    step that ranks cis genes by WFTM and flags the Rank-1 driver
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Runtime environment in which iEDGE and calc_sobel are executed (library(iEDGE))
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:a5492ae1222a647fe5fba98b7d0963cefa61ddde4ef14862a91b500ec5d17d59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - Biobase
  - R
  - calc_sobel
  - iEDGE
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# sobel-mediation-testing

## Summary

Statistical mediation testing via the Sobel test, as implemented in the iEDGE R package's calc_sobel function, to determine whether cis genes (mediators) explain the trans gene expression effects of epi-DNA alterations, and to rank cis candidate cancer drivers by extent of trans mediation (WFTM/Rank-1).

## When to use

Use when you have paired copy-number/epi-DNA alteration states and gene expression data and need to test whether a cis gene mediates alteration-to-trans expression effects, or to rank candidate cis drivers.

## When NOT to use

- Sample size is small or the correlation between the alteration and the mediator/expression is weak — simulations showed sensitivity drops under low correlation and low N
- The goal is differential expression alone (cis/trans signatures) without needing to rank candidate mediators — run_iEDGE/iEDGE_DE suffices
- Paired alteration and expression data are not available from the same samples, since mediation testing requires the triplet alteration–cis–trans per sample

## Inputs

- Binary epi-DNA alteration (e.g. GISTIC2.0 SCNA amplification/deletion) status matrix per sample
- Gene expression matrix (e.g. TCGA RNASeq) for the same paired samples
- Candidate cis gene list and significant trans gene list from iEDGE cis/trans differential expression analysis
- Simulated mediation datasets (true-positive cis-trans pairs with beta1=beta2=0.7 plus Gaussian noise) for performance evaluation

## Outputs

- Sobel mediation statistics and p-values per SCNA-cis-trans gene pair (calc_sobel output)
- Pruned significant mediation connections (res$pruning$sig)
- WFTM score per cis gene and Rank-1 candidate driver designation per alteration
- Performance metrics (AUC, sensitivity, specificity) grids over correlationXY and sample size N when applied to simulated data

## How to apply

Run calc_sobel(x, y, z, y.names, z.names) from the iEDGE package for every SCNA-trans gene pair, passing candidate cis genes as mediators to obtain Sobel test statistics and p-values (optionally bipartite=TRUE). Select the single most significant cis mediator per SCNA-trans pair based on Sobel-test p-value/FDR, then aggregate weighted fractional mediation effects into a per-cis-gene WFTM score; flag the top-scoring cis gene as the Rank-1 candidate driver. In practice the mediation step follows cis/trans differential expression analysis (e.g. run_iEDGE with fdr.cis.cutoff=0.25, fdr.trans.cutoff=0.01, fc.cis=1.2, fc.trans=1.5), and Rank-1 pruning is applied with prune.col='fdr' and prune.thres=0.05. Because simulation studies showed the Sobel test only achieves high AUC/sensitivity under adequate sample size and strong correlation between alteration and expression, verify these assumptions before trusting mediation calls.

## Related tools

- **iEDGE** (R package providing calc_sobel for Sobel mediation testing and the pruning step that ranks cis genes by WFTM and flags the Rank-1 driver) — https://github.com/montilab/iEDGE
- **R** (Runtime environment in which iEDGE and calc_sobel are executed (library(iEDGE)))

## Examples

```
library(iEDGE); sob <- calc_sobel(x, y, z, y.names, z.names); res <- run_iEDGE(dat = dat, header = "testrun", outdir = ".")
```

## Evaluation signals

- On simulated data (1000 true-positive mediated pairs, beta1=beta2=0.7), Sobel AUC should increase with sample size N and with correlationXY between X and Y
- Specificity should remain high across all tested (correlationXY, N) grid cells; sensitivity dropping only at low correlation/low N
- Recovered significant cn-cis-trans connections should reproduce the known simulated mediations and match res$pruning$sig from the iEDGE pruning step
- Within each SCNA, exactly one cis gene is flagged as Rank-1, and it should be the cis gene with the highest WFTM / most significant Sobel-test FDR
- Pan-cancer Rank-1 driver lists should show significant enrichment for known cancer drivers (FDR < 0.05) in the majority of TCGA types (15/19 in the paper, KS test p=2.5e-13)

## Limitations

- The Sobel test achieved high AUC only under adequate sample size and strong correlation assumptions, conditions which were met in most but not all TCGA datasets used in the study
- Sensitivity drops under conditions of low correlation and low sample size, while specificity remains high
- The assumption that the number of mediated trans genes (WFTM) proxies driver importance may not hold for specific use cases

## Evidence

- [methods] Sobel test call signature: "calc_sobel(x, y, z, y.names, z.names)"
- [discussion] Simulation validation of Sobel performance: "the Sobel test of mediation achieved high AUC for uncovering true mediation effects"
- [discussion] Simulation caveat: "conditions which were met in most of the TCGA datasets used in this study"
- [abstract] Mediation step in pipeline: "It then applies tests of statistical mediation to determine the cis genes predictive of the trans gene expression"
- [readme] Package availability: "iEDGE is an R-package for performing integrative analysis of (epi-)DNA and gene expression data"
