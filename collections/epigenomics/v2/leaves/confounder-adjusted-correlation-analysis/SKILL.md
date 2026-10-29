---
name: confounder-adjusted-correlation-analysis
description: Use when inferring whether transcriptional regulators (TRs) are associated
  with drug response from pharmacogenomic data (expression, CNV, mutation, pharmacology
  profiles) plus ChIP-seq-derived target information, and naive mRNA-level correlation
  would be confounded by copy number or mutations.
when_to_use_negative:
- No reliable ChIP-seq target information is available, or ChIP-seq data have too
  few targets — RePhine would mislead TR identification since it identifies TRs through
  their targets
- The goal is to detect secondary/acquired resistance arising from post-treatment
  events (e.g. acquired STAT3 activation under erlotinib), which RePhine cannot detect
  due to lack of post-treatment data in CCLE
- The question is causal mechanism rather than association — the detected TR–drug
  relationships may not be causal and require knockout/activation validation
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: RePhine
  role: R package implementing the confounder-adjusted partial correlation, uniP/multiP
    likelihood ratio testing, and modified GSEA filtering
  repo: https://github.com/coexps/RePhine
- name: Python
  role: Modified Python script (RP_calculation_modified.py) computes regulation potential
    scores from ChIP-seq peak bed files considering both distance and peak strength
  repo: https://github.com/coexps/RePhine
- name: R
  role: Runtime environment for the RePhine package (requires glmnet, MASS, lmtest,
    ppcor, psych, ROCR) and for running drug_code_R.R
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:8f1c2947e456646c81b5100f8ae875a859bc94318f5e2d367dacd079ac24f421
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - Python
  - R
  - RePhine
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# confounder-adjusted-correlation-analysis

## Summary

RePhine's regression-based strategy for measuring gene–drug response correlations while removing confounding effects (CNVs, kinase/tumor-suppressor mutations, cancer type), used to identify transcriptional regulators associated with drug response from CCLE pharmacogenomic and ENCODE ChIP-seq data.

## When to use

Use when inferring whether transcriptional regulators (TRs) are associated with drug response from pharmacogenomic data (expression, CNV, mutation, pharmacology profiles) plus ChIP-seq-derived target information, and naive mRNA-level correlation would be confounded by copy number or mutations.

## When NOT to use

- No reliable ChIP-seq target information is available, or ChIP-seq data have too few targets — RePhine would mislead TR identification since it identifies TRs through their targets
- The goal is to detect secondary/acquired resistance arising from post-treatment events (e.g. acquired STAT3 activation under erlotinib), which RePhine cannot detect due to lack of post-treatment data in CCLE
- The question is causal mechanism rather than association — the detected TR–drug relationships may not be causal and require knockout/activation validation

## Inputs

- CCLE gene expression matrix
- CCLE copy number variation data
- CCLE mutation status data
- Cancer type information per cell line
- CCLE drug sensitivity (pharmacological profiles), e.g. PLX4720 response data
- ENCODE ChIP-seq peak bed files for TRs (or precomputed RP scores)

## Outputs

- Per-TR RePhine coefficient with uniP, multiP, and permuP statistics (e.g. tr_ranking_plx4720.csv)
- Ranked list of negatively- and positively-correlated independent TRs (e.g. GTF3C2, YY1, ESR1, E2F1, MYC... as negative TRs for PLX4720)
- RP scores per cell line and ChIP-seq dataset

## How to apply

First remove copy number effects by estimating adjusted expression for each gene through linear regression, and identify drug-response-related mutated genes from CCLE mutation data via an adaptive lasso model and likelihood ratio test. Compute the partial correlation between each specific gene and the drug while controlling the confounding factors (including kinase/tumor-suppressor mutations and cancer type), then down-weight non-cancer-related genes. Evaluate significance (uniP) with a likelihood ratio test for TRs whose targets show concordant positive or negative partial correlations, and use elastic net combined with a likelihood ratio test (multiP) to select independent TRs; filter candidates with a modified GSEA. Targets are quantitatively inferred from ENCODE ChIP-seq by considering both peak signals and peak-to-target distances, captured as regulation potential (RP) scores computed with the modified Python script. For the PLX4720 CCLE application, significant resistance-related TRs pass uniP < 1E-5 and multiP < 0.005.

## Related tools

- **RePhine** (R package implementing the confounder-adjusted partial correlation, uniP/multiP likelihood ratio testing, and modified GSEA filtering) — https://github.com/coexps/RePhine
- **Python** (Modified Python script (RP_calculation_modified.py) computes regulation potential scores from ChIP-seq peak bed files considering both distance and peak strength) — https://github.com/coexps/RePhine
- **R** (Runtime environment for the RePhine package (requires glmnet, MASS, lmtest, ppcor, psych, ROCR) and for running drug_code_R.R)

## Examples

```
Rscript drug_code_R.R
```

## Evaluation signals

- PLX4720 application recovers GTF3C2, YY1, ESR1, E2F1, MYC, GATA3, RBBP5, EZH2, E2F4, ZEB1 and ZNF217 as negative (resistance-related) TRs at uniP < 1e-5 and multiP < 0.005, matching Figure 4A / Table S8
- SPI1, CEBPB, EP300 appear as positive TRs for PLX4720 at the same thresholds
- Significant TRs additionally pass permutation testing (permuP) implemented in the RePhine R package
- Adjusted expression no longer shows residual association with copy number after the linear regression removal step
- Modified GSEA filtering retains only top candidates whose target partial correlations show concordant sign patterns

## Limitations

- The relationship between TRs and drug response may not be causal; upstream regulators or kinases may influence both response and downstream TRs, so TR knockout/activation experiments combined with drug response examination are required
- Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead the identification of TRs
- Detection of acquired resistance is limited due to lack of post-treatment data in CCLE (e.g. secondary resistance to erlotinib through acquired STAT3 activation could not be detected)
- On simulated data with noises/confounders, performance advantages were benchmarked against Pearson correlation, logistic regression, and GSEA — residual noise can still affect identification

## Evidence

- [readme] Compute the partial correlation between a specific gene and a drug by controlling the confounding factors.: "Compute the partial correlation between a specific gene and a drug by controlling the confounding factors."
- [readme] Copy number adjustment step: "Estimate the adjusted expression for each gene by removing the effects of copy number through linear regression."
- [readme] uniP/multiP testing and elastic net selection: "Use likelihood ratio test to evaluate significance (uniP) of the TRs whose targets have concordant positive or negative partial correlations with drug response"
- [discussion] Rationale for confounder adjustment: "an accurate measurement of correlation patterns by adjusting all potential CFs that are not under the impact of TR regulation"
- [abstract] Method definition: "we developed a regression-based pharmacogenomic and ChIP-seq data integration method (RePhine) to infer the impact of TRs on drug response"
