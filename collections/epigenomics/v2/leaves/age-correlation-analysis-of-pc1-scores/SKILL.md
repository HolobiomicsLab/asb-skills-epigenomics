---
name: age-correlation-analysis-of-pc1-scores
description: Use when donor-level PCA coordinates (e.g., PC1 scores) from ATAC-seq
  or RNA-seq are available together with donor age and sex metadata, and the goal
  is to confirm that the top PC axis tracks aging and sex-dimorphism.
when_to_use_negative:
- Input data lacks donor age or sex metadata, so the age correlation and older-cohort
  sex contrast cannot be computed
- The analysis targets single-cell resolution or sorted-cell data, which the paper
  explicitly defers to future studies
- Only flow cytometry cell-frequency data are available without genome-wide ATAC-seq/RNA-seq
  PCA coordinates
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3173
tools:
- name: R Shiny
  role: Searchable application hosting the PBMC aging/sex-dimorphism results that
    this PC1 verification skill re-derives from the Source Data tables
  repo: https://immune-aging.jax.org/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:8220482a3a1719d1676978b31ea9eba024b8aa71a75f1cd0e16c366699438312
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# age-correlation-analysis-of-pc1-scores

## Summary

Verifies that PC1 from PCA of PBMC ATAC-seq and RNA-seq data captures age-driven variation by correlating per-donor PC1 scores with age, checking variance explained (~15% ATAC-seq, ~18.6% RNA-seq), and testing whether older men show higher PC1 than older women via a Wilcoxon rank-sum test.

## When to use

Use when donor-level PCA coordinates (e.g., PC1 scores) from ATAC-seq or RNA-seq are available together with donor age and sex metadata, and the goal is to confirm that the top PC axis tracks aging and sex-dimorphism.

## When NOT to use

- Input data lacks donor age or sex metadata, so the age correlation and older-cohort sex contrast cannot be computed
- The analysis targets single-cell resolution or sorted-cell data, which the paper explicitly defers to future studies
- Only flow cytometry cell-frequency data are available without genome-wide ATAC-seq/RNA-seq PCA coordinates

## Inputs

- Per-individual PC1 scores from ATAC-seq PCA (Source Data table for Fig. 1b)
- Per-individual PC1 scores from RNA-seq PCA (Source Data table for Fig. 1b)
- Donor metadata: age (22-93 years) and sex for 172 healthy adults

## Outputs

- Pearson/Spearman correlation coefficients between PC1 and age for ATAC-seq and RNA-seq
- Variance-explained values for PC1 per assay compared against ~15% (ATAC-seq) and ~18.6% (RNA-seq)
- Wilcoxon rank-sum statistic and p-value for PC1 comparison of older men (>=65) vs older women (>=65)
- fig1b_pc1_verification.csv with per-check pass/fail verdicts

## How to apply

Load the Source Data table for Fig. 1b, which contains per-individual PC1 scores from the ATAC-seq and RNA-seq PCA along with donor age and sex, using pandas. Compute Pearson/Spearman correlations between PC1 score and age separately for each assay and confirm PC1 increases with age in both ATAC-seq (PC1 ~15% variance) and RNA-seq (PC1 ~18.6% variance). Check the computed variance explained against the published ~15% (ATAC-seq) and ~18.6% (RNA-seq) values. Then subset donors aged 65 and older and run a two-sided Wilcoxon rank-sum (Mann-Whitney U) test on PC1 scores of older men versus older women with scipy.stats, expecting men to show significantly higher PC1. Record correlation coefficients, variance-explained values, Wilcoxon statistics and p-values, and pass/fail verdicts (e.g., into fig1b_pc1_verification.csv).

## Related tools

- **R Shiny** (Searchable application hosting the PBMC aging/sex-dimorphism results that this PC1 verification skill re-derives from the Source Data tables) — https://immune-aging.jax.org/

## Examples

```
from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(pc1_men65, pc1_women65, alternative='two-sided')
```

## Evaluation signals

- Pearson/Spearman correlation between PC1 and age is positive for both ATAC-seq and RNA-seq PC1 scores
- PC1 variance explained reproduces published values: ~15% for ATAC-seq and ~18.6% for RNA-seq
- Two-sided Mann-Whitney U test on PC1 of donors aged >=65 yields significantly higher PC1 in older men than older women (p < 0.05)
- Verification table fig1b_pc1_verification.csv contains entries for every check (correlations, variance explained, Wilcoxon statistics, p-values, verdicts) for both assays
- Donor counts match the published cohort of 172 healthy adults aged 22-93

## Limitations

- Bulk PBMC annotation approach is prone to biases in references, e.g., differences in data quality and limitation to cell types available in references
- Not all sex-biased differences in older adults are attributable to cell composition: monocyte cell numbers/frequencies did not differ between older men and women, implying cell-intrinsic changes
- Sex-specific aging signatures may not be conserved across ethnicities/populations, which requires further investigation

## Evidence

- [results] PC1 captured 15% of variation in ATAC-seq and 18.6% in RNA-seq data, associated with age groups, with older men showing higher PC1 scores than older women: [evidence span withheld — anchor status: not_found]
- [abstract] we characterize peripheral blood mononuclear cells from 172 healthy adults 22-93 years of age using ATAC-seq, RNA-seq, and flow cytometry: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry."
- [abstract] genomic differences between sexes increase after age 65, with men having higher innate and pro-inflammatory activity and lower adaptive activity: "genomic differences between sexes increase after age 65, with men having higher innate and pro-inflammatory activity and lower adaptive activity."
- [readme] The code can be found under seperate folders: "The code can be found under seperate folders: 
1. FlowData: Code used for flow cytometry data analyses"
- [intro] These results are shared via a searchable R Shiny application: "These results are shared via a searchable R Shiny application (https://immune-aging.jax.org/)."
