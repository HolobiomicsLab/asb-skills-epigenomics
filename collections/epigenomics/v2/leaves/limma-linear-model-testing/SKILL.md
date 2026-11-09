---
name: limma-linear-model-testing
description: Use when paired copy-number/alteration calls (e.g., GISTIC2.0 focal SCNA
  peaks) and a log2 gene-expression matrix over the same tumor samples must be tested
  for cis/trans differential expression with limma linear models.
when_to_use_negative:
- Samples are unpaired or alteration status and expression profiles do not share sample
  identifiers, so alteration-positive/negative groups cannot be formed
- The goal is mediation analysis of cis-to-trans effects rather than the upstream
  cis/trans differential-expression signature (use the iEDGE mediation/Sobel module
  instead)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_3308
tools:
- name: iEDGE
  role: R package whose iEDGE_DE module runs the limma-based cis/trans differential-expression
    linear models via run_iEDGE
  repo: https://github.com/montilab/iEDGE
- name: limma
  role: Imported dependency providing the linear-model differential-expression framework
    underlying cis/trans testing
  repo: https://github.com/montilab/iEDGE
- name: Biobase
  role: Dependency providing the ExpressionSet-style data containers for the cn/gep
    inputs
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Execution environment for the iEDGE pipeline and rmarkdown reporting
  repo: https://github.com/montilab/iEDGE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:800f8fd583277967171ddfe5d5e15a47f011a13f20fc7d347d38dbc1a72286ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - Biobase
  - R
  - iEDGE
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# limma-linear-model-testing

## Summary

Differential-expression testing of cis and trans genes against epi-DNA alteration status using limma-based linear models, as implemented in iEDGE's iEDGE_DE module (e.g., reproducing TCGA BRCA focal SCNA signatures in Supplemental Tables S1 and S2).

## When to use

Use when paired copy-number/alteration calls (e.g., GISTIC2.0 focal SCNA peaks) and a log2 gene-expression matrix over the same tumor samples must be tested for cis/trans differential expression with limma linear models.

## When NOT to use

- Samples are unpaired or alteration status and expression profiles do not share sample identifiers, so alteration-positive/negative groups cannot be formed
- The goal is mediation analysis of cis-to-trans effects rather than the upstream cis/trans differential-expression signature (use the iEDGE mediation/Sobel module instead)

## Inputs

- GISTIC2.0 focal SCNA calls (e.g., 29 amplifications and 40 deletions from TCGA BRCA) with direction annotations in cn.fdat
- RSEM log2 gene-expression matrix over paired primary-tumor samples (e.g., 1050 TCGA BRCA samples with pam50/HER2 subtype labels in gep.pdat)
- Per-alteration cis gene lists (genes within each focal peak)
- iEDGE object built via construct_iEDGE

## Outputs

- Full cis differential-expression table per alteration (res$de$cis$full) with FDR-adjusted p-values and fold changes
- Full trans differential-expression table per alteration (res$de$trans$full)
- Filtered significant-gene tables at cis FDR<0.25/fc>1.2 and trans FDR<0.01/fc>1.5, exportable as Supplemental Table S1/S2-style tabular files

## How to apply

Assemble the alteration calls (e.g., Firehose GISTIC2.0 focal peaks with direction annotations in cn.fdat) and the RSEM log2 expression matrix into an iEDGE object with construct_iEDGE, supplying per-alteration cis gene lists. Run iEDGE_DE via run_iEDGE with limma-based linear models comparing alteration-positive versus alteration-negative samples: for cis genes use one-sided UP/DOWN tests (onesided.cis=TRUE, uptest='Amplification', downtest='Deletion') with FDR<0.25 and fold change>1.2; for trans genes use two-sided tests (onesided.trans=FALSE) with FDR<0.01 and fold change>1.5, enforcing a minimum group size min.group=2. Retrieve full statistics tables from res$de$cis$full and res$de$trans$full and export the filtered significant-gene tables as tabular files.

## Related tools

- **iEDGE** (R package whose iEDGE_DE module runs the limma-based cis/trans differential-expression linear models via run_iEDGE) — https://github.com/montilab/iEDGE
- **limma** (Imported dependency providing the linear-model differential-expression framework underlying cis/trans testing) — https://github.com/montilab/iEDGE
- **Biobase** (Dependency providing the ExpressionSet-style data containers for the cn/gep inputs) — https://github.com/montilab/iEDGE
- **R** (Execution environment for the iEDGE pipeline and rmarkdown reporting) — https://github.com/montilab/iEDGE

## Examples

```
res <- run_iEDGE(dat = dat, header = "testrun", outdir = ".", fdr.cis.cutoff = 0.25, fdr.trans.cutoff = 0.01, fc.cis = 1.2, fc.trans = 1.5, onesided.cis = TRUE, onesided.trans = FALSE, uptest = "Amplification", downtest = "Deletion", min.group = 2)
```

## Evaluation signals

- Output tables contain FDR-adjusted p-values and fold changes for every gene per alteration
- Cis genes are called significant only at FDR<0.25 with fold change>1.2; trans genes only at FDR<0.01 with fold change>1.5
- For TCGA BRCA focal SCNAs, totals match reported counts: 1330 significant cis genes (269 in amplifications, 1061 in deletions), average 20 (median 8) genes per SCNA over 69 alterations
- Directionality is respected: amplified regions are tested with one-sided UP tests and deleted regions with one-sided DOWN tests for cis genes
- Exported tables reproduce the content of Supplemental Tables S1 (cis) and S2 (trans)

## Limitations

- Sobel/mediation validity concerns aside, limma group comparisons require adequate samples on both sides of the alteration split; iEDGE enforces min.group=2 but small groups reduce power
- One-sided cis tests assume a known direction from the cn.fdat annotation; mislabeled amplification/deletion directions invalidate the significance calls
- Testing is restricted to genes within the focal peak for cis; broader driver effects outside peaks require the trans or mediation modules

## Evidence

- [methods] limma-based linear models with one-sided UP/DOWN tests for cis and two-sided tests for trans, as encoded in the direction column of cn.fdat: [evidence span withheld — anchor status: not_found]
- [methods] iEDGE_DE signature: "iEDGE_DE(cn, gep, cisgenes, transgenes, header, gepid, cnid, f.dir.out"
- [abstract] abstract cis/trans signature definition: "iEDGE first identifies the cis and trans gene expression signatures associated with the presence/absence of a particular epi-DNA alteration across samples"
- [discussion] discussion DE role: "uses differential expression analysis to determine the cis and trans genes that are associated with the presence/absence of a particular epi-DNA alteration across samples"
- [readme] README module list: "cis/trans differential expression analysis of epi-DNA and gene expression data"
