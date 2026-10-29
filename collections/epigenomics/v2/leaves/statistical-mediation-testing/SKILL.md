---
name: statistical-mediation-testing
description: Use when you have paired copy-number-alteration (or epi-DNA) and gene-expression
  data with candidate cis genes and need to rank cis genes by how much of the trans
  expression effect they mediate, or to re-derive Rank-1 cis gene calls under resampling.
when_to_use_negative:
- Datasets where sample size or correlation effects are inadequate — the article notes
  the Sobel test's validity depends on these conditions, which were not met in all
  datasets
- Use cases where the number of mediated trans genes is not an appropriate proxy for
  driver importance, as the authors caveat
- Problems requiring comparison with CONEXIC, whose software is not available
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0622
tools:
- name: iEDGE
  role: R package whose mediation module prunes cis/trans epi-DNA to gene-expression
    connections using the statistical mediation model and ranks cis genes
  repo: https://github.com/montilab/iEDGE
- name: limma
  role: Used upstream in iEDGE differential expression analysis to define the cis/trans
    gene sets that mediation is tested on
- name: Biobase
  role: Bioconductor base dependency supporting the iEDGE data-object structure used
    in mediation analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:aabfa59815ae8d59f02c20e04320ce239157b483264b6b1785a0ed6173824d30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - Biobase
  - R
  - iEDGE
  - limma
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# statistical-mediation-testing

## Summary

Sobel-test statistical mediation analysis used by iEDGE to rank cis genes by the extent to which they mediate the trans gene expression effects of epi-DNA alterations (e.g., SCNAs), thereby prioritizing candidate cis driver genes.

## When to use

Use when you have paired copy-number-alteration (or epi-DNA) and gene-expression data with candidate cis genes and need to rank cis genes by how much of the trans expression effect they mediate, or to re-derive Rank-1 cis gene calls under resampling.

## When NOT to use

- Datasets where sample size or correlation effects are inadequate — the article notes the Sobel test's validity depends on these conditions, which were not met in all datasets
- Use cases where the number of mediated trans genes is not an appropriate proxy for driver importance, as the authors caveat
- Problems requiring comparison with CONEXIC, whose software is not available

## Inputs

- iEDGE object (constructed via construct_iEDGE) containing copy-number alteration matrix, gene expression matrix, and cisgene annotations
- Differential expression results: significant cis genes (cis FDR 0.25) and trans genes (trans FDR 0.01) per alteration
- Bootstrap resampled datasets (e.g., 100 resamples with replacement) for reproducibility assessment

## Outputs

- Per-alteration cis gene ranking by number of mediated trans genes (Rank-1 cis gene call)
- Sobel mediation test statistics per cis gene (calc_sobel output)
- Per-gene fraction-of-inclusion table across resamples (e.g., bootstrap_reproducibility_fractions.csv)

## How to apply

First run differential expression (iEDGE_DE / limma) with cis FDR cutoff 0.25 and trans FDR cutoff 0.01 to identify cis and trans genes associated with presence/absence of each epi-DNA alteration. Then apply the Sobel test of mediation (calc_sobel) for each cis gene to quantify whether it is predictive of the trans gene expression, predicated on the hypothesis that true drivers mediate a larger proportion of the downstream trans effect than non-drivers. Rank the cis genes per alteration by the number of trans genes they mediate; the top-ranked gene is the Rank-1 cis gene. In the BRCA reproducibility task, rerun this mediation ranking over 100 bootstrap resamples (resample size equal to the original cohort, with replacement) and compute each Rank-1 gene's fraction of inclusion. The approach is justified by simulations showing the Sobel test achieved high AUC when sample size and correlation effects are adequate.

## Related tools

- **iEDGE** (R package whose mediation module prunes cis/trans epi-DNA to gene-expression connections using the statistical mediation model and ranks cis genes) — https://github.com/montilab/iEDGE
- **limma** (Used upstream in iEDGE differential expression analysis to define the cis/trans gene sets that mediation is tested on)
- **Biobase** (Bioconductor base dependency supporting the iEDGE data-object structure used in mediation analysis)

## Examples

```
res <- run_iEDGE(dat = dat, header = "testrun", outdir = ".")  # after calc_sobel(x, y, z, y.names, z.names) mediation testing
```

## Evaluation signals

- Reproducing the reported result that 69.2% of TCGA BRCA Rank-1 cis genes have bootstrap inclusion fraction > 0.75 within Monte-Carlo sampling variability
- Rank-1 recovery is not biased by SCNA type (amplification vs. deletion)
- Sobel test achieves high AUC in simulated data with known mediations at adequate sample size and strong correlation effects
- Per-gene fraction-of-inclusion table is saved as bootstrap_reproducibility_fractions.csv with one inclusion fraction per Rank-1 cis gene
- Mediated trans gene counts follow cis/trans differential expression FDR cutoffs (cis 0.25, trans 0.01) before mediation ranking

## Limitations

- Sobel test validity depends on adequate sample size and strong correlation effects, conditions not met in all datasets
- Assumption that the number of mediated trans genes proxies driver importance may not hold for specific use cases
- Further experimental studies are needed to validate and characterize predicted drivers

## Evidence

- [intro] The approach is predicated on the hypothesis that SCNA-related drivers of tumorigenesis will mediate a larger proportion of the downstream effect observable by trans gene expression than non-drivers.: "The approach is predicated on the hypothesis that SCNA-related drivers of tumorigenesis will mediate a larger proportion of the downstream effect observable by trans gene expression than non-drivers."
- [discussion] the mediation analysis, ranks the set of cis genes by the extent of their mediation of trans genes: "the mediation analysis, ranks the set of cis genes by the extent of their mediation of trans genes"
- [discussion] the Sobel test of mediation achieved high AUC for uncovering true mediation effects: "the Sobel test of mediation achieved high AUC for uncovering true mediation effects"
- [methods] calc_sobel(x, y, z, y.names, z.names): "calc_sobel(x, y, z, y.names, z.names)"
- [abstract] It then applies tests of statistical mediation to determine the cis genes predictive of the trans gene expression: "It then applies tests of statistical mediation to determine the cis genes predictive of the trans gene expression"
