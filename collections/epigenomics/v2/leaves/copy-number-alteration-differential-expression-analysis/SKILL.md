---
name: copy-number-alteration-differential-expression-analysis
description: Use when you have paired copy-number alteration data (e.g. Firehose GISTIC2.0
  lesion files) and matched gene expression data per tumor sample, and need to detect
  cis/trans genes associated with amplifications or deletions.
when_to_use_negative:
- Input lacks paired samples with matched (epi-)DNA and gene-expression profiles —
  iEDGE requires paired samples from the same tumors
- The alteration data are not copy-number lesions and no make_GISTIC2-compatible lesion
  file can be produced (e.g. unprocessed raw segment files)
- Goal is pathway enrichment of already-called cis/trans gene lists rather than deriving
  the DE signatures themselves — use run_hyperEnrichment instead
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: iEDGE
  role: R package providing the cis/trans differential expression pipeline (construct_iEDGE,
    iEDGE_DE, run_iEDGE) over epi-DNA and gene expression data
  repo: https://github.com/montilab/iEDGE
- name: limma
  role: Underlying differential expression engine imported by iEDGE for testing genes
    associated with alteration presence/absence
  repo: https://github.com/montilab/iEDGE
- name: Biobase
  role: Dependency providing the ExpressionSet containers for the gep/cn data in iEDGE
    objects
  repo: https://github.com/montilab/iEDGE
- name: GISTIC2.0
  role: Source of Firehose lesion files converted into the cn alteration matrix via
    make_GISTIC2/make_GISTIC2_focal
- name: R
  role: Runtime environment for the iEDGE package and rmarkdown reporting
  repo: https://github.com/montilab/iEDGE
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:60c0cbdc5c1cbf66a6fc7364756d842d6ca6baf43d8135592bdb39f72c8079ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - Biobase
  - GISTIC2.0
  - R
  - iEDGE
  - limma
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# copy-number-alteration-differential-expression-analysis

## Summary

Identify cis and trans gene expression signatures associated with the presence/absence of a copy-number alteration (SCNA) using the iEDGE R package's differential expression module, built on limma, to define candidate cis drivers and downstream trans effects.

## When to use

Use when you have paired copy-number alteration data (e.g. Firehose GISTIC2.0 lesion files) and matched gene expression data per tumor sample, and need to detect cis/trans genes associated with amplifications or deletions.

## When NOT to use

- Input lacks paired samples with matched (epi-)DNA and gene-expression profiles — iEDGE requires paired samples from the same tumors
- The alteration data are not copy-number lesions and no make_GISTIC2-compatible lesion file can be produced (e.g. unprocessed raw segment files)
- Goal is pathway enrichment of already-called cis/trans gene lists rather than deriving the DE signatures themselves — use run_hyperEnrichment instead

## Inputs

- Firehose GISTIC2.0 lesion file per TCGA cancer type
- RNASeq gene expression matrix (gep) matched to the same samples
- Copy-number alteration matrix (cn) with amplification/deletion lesion annotations
- cisgene annotations mapping lesions to genes within the alteration

## Outputs

- iEDGE object (.rds) holding cn, gep, cisgenes slots
- cis differential expression signature per epi-DNA alteration (FDR < 0.25, |FC| > 1.2)
- trans differential expression signature per epi-DNA alteration (FDR < 0.01, |FC| > 1.5)
- run_iEDGE output tables (.txt) and html reports with interactive plots

## How to apply

Preprocess Firehose GISTIC2.0 lesion files with the iEDGE converters make_GISTIC2 (arm-level, with amp_thres_arm/del_thres_arm) or make_GISTIC2_focal (focal, with qvalue_focal) to build a cn matrix paired to a gep expression matrix plus cisgene annotations. Assemble these into an iEDGE object via construct_iEDGE(cn, gep, cisgenes, ...) and load with readRDS. Run run_iEDGE with parameters fdr.cis.cutoff=0.25, fdr.trans.cutoff=0.01, fc.cis=1.2, fc.trans=1.5, min.group=2, onesided.cis=TRUE, onesided.trans=FALSE, uptest="Amplification", downtest="Deletion"; the underlying iEDGE_DE step uses limma for differential expression between altered and unaltered sample groups. The rationale is that SCNA-related drivers mediate a larger proportion of downstream trans expression effects than non-drivers, so the cis/trans signatures feed the subsequent Sobel mediation ranking (calc_sobel).

## Related tools

- **iEDGE** (R package providing the cis/trans differential expression pipeline (construct_iEDGE, iEDGE_DE, run_iEDGE) over epi-DNA and gene expression data) — https://github.com/montilab/iEDGE
- **limma** (Underlying differential expression engine imported by iEDGE for testing genes associated with alteration presence/absence) — https://github.com/montilab/iEDGE
- **Biobase** (Dependency providing the ExpressionSet containers for the gep/cn data in iEDGE objects) — https://github.com/montilab/iEDGE
- **GISTIC2.0** (Source of Firehose lesion files converted into the cn alteration matrix via make_GISTIC2/make_GISTIC2_focal)
- **R** (Runtime environment for the iEDGE package and rmarkdown reporting) — https://github.com/montilab/iEDGE

## Examples

```
dat <- construct_iEDGE(cn, gep, cisgenes, transgenes = NA, cn.fdat, gep.fdat); res <- run_iEDGE(dat = dat, header = "testrun", outdir = ".")
```

## Evaluation signals

- The iEDGE object built via construct_iEDGE contains non-empty cn, gep, and cisgenes slots and loads with readRDS without error
- Cis signatures are called at fdr.cis.cutoff=0.25 and trans signatures at fdr.trans.cutoff=0.01, with fold-change cutoffs fc.cis=1.2 and fc.trans=1.5 and min.group=2
- Reproducing the TCGA pan-cancer run should yield Rank-1 cis genes significantly enriched for known drivers (FDR < 0.05) in 15 of 19 cancer types (KS test p=2.5e-13), confirming the DE signatures feed correct mediation analysis
- In the TCGA BRCA bootstrap check, ~69.2% of Rank-1 cis genes should be recovered with inclusion fraction > 0.75 across 100 resamples, indicating stable downstream DE/mediation results
- Output matches the reference Supplemental Tables (S7, S8) when running the scripts_TCGA_pancancer preprocessing and pipeline

## Limitations

- The Sobel-test mediation downstream of the DE step is valid only under adequate sample size and strong correlation effects, conditions not met in all datasets
- Assuming the number of mediated trans genes proxies driver importance may not hold for specific use cases
- GISTIC2.0-only approaches (and by extension SCNA-within-lesion gene lists) generally miss trans effects captured by iEDGE, so the DE step should not be replaced by lesion-gene annotation alone

## Evidence

- [abstract] iEDGE first identifies the cis and trans gene expression signatures associated with the presence/absence of a particular epi-DNA alteration across samples: "iEDGE first identifies the cis and trans gene expression signatures associated with the presence/absence of a particular epi-DNA alteration across samples"
- [discussion] uses differential expression analysis to determine the cis and trans genes that are associated with the presence/absence of a particular epi-DNA alteration across samples: "uses differential expression analysis to determine the cis and trans genes that are associated with the presence/absence of a particular epi-DNA alteration across samples"
- [methods] dat <- construct_iEDGE(cn, gep, cisgenes, transgenes = NA, cn.fdat, gep.fdat, cn.pdat = NA, gep.pdat = NA): "dat <- construct_iEDGE(cn, gep, cisgenes, transgenes = NA, cn.fdat, gep.fdat, cn.pdat = NA, gep.pdat = NA)"
- [readme] cis/trans differential expression analysis of epi-DNA and gene expression data: "1. cis/trans differential expression analysis of epi-DNA and gene expression data"
