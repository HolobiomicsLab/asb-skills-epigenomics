---
name: cis-trans-mediation-ranking
description: Use when a cis/trans differential-expression signature table exists for
  SCNA events and candidate cis genes must be ranked by their trans mediation to designate
  the Rank-1 driver per alteration.
when_to_use_negative:
- Sample size is small or cis-trans correlation effects are weak — the Sobel test
  achieved high AUC only under adequate sample size and strong correlation assumptions,
  conditions not met in all datasets
- The goal is identifying altered regions or genes within the alteration only (a GISTIC2.0-style
  analysis) with no need to rank cis drivers by downstream trans mediation
- The assumption that the number of mediated trans genes proxies driver importance
  is not appropriate for the specific use case
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3053
tools:
- name: iEDGE
  role: R package whose pruning/mediation module provides calc_sobel for per-pair
    Sobel testing, WFTM aggregation, and Rank-1 cis driver flagging within run_iEDGE
  repo: https://github.com/montilab/iEDGE
- name: R
  role: runtime environment in which the iEDGE mediation step is executed (library(iEDGE))
- name: Biobase
  role: declared dependency backing the expression/phenotype data containers from
    which the iEDGE object used by the mediation step is built
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:3c21c9a279b0e8df9a311ccf967ed51332adf0ff962d3aa89f8827e6fab289cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - Biobase
  - R
  - iEDGE
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cis-trans-mediation-ranking

## Summary

The mediation/pruning module of iEDGE: for each (epi-)DNA alteration (SCNA), it applies Sobel tests of statistical mediation to determine which candidate cis genes predict trans gene expression, then aggregates per-pair results into a Weighted Fraction of Trans Mediation (WFTM) score to rank cis genes and flag the Rank-1 candidate driver.

## When to use

Use when a cis/trans differential-expression signature table exists for SCNA events and candidate cis genes must be ranked by their trans mediation to designate the Rank-1 driver per alteration.

## When NOT to use

- Sample size is small or cis-trans correlation effects are weak — the Sobel test achieved high AUC only under adequate sample size and strong correlation assumptions, conditions not met in all datasets
- The goal is identifying altered regions or genes within the alteration only (a GISTIC2.0-style analysis) with no need to rank cis drivers by downstream trans mediation
- The assumption that the number of mediated trans genes proxies driver importance is not appropriate for the specific use case

## Inputs

- iEDGE data object: binary SCNA alteration matrix (cn), paired gene expression matrix (gep), and cis gene list (from construct_iEDGE)
- cis/trans differential expression results (iEDGE_DE output) providing candidate mediators and mediated trans genes
- List of candidate cis genes within each SCNA
- Set of significantly expressed trans genes associated with each alteration

## Outputs

- Sobel mediation statistics (test statistics, p-values/FDR) for each SCNA-trans-cis gene triplet
- Per SCNA-trans pair, the single most significant cis mediator selected by Sobel-test p-value
- WFTM (Weighted Fraction of Trans Mediation) score per cis gene per SCNA
- Ranked cis gene list per alteration with the Rank-1 candidate driver flagged, plus significant connections reported by the pruning step (res$pruning$sig)

## How to apply

Load the iEDGE data object (binary SCNA alteration matrix cn, gene expression matrix gep, cis gene list, built via construct_iEDGE) together with the cis/trans differential-expression results that define candidate mediators and mediated genes. For every SCNA-trans gene pair, run calc_sobel(x, y, z, y.names, z.names) over the candidate cis genes to obtain Sobel mediation statistics, and keep the single most significant cis mediator per pair by Sobel-test p-value/FDR. Aggregate the per-pair results into each cis gene's WFTM — the total fractional weighted mediation effect across the m significantly expressed trans genes — weighting each mediated trans gene by its mediation significance. Rank the cis genes within each alteration by WFTM and flag the highest scorer as the Rank-1 candidate driver, per the hypothesis that true SCNA-related drivers mediate a larger proportion of downstream trans effects than non-drivers. Validate by checking that the recovered significant cn-cis-trans connections reproduce the known simulated mediations and match the significant connections reported by the pruning step (res$pruning$sig).

## Related tools

- **iEDGE** (R package whose pruning/mediation module provides calc_sobel for per-pair Sobel testing, WFTM aggregation, and Rank-1 cis driver flagging within run_iEDGE) — https://github.com/montilab/iEDGE
- **R** (runtime environment in which the iEDGE mediation step is executed (library(iEDGE)))
- **Biobase** (declared dependency backing the expression/phenotype data containers from which the iEDGE object used by the mediation step is built)

## Examples

```
calc_sobel(x, y, z, y.names, z.names)
```

## Evaluation signals

- Recovered significant cn-cis-trans connections reproduce the known simulated mediations
- Significant connections match those reported by the pruning step (res$pruning$sig)
- Exactly one most-significant cis mediator is selected per SCNA-trans pair by Sobel-test p-value
- Rank-1 cis drivers are enriched for known driver genes from multiple cancer driver databases
- In simulation benchmarks, the Sobel test of mediation achieves high AUC for uncovering true mediation effects under adequate sample size and strong correlation

## Limitations

- Sobel test validity depends on adequate sample size and strong correlation effects; these conditions were met in most, but not all, of the TCGA datasets used in the study
- The assumption that the number of mediated trans genes proxies driver importance 'may not be an appropriate assumption for specific use cases'
- Further experimental studies are needed to validate and characterize the predicted drivers

## Evidence

- [abstract] It then applies tests of statistical mediation to determine the cis genes predictive of the trans gene expression: "It then applies tests of statistical mediation to determine the cis genes predictive of the trans gene expression"
- [intro] SCNA-related drivers of tumorigenesis will mediate a larger proportion of the downstream effect observable by trans gene expression than non-drivers.: "SCNA-related drivers of tumorigenesis will mediate a larger proportion of the downstream effect observable by trans gene expression than non-drivers."
- [methods] calc_sobel(x, y, z, y.names, z.names): "calc_sobel(x, y, z, y.names, z.names)"
- [discussion] the mediation analysis, ranks the set of cis genes by the extent of their mediation of trans genes: "the mediation analysis, ranks the set of cis genes by the extent of their mediation of trans genes"
- [discussion] simulations were performed to validate the performance of the Sobel test of mediation: "simulations were performed to validate the performance of the Sobel test of mediation"
- [readme] 3. pruning of cis/trans epi-DNA to gene expression connections using a statistical mediation model: "3. pruning of cis/trans epi-DNA to gene expression connections using a statistical mediation model"
