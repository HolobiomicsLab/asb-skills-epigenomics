---
name: kolmogorov-smirnov-pancancer-enrichment-aggregation
description: 'Use when replicating the pan-cancer iEDGE evaluation: per-type Rank-1
  cis driver lists are available and you must test known-driver enrichment per cancer
  type and then assess aggregate significance (15/19 types FDR<0.05, KS p=2.5e-13).'
when_to_use_negative:
- You have only a single cancer type — the KS aggregation step requires p-values pooled
  across multiple (here 19) cancer types
- Input cis gene lists have not been ranked by Sobel mediation pruning (Rank-1 selection),
  since the test specifically contrasts Rank-1 vs non-Rank-1 drivers
- Small cohorts where the Sobel mediation assumptions (adequate sample size, strong
  correlation effects) are not met, as noted in the article's discussion
tools:
- name: iEDGE
  role: Provides run_iEDGE, calc_sobel, prune.col/prune.thres Rank-1 pruning, and
    run_hyperEnrichment for per-type driver enrichment
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Language for running the iEDGE pipeline and performing the aggregate KS test
  repo: https://github.com/montilab/iEDGE
- name: limma
  role: Supports the cis/trans differential expression analysis that feeds the Rank-1
    driver lists
  repo: https://github.com/montilab/iEDGE
- name: Biobase
  role: Dependency underlying iEDGE expression data objects (gep, cn)
  repo: https://github.com/montilab/iEDGE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:1b9201efa1cae7f9601bbd5c8872e114c02e1fc8c985ebf7f8d9213186be94fe
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# kolmogorov-smirnov-pancancer-enrichment-aggregation

## Summary

Aggregates per-cancer-type driver enrichment statistics across the 19 TCGA cancer types and assesses whether iEDGE Rank-1 cis driver predictions are enriched for known cancer drivers, using one-sided Fisher/hypergeometric tests per type and a Kolmogorov-Smirnov test over the aggregate p-values.

## When to use

Use when replicating the pan-cancer iEDGE evaluation: per-type Rank-1 cis driver lists are available and you must test known-driver enrichment per cancer type and then assess aggregate significance (15/19 types FDR<0.05, KS p=2.5e-13).

## When NOT to use

- You have only a single cancer type — the KS aggregation step requires p-values pooled across multiple (here 19) cancer types
- Input cis gene lists have not been ranked by Sobel mediation pruning (Rank-1 selection), since the test specifically contrasts Rank-1 vs non-Rank-1 drivers
- Small cohorts where the Sobel mediation assumptions (adequate sample size, strong correlation effects) are not met, as noted in the article's discussion

## Inputs

- Firehose GISTIC2.0 lesion files and RNASeq expression data for each of the 19 TCGA cancer types
- iEDGE objects (cn, gep, cisgenes) built with make_GISTIC2/make_GISTIC2_focal converters and loaded via readRDS
- Known cancer driver gene sets from multiple driver-database compendia

## Outputs

- Per-cancer-type one-sided Fisher/hypergeometric enrichment p-values for Rank-1 driver lists (run_hyperEnrichment output)
- Kolmogorov-Smirnov test statistics and p-values comparing Rank-1 vs non-Rank-1 driver enrichment across the 19 types
- Reproduced Supplemental Table S7 (15/19 types significant, KS p=2.5e-13) and Table S8 (8/19 amplification-dependency, KS p=0.00287)

## How to apply

For each of the 19 TCGA cancer types, run run_iEDGE (fdr.cis.cutoff=0.25, fc.cis=1.2, onesided.cis=TRUE, uptest='Amplification', downtest='Deletion'), then calc_sobel with bipartite=TRUE and Rank-1 pruning via prune.col='fdr', prune.thres=0.05 to rank cis candidate drivers by their mediation of trans effects. Test each per-type Rank-1 driver list against known driver compendia using run_hyperEnrichment with min.drawsize=3 and hyperthres=0.25, applying a one-sided Fisher exact test with FDR correction. Finally aggregate the 19 per-type Fisher test statistics and compute a Kolmogorov-Smirnov test comparing Rank-1 versus non-Rank-1 driver enrichment, separately for overall driver enrichment and the amplification-dependency subset. Correctness is judged by matching the published Supplemental Tables S7 (15/19 types, KS p=2.5e-13) and S8 (8/19 types amplification-dependent, KS p=0.00287).

## Related tools

- **iEDGE** (Provides run_iEDGE, calc_sobel, prune.col/prune.thres Rank-1 pruning, and run_hyperEnrichment for per-type driver enrichment) — https://github.com/montilab/iEDGE
- **R** (Language for running the iEDGE pipeline and performing the aggregate KS test) — https://github.com/montilab/iEDGE
- **limma** (Supports the cis/trans differential expression analysis that feeds the Rank-1 driver lists) — https://github.com/montilab/iEDGE
- **Biobase** (Dependency underlying iEDGE expression data objects (gep, cn)) — https://github.com/montilab/iEDGE

## Examples

```
run_hyperEnrichment(drawn = rank1_drivers, categories = known_drivers_gmt, ntotal = ntotal, min.drawsize = 3, mht = TRUE, hypercol = 'hypergeom_p', hyperthres = 0.25)
```

## Evaluation signals

- Reproduced Supplemental Table S7 shows 15/19 TCGA cancer types significantly enriched for known drivers (FDR < 0.05) with KS p = 2.5e-13
- Reproduced Supplemental Table S8 shows 8/19 types with amplification-dependency enrichment and KS p = 0.00287
- Per-type enrichment uses the one-sided Fisher exact test with FDR correction, min.drawsize=3, hyperthres=0.25 — check these parameters match run_hyperEnrichment output
- output_matches_reference check against the published S7/S8 tables for the 19 cancer types

## Limitations

- Sobel test validity depends on adequate sample size and strong correlation effects, conditions not met in all datasets
- The assumption that the number of mediated trans genes proxies driver importance may not hold for specific use cases
- A side-by-side comparison with CONEXIC was not possible because its software is not available

## Evidence

- [abstract] Per-type Fisher enrichment and aggregate KS: "cis gene drivers identified by iEDGE were found to be significantly enriched for known driver genes from multiple compendia of validated oncogenes and tumor suppressors"
- [methods] run_hyperEnrichment signature: "run_hyperEnrichment(drawn, categories, ntotal, min.drawsize, mht, verbose, order, hypercol, hyperthres)"
- [methods] Sobel mediation as the basis of driver ranking: "calc_sobel(x, y, z, y.names, z.names)"
- [discussion] In silico validation via driver-database enrichment: "first testing for their enrichment with known drivers from multiple cancer driver databases"
- [readme] iEDGE as the pipeline the enrichment tests validate: "iEDGE is an R-package for performing integrative analysis of (epi-)DNA and gene expression data"
