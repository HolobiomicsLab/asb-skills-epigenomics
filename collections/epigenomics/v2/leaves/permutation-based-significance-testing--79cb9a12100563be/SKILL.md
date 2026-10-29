---
name: permutation-based-significance-testing
description: Use when regressing CCLE pharmacogenomic drug response (e.g. PLX4720
  sensitivity) against regulation-potential (RP) scores of TR targets and you need
  empirical significance beyond parametric uniP/multiP likelihood-ratio tests.
when_to_use_negative:
- Lack of ChIP-seq data or ChIP-seq data with too few targets, which would mislead
  the identification of TRs
- Detection of acquired (secondary) resistance from CCLE, since lack of post-treatment
  data prevents integrating drug-perturbation effects on genes
- Cases where reliable TR target information is unavailable, since RePhine identifies
  drug response-related TRs through the targets
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: RePhine
  role: R package that runs the regression, likelihood-ratio (uniP/multiP) tests,
    permutation testing (permuP), and modified GSEA filtering on CCLE pharmacogenomic
    plus ChIP-seq data
  repo: https://github.com/coexps/RePhine
- name: R
  role: Runtime for the RePhine package (with glmnet, MASS, lmtest, ppcor, psych,
    ROCR) executing drug_code_R.R for significance testing
  repo: https://github.com/coexps/RePhine
- name: Python
  role: Runs RP_calculation_modified.py to compute regulatory potential scores from
    ChIP-seq peak bed files used as regression inputs
  repo: https://github.com/coexps/RePhine
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:f2b785f57f8644d7b4bd1f315e91295419a4b1ba9819e3f85313d4df76e35338
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# permutation-based-significance-testing

## Summary

Assess the statistical significance of transcriptional regulator (TR) associations with drug response in the RePhine framework, where each TR's regression yields a RePhine coefficient with uniP and multiP likelihood-ratio statistics that are further checked against a null distribution via permutation testing (permuP).

## When to use

Use when regressing CCLE pharmacogenomic drug response (e.g. PLX4720 sensitivity) against regulation-potential (RP) scores of TR targets and you need empirical significance beyond parametric uniP/multiP likelihood-ratio tests.

## When NOT to use

- Lack of ChIP-seq data or ChIP-seq data with too few targets, which would mislead the identification of TRs
- Detection of acquired (secondary) resistance from CCLE, since lack of post-treatment data prevents integrating drug-perturbation effects on genes
- Cases where reliable TR target information is unavailable, since RePhine identifies drug response-related TRs through the targets

## Inputs

- CCLE gene expression matrix and PLX4720 pharmacological (drug sensitivity) profiles
- CCLE copy number and mutation status data for confounder adjustment
- ENCODE ChIP-seq peak bed files (or precomputed regulatory potential scores from the RePhine Google Drive resource)
- TR-target assignment via RP scores computed by RP_calculation_modified.py

## Outputs

- Per-TR statistics table (RePhine coefficient, uniP, multiP, permuP), e.g. tr_ranking_plx4720.csv
- Ranked list of independent resistance-related (negative) and sensitivity-related (positive) TRs at uniP < 1E-5, multiP < 0.005
- Filtered top-candidate TR list after modified GSEA

## How to apply

First compute RP scores per cell line using RP_calculation_modified.py, which considers both ChIP-seq peak strength and peak-to-target distance. Run the RePhine regression in R (drug_code_R.R) for each TR, removing copy-number effects via linear regression, identifying mutated genes related to drug response with an adaptive lasso model and likelihood ratio test, and computing partial correlations controlling confounding factors (CNVs, kinase/tumor-suppressor mutations). Evaluate TR significance (uniP) with a likelihood ratio test on concordantly correlated targets, and select independent TRs (multiP) via elastic net combined with likelihood ratio testing. Then assess significance by permutation testing (permuP) as implemented in the RePhine R package, and apply the modified GSEA filter to top candidates. For PLX4720, call a TR significant at uniP < 1E-5 and multiP < 0.005.

## Related tools

- **RePhine** (R package that runs the regression, likelihood-ratio (uniP/multiP) tests, permutation testing (permuP), and modified GSEA filtering on CCLE pharmacogenomic plus ChIP-seq data) — https://github.com/coexps/RePhine
- **R** (Runtime for the RePhine package (with glmnet, MASS, lmtest, ppcor, psych, ROCR) executing drug_code_R.R for significance testing) — https://github.com/coexps/RePhine
- **Python** (Runs RP_calculation_modified.py to compute regulatory potential scores from ChIP-seq peak bed files used as regression inputs) — https://github.com/coexps/RePhine

## Examples

```
Rscript drug_code_R.R  # run RePhine on CCLE PLX4720 data to obtain RePhine coefficient, uniP, multiP and permuP per TR
```

## Evaluation signals

- Recovered negative (resistance-related) independent TRs GTF3C2, YY1, ESR1, E2F1, MYC, GATA3, RBBP5, EZH2, E2F4, ZEB1 and ZNF217 for PLX4720 at uniP < 1e-5 and multiP < 0.005
- Recovered positive (sensitivity-related) TRs SPI1, CEBPB, EP300 for PLX4720, matching Figure 4A / Table S8
- uniP and multiP values derive from likelihood ratio tests and elastic-net likelihood ratio testing, not from ordinary Pearson correlation of TR mRNA with drug response
- Output table contains all four columns (RePhine coefficient, uniP, multiP, permuP) for every tested TR
- Simulated benchmarks should show improved performance over Pearson correlation, logistic regression, and GSEA on data with noise/confounders

## Limitations

- The relationship between identified TRs and drug response may not be causal; upstream regulators or kinases influencing both response and downstream TRs can confound results
- Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead the identification of TRs
- Detection of acquired resistance is limited due to lack of post-treatment data in CCLE; secondary resistance to erlotinib via acquired STAT3 activation could not be detected

## Evidence

- [readme] README method step 6: parametric significance framework permuted against: "Use likelihood ratio test to evaluate significance (uniP) of the TRs whose targets have concordant positive or negative partial correlations"
- [readme] README method step 6b: multiP selection via elastic net and likelihood ratio test: "apply the elastic net analysis combined with likelihood ratio test (multiP) to select the independent TRs"
- [abstract] Abstract: benchmark advantage of the RePhine statistics over alternative tests: "RePhine demonstrated an improved performance in comparison with three commonly used methods (including Pearson correlation analysis, logistic regression model, and gene set"
- [discussion] Discussion: confounder adjustment rationale for the regression underlying the significance tests: "an accurate measurement of correlation patterns by adjusting all potential CFs that are not under the impact of TR regulation"
- [discussion] Discussion: permutation/filter step applied after candidate selection: [evidence span withheld — anchor status: external_doc]
