---
name: differential-gene-expression-analysis
description: Use when you have paired copy-number/epi-DNA alteration calls (e.g. GISTIC2.0
  focal SCNAs) and matched RSEM log2 expression for the same tumor samples, and need
  cis (in-peak) and trans (genome-wide) DE genes per alteration.
when_to_use_negative:
- When you only need genes prioritized as drivers by their mediation of trans effects
  — use the iEDGE mediation (calc_sobel) module instead of raw DE
- When alterations are broad arm-level events rather than focal peaks — iEDGE_DE is
  designed around focal peak cis gene lists; use make_GISTIC2 arm-level processing
  instead
- When expression and alteration data come from unpaired or mismatched samples — the
  design requires paired omics profiles from the same specimens
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: iEDGE
  role: R package providing construct_iEDGE and run_iEDGE/iEDGE_DE for cis/trans differential
    expression of epi-DNA alterations vs expression
  repo: https://github.com/montilab/iEDGE
- name: limma
  role: Linear-model differential expression engine imported by iEDGE for testing
    alteration-positive vs alteration-negative groups
- name: R
  role: Runtime environment for the iEDGE package and analysis scripts
- name: Biobase
  role: Bioconductor base package (Depends) used for expression/phenotype data containers
    in iEDGE
- name: GISTIC2.0
  role: Upstream source of focal SCNA calls defining cis gene lists per alteration
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:09eb5312c9b20d18861eee98888abdecb9d3f41387796e56bf39754621607322
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# differential-gene-expression-analysis

## Summary

limma-based cis/trans differential expression analysis of gene expression between samples with and without a focal (epi-)DNA alteration, as implemented in the iEDGE R package's iEDGE_DE engine to identify alteration-associated cis and trans gene signatures.

## When to use

Use when you have paired copy-number/epi-DNA alteration calls (e.g. GISTIC2.0 focal SCNAs) and matched RSEM log2 expression for the same tumor samples, and need cis (in-peak) and trans (genome-wide) DE genes per alteration.

## When NOT to use

- When you only need genes prioritized as drivers by their mediation of trans effects — use the iEDGE mediation (calc_sobel) module instead of raw DE
- When alterations are broad arm-level events rather than focal peaks — iEDGE_DE is designed around focal peak cis gene lists; use make_GISTIC2 arm-level processing instead
- When expression and alteration data come from unpaired or mismatched samples — the design requires paired omics profiles from the same specimens

## Inputs

- GISTIC2.0 focal SCNA calls per sample (amplifications and deletions with peak boundaries)
- RSEM log2 gene expression matrix with paired samples and phenotype labels (e.g. pam50/HER2 subtype in gep.pdat)
- Per-alteration cis gene lists (genes within each focal peak)
- cn.fdat annotation frame with alteration direction (Amplification/Deletion)

## Outputs

- Full cis differential expression table per alteration (limma statistics: fold change, p-value, FDR) for all genes
- Filtered significant cis gene table (FDR < 0.25, fold change > 1.2, one-directional)
- Full and filtered trans differential expression tables (FDR < 0.01, fold change > 1.5, two-sided)
- Tabular export files reproducing Supplemental Tables S1 (cis) and S2 (trans)

## How to apply

Assemble alteration calls (e.g. Firehose-processed TCGA BRCA GISTIC2.0 focal peaks: 29 amplifications, 40 deletions) and an RSEM log2 expression matrix (1050 paired primary-tumor samples with pam50/HER2 subtype labels) into an iEDGE object with construct_iEDGE, supplying per-alteration cis gene lists and cn.fdat direction annotations. Run the iEDGE_DE engine via run_iEDGE with cis thresholds fdr.cis.cutoff=0.25, fc.cis=1.2, onesided.cis=TRUE (one-directional UP for amplifications, DOWN for deletions) and trans thresholds fdr.trans.cutoff=0.01, fc.trans=1.5, onesided.trans=FALSE, using limma linear models to compare alteration-positive vs alteration-negative samples (min.group=2). Collect full statistics tables from res$de$cis$full and res$de$trans$full and the filtered significant-gene tables, then export them as tabular files. Rationale: the one-sided cis test encodes the expected direction of the copy-number effect, while trans effects are tested two-sided since direction is not assumed a priori.

## Related tools

- **iEDGE** (R package providing construct_iEDGE and run_iEDGE/iEDGE_DE for cis/trans differential expression of epi-DNA alterations vs expression) — https://github.com/montilab/iEDGE
- **limma** (Linear-model differential expression engine imported by iEDGE for testing alteration-positive vs alteration-negative groups)
- **R** (Runtime environment for the iEDGE package and analysis scripts)
- **Biobase** (Bioconductor base package (Depends) used for expression/phenotype data containers in iEDGE)
- **GISTIC2.0** (Upstream source of focal SCNA calls defining cis gene lists per alteration)

## Examples

```
dat <- construct_iEDGE(cn, gep, cisgenes, transgenes = NA, cn.fdat, gep.fdat); res <- run_iEDGE(dat = dat, header = "testrun", outdir = ".")
```

## Evaluation signals

- Output tables contain FDR-adjusted p-values and fold changes for each of the 69 alterations
- Cis genes are called significant only at FDR < 0.25 with fold change > 1.2 and in the expected one-sided direction (UP for amplifications, DOWN for deletions per cn.fdat)
- Trans genes are called significant only at FDR < 0.01 with fold change > 1.5 under two-sided testing
- Against the TCGA BRCA benchmark, the analysis reproduces Supplemental Table S1: 1330 significant cis genes (269 amplification, 1061 deletion), averaging 20 genes per SCNA (median 8)
- res$de$cis$full and res$de$trans$full contain statistics for all tested genes, not only the filtered significant subset

## Limitations

- Significant cis gene counts vary widely across alterations (median 8 vs mean 20), so alterations with few informative samples may yield sparse signatures
- The one-sided cis test assumes the direction of the copy-number effect; it is inappropriate if direction is unknown or inconsistent
- This DE step alone identifies associated genes, not drivers — iEDGE's mediation analysis (Sobel test) is required for driver prioritization, and the Sobel test requires adequate sample size and strong correlation effects to be valid

## Evidence

- [abstract] iEDGE first identifies the cis and trans gene expression signatures associated with the presence/absence of a particular epi-DNA alteration across samples: "iEDGE first identifies the cis and trans gene expression signatures associated with the presence/absence of a particular epi-DNA alteration across samples"
- [readme] cis/trans DE is the first iEDGE module: "1. cis/trans differential expression analysis of epi-DNA and gene expression data"
- [methods] construct_iEDGE builds the object from standard matrices: "You can use `construct_iEDGE` to generate the iEDGE dataset from standard matrices"
- [methods] run_iEDGE invocation signature: "res <- run_iEDGE(dat = dat, header = "testrun", outdir = ".""
- [methods] DE engine named in article methods: "iEDGE_DE(cn, gep, cisgenes, transgenes, header, gepid, cnid, f.dir.out"
