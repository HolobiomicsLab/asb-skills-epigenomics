---
name: bootstrap-resampling-reproducibility-assessment
description: Use when you need to test whether ranked cis driver predictions from
  an iEDGE-style mediation analysis (Rank-1 cis gene per SCNA) are stable under sample
  resampling rather than artifacts of a single cohort draw.
when_to_use_negative:
- The goal is primary driver discovery rather than assessing the stability of already-derived
  Rank-1 cis gene rankings
- The cohort is too small for the Sobel test to be valid — the paper notes the test
  requires adequate sample size and strong correlation effects, conditions not met
  in all datasets
- The use case assumes the number of mediated trans genes proxies driver importance
  — the authors caution this 'may not be an appropriate assumption for specific use
  cases'
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: iEDGE
  role: R package whose pipeline (differential expression + Sobel mediation) is rerun
    on each bootstrap resample to re-derive the Rank-1 cis gene per alteration
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Language in which the bootstrap resampling loop and fraction-of-inclusion
    computation are implemented
- name: Biobase
  role: Dependency of iEDGE supporting the expression-set data structures used in
    the reruns
- name: limma
  role: Imported by iEDGE for the cis/trans differential expression step executed
    within each resample
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:3d3c76ded57f58aab0fc941c310e4b8446ce8769b01e841eb6e302ea4757fcdc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - Biobase
  - R
  - iEDGE
  - limma
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bootstrap-resampling-reproducibility-assessment

## Summary

Quantifies reproducibility of iEDGE Rank-1 cis driver calls in TCGA BRCA by re-deriving the Sobel-mediated ranking across 100 bootstrap resamples and computing per-gene fraction-of-inclusion.

## When to use

Use when you need to test whether ranked cis driver predictions from an iEDGE-style mediation analysis (Rank-1 cis gene per SCNA) are stable under sample resampling rather than artifacts of a single cohort draw.

## When NOT to use

- The goal is primary driver discovery rather than assessing the stability of already-derived Rank-1 cis gene rankings
- The cohort is too small for the Sobel test to be valid — the paper notes the test requires adequate sample size and strong correlation effects, conditions not met in all datasets
- The use case assumes the number of mediated trans genes proxies driver importance — the authors caution this 'may not be an appropriate assumption for specific use cases'

## Inputs

- Preprocessed iEDGE object for TCGA BRCA (copy-number alteration matrix, gene expression matrix, cisgene annotations)
- Bootstrap resample specification: 100 resamples, sample size equal to original cohort, drawn with replacement
- iEDGE run parameters: cis FDR cutoff 0.25, trans FDR cutoff 0.01, Sobel-test mediation

## Outputs

- Per-gene fraction-of-inclusion table (bootstrap_reproducibility_fractions.csv)
- Fig. 5A-style distribution of fraction-of-inclusion values across Rank-1 cis genes
- Proportion of Rank-1 genes with inclusion fraction >0.75 (reported: 69.2%)

## How to apply

Load the preprocessed iEDGE object for TCGA BRCA containing the copy-number alteration matrix, gene expression matrix, and cisgene annotations, then generate 100 bootstrap resamples of samples at fixed resample size equal to the original cohort, drawn with replacement. For each resample, rerun the iEDGE pipeline — differential expression with cis FDR cutoff 0.25 and trans FDR cutoff 0.01, followed by Sobel-test mediation analysis — and record the Rank-1 cis gene per alteration, i.e. the cis gene mediating the largest number of trans genes. For each cis gene, compute the fraction of the 100 resamples in which it is recovered as Rank-1, save the per-gene table as bootstrap_reproducibility_fractions.csv, and reconstruct the Fig. 5A distribution of fraction-of-inclusion values. Accept the analysis if the proportion of Rank-1 genes with inclusion fraction >0.75 reproduces the reported 69.2% within Monte-Carlo sampling variability, and confirm this fraction is not biased by SCNA type (amplification vs deletion).

## Related tools

- **iEDGE** (R package whose pipeline (differential expression + Sobel mediation) is rerun on each bootstrap resample to re-derive the Rank-1 cis gene per alteration) — https://github.com/montilab/iEDGE
- **R** (Language in which the bootstrap resampling loop and fraction-of-inclusion computation are implemented)
- **Biobase** (Dependency of iEDGE supporting the expression-set data structures used in the reruns)
- **limma** (Imported by iEDGE for the cis/trans differential expression step executed within each resample)

## Examples

```
for (b in 1:100) { res <- run_iEDGE(dat = dat[b,], header = paste0("boot", b), outdir = ".") }
```

## Evaluation signals

- Computed proportion of Rank-1 genes with inclusion fraction >0.75 reproduces the reported 69.2% within Monte-Carlo sampling variability
- Per-gene fraction-of-inclusion table is saved as bootstrap_reproducibility_fractions.csv
- Distribution of inclusion fractions shows no bias by SCNA type (amplification vs deletion)
- Rank-1 definition is honored: the recorded cis gene per alteration mediates the largest number of trans genes in each resample
- Resampling design matches spec: 100 resamples, fixed size equal to the original cohort, drawn with replacement

## Limitations

- Sobel test validity depends on adequate sample size and strong correlation effects, conditions not met in all datasets
- Assumption that the number of mediated trans genes proxies driver importance may not hold for specific use cases
- Bootstrap reproducibility estimates carry Monte-Carlo sampling variability inherent to the 100-resample design

## Evidence

- [discussion] Rank-1 ranking rationale: "the mediation analysis, ranks the set of cis genes by the extent of their mediation of trans genes"
- [discussion] Sobel validation precedent: "simulations were performed to validate the performance of the Sobel test of mediation"
- [methods] iEDGE object construction entry point: "dat <- construct_iEDGE(cn, gep, cisgenes, transgenes = NA, cn.fdat, gep.fdat, cn.pdat = NA, gep.pdat = NA)"
- [readme] Package scope: "iEDGE is an R-package for performing integrative analysis of (epi-)DNA and gene expression data"
