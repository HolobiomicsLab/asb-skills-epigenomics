---
name: sex-dimorphism-pca-interpretation
description: Use when PCA has been computed on age-stratified PBMC ATAC-seq or RNA-seq
  samples and you must verify that PC1 scores correlate with donor age, match reported
  variance-explained, and differ between older men and older women.
when_to_use_negative:
- The question concerns cell-type-specific or single-cell resolution sex differences,
  which the authors explicitly defer to future studies
- Data are from a single sex or lack donor age/sex metadata, making the older-men-vs-older-women
  comparison impossible
- You need mechanistic explanation of inflammaging differences (sex hormones, DAMPs/PAMPs),
  which the paper leaves unexplored
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: R Shiny
  role: Searchable web application (https://immune-aging.jax.org/) sharing the PC1
    scores, PCA results, and aging/sex-difference results used for verification
  repo: github.com/UcarLab/SexDimorphismNatureCommunications
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:dca14e6d56a44dbdab015acfe9d88a553ea594336f085f80b61a2cfd0d25bd93
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sex-dimorphism-pca-interpretation

## Summary

Interpret PC1 of PCA on paired PBMC ATAC-seq/RNA-seq data as an aging gradient and test for a sex difference within older donors, as in Márquez et al. 2020 (Nature Communications). The skill reproduces and validates the claim that age drives shared epigenomic/transcriptomic variation and that men and women diverge after age 65.

## When to use

Use when PCA has been computed on age-stratified PBMC ATAC-seq or RNA-seq samples and you must verify that PC1 scores correlate with donor age, match reported variance-explained, and differ between older men and older women.

## When NOT to use

- The question concerns cell-type-specific or single-cell resolution sex differences, which the authors explicitly defer to future studies
- Data are from a single sex or lack donor age/sex metadata, making the older-men-vs-older-women comparison impossible
- You need mechanistic explanation of inflammaging differences (sex hormones, DAMPs/PAMPs), which the paper leaves unexplored

## Inputs

- Source Data table for Fig. 1b with per-individual PC1 scores from ATAC-seq PCA
- Source Data table for Fig. 1b with per-individual PC1 scores from RNA-seq PCA
- Donor metadata: age and sex for 172 healthy adults aged 22-93
- Reported variance-explained values (15% ATAC-seq, 18.6% RNA-seq) for comparison

## Outputs

- Pearson and Spearman correlation coefficients between PC1 and age per assay
- Variance-explained verification values vs. published 15% (ATAC-seq) and 18.6% (RNA-seq)
- Wilcoxon rank-sum statistics and p-values for older men vs. older women PC1 scores
- Verification table (e.g. fig1b_pc1_verification.csv) with pass/fail verdicts per check

## How to apply

Load the Source Data table for Fig. 1b containing per-individual PC1 scores from the ATAC-seq and RNA-seq PCA together with donor age and sex using pandas. Compute Pearson and Spearman correlations between PC1 score and age separately per assay, confirming PC1 increases with age (172 healthy adults aged 22-93). Check variance explained against the published values of ~15% for ATAC-seq and ~18.6% for RNA-seq. Then subset donors aged 65 and older and run a two-sided Wilcoxon rank-sum (Mann-Whitney U) test with scipy.stats on PC1 scores of older men versus older women, confirming men show significantly higher PC1, consistent with greater age-related changes in male PBMCs. Record correlation coefficients, variance-explained values, Wilcoxon statistics and p-values, and pass/fail verdicts (e.g. in fig1b_pc1_verification.csv); interpret significant PC1-associated loci via module shifts (declining naive T cell, increasing myeloid/inflammation modules).

## Related tools

- **R Shiny** (Searchable web application (https://immune-aging.jax.org/) sharing the PC1 scores, PCA results, and aging/sex-difference results used for verification) — github.com/UcarLab/SexDimorphismNatureCommunications

## Examples

```
from scipy.stats import spearmanr, mannwhitneyu; rho, p = spearmanr(df.PC1, df.age); u, pw = mannwhitneyu(df[(df.age>=65)&(df.sex=='M')].PC1, df[(df.age>=65)&(df.sex=='F')].PC1, alternative='two-sided')
```

## Evaluation signals

- Pearson/Spearman correlation between PC1 and age is positive and significant for both ATAC-seq and RNA-seq PC1 scores
- Variance explained by PC1 reproduces ~15% (ATAC-seq) and ~18.6% (RNA-seq) within a reasonable tolerance
- Two-sided Wilcoxon rank-sum test on PC1 scores of donors aged >= 65 yields significantly higher PC1 in older men than older women
- Verification CSV (fig1b_pc1_verification.csv) contains one row per check with statistic, p-value, and pass/fail verdict
- Directionality of module interpretation matches the paper: declines in T cell module and gains in myeloid/inflammation modules along PC1

## Limitations

- Sex-biased differences in older adults are not fully attributable to cell composition: CD14+ and CD16+ monocyte numbers/frequencies did not differ between older men and women, implying cell-intrinsic changes that require further study
- Bulk PBMC annotation is prone to biases in reference datasets, including data-quality differences and limited available cell types
- Sex-specific aging signatures may not be conserved across ethnicities/populations; this requires further investigation
- Sex differences in immune aging were not described at single-cell resolution or in sorted cells, and functional implications remain unestablished

## Evidence

- [results] PC1 captured 15% of variation in ATAC-seq and 18.6% in RNA-seq data, associated with age groups, with older men showing higher PC1 scores than older women: [evidence span withheld — anchor status: not_found]
- [abstract] Characterization of PBMCs from 172 adults: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry."
- [abstract] Shared aging signature of naive T cell decline and myeloid gain: "These data reveal a shared epigenomic signature of aging including declining naïve T cell and increasing monocyte and cytotoxic cell functions."
- [abstract] Genomic sex differences increase after age 65: "genomic differences between sexes increase after age 65, with men having higher innate and pro-inflammatory activity and lower adaptive activity."
- [intro] R Shiny app sharing results: "These results are shared via a searchable R Shiny application (https://immune-aging.jax.org/)."
- [discussion] Monocyte differences not explaining older-adult sex bias: "we did not detect significant differences between older men and older women in CD14+ and CD16+ monocyte cell numbers/frequencies"
- [readme] Repository contains Shiny code: "4. Shiny: Code used for R Shiny app."
