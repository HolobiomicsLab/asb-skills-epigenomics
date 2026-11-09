---
name: rank1-driver-pruning-fdr-thresholding
description: Use when candidate cis genes from SCNA/gene-expression integrative analysis
  must be prioritized by their trans mediation signal, and the pruned Rank-1 lists
  need in silico validation against known driver databases.
when_to_use_negative:
- Transgenes are not defined or downstream trans effects cannot be measured — mediation
  ranking of cis genes requires trans gene-expression signatures
- Sample size is small or copy-number/gene-expression correlations are weak — the
  Sobel test only achieved high AUC under adequate sample size and strong correlation
  effects
- The goal is cataloguing all cis genes within an alteration rather than prioritizing
  drivers (tools like GISTIC2.0 that only consider genes within the alterations)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: iEDGE
  role: Provides run_iEDGE, calc_sobel, and prune thresholds (prune.col='fdr', prune.thres=0.05)
    for mediation-based Rank-1 driver pruning; run_hyperEnrichment validates pruned
    lists
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Runtime environment for executing the iEDGE package and analysis scripts
  repo: https://github.com/montilab/iEDGE
- name: limma
  role: Imported dependency supporting the differential-expression statistics whose
    cis/trans signatures feed the mediation test
- name: Biobase
  role: Dependency for the ExpressionSet-based iEDGE data containers (gep/cn objects)
    consumed by run_iEDGE and calc_sobel
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:c21ac156c623ccb45e01387a2da6f3c3a6758a169aa28eac4830816e7b685aa4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - Biobase
  - R
  - iEDGE
  - limma
  - rmarkdown
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Rank-1 driver pruning via mediation with FDR thresholding

## Summary

Ranks iEDGE cis candidate drivers by Sobel statistical mediation of trans gene-expression effects and retains only the Rank-1 (top FDR, e.g. prune.thres=0.05) driver per cancer type, then validates the pruned list by hypergeometric enrichment against known cancer driver compendia.

## When to use

Use when candidate cis genes from SCNA/gene-expression integrative analysis must be prioritized by their trans mediation signal, and the pruned Rank-1 lists need in silico validation against known driver databases.

## When NOT to use

- Transgenes are not defined or downstream trans effects cannot be measured — mediation ranking of cis genes requires trans gene-expression signatures
- Sample size is small or copy-number/gene-expression correlations are weak — the Sobel test only achieved high AUC under adequate sample size and strong correlation effects
- The goal is cataloguing all cis genes within an alteration rather than prioritizing drivers (tools like GISTIC2.0 that only consider genes within the alterations)

## Inputs

- iEDGE object (cn, gep, cisgenes, transgenes) built via construct_iEDGE or GISTIC2.0 converters (make_GISTIC2/make_GISTIC2_focal)
- run_iEDGE output: per-lesion cis and trans differential-expression signatures with p/fdr columns
- Gene sets of known cancer drivers from multiple compendia (e.g. .gmt files) for hypergeometric validation

## Outputs

- Per-alteration Sobel mediation statistics with FDR-corrected significance for cis genes
- Pruned Rank-1 cis driver list (one top driver per lesion/cancer type)
- run_hyperEnrichment table of driver-database enrichment p-values/FDR per Rank-1 list (e.g. Supplemental Tables S7/S8)
- Pan-cancer KS test statistic comparing Rank-1 vs non-Rank-1 enrichment of p-values

## How to apply

After run_iEDGE produces cis and trans differential-expression signatures (e.g. fdr.cis.cutoff=0.25, fdr.trans.cutoff=0.01, fc.cis=1.2, fc.trans=1.5, min.group=2), compute mediation statistics for each cis gene with calc_sobel using bipartite=TRUE, since the hypothesis is that SCNA-related drivers mediate a larger proportion of downstream trans expression effects than non-drivers. Prune the cis/trans connection matrix with prune.col='fdr' and prune.thres=0.05 to keep the Rank-1 driver per alteration/cancer type. Validate each per-type Rank-1 list with run_hyperEnrichment (min.drawsize=3, hyperthres=0.25, one-sided Fisher exact test with FDR correction) against known oncogene/tumor-suppressor compendia. Apply correctly if Rank-1 lists are significantly enriched for known drivers (FDR < 0.05) in most cancer types — e.g. 15/19 TCGA types in the pan-cancer reproduction, KS test against uniform p=2.5e-13.

## Related tools

- **iEDGE** (Provides run_iEDGE, calc_sobel, and prune thresholds (prune.col='fdr', prune.thres=0.05) for mediation-based Rank-1 driver pruning; run_hyperEnrichment validates pruned lists) — https://github.com/montilab/iEDGE
- **R** (Runtime environment for executing the iEDGE package and analysis scripts) — https://github.com/montilab/iEDGE
- **limma** (Imported dependency supporting the differential-expression statistics whose cis/trans signatures feed the mediation test)
- **Biobase** (Dependency for the ExpressionSet-based iEDGE data containers (gep/cn objects) consumed by run_iEDGE and calc_sobel)

## Examples

```
res <- run_iEDGE(dat = dat, header = "testrun", outdir = "."); sob <- calc_sobel(x, y, z, y.names, z.names)
```

## Evaluation signals

- Pruned Rank-1 driver lists show significant enrichment for known cancer drivers (one-sided Fisher exact, FDR < 0.05) — reproduced pan-cancer result: 15/19 TCGA cancer types significant
- KS test of Rank-1 vs non-Rank-1 driver enrichment p-values against uniform gives p ≈ 2.5e-13 (Supplemental Table S7 reproduction)
- Amplification-dependency subset shows enrichment in 8/19 types with KS p = 0.00287 (Supplemental Table S8 reproduction)
- Every pruned cis entry has a computable Sobel statistic (no NA mediation scores) and FDR below the prune.thres=0.05 cutoff
- No Rank-1 list falls below min.drawsize=3 when passed to run_hyperEnrichment; hyperthres=0.25 draw coverage respected

## Limitations

- Sobel test validity depends on adequate sample size and strong correlation effects, conditions which were met in most but not all TCGA datasets used in the study
- The assumption that the number of mediated trans genes proxies driver importance may not hold for specific use cases
- Further experimental studies are needed to validate and characterize predicted drivers; enrichment against driver databases is only in silico validation

## Evidence

- [abstract] It then applies tests of statistical mediation to determine the cis genes predictive of the trans gene expression: "It then applies tests of statistical mediation to determine the cis genes predictive of the trans gene expression"
- [methods] calc_sobel signature: "calc_sobel(x, y, z, y.names, z.names)"
- [discussion] Mediation ranking in discussion: "the mediation analysis, ranks the set of cis genes by the extent of their mediation of trans genes"
- [discussion] In silico driver-database validation: "first testing for their enrichment with known drivers from multiple cancer driver databases"
- [readme] README module description: "pruning of cis/trans epi-DNA to gene expression connections using a statistical mediation model"
