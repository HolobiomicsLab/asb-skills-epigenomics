---
name: multi-omics-data-object-construction
description: Use when you have paired sample-level copy-number/epi-DNA alteration
  calls and matched gene expression matrices and need to package them, with cis gene
  lists and sample/feature annotations, into the iEDGE object required by run_iEDGE.
when_to_use_negative:
- Input data are not sample-paired — iEDGE integrates (epi-)DNA and expression from
  the same samples, so unpaired alteration and expression datasets cannot be combined
  into a single iEDGE object.
- You only need a differential expression table and already have a run_iEDGE result
  object (res$de$cis$full / res$de$trans$full) — construction is already done.
- Alterations are not discrete calls (e.g., continuous copy-number values); construct_iEDGE
  expects alteration status calls per sample per feature.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: iEDGE
  role: R package providing construct_iEDGE to build the integrated (epi-)DNA + gene
    expression data object from standard matrices
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Runtime environment in which the iEDGE object is constructed and annotated
- name: Biobase
  role: Dependency of iEDGE supporting the expression/annotation data structures used
    in the constructed object
- name: GISTIC2.0
  role: Upstream producer of the focal SCNA calls (cn matrix and per-peak cis gene
    lists) that populate the constructed object
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:fd41ef4b090d95c1fbe80ae8a0ba5543b3d4ce3bacb464f206ed5892b418eaa7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - Biobase
  - GISTIC2.0
  - R
  - iEDGE
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# multi-omics-data-object-construction

## Summary

Assemble paired (epi-)DNA alteration calls (e.g., GISTIC2.0 focal SCNA peak calls) and matched gene expression data (e.g., RSEM log2 matrix) into a single iEDGE object using construct_iEDGE, supplying per-alteration cis gene lists and direction annotations, so downstream cis/trans differential expression and mediation analyses can run within the iEDGE R package.

## When to use

Use when you have paired sample-level copy-number/epi-DNA alteration calls and matched gene expression matrices and need to package them, with cis gene lists and sample/feature annotations, into the iEDGE object required by run_iEDGE.

## When NOT to use

- Input data are not sample-paired — iEDGE integrates (epi-)DNA and expression from the same samples, so unpaired alteration and expression datasets cannot be combined into a single iEDGE object.
- You only need a differential expression table and already have a run_iEDGE result object (res$de$cis$full / res$de$trans$full) — construction is already done.
- Alterations are not discrete calls (e.g., continuous copy-number values); construct_iEDGE expects alteration status calls per sample per feature.

## Inputs

- Copy-number / epi-DNA alteration matrix (cn), e.g., GISTIC2.0 focal SCNA peak calls per sample
- Gene expression matrix (gep), e.g., RSEM log2 expression values for paired samples
- Per-alteration cis gene lists (genes within each focal peak)
- cn.fdat: feature-level data frame annotating alteration direction (amplification/deletion)
- gep.fdat: expression feature annotations
- Optional cn.pdat / gep.pdat: sample-level annotations (e.g., pam50/HER2 subtype labels)

## Outputs

- iEDGE object (dat) bundling paired cn and gep matrices, cis/trans gene lists, and feature/sample annotations, ready for run_iEDGE

## How to apply

Load the alteration matrix (e.g., Firehose-processed TCGA BRCA GISTIC2.0 focal SCNA calls — 29 amplifications and 40 deletions) and the paired gene expression matrix (e.g., RSEM log2 values from 1050 primary tumor samples) into R. Assemble per-alteration cis gene lists (genes within each focal peak) and annotate alteration direction (amplification vs. deletion) in the cn.fdat data frame, plus optional sample annotations in cn.pdat/gep.pdat. Call construct_iEDGE(cn, gep, cisgenes, transgenes = NA, cn.fdat, gep.fdat, cn.pdat = NA, gep.pdat = NA) to generate the iEDGE dataset object from these standard matrices, as documented in the article's methods. The rationale is that iEDGE requires sample-paired multi-omics inputs bundled in one object so that differential expression (iEDGE_DE) and mediation (calc_sobel) modules can consistently match alteration status to expression across the same samples. Validate that sample identifiers in cn and gep align (paired samples) and that each cis gene list corresponds to its alteration row before proceeding to run_iEDGE with thresholds such as fdr.cis.cutoff=0.25, fc.cis=1.2, fdr.trans.cutoff=0.01, fc.trans=1.5.

## Related tools

- **iEDGE** (R package providing construct_iEDGE to build the integrated (epi-)DNA + gene expression data object from standard matrices) — https://github.com/montilab/iEDGE
- **R** (Runtime environment in which the iEDGE object is constructed and annotated)
- **Biobase** (Dependency of iEDGE supporting the expression/annotation data structures used in the constructed object)
- **GISTIC2.0** (Upstream producer of the focal SCNA calls (cn matrix and per-peak cis gene lists) that populate the constructed object)

## Examples

```
dat <- construct_iEDGE(cn, gep, cisgenes, transgenes = NA, cn.fdat, gep.fdat, cn.pdat = NA, gep.pdat = NA)
```

## Evaluation signals

- The returned object is a valid iEDGE dataset whose cn and gep matrices share the same paired samples (e.g., the 1050 TCGA BRCA primary-tumor samples).
- Every alteration row in cn has a non-empty cis gene list and a direction annotation in cn.fdat (amplification vs. deletion, e.g., 29 amplifications and 40 deletions for TCGA BRCA).
- The constructed object passes directly into run_iEDGE(dat = dat, header = ..., outdir = ...) without further reshaping, and produces cis/trans DE tables with FDR-adjusted p-values and fold changes.
- Cis genes called downstream are restricted to genes within the focal GISTIC2.0 peaks, and significance respects fdr.cis.cutoff=0.25 with fc.cis=1.2 and onesided.cis=TRUE.

## Limitations

- construct_iEDGE assumes alteration calls and expression come from the same, sample-paired cohort; unpaired or batch-mismatched omics data cannot be validly bundled.
- Downstream iEDGE results depend on upstream GISTIC2.0 peak definitions and thresholds (e.g., make_GISTIC2_focal qvalue_focal), so errors in peak calling propagate into the constructed object.
- Trans gene sets may be left as NA (transgenes = NA), in which case only cis-level analysis is fully specified at construction time.

## Evidence

- [methods] You can use `construct_iEDGE` to generate the iEDGE dataset from standard matrices: "You can use `construct_iEDGE` to generate the iEDGE dataset from standard matrices"
- [methods] construct_iEDGE call signature: "dat <- construct_iEDGE(cn, gep, cisgenes, transgenes = NA, cn.fdat, gep.fdat, cn.pdat = NA, gep.pdat = NA)"
- [readme] iEDGE integrates paired samples: "gene-expression data from paired samples, and consists of 3 major modules"
- [readme] cis/trans DE module: "cis/trans differential expression analysis of epi-DNA and gene expression data"
- [abstract] iEDGE identifies cis and trans signatures per alteration: "iEDGE first identifies the cis and trans gene expression signatures associated with the presence/absence of a particular epi-DNA alteration across samples"
