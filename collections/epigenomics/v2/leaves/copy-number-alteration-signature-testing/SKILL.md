---
name: copy-number-alteration-signature-testing
description: Use when you have per-sample SCNA alteration calls (e.g. GISTIC2.0 focal
  peaks) and matched gene expression data (e.g. RSEM log2) and need to identify cis
  genes within each peak and trans genes genome-wide whose expression shifts with
  the alteration.
when_to_use_negative:
- When the goal is to rank cis genes by mediation of trans expression rather than
  to test for alteration-associated differential expression itself (use the mediation
  module instead)
- When no paired sample-level SCNA and expression data are available (e.g. only aggregate
  peak summaries without alteration status per sample)
- When alterations are broad arm-level events processed without focal-peak cis gene
  definitions and direction annotations
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: iEDGE
  role: R package providing construct_iEDGE and the iEDGE_DE differential expression
    engine invoked via run_iEDGE to test cis/trans signatures per SCNA
  repo: https://github.com/montilab/iEDGE
- name: limma
  role: Linear-model framework underlying iEDGE_DE's one-sided cis and two-sided trans
    differential expression tests
- name: Biobase
  role: Base infrastructure on which the iEDGE package depends for expression data
    containers
- name: GISTIC2.0
  role: Source of the focal SCNA peak calls that define the per-alteration cis gene
    lists and alteration status per sample
- name: R
  role: Runtime environment in which the iEDGE pipeline scripts are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:395b056e41674ebb6088a3da8fb6dafb981cd4734872cd8991b54864ec0d5bc4
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
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# copy-number-alteration-signature-testing

## Summary

Tests each focal somatic copy-number alteration (SCNA) for cis and trans differential gene expression between alteration-positive and alteration-negative tumor samples using the iEDGE package's limma-based differential expression engine, as applied to TCGA breast cancer GISTIC2.0 calls.

## When to use

Use when you have per-sample SCNA alteration calls (e.g. GISTIC2.0 focal peaks) and matched gene expression data (e.g. RSEM log2) and need to identify cis genes within each peak and trans genes genome-wide whose expression shifts with the alteration.

## When NOT to use

- When the goal is to rank cis genes by mediation of trans expression rather than to test for alteration-associated differential expression itself (use the mediation module instead)
- When no paired sample-level SCNA and expression data are available (e.g. only aggregate peak summaries without alteration status per sample)
- When alterations are broad arm-level events processed without focal-peak cis gene definitions and direction annotations

## Inputs

- GISTIC2.0 focal SCNA calls (e.g. 29 amplifications and 40 deletions from TCGA BRCA) with direction annotations in cn.fdat
- RSEM log2 gene expression matrix from paired primary-tumor samples (e.g. 1050 TCGA BRCA samples) with subtype labels in gep.pdat
- Per-alteration cis gene lists (genes within each focal peak)

## Outputs

- Per-alteration cis differential expression tables with FDR-adjusted p-values and fold changes, filtered at FDR<0.25 and fold change>1.2 (res$de$cis$full)
- Per-alteration trans differential expression tables, filtered at FDR<0.01 and fold change>1.5 (res$de$trans$full)
- Tabular text files reproducing Supplemental Tables S1 (cis) and S2 (trans), e.g. 1330 significant cis genes (269 amplification, 1061 deletion)

## How to apply

Assemble GISTIC2.0 SCNA calls (direction-annotated via cn.fdat), the expression matrix, per-alteration cis gene lists, and sample annotations into an iEDGE object with construct_iEDGE. Run iEDGE_DE via run_iEDGE with cis thresholds fdr.cis.cutoff=0.25 and fc.cis=1.2 using one-sided UP/DOWN tests (uptest='Amplification', downtest='Deletion', onesided.cis=TRUE), and trans thresholds fdr.trans.cutoff=0.01 and fc.trans=1.5 with two-sided tests (onesided.trans=FALSE), requiring min.group=2 samples per group. The limma-based linear models compare alteration-positive versus alteration-negative samples for genes within the focal peak (cis) and all other genes (trans). Retrieve full and filtered statistics tables from res$de$cis$full and res$de$trans$full and export them as tabular files. Rationale: one-sided cis tests encode the expected direction of the dosage effect, while trans effects are tested in both directions.

## Related tools

- **iEDGE** (R package providing construct_iEDGE and the iEDGE_DE differential expression engine invoked via run_iEDGE to test cis/trans signatures per SCNA) — https://github.com/montilab/iEDGE
- **limma** (Linear-model framework underlying iEDGE_DE's one-sided cis and two-sided trans differential expression tests)
- **Biobase** (Base infrastructure on which the iEDGE package depends for expression data containers)
- **GISTIC2.0** (Source of the focal SCNA peak calls that define the per-alteration cis gene lists and alteration status per sample)
- **R** (Runtime environment in which the iEDGE pipeline scripts are executed)

## Examples

```
res <- run_iEDGE(dat = dat, header = "testrun", outdir = "."
```

## Evaluation signals

- Output tables contain FDR-adjusted p-values and fold changes for every gene per alteration
- Cis genes are called significant exactly at FDR<0.25 and fold change>1.2, trans genes at FDR<0.01 and fold change>1.5, matching the paper's thresholds
- Significant cis genes are direction-consistent with the cn.fdat annotation (amplifications UP-tested, deletions DOWN-tested)
- All 69 alterations (29 amplifications, 40 deletions) yield results with no empty or malformed tables
- Aggregate counts are reproducible against the paper's report: an average of 20 genes (median 8) per SCNA, 1330 total significant cis genes (269 in amplifications, 1061 in deletions)

## Limitations

- The same alteration-positive/negative comparison framework assumes adequate samples per group (min.group=2 guards, but small groups reduce power)
- Results depend on upstream GISTIC2.0 peak calling; genes outside the defined focal peaks and genes misassigned to peaks will distort cis/trans labeling
- Trans testing at fold change>1.5 and FDR<0.01 is two-sided and genome-wide, so it is more conservative and may miss weaker distal effects than the one-sided cis test

## Evidence

- [abstract] cis/trans signatures via iEDGE_DE: "iEDGE first identifies the cis and trans gene expression signatures associated with the presence/absence of a particular epi-DNA alteration across samples"
- [methods] construct_iEDGE call: "dat <- construct_iEDGE(cn, gep, cisgenes, transgenes = NA, cn.fdat, gep.fdat, cn.pdat = NA, gep.pdat = NA)"
- [methods] iEDGE_DE engine signature: "iEDGE_DE(cn, gep, cisgenes, transgenes, header, gepid, cnid, f.dir.out"
- [readme] README module list: "1. cis/trans differential expression analysis of epi-DNA and gene expression data"
