---
name: cis-driver-gene-rank-derivation
description: Use when assessing the reproducibility of iEDGE Rank-1 cis driver predictions
  across bootstrapped resamples of a TCGA-style SCNA/gene-expression cohort, or when
  a stability measure for mediation-ranked cis genes is required.
when_to_use_negative:
- Cohort is too small or correlation effects too weak for the Sobel test's validity
  assumptions — the article notes these conditions were not met in all datasets.
- The goal is single-run discovery of novel cis drivers (with pathway enrichment via
  run_hyperEnrichment), not stability testing of already-derived Rank-1 calls.
- Data are unpaired or lack copy-number alteration and gene expression measured in
  the same samples, so the cis/trans mediation structure cannot be re-estimated per
  resample.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: iEDGE
  role: Rerun per bootstrap resample to recompute cis/trans differential expression
    and Sobel mediation and re-derive the Rank-1 cis gene per alteration
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Host language for the 100-resample bootstrap loop and the fraction-of-inclusion
    summarization
- name: limma
  role: Differential expression engine called within iEDGE to derive cis (FDR 0.25)
    and trans (FDR 0.01) signatures in each resample
- name: Biobase
  role: Provides the ExpressionSet-based containers holding the BRCA expression data
    inside the iEDGE object
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:afd8a8ba80e5d0e3cb802dc6d1610b2e0e1e983e85a6d7344da0862d891b1591
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - Biobase
  - R
  - iEDGE
  - limma
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cis-driver-gene-rank-derivation

## Summary

Quantifies bootstrap reproducibility of iEDGE's Rank-1 cis driver calls: 100 with-replacement resamples of a TCGA BRCA-style cohort are each rerun through the cis FDR 0.25 / trans FDR 0.01 differential-expression and Sobel mediation pipeline, and per-gene fraction-of-inclusion values test the reported stability (69.2% of Rank-1 genes above 0.75).

## When to use

Use when assessing the reproducibility of iEDGE Rank-1 cis driver predictions across bootstrapped resamples of a TCGA-style SCNA/gene-expression cohort, or when a stability measure for mediation-ranked cis genes is required.

## When NOT to use

- Cohort is too small or correlation effects too weak for the Sobel test's validity assumptions — the article notes these conditions were not met in all datasets.
- The goal is single-run discovery of novel cis drivers (with pathway enrichment via run_hyperEnrichment), not stability testing of already-derived Rank-1 calls.
- Data are unpaired or lack copy-number alteration and gene expression measured in the same samples, so the cis/trans mediation structure cannot be re-estimated per resample.

## Inputs

- Preprocessed iEDGE object for TCGA BRCA containing the copy-number alteration matrix, gene expression matrix and cisgene annotations
- Sample-level cohort definition used to draw the 100 with-replacement bootstrap resamples
- SCNA type labels (amplification vs deletion) for the SCNA-bias check of inclusion fractions

## Outputs

- bootstrap_reproducibility_fractions.csv — per-cis-gene table of fraction-of-inclusion values across the 100 resamples
- Fig. 5A-style distribution of fraction-of-inclusion values across Rank-1 cis genes
- Proportion of Rank-1 cis genes with inclusion fraction >0.75 (reported value: 69.2%)

## How to apply

Load the preprocessed iEDGE object for the cohort (TCGA BRCA copy-number alteration matrix, gene expression matrix and cisgene annotations), built with construct_iEDGE. Generate 100 bootstrap resamples of the samples at fixed size equal to the original cohort, drawn with replacement. For each resample rerun the iEDGE pipeline — differential expression with cis FDR cutoff 0.25 and trans FDR cutoff 0.01 followed by Sobel-test mediation (calc_sobel / run_iEDGE) — and record the Rank-1 cis gene per alteration, defined as the cis gene mediating the largest number of trans genes. For each cis gene compute the fraction of the 100 resamples in which it is recovered as Rank-1 and save the per-gene table as bootstrap_reproducibility_fractions.csv. Reconstruct the Fig. 5A distribution of fraction-of-inclusion values and accept the analysis if the proportion of Rank-1 genes with inclusion fraction >0.75 reproduces the reported 69.2% within Monte-Carlo sampling variability; the ranking rationale is the article's hypothesis that SCNA-related drivers mediate a larger proportion of downstream trans gene expression effects than non-drivers.

## Related tools

- **iEDGE** (Rerun per bootstrap resample to recompute cis/trans differential expression and Sobel mediation and re-derive the Rank-1 cis gene per alteration) — https://github.com/montilab/iEDGE
- **R** (Host language for the 100-resample bootstrap loop and the fraction-of-inclusion summarization)
- **limma** (Differential expression engine called within iEDGE to derive cis (FDR 0.25) and trans (FDR 0.01) signatures in each resample)
- **Biobase** (Provides the ExpressionSet-based containers holding the BRCA expression data inside the iEDGE object)

## Examples

```
dat <- construct_iEDGE(cn, gep, cisgenes, transgenes = NA, cn.fdat, gep.fdat, cn.pdat = NA, gep.pdat = NA); res <- run_iEDGE(dat = dat, header = "testrun", outdir = ".")
```

## Evaluation signals

- Computed proportion of Rank-1 cis genes with inclusion fraction >0.75 reproduces the reported 69.2% within Monte-Carlo sampling variability
- bootstrap_reproducibility_fractions.csv contains one fraction-of-inclusion value per Rank-1 cis gene, each bounded in [0,1]
- Exactly 100 resamples at fixed size equal to the original cohort, drawn with replacement, were processed
- Each resample used the same thresholds (cis FDR 0.25, trans FDR 0.01) and yielded exactly one Rank-1 cis gene per alteration — the cis gene mediating the largest number of trans genes
- Inclusion-fraction results show no bias by SCNA type (amplification vs deletion)

## Limitations

- Sobel test validity depends on adequate sample size and strong correlation effects; the article notes these conditions were not met in all datasets (met in most of the TCGA datasets used).
- The assumption that the number of mediated trans genes proxies driver importance may not hold for specific use cases.
- The 69.2% figure is a Monte-Carlo estimate over 100 resamples and carries sampling variability; replicate bootstrap runs will not give identical fractions.

## Evidence

- [intro] Mediation ranking rationale: "SCNA-related drivers of tumorigenesis will mediate a larger proportion of the downstream effect observable by trans gene expression than non-drivers"
- [abstract] Mediation-based cis ranking step: "It then applies tests of statistical mediation to determine the cis genes predictive of the trans gene expression"
- [discussion] Rank-1 derivation step: "the mediation analysis, ranks the set of cis genes by the extent of their mediation of trans genes"
- [discussion] Sobel test performance: "the Sobel test of mediation achieved high AUC for uncovering true mediation effects"
- [methods] Pipeline rerun call: "res <- run_iEDGE(dat = dat, header = "testrun", outdir = ".""
- [readme] Mediation module description: "pruning of cis/trans epi-DNA to gene expression connections using a statistical mediation model"
