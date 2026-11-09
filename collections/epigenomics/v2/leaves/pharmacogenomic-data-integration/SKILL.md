---
name: pharmacogenomic-data-integration
description: Use when you have cell-line pharmacogenomic data (expression, CNV, mutation,
  drug sensitivity such as CCLE PLX4720 profiles) plus ChIP-seq or RP-score target
  maps and need to find drug-response TRs while adjusting confounders.
when_to_use_negative:
- When no ChIP-seq data or only ChIP-seq data with too few targets are available for
  the TRs of interest, which would mislead TR identification
- When reliable target information is unavailable, since RePhine identifies TRs through
  their targets
- When studying acquired/secondary resistance requiring post-treatment data, which
  CCLE lacks (e.g., erlotinib resistance via acquired STAT3 activation could not be
  detected)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3517
tools:
- name: RePhine
  role: 'R package implementing the full pipeline: confounder adjustment, partial
    correlation computation, uniP/multiP testing, and modified GSEA filtering'
  repo: https://github.com/coexps/RePhine
- name: R
  role: Runtime environment for the RePhine package; glmnet (Adaptive Lasso/elastic
    net), lmtest (likelihood-ratio tests), ppcor (partial correlation), MASS, psych,
    ROCR packages required
  repo: https://github.com/coexps/RePhine
- name: Python
  role: Runs RP_calculation_modified.py to compute regulatory potential scores from
    ChIP-seq peak BED files using both peak strength and distance
  repo: https://github.com/coexps/RePhine
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_006
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:5ea490b40b9a7c5dd1da47b0943ecb220a15ac6e06de8d4fddbd1d48416d24f2
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
yes_rate: 0.6666666666666666
n_claims: 9
---

# pharmacogenomic-data-integration

## Summary

Integrate CCLE pharmacogenomic data (gene expression, copy number, mutation status, pharmacological profiles) with ENCODE ChIP-seq–derived regulatory potential (RP) scores via the RePhine regression framework to identify transcriptional regulators (TRs) associated with drug response.

## When to use

Use when you have cell-line pharmacogenomic data (expression, CNV, mutation, drug sensitivity such as CCLE PLX4720 profiles) plus ChIP-seq or RP-score target maps and need to find drug-response TRs while adjusting confounders.

## When NOT to use

- When no ChIP-seq data or only ChIP-seq data with too few targets are available for the TRs of interest, which would mislead TR identification
- When reliable target information is unavailable, since RePhine identifies TRs through their targets
- When studying acquired/secondary resistance requiring post-treatment data, which CCLE lacks (e.g., erlotinib resistance via acquired STAT3 activation could not be detected)

## Inputs

- CCLE DepMap gene expression matrix
- CCLE copy-number segments
- CCLE mutation calls (including kinase/tumor-suppressor and H3K27me3/PRC2 gene alterations)
- Drug pharmacology / sensitivity profiles (e.g., PLX4720 response values)
- ENCODE ChIP-seq peak BED files or precomputed regulatory potential (RP) scores for ~1000 ENCODE ChIP-seq datasets
- TCGA differentially expressed gene sets

## Outputs

- Per-gene partial-correlation coefficient table (e.g., PLX4720_partial_correlations.csv)
- Selected mutation-confounder gene list (e.g., PLX4720_confounder_genes.txt)
- TR ranking table with RePhine coefficient, uniP, multiP, and permuP (e.g., tr_ranking_plx4720.csv)
- 160-TR by N-drug -Log10(uniP) response signature matrix for drug clustering

## How to apply

First, estimate CNV-adjusted expression for each gene by regressing expression on copy number with a linear model and retaining residuals, removing CNV-driven expression variation. Second, identify drug-response mutation confounders from CCLE mutation data and pharmacology profiles using an Adaptive Lasso (glmnet) with likelihood-ratio tests. Third, compute partial correlations (R ppcor package) between each gene's CNV-adjusted expression and the drug response (e.g., PLX4720), controlling for the selected mutation confounders, repeating per gene. Fourth, down-weight non-cancer-related genes by scoring against TCGA differentially expressed gene sets (limma). Fifth, evaluate TRs whose targets show concordant positive or negative partial correlations using likelihood-ratio tests for uniP, and elastic net combined with likelihood-ratio tests for multiP; apply a modified GSEA to filter top candidates. RP scores must be computed with the modified Python script (RP_calculation_modified.py), which considers both ChIP-seq peak strength and peak-to-target distance. Significance thresholds used in the CCLE PLX4720 application are uniP < 1E-5 and multiP < 0.005, supplemented by permutation testing (permuP).

## Related tools

- **RePhine** (R package implementing the full pipeline: confounder adjustment, partial correlation computation, uniP/multiP testing, and modified GSEA filtering) — https://github.com/coexps/RePhine
- **R** (Runtime environment for the RePhine package; glmnet (Adaptive Lasso/elastic net), lmtest (likelihood-ratio tests), ppcor (partial correlation), MASS, psych, ROCR packages required) — https://github.com/coexps/RePhine
- **Python** (Runs RP_calculation_modified.py to compute regulatory potential scores from ChIP-seq peak BED files using both peak strength and distance) — https://github.com/coexps/RePhine

## Examples

```
Rscript -e 'install.packages("Rephine_1.1.0.tar.gz", repos=NULL, type="source")'; python RP_calculation_modified.py --peaks peaks.bed
```

## Evaluation signals

- RePhine run on CCLE/ENCODE PLX4720 data should recover GTF3C2, YY1, ESR1, E2F1, MYC, GATA3, RBBP5, EZH2, E2F4, ZEB1, ZNF217 as negatively correlated TRs and SPI1, CEBPB, EP300 as positive TRs at uniP < 1e-5 and multiP < 0.005, matching Figure 4A / Table S8
- Hierarchical clustering (ward.D / complete linkage in R hclust) of the -Log10(uniP) signature matrix must reproduce Figure 3A: MEK inhibitors PD-0325901 and AZD6244, RAF inhibitors PLX4720 and RAF265, and MET inhibitor PHA-665752 co-cluster in cluster 2
- On simulated data with noises/confounders (simulation_code.R), RePhine should outperform Pearson correlation, logistic regression, and GSEA
- The partial-correlation step must regress out copy number residuals and selected mutation confounders; the double-mutant (BRAF + H3K27me3/PRC2, e.g. EZH2) validation t-test should yield P = 0.044 with lower PLX4720 response in the double-mutant group
- RP scores must be computed accounting for both ChIP-seq peak signal and peak-to-target distance, not from raw peak BED files alone

## Limitations

- Correlations detected through targets may not be causal; upstream regulators or kinases influencing both drug response and downstream TRs may confound results
- Restricted to cases where reliable target information is available; lack of ChIP-seq data or ChIP-seq with too few targets misleads identification
- Lack of post-treatment data in CCLE limits detection of acquired resistance and integration of drug perturbation effects on genes (e.g., secondary erlotinib resistance via acquired STAT3 activation not detectable)

## Evidence

- [readme] Estimate the adjusted expression for each gene by removing the effects of copy number through linear regression.: "Estimate the adjusted expression for each gene by removing the effects of copy number through linear regression."
- [intro] RePhine takes pharmacogenomic and ChIP-seq data as input and identifies TRs associated with response to pharmacological drug therapy.: "we developed the regression-based pharmacogenomic and ChIP-seq data integration method (RePhine) which takes pharmacogenomic and ChIP-seq data as input and identifies TRs associated with response to"
- [readme] Compute the partial correlation between a specific gene and a drug by controlling the confounding factors.: "Compute the partial correlation between a specific gene and a drug by controlling the confounding factors."
- [readme] Use likelihood ratio test to evaluate significance (uniP) of the TRs whose targets have concordant positive or negative partial correlations with drug response and apply the elastic net analysis combined with likelihood ratio test (multiP).: "Use likelihood ratio test to evaluate significance (uniP) of the TRs whose targets have concordant positive or negative partial correlations with drug response and apply the elastic net analysis"
- [discussion] It is restricted to the cases where reliable target information is available.: "because RePhine identifies drug response-related TRs through the targets, it is restricted to the cases where reliable target information is available"
- [abstract] RePhine demonstrated an improved performance in comparison with three commonly used methods (including Pearson correlation analysis, logistic regression model, and gene set enrichment analysis).: "RePhine demonstrated an improved performance in comparison with three commonly used methods (including Pearson correlation analysis, logistic regression model, and gene set enrichment analysis)"
