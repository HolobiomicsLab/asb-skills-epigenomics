---
name: drug-response-biomarker-identification
description: Use when you need to identify transcriptional regulators independently
  associated with pharmacological drug response from cell line pharmacogenomic data
  (expression, CNV, mutation) plus ChIP-seq target information, as with CCLE/ENCODE
  and PLX4720.
when_to_use_negative:
- No ChIP-seq data are available for a TR, or the ChIP-seq data have too few targets,
  which would mislead target-based identification
- The question is secondary/acquired resistance arising post-treatment (e.g., acquired
  STAT3 activation in erlotinib resistance) — RePhine uses pre-treatment CCLE data
  and cannot detect it
- Reliable target information is unavailable for the TRs of interest, since RePhine
  identifies drug response-related TRs only through their targets
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3517
tools:
- name: RePhine
  role: 'R package implementing the regression-based pharmacogenomic and ChIP-seq
    integration: CNV-adjusted expression, partial correlations, uniP/multiP tests,
    and GSEA filtering'
  repo: https://github.com/coexps/RePhine
- name: R
  role: Runtime for the RePhine package, simulation_code.R, and drug_code_R.R; required
    packages glmnet, MASS, lmtest, ppcor, psych, ROCR
  repo: https://github.com/coexps/RePhine
- name: Python
  role: Runs RP_calculation_modified.py to compute regulatory potential scores from
    ChIP-seq peak bed files considering both distance and peak strength
  repo: https://github.com/coexps/RePhine
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:d7b05b2014b59ad548fa2e000f506964bdb198a66da5ff3ff523b645a654343f
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# drug-response-biomarker-identification

## Summary

Regression-based identification of drug response-related transcriptional regulators (TRs) by integrating CCLE pharmacogenomic data with ENCODE ChIP-seq-derived regulation potential (RP) scores, as implemented in the RePhine R package. It recovers resistance- and sensitivity-associated TRs (e.g., for BRAF inhibitor PLX4720) while adjusting confounders such as CNVs and kinase/tumor-suppressor mutations.

## When to use

Use when you need to identify transcriptional regulators independently associated with pharmacological drug response from cell line pharmacogenomic data (expression, CNV, mutation) plus ChIP-seq target information, as with CCLE/ENCODE and PLX4720.

## When NOT to use

- No ChIP-seq data are available for a TR, or the ChIP-seq data have too few targets, which would mislead target-based identification
- The question is secondary/acquired resistance arising post-treatment (e.g., acquired STAT3 activation in erlotinib resistance) — RePhine uses pre-treatment CCLE data and cannot detect it
- Reliable target information is unavailable for the TRs of interest, since RePhine identifies drug response-related TRs only through their targets

## Inputs

- CCLE gene expression matrix (cell lines x genes)
- CCLE pharmacological/drug sensitivity profiles (e.g., PLX4720 IC50/AUC)
- CCLE copy number variation data
- CCLE mutation data (kinase and tumor suppressor genes as confounders)
- Cancer type / lineage annotations for cell lines
- ENCODE ChIP-seq peak bed files (or pre-computed regulatory potential RP scores from the RePhine Google Drive resource)

## Outputs

- Ranked table of TRs per drug with RePhine coefficient, uniP, multiP, and permuP statistics (e.g., tr_ranking_plx4720.csv)
- Independent resistance-related (negative) and sensitivity-related (positive) TR lists at uniP < 1E-5 and multiP < 0.005
- RePhine-derived TR signatures usable for clustering drugs by mechanism of action
- Optional enrichment-pattern visualizations between TRs and drugs of interest (RePhine v1.1.0 feature)

## How to apply

Install RePhine (R package with glmnet, MASS, lmtest, ppcor, psych, ROCR) and obtain pre-computed RP scores for >1000 ENCODE ChIP-seq datasets or recompute them with RP_calculation_modified.py, which weighs both peak strength and peak-to-target distance. Adjust each gene's expression by regressing out copy number, then identify drug-response-related mutated genes via adaptive lasso with likelihood ratio test, and compute gene-drug partial correlations controlling for confounding factors. For each TR, evaluate uniP by likelihood ratio test over targets with concordant partial correlations, select independent TRs by elastic net with multiP, and filter top candidates with a modified GSEA step. Rank and threshold TRs (e.g., PLX4720 negatives at uniP < 1E-5 and multiP < 0.005: GTF3C2, YY1, ESR1, E2F1, MYC, GATA3, RBBP5, EZH2, E2F4, ZEB1, ZNF217; positives: SPI1, CEBPB, EP300), optionally adding permutation testing (permuP) and cross-checking against Figure 4A / Table S8.

## Related tools

- **RePhine** (R package implementing the regression-based pharmacogenomic and ChIP-seq integration: CNV-adjusted expression, partial correlations, uniP/multiP tests, and GSEA filtering) — https://github.com/coexps/RePhine
- **R** (Runtime for the RePhine package, simulation_code.R, and drug_code_R.R; required packages glmnet, MASS, lmtest, ppcor, psych, ROCR) — https://github.com/coexps/RePhine
- **Python** (Runs RP_calculation_modified.py to compute regulatory potential scores from ChIP-seq peak bed files considering both distance and peak strength) — https://github.com/coexps/RePhine

## Examples

```
Rscript drug_code_R.R  # run RePhine on CCLE pharmacogenomic + RP score inputs; see RePhine User Guide.pdf
```

## Evaluation signals

- Recovery of the reported PLX4720 negative TRs (GTF3C2, YY1, ESR1, E2F1, MYC, GATA3, RBBP5, EZH2, E2F4, ZEB1, ZNF217) and positive TRs (SPI1, CEBPB, EP300) at uniP < 1e-5 and multiP < 0.005, matching Figure 4A / Table S8
- All candidate TRs in the final ranking carry both uniP and multiP values, plus permuP from permutation testing
- Expression adjustment step shows copy-number effects removed via linear regression before correlation computation
- RePhine-derived TR signatures cluster drugs with different mechanisms of action when applied to CCLE data (paper's benchmark behavior)
- Simulated-data benchmark should outperform Pearson correlation, logistic regression, and GSEA baselines on data with noise/confounders

## Limitations

- Associations detected through targets may not be causal; upstream regulators or kinases may drive both drug response and downstream TR regulation, requiring TR knockout/activation experiments for causal validation
- Detection of acquired resistance is limited due to lack of post-treatment data in CCLE (e.g., secondary erlotinib resistance via acquired STAT3 activation could not be detected)
- Method is restricted to cases where reliable target information is available; lack of ChIP-seq data or too few targets misleads identification

## Evidence

- [abstract] we developed a regression-based pharmacogenomic and ChIP-seq data integration method (RePhine) to infer the impact of TRs on drug response: "we developed a regression-based pharmacogenomic and ChIP-seq data integration method (RePhine) to infer the impact of TRs on drug response"
- [readme] README method steps: "Estimate the adjusted expression for each gene by removing the effects of copy number through linear regression."
- [readme] README uniP/multiP definition: "Use likelihood ratio test to evaluate significance (uniP) of the TRs whose targets have concordant positive or negative partial correlations with drug response"
- [discussion] RP score calculation rationale: "application of quantitative and informative target inference by considering both ChIP-seq signals and the distances from peaks to the targets"
- [readme] Resource availability: [evidence span withheld — anchor status: external_doc]
