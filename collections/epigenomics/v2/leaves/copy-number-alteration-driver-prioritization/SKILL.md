---
name: copy-number-alteration-driver-prioritization
description: Use when binary copy-number alteration events (e.g., GISTIC arm/focal
  SCNAs) are paired with matched gene expression data and the goal is to prioritize
  the cis driver gene most likely to mediate the alteration's trans effects.
when_to_use_negative:
- When the alteration's trans effects are weak or sample size is small — the Sobel
  test only achieves high AUC under adequate sample size and strong correlation conditions
- When the assumption that the number of mediated trans genes proxies driver importance
  does not hold for the use case at hand
- When only genes within the alteration are considered (GISTIC2.0-style analysis)
  and no trans expression data are available to support mediation
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: iEDGE
  role: R package providing construct_iEDGE, run_iEDGE, and calc_sobel to perform
    mediation-based cis driver ranking and pruning of cn-trans connections
  repo: https://github.com/montilab/iEDGE
- name: Biobase
  role: Package dependency (Depends) of iEDGE used for expression data containers
- name: limma
  role: Package dependency (Imports) of iEDGE used for differential expression of
    cis/trans genes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:6eb43eb0b9f3e0dd2361cb7d1bbe12ccd424ec72f222669e5747125798d991da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - Biobase
  - R
  - iEDGE
  - limma
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# copy-number-alteration-driver-prioritization

## Summary

Ranks cis genes within each SCNA by their ability to statistically mediate downstream trans gene expression effects (via Sobel-test FDR and Weighted Fraction of Trans Mediation, WFTM), flagging the top-scoring cis gene as the Rank-1 candidate driver of the alteration.

## When to use

Use when binary copy-number alteration events (e.g., GISTIC arm/focal SCNAs) are paired with matched gene expression data and the goal is to prioritize the cis driver gene most likely to mediate the alteration's trans effects.

## When NOT to use

- When the alteration's trans effects are weak or sample size is small — the Sobel test only achieves high AUC under adequate sample size and strong correlation conditions
- When the assumption that the number of mediated trans genes proxies driver importance does not hold for the use case at hand
- When only genes within the alteration are considered (GISTIC2.0-style analysis) and no trans expression data are available to support mediation

## Inputs

- iEDGE data object from construct_iEDGE: binary alteration (cn) matrix, gene expression (gep) matrix, cis gene list, sample annotations (cn.fdat, gep.fdat)
- cis/trans differential expression results (significant candidate cis mediators and mediated trans genes)
- GMT gene sets (h.all.v5.0.symbols.gmt, c2.cp.v5.0.symbols.gmt, c3.tft.v5.0.symbols.gmt) for downstream annotation

## Outputs

- Per-SCNA cis gene ranking with WFTM scores and Rank-1 candidate driver designation
- Significant cn-cis-trans mediation connections (res$pruning$sig)
- Sobel mediation statistics per SCNA-trans gene pair

## How to apply

Load the iEDGE data object (binary alteration matrix, expression matrix, cis gene list) built with `construct_iEDGE`, and run `run_iEDGE` so the cis/trans differential expression step yields candidate mediators and mediated trans genes. For each SCNA-trans gene pair, apply `calc_sobel(x, y, z, y.names, z.names)` over all candidate cis genes to obtain Sobel mediation statistics, and select the single most significant cis mediator per pair by Sobel p-value. Aggregate per-pair results into a WFTM score per cis gene — the total fractional weighted mediation effect across the m significantly expressed trans genes, each weighted by mediation significance. Rank cis genes within each alteration by WFTM and flag the highest-scoring gene as the Rank-1 candidate driver; validate by confirming recovered cn-cis-trans connections match the significant connections in `res$pruning$sig` and reproduce known simulated mediations.

## Related tools

- **iEDGE** (R package providing construct_iEDGE, run_iEDGE, and calc_sobel to perform mediation-based cis driver ranking and pruning of cn-trans connections) — https://github.com/montilab/iEDGE
- **Biobase** (Package dependency (Depends) of iEDGE used for expression data containers)
- **limma** (Package dependency (Imports) of iEDGE used for differential expression of cis/trans genes)

## Examples

```
res <- run_iEDGE(dat = dat, header = "testrun", outdir = ".")
```

## Evaluation signals

- Recovered significant cn-cis-trans connections reproduce the known simulated mediations injected into simulated expression data
- Significant mediation connections match the significant connections reported by the pruning step (res$pruning$sig)
- Exactly one cis gene per SCNA is flagged as Rank-1 (the maximum-WFTM gene), and Sobel FDR ranking selects a single mediator per SCNA-trans pair
- iEDGE-identified cis drivers are significantly enriched for known driver genes from validated oncogene/tumor suppressor compendia
- Sobel test performance validated by simulation under adequate sample size and strong correlation assumptions

## Limitations

- Sobel test validity depends on adequate sample size and strong correlation effects, conditions not met in all datasets
- The assumption that the number of mediated trans genes proxies driver importance may not hold for specific use cases
- Further experimental validation of predicted drivers is still required

## Evidence

- [discussion] the mediation analysis, ranks the set of cis genes by the extent of their mediation of trans genes: "the mediation analysis, ranks the set of cis genes by the extent of their mediation of trans genes"
- [abstract] It then applies tests of statistical mediation to determine the cis genes predictive of the trans gene expression: "It then applies tests of statistical mediation to determine the cis genes predictive of the trans gene expression"
- [methods] calc_sobel(x, y, z, y.names, z.names): "calc_sobel(x, y, z, y.names, z.names)"
- [discussion] Sobel test of mediation achieved high AUC for uncovering true mediation effects: "the Sobel test of mediation achieved high AUC for uncovering true mediation effects"
- [readme] package provides the generalized pipeline including pruning of cis/trans connections using statistical mediation: "pruning of cis/trans epi-DNA to gene expression connections using a statistical mediation model"
