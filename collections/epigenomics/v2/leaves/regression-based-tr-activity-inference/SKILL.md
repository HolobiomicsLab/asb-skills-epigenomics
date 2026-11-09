---
name: regression-based-tr-activity-inference
description: Use when you have cell-line gene expression, copy number, mutation, and
  pharmacological sensitivity data plus ChIP-seq (or precomputed RP) targets, and
  want to infer which transcriptional regulators drive resistance or sensitivity to
  a specific drug.
when_to_use_negative:
- No ChIP-seq data exist for the TR of interest, or available ChIP-seq has too few
  targets — RePhine identifies TRs through targets and such data would mislead identification
- The question is detecting acquired/secondary resistance (e.g., erlotinib resistance
  via acquired STAT3 activation) — CCLE lacks post-treatment data, so pre-treatment-based
  RePhine cannot capture these mechanisms
- The goal is establishing causality rather than association — correlations detected
  through targets may not be causal and require TR knockout/activation experiments
  with drug response testing
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3301
tools:
- name: RePhine
  role: Core R package that computes partial correlations, uniP/multiP statistics,
    and permutation testing to identify drug response-related TRs
  repo: https://github.com/coexps/RePhine
- name: R
  role: Runtime environment for the RePhine package; requires packages glmnet, MASS,
    lmtest, ppcor, psych, ROCR
  repo: https://github.com/coexps/RePhine
- name: Python
  role: Runs RP_calculation_modified.py to compute regulatory potential scores from
    ChIP-seq peak BED files
  repo: https://github.com/coexps/RePhine
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:71821bdb9d2bd16ef93d94ae3bb2d8fb8aa238a20af8733772418270935191bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - Python
  - R
  - RePhine
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3301
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# regression-based-tr-activity-inference

## Summary

Applies RePhine, a regression-based integration of CCLE pharmacogenomic data and ENCODE ChIP-seq regulatory potential (RP) scores, to identify transcriptional regulators independently associated with response to a given drug (e.g., BRAF inhibitor PLX4720). It models concordant partial correlations of TR target sets while adjusting confounders such as CNVs and kinase/tumor-suppressor mutations.

## When to use

Use when you have cell-line gene expression, copy number, mutation, and pharmacological sensitivity data plus ChIP-seq (or precomputed RP) targets, and want to infer which transcriptional regulators drive resistance or sensitivity to a specific drug.

## When NOT to use

- No ChIP-seq data exist for the TR of interest, or available ChIP-seq has too few targets — RePhine identifies TRs through targets and such data would mislead identification
- The question is detecting acquired/secondary resistance (e.g., erlotinib resistance via acquired STAT3 activation) — CCLE lacks post-treatment data, so pre-treatment-based RePhine cannot capture these mechanisms
- The goal is establishing causality rather than association — correlations detected through targets may not be causal and require TR knockout/activation experiments with drug response testing

## Inputs

- CCLE gene expression matrix (cell lines x genes)
- CCLE drug sensitivity / pharmacological profiles (e.g., PLX4720 IC50 or AUC)
- CCLE copy number data
- CCLE mutation status (kinase genes, tumor suppressor genes, cancer type information)
- ChIP-seq peak BED files or precomputed regulatory potential (RP) scores from ENCODE (>1000 ChIP-seq datasets)
- Optional: CRISPR screening data processed by MaGeck (e.g., PLX4720 results)

## Outputs

- Ranked table of transcriptional regulators with RePhine coefficient, uniP, multiP, and permuP statistics (e.g., tr_ranking_plx4720.csv)
- Resistance- vs sensitivity-associated TR lists at uniP < 1e-5 and multiP < 0.005
- RePhine-derived TR signatures usable for drug clustering by mechanism of action

## How to apply

Download the RePhine R package (v1.1.0) and the precomputed RP scores covering >1000 ENCODE ChIP-seq datasets; compute RP scores per cell line with RP_calculation_modified.py, which weighs both peak strength and peak-to-target distance. First remove copy-number effects from expression via linear regression, then use an adaptive lasso plus likelihood ratio test to flag mutated genes related to drug response and treat them as confounding factors. For each TR, compute partial correlations between its target genes and the drug response controlling for those confounders, evaluate uniP via likelihood ratio test on targets with concordant partial correlations, and run elastic net plus likelihood ratio test to derive multiP for independent TRs. Rank and filter TRs using the thresholds validated against PLX4720 results (uniP < 1e-5, multiP < 0.005), optionally applying the modified GSEA step to further filter top candidates; a modified GSEA algorithm is also used to filter candidates.

## Related tools

- **RePhine** (Core R package that computes partial correlations, uniP/multiP statistics, and permutation testing to identify drug response-related TRs) — https://github.com/coexps/RePhine
- **R** (Runtime environment for the RePhine package; requires packages glmnet, MASS, lmtest, ppcor, psych, ROCR) — https://github.com/coexps/RePhine
- **Python** (Runs RP_calculation_modified.py to compute regulatory potential scores from ChIP-seq peak BED files) — https://github.com/coexps/RePhine

## Examples

```
install.packages('Rephine_1.1.0.tar.gz', repos=NULL, type='source'); library(RePhine)  # then run drug_code_R.R per the RePhine User Guide
```

## Evaluation signals

- PLX4720 analysis recovers GTF3C2, YY1, ESR1, E2F1, MYC, GATA3, RBBP5, EZH2, E2F4, ZEB1, ZNF217 as negatively-correlated (resistance) independent TRs and SPI1, CEBPB, EP300 as positive TRs at uniP < 1e-5 and multiP < 0.005, matching Figure 4A / Table S8
- Ranking table contains the four statistics (RePhine coefficient, uniP, multiP, permuP) for every TR and is sorted by significance
- Expression adjustment step removes copy-number effects before any partial correlation is computed (linear regression step 1)
- Permutation testing (permuP) has been run in addition to uniP/multiP to confirm TR significance
- RePhine-derived TR signatures should cluster drugs sharing mechanisms of action when applied across CCLE compounds

## Limitations

- Identified TR-drug associations may not be causal; upstream regulators or kinases could influence drug response and simultaneously regulate downstream TRs
- Lack of ChIP-seq data or ChIP-seq with too few targets would mislead TR identification
- Detection of acquired resistance is limited because CCLE lacks post-treatment data; secondary resistance to erlotinib via acquired STAT3 activation could not be detected
- Causal claims require additional TR knockout/activation experiments combined with drug response examination

## Evidence

- [abstract] we developed a regression-based pharmacogenomic and ChIP-seq data integration method (RePhine) to infer the impact of TRs on drug response: "we developed a regression-based pharmacogenomic and ChIP-seq data integration method (RePhine) to infer the impact of TRs on drug response"
- [readme] README method steps: "Use likelihood ratio test to evaluate significance (uniP) of the TRs whose targets have concordant positive or negative partial correlations with drug response and apply the elastic net"
- [readme] README target inference: [evidence span withheld — anchor status: external_doc]
- [discussion] Workflow confounder adjustment: "an accurate measurement of correlation patterns by adjusting all potential CFs that are not under the impact of TR regulation"
- [discussion] Limitation: target availability: "because RePhine identifies drug response-related TRs through the targets, it is restricted to the cases where reliable target information is available"
- [discussion] Limitation: acquired resistance: "detection of acquired resistance is limited due to lack of post-treatment data in CCLE"
