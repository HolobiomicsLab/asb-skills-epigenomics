---
name: fdr-multiple-testing-correction
description: Use when many parallel hypothesis tests are performed (e.g., limma DE
  tests across 69 SCNAs or hypergeometric enrichment of gene sets) and per-test p-values
  must be adjusted to FDR before calling significant genes or pathways.
when_to_use_negative:
- When only a small number of pre-specified hypotheses are tested (no genome-wide
  multiplicity), where uncorrected or Bonferroni-style control may be more appropriate
- When raw p-values are desired for downstream combination (e.g., feeding into meta-analysis
  or custom correction), since iEDGE tables report FDR-adjusted values
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: iEDGE
  role: R package whose run_iEDGE/iEDGE_DE functions compute FDR-adjusted p-values
    for cis/trans DE and whose run_hyperEnrichment applies multiple-testing correction
    to hypergeometric pathway tests
  repo: https://github.com/montilab/iEDGE
- name: limma
  role: Linear-model differential expression engine imported by iEDGE whose per-gene
    p-values are the input to FDR adjustment
- name: R
  role: Runtime environment in which the iEDGE FDR correction and threshold filtering
    are executed
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:55e9c43eec8ad3a298ea06bd0c41cfdbcb0c25dd0fb74a9d5e704bd8bdff86c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - Biobase
  - R
  - iEDGE
  - limma
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# FDR multiple-testing correction

## Summary

Applies false discovery rate correction to control the expected proportion of false positives when testing thousands of genes or gene sets, as used in the iEDGE pipeline for cis/trans differential expression (limma) and hypergeometric pathway enrichment in TCGA cancer data.

## When to use

Use when many parallel hypothesis tests are performed (e.g., limma DE tests across 69 SCNAs or hypergeometric enrichment of gene sets) and per-test p-values must be adjusted to FDR before calling significant genes or pathways.

## When NOT to use

- When only a small number of pre-specified hypotheses are tested (no genome-wide multiplicity), where uncorrected or Bonferroni-style control may be more appropriate
- When raw p-values are desired for downstream combination (e.g., feeding into meta-analysis or custom correction), since iEDGE tables report FDR-adjusted values

## Inputs

- Raw p-values from limma-based differential expression tests per gene per alteration (cis and trans)
- Raw p-values from hypergeometric over-rerepresentation tests over MSigDB gene sets (h.all.v5.0.symbols.gmt, c2.cp.v5.0.symbols.gmt, c3.tft.v5.0.symbols.gmt)
- Threshold parameters: fdr.cis.cutoff=0.25, fdr.trans.cutoff=0.01, hyperthres=0.25

## Outputs

- Differential expression tables with FDR-adjusted p-values and fold changes per gene per alteration (Supplemental Tables S1 and S2 equivalents)
- Filtered significant cis gene sets (FDR<0.25, |FC|>1.2) and trans gene sets (FDR<0.01, |FC|>1.5)
- Significant pathway enrichment results table (e.g., enrichment_results.tsv) with corrected enrichment p-values

## How to apply

In iEDGE, FDR-adjusted p-values are computed by the differential expression engine (iEDGE_DE, which wraps limma) rather than applied post hoc: cis genes are called significant at FDR < 0.25 with fold change > 1.2 (one-sided UP/DOWN tests for amplification/deletion), while trans genes require FDR < 0.01 with fold change > 1.5 (two-sided tests). These thresholds are passed directly to run_iEDGE as fdr.cis.cutoff=0.25, fdr.trans.cutoff=0.01, fc.cis=1.2, fc.trans=1.5. For pathway enrichment, run_hyperEnrichment performs a hypergeometric over-representation test with multiple-testing correction (mht argument) and calls enriched pathways at hyperthres=0.25 with a minimum drawn set size of min.drawsize=3. The rationale is that genome-wide testing of ~20,000 genes per alteration yields enormous numbers of raw p-values, so significance must be judged on FDR-adjusted values reported in the output tables (res$de$cis$full, res$de$trans$full) alongside fold changes.

## Related tools

- **iEDGE** (R package whose run_iEDGE/iEDGE_DE functions compute FDR-adjusted p-values for cis/trans DE and whose run_hyperEnrichment applies multiple-testing correction to hypergeometric pathway tests) — https://github.com/montilab/iEDGE
- **limma** (Linear-model differential expression engine imported by iEDGE whose per-gene p-values are the input to FDR adjustment)
- **R** (Runtime environment in which the iEDGE FDR correction and threshold filtering are executed)

## Examples

```
res <- run_iEDGE(dat = dat, header = "testrun", outdir = ".", fdr.cis.cutoff = 0.25, fdr.trans.cutoff = 0.01, fc.cis = 1.2, fc.trans = 1.5)
```

## Evaluation signals

- Cis DE output tables contain FDR-adjusted p-values and fold changes, with significance called at FDR<0.25 and FC>1.2
- Trans DE output tables show significance at FDR<0.01 and FC>1.5
- Reproduced BRCA cis counts match reported scale: average 20 genes (median 8) per SCNA, 1330 total significant cis genes (269 amplifications, 1061 deletions)
- Pathway enrichment table lists significant gene sets only after multiple-testing correction at hyperthres=0.25 with min.drawsize=3
- No significant gene/pathway is reported that fails its assigned FDR and fold-change threshold in the exported tables

## Limitations

- Thresholds are dataset- and goal-specific: the lenient cis FDR of 0.25 (vs strict trans FDR of 0.01) reflects the cis-discovery vs trans-validation tradeoff and should not be transplanted blindly
- Sobel mediation validity depends on adequate sample size and strong correlation effects, conditions not met in all datasets, which affects downstream FDR-filtered driver rankings

## Evidence

- [abstract] iEDGE first identifies the cis and trans gene expression signatures associated with the presence/absence of a particular epi-DNA alteration across samples: "iEDGE first identifies the cis and trans gene expression signatures associated with the presence/absence of a particular epi-DNA alteration across samples"
- [methods] fdr.cis.cutoff=0.25, fdr.trans.cutoff=0.01: [evidence span withheld — anchor status: not_found]
- [methods] run_hyperEnrichment signature with mht argument: "run_hyperEnrichment(drawn, categories, ntotal, min.drawsize, mht, verbose, order, hypercol, hyperthres)"
- [discussion] uses differential expression analysis to determine the cis and trans genes: "uses differential expression analysis to determine the cis and trans genes that are associated with the presence/absence of a particular epi-DNA alteration across samples"
- [abstract] cis and trans effects are annotated by pathway enrichment analysis: "cis and trans effects are annotated by pathway enrichment analysis to gain insights into the underlying regulatory networks"
