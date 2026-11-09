---
name: weighted-trans-mediation-fraction-computation
description: Use when a copy-number (SCNA) or epi-DNA alteration has both a cis gene
  candidate set and a trans gene expression signature, and you must rank cis genes
  by how much of the trans effect they statistically mediate to flag the Rank-1 driver.
when_to_use_negative:
- No trans gene expression signature is associated with the alteration (mediation
  step requires mediated trans genes to compute WFTM)
- Sample size or correlation effects are insufficient for valid Sobel-test mediation
  inference — the article notes the Sobel test achieved high AUC only under adequate
  sample size and strong correlation assumptions
- The goal is driver detection considering only genes within the alteration boundaries
  — the article contrasts iEDGE with tools like GISTIC2.0 and expects trans mediation
  signal beyond the alteration
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: iEDGE
  role: R package providing the mediation/pruning module, calc_sobel, and WFTM-based
    Rank-1 cis driver ranking
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Execution environment for the iEDGE package and mediation analysis scripts
- name: Biobase
  role: Dependency of iEDGE for handling expression data objects
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:b988da2ffa639f89cac82dfd071bed175455ef061b9dd2b74bf6f355b025a562
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - Biobase
  - R
  - iEDGE
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# weighted-trans-mediation-fraction-computation

## Summary

Computation of the Weighted Fraction of Trans Mediation (WFTM) score in iEDGE: for each SCNA, Sobel mediation statistics rank cis gene mediators of trans gene expression, and each cis gene's fractional weighted mediation effect across significantly mediated trans genes identifies the Rank-1 candidate driver.

## When to use

Use when a copy-number (SCNA) or epi-DNA alteration has both a cis gene candidate set and a trans gene expression signature, and you must rank cis genes by how much of the trans effect they statistically mediate to flag the Rank-1 driver.

## When NOT to use

- No trans gene expression signature is associated with the alteration (mediation step requires mediated trans genes to compute WFTM)
- Sample size or correlation effects are insufficient for valid Sobel-test mediation inference — the article notes the Sobel test achieved high AUC only under adequate sample size and strong correlation assumptions
- The goal is driver detection considering only genes within the alteration boundaries — the article contrasts iEDGE with tools like GISTIC2.0 and expects trans mediation signal beyond the alteration

## Inputs

- iEDGE data object from construct_iEDGE(cn, gep, cisgenes, transgenes, cn.fdat, gep.fdat): binary alteration matrix, gene expression matrix, cis gene list
- Cis/trans differential expression results (significant cis candidate mediators and trans genes) from iEDGE_DE

## Outputs

- Sobel mediation statistics per SCNA-cis-trans pair
- WFTM score per cis gene within each SCNA
- Rank-1 candidate driver gene designation per alteration
- Pruned significant cn-cis-trans mediation connections (res$pruning$sig)

## How to apply

Load the iEDGE data object (binary alteration matrix, expression matrix, cis gene list) built via construct_iEDGE, together with the cis/trans differential expression results (iEDGE_DE) as candidate mediators and mediated genes. For every SCNA-trans gene pair, call calc_sobel(x, y, z, y.names, z.names) over the candidate cis genes to obtain Sobel mediation statistics, and select the single most significant cis mediator per pair by Sobel-test p-value/FDR. Aggregate the per-pair results into a WFTM score per cis gene — the total fractional weighted mediation effect across the m significantly expressed trans genes, weighting each mediated trans gene by its mediation significance. Rank cis genes per alteration by extent of mediation and flag the top-scoring gene as the Rank-1 candidate driver; this typically runs as part of the run_iEDGE pruning module, whose significant connections should be reproduced (res$pruning$sig).

## Related tools

- **iEDGE** (R package providing the mediation/pruning module, calc_sobel, and WFTM-based Rank-1 cis driver ranking) — https://github.com/montilab/iEDGE
- **R** (Execution environment for the iEDGE package and mediation analysis scripts)
- **Biobase** (Dependency of iEDGE for handling expression data objects)

## Examples

```
res <- run_iEDGE(dat = dat, header = "testrun", outdir = "."); calc_sobel(x, y, z, y.names, z.names)
```

## Evaluation signals

- Recovered significant cn-cis-trans mediation connections reproduce the known simulated mediations used to simulate the input data
- Significant connections match those reported by the pruning step (res$pruning$sig) in the run_iEDGE results object
- The cis gene with the highest WFTM score per SCNA is the one flagged as Rank-1, and the single most significant Sobel mediator per SCNA-trans pair matches the selected mediator
- WFTM values are fractional (bounded by the total weighted mediation effect across the m significant trans genes) and are computable for each cis gene within the alteration
- Simulation shows the Sobel test of mediation achieves high AUC under adequate sample size and strong correlation, so inferred mediations should be reproducible on simulated ground truth

## Limitations

- The Sobel test's validity depends on adequate sample size and strong correlation effects, conditions not met in all datasets
- The assumption that the number of mediated trans genes proxies driver importance may not hold for specific use cases
- Further experimental studies are needed to validate predicted drivers

## Evidence

- [methods] calc_sobel in iEDGE methods: "calc_sobel(x, y, z, y.names, z.names)"
- [discussion] mediation ranking rationale: "the mediation analysis, ranks the set of cis genes by the extent of their mediation of trans genes"
- [intro] trans mediation hypothesis: "SCNA-related drivers of tumorigenesis will mediate a larger proportion of the downstream effect observable by trans gene expression than non-drivers"
- [readme] package structure: "pruning of cis/trans epi-DNA to gene expression connections using a statistical mediation model"
- [discussion] simulation validation: "simulations were performed to validate the performance of the Sobel test of mediation"
