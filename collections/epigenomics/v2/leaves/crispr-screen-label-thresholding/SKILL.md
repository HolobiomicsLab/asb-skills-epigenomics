---
name: crispr-screen-label-thresholding
description: Use when sgRNA screening datasets provide continuous phenotype scores
  or activity scores that must be converted to binary high-activity labels for classification
  model training or independent validation.
when_to_use_negative:
- The downstream task is regression of sgRNA activity (use Spearman correlation on
  continuous scores instead of binarizing).
- The screening table already contains binary activity calls or the analysis goal
  does not require a classification label schema.
- The dataset is not a CRISPRi/CRISPRoff/CRISPRa gene silencing or activation screen,
  so the -0.1 phenotype / 0.75 activity-score cutoffs are not applicable.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Python 3.6
  role: Custom scripting to normalize phenotype scores and threshold them into binary
    high-activity labels
- name: scikit-learn
  role: Backend for downstream classification metrics (ROC-AUC) computed on the thresholded
    labels
- name: pandas
  role: Per-README dependency (1.2.4) used for loading and manipulating sgRNA screening
    tables during labeling
  repo: https://github.com/yangqianq/EpiCas-DL
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:74585d980d7914aaec42937f0e92aadf2262b94aca1bbf94b54575c8ac91fe3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - Python 3.6
  - pandas
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# crispr-screen-label-thresholding

## Summary

Binarizes continuous sgRNA screening measurements (phenotype scores or activity scores) into high-activity ('1') versus low-activity labels so CRISPRi/CRISPRoff/CRISPRa datasets can be used for classification-style sgRNA efficacy prediction (e.g., EpiCas-DL ROC-AUC evaluation).

## When to use

Use when sgRNA screening datasets provide continuous phenotype scores or activity scores that must be converted to binary high-activity labels for classification model training or independent validation.

## When NOT to use

- The downstream task is regression of sgRNA activity (use Spearman correlation on continuous scores instead of binarizing).
- The screening table already contains binary activity calls or the analysis goal does not require a classification label schema.
- The dataset is not a CRISPRi/CRISPRoff/CRISPRa gene silencing or activation screen, so the -0.1 phenotype / 0.75 activity-score cutoffs are not applicable.

## Inputs

- sgRNA-level screening tables with continuous phenotype scores (e.g., CRISPRoff_genome, CRISPRi_K562, hCRISPRi-v2, CRISPRi_genome)
- sgRNA-level tables with precomputed activity scores (CRISPRi_activityscore, CRISPRa_activityscore)
- per-gene top-3 sgRNA average scores for phenotype-score normalization

## Outputs

- Binary high-activity/low-activity label column ('1' = highly active) per sgRNA
- Labeled dataset matrices (11 x 40 input rows) ready for ROC-AUC classification evaluation (independent_validation_metrics.csv)

## How to apply

First normalize raw phenotype scores into activity scores by dividing each sgRNA's phenotype score by the absolute average score of the top 3 sgRNAs for the same gene. For gene-silencing datasets (CRISPRi_activityscore, CRISPRi_K562, CRISPRoff_genome, hCRISPRi-v2, CRISPRi_genome), label an sgRNA as highly active ('1') when its phenotype score is below -0.1. For activity-score datasets such as CRISPRa_activityscore (2,779 sgRNAs on 236 genes), apply the reported criterion of activity scores above 0.75 as highly active. Store the resulting binary labels alongside the sgRNA sequence and epigenetic features (nucleosome positioning, chromatin accessibility, DNA methylation, RNA expression) so they can feed the EpiCas-DL classification schema and ROC-AUC computation.

## Related tools

- **Python 3.6** (Custom scripting to normalize phenotype scores and threshold them into binary high-activity labels)
- **scikit-learn** (Backend for downstream classification metrics (ROC-AUC) computed on the thresholded labels)
- **pandas** (Per-README dependency (1.2.4) used for loading and manipulating sgRNA screening tables during labeling) — https://github.com/yangqianq/EpiCas-DL

## Examples

```
df['label'] = ((df['phenotype_score'] < -0.1) if assay=='silencing' else (df['activity_score'] > 0.75)).astype(int)
```

## Evaluation signals

- Label column contains only {0, 1} values, with '1' assigned exactly where phenotype score < -0.1 (silencing datasets) or activity score > 0.75 (activityscore datasets).
- Labeled CRISPRa_activityscore set spans 2,779 sgRNAs on 236 genes, matching the reported dataset size.
- Positive-label fraction is a plausible minority of sgRNAs (highly active sgRNAs are rare in genome-wide screens).
- Resulting ROC-AUC computed on these labels falls within [0, 1] and reproduces the reported independent-validation performance (overall ROC-AUC of 0.746).
- Activity-score normalization is reproducible: each sgRNA's score equals its phenotype score divided by the absolute average of the top 3 sgRNAs for the same gene.

## Limitations

- Gene silencing and activation screening datasets may contain noise in the process of measuring sgRNA knock-down or knock-up efficacy, so thresholded labels inherit that noise.
- The -0.1 phenotype-score and 0.75 activity-score cutoffs were defined for specific reported screens; applying them to other screening assays may mislabel sgRNAs.
- Differences between CRISPRoff and CRISPRi systems might affect label comparability across datasets from different cell lines.

## Evidence

- [methods] normalizing the phenotype score of each sgRNA with the absolute average score of the top 3 sgRNAs for each gene: "normalizing the phenotype score of each sgRNA with the absolute average score of the top 3 sgRNAs for each gene"
- [other] activity scores above 0.75 for the activityscore datasets: [evidence span withheld — anchor status: not_found]
- [methods] CRISPRa_activescore contains 2779 sgRNAs on 236 genes: "CRISPRa_activescore contains 2779 sgRNAs on 236 genes"
- [discussion] may contain noises in the process of measuring the sgRNA knock-down or knock-up efficacy: "The gene silencing and activation screening datasets may contain noises in the process of measuring the sgRNA knock-down or knock-up efficacy"
