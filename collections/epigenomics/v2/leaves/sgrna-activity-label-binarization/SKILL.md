---
name: sgrna-activity-label-binarization
description: Use when a CRISPRoff/CRISPRi/CRISPRa sgRNA screening dataset provides
  continuous phenotype or activity scores per sgRNA and a classification task (e.g.,
  ROC-AUC evaluation) requires dichotomized activity labels.
when_to_use_negative:
- The downstream task is regression of continuous sgRNA activity/efficiency (e.g.,
  Spearman correlation evaluation) rather than binary classification
- The dataset already provides binary activity calls or the analysis specifies a different,
  dataset-specific activity score definition
- Applying a CRISPRoff-derived cutoff to CRISPRa/CRISPRi data without checking that
  the phenotype score scale and screen type are comparable
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: EpiCas-DL
  role: Consumes the binarized labels as classification targets for its CNN and for
    the ROC-AUC benchmark against baselines
  repo: https://github.com/yangqianq/EpiCas-DL
- name: Python 3.6
  role: Custom scripting environment for loading screening score tables and applying
    the -0.1 binarization cutoff
- name: pybedtools 0.9.0
  role: Genomic interval handling when mapping sgRNA coordinates and TSS-proximal
    sites prior to labeling
- name: scikit-learn 0.24.2
  role: Backend for baseline classifiers trained on the identically binarized labels
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:63875a2d89e22f8617a5a99087acabf317d3662589ad6c46e4fb45cefe04a2d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - GPyOpt
  - Python 3.6
  - pybedtools 0.9.0
  - scikit-learn
  - scikit-learn 0.24.2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sgrna-activity-label-binarization

## Summary

Converts continuous sgRNA screening phenotype scores (e.g., from CRISPRoff tiling screens) into binary 'highly active' vs. inactive labels using the published cutoff (phenotype score < -0.1 -> class '1'), enabling downstream classification models such as EpiCas-DL's ROC-AUC benchmark against DeepHF, C-RNNcrispr, Random Forest, and Gradient Boosting.

## When to use

Use when a CRISPRoff/CRISPRi/CRISPRa sgRNA screening dataset provides continuous phenotype or activity scores per sgRNA and a classification task (e.g., ROC-AUC evaluation) requires dichotomized activity labels.

## When NOT to use

- The downstream task is regression of continuous sgRNA activity/efficiency (e.g., Spearman correlation evaluation) rather than binary classification
- The dataset already provides binary activity calls or the analysis specifies a different, dataset-specific activity score definition
- Applying a CRISPRoff-derived cutoff to CRISPRa/CRISPRi data without checking that the phenotype score scale and screen type are comparable

## Inputs

- Per-sgRNA continuous phenotype scores from a CRISPRoff tiling screen (e.g., CRISPRoff_tiling: 111,682 sgRNAs on 520 genes)
- Published binarization cutoff (phenotype score threshold of -0.1)
- Optional per-gene normalization reference (absolute average score of top 3 sgRNAs per gene)

## Outputs

- Binary activity label vector (1 = highly active, 0 = inactive) aligned one-to-one with sgRNA rows
- Labeled sgRNA table ready for classification input assembly (11 x 40 sequence-plus-epigenetic matrix with labels)

## How to apply

Start from per-sgRNA phenotype scores in the screening dataset (e.g., CRISPRoff_tiling: 111,682 sgRNAs within +/- 2.5 kb of the TSS of 520 genes). Optionally normalize phenotype scores first, following the article's scheme of normalizing each sgRNA's score by the absolute average score of the top 3 sgRNAs for each gene. Apply the published binarization cutoff: label sgRNAs with phenotype scores lower than -0.1 as highly active (class '1'), all others as '0'. Carry these binary labels alongside the 11 x 40 input matrix (one-hot encoded 40 bp sequence plus min-max normalized epigenetic channels) into the 9:1 train/test split and 10-fold cross-validated ROC-AUC evaluation. Keep the cutoff fixed across all compared models so baselines (DeepHF, C-RNNcrispr, Random Forest, Gradient Boosting) and EpiCas-DL are evaluated on identical labels.

## Related tools

- **EpiCas-DL** (Consumes the binarized labels as classification targets for its CNN and for the ROC-AUC benchmark against baselines) — https://github.com/yangqianq/EpiCas-DL
- **Python 3.6** (Custom scripting environment for loading screening score tables and applying the -0.1 binarization cutoff)
- **pybedtools 0.9.0** (Genomic interval handling when mapping sgRNA coordinates and TSS-proximal sites prior to labeling)
- **scikit-learn 0.24.2** (Backend for baseline classifiers trained on the identically binarized labels)

## Examples

```
df['label'] = (df['phenotype_score'] < -0.1).astype(int)  # highly active sgRNAs = 1
```

## Evaluation signals

- All 111,682 CRISPRoff_tiling sgRNAs receive exactly one binary label; label count equals sgRNA count with no missing values
- Every sgRNA with phenotype score < -0.1 is labeled '1'; spot-check the boundary: scores exactly at or above -0.1 must be '0'
- Downstream classification on these labels reproduces baseline ROC-AUCs: DeepHF 0.751, C-RNNcrispr 0.790, Random Forest 0.767, Gradient Boosting 0.755, with EpiCas-DL at 0.879 under 10-fold cross-validation on the 9:1 split
- Both classes are non-degenerate (the highly active class is a minority but non-empty subset of the 520-gene tiling set)
- auc_comparison_table.csv contains ROC-AUC entries for all five models trained on the same labels

## Limitations

- Gene silencing and activation screening datasets may contain noise in measuring sgRNA knock-down or knock-up efficacy, so the binary labels inherit measurement noise
- The -0.1 cutoff is published for the CRISPRoff-style phenotype score scale; applying it to other score normalizations or screen types may mislabel sgRNAs
- Small CRISPRoff screening datasets mainly in HEK293T and K562 cell lines increase overfitting risk and limit generalization of models trained on these labels

## Evidence

- [other] Binarize sgRNA labels with the published cutoff: phenotype scores lower than -0.1 are labeled as highly active ('1'): [evidence span withheld — anchor status: not_found]
- [methods] Phenotype scores are normalized per gene using the top 3 sgRNAs before activity labeling: "normalizing the phenotype score of each sgRNA with the absolute average score of the top 3 sgRNAs for each gene"
- [results] The 90/10 split with ROC-AUC evaluation defines the classification setting the binarized labels serve: "We randomly divided the dataset into a training dataset (90%) and a testing dataset (10%) and calculated the area under the receiver operating characteristic (ROC) curve (AUC)"
- [discussion] Screening efficacy measurements may be noisy, affecting label reliability: [evidence span withheld — anchor status: not_found]
- [methods] The CRISPRoff_tiling dataset the binarization is applied to contains 111,682 sgRNAs on 520 genes: "CRISPRoff_tiling dataset contains 111,682 sgRNAs within ± 2.5 kb from the TSS of 520 genes"
