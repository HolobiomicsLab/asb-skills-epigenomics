---
name: cross-validation-performance-evaluation
description: Use when benchmarking a DNA 6mA (or similar sequence-classification)
  predictor against an existing tool under matched 5-fold cross-validation on benchmark
  datasets such as cross-species, rice, or M. musculus.
when_to_use_negative:
- Evaluating generalization on held-out data — use the independent test set via csDMA_indepedent_testing.py
  instead of cross-validation
- The input is already a per-sample prediction score file rather than raw 41nt sequences
  needing Motif/Kmer/Binary encoding
- The baseline method is unavailable for direct comparison (e.g. i6mA-Pred, which
  was only known from its online abstract with jackknife ACC 0.831)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: csDMA
  role: ExtraTrees-based 6mA predictor whose model is evaluated by 5-fold cross-validation
  repo: https://github.com/liuze-nwafu/csDMA
- name: iDNA6mA-PseKNC
  role: Reimplemented SVM baseline (C=0.336, gamma=0.02) evaluated on the same feature
    matrices for comparison
- name: classifier_selection.py
  role: Trains RandomForest, GradientBoosting, AdaBoost, ExtraTrees, and SVM models
    and evaluates them by 5-fold cross-validation
  repo: github.com/NWAFU-LiuLab/csDMA
- name: feature_selection.py
  role: Implements the Motif + Kmer + Binary feature encoding scheme consumed by the
    cross-validation scripts
  repo: github.com/NWAFU-LiuLab/csDMA
- name: CD-HIT-EST
  role: Reduces sequence redundancy of the cross-species benchmark at threshold 0.80
    before evaluation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-019-49430-4
    title: 'csDMA: an improved bioinformatics tool for identifying DNA 6 mA modifications
      via Chou’s 5-step rule'
  source_content_hash: sha256:d7cd587a37edc330a4e71162b942cd4e10ed24db383b6e68e081228b6db0f9c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liuze-nwafu/csDMA
  tools:
  - CD-HIT-EST
  - classifier_selection.py
  - csDMA
  - feature_selection.py
  - iDNA6mA-PseKNC
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# cross-validation-performance-evaluation

## Summary

Reproduces the csDMA evaluation protocol: k-mer/Motif/Binary-encoded 41nt DNA 6mA sequence matrices are scored by 5-fold cross-validation with the ExtraTrees classifier and compared against the iDNA6mA-PseKNC SVM baseline, reporting AUC, ACC, Sn, Sp, and MCC per dataset.

## When to use

Use when benchmarking a DNA 6mA (or similar sequence-classification) predictor against an existing tool under matched 5-fold cross-validation on benchmark datasets such as cross-species, rice, or M. musculus.

## When NOT to use

- Evaluating generalization on held-out data — use the independent test set via csDMA_indepedent_testing.py instead of cross-validation
- The input is already a per-sample prediction score file rather than raw 41nt sequences needing Motif/Kmer/Binary encoding
- The baseline method is unavailable for direct comparison (e.g. i6mA-Pred, which was only known from its online abstract with jackknife ACC 0.831)

## Inputs

- Training positive and negative 41nt DNA 6mA sequence datasets for cross-species, rice, and M. musculus benchmarks
- Motif feature score matrices (matrix_motif1, matrix_motif2) from feature_selection.py

## Outputs

- Per-dataset cross-validation metrics (AUC, ACC, Sn, Sp, MCC) for csDMA and iDNA6mA-PseKNC
- Comparison table mirroring Table 4, e.g. table4_cross_validation_results.csv

## How to apply

Load the training positive/negative FASTA-style sequence datasets (cross-species 2,768 pos/2,716 neg; rice 880/880; M. musculus 1,934/1,934, all 41nt) and generate the Motif score matrices (matrix_motif1, matrix_motif2) with feature_selection.py, which encodes Motif + Kmer (k=2,3,4; 336 k-mer features) + Binary features. Run 5-fold cross-validation of the csDMA ExtraTrees model via classifier_selection.py, and run the iDNA6mA-PseKNC baseline reimplemented as an SVM with C=0.336 and gamma=0.02 on the same feature matrices. Compute AUC, ACC, Sn, Sp, and MCC per dataset from the cross-validation score files and assemble them side-by-side (csDMA AUC 0.879/0.923/0.965 vs iDNA6mA-PseKNC 0.844/0.896/0.974 on cross-species/rice/M. musculus). The rationale is Chou's 5-step rule: cross-validation on a reduced-redundancy gold-standard dataset (CD-HIT-EST threshold 0.80) provides a fair, data-identical comparison between predictors.

## Related tools

- **csDMA** (ExtraTrees-based 6mA predictor whose model is evaluated by 5-fold cross-validation) — https://github.com/liuze-nwafu/csDMA
- **iDNA6mA-PseKNC** (Reimplemented SVM baseline (C=0.336, gamma=0.02) evaluated on the same feature matrices for comparison)
- **classifier_selection.py** (Trains RandomForest, GradientBoosting, AdaBoost, ExtraTrees, and SVM models and evaluates them by 5-fold cross-validation) — github.com/NWAFU-LiuLab/csDMA
- **feature_selection.py** (Implements the Motif + Kmer + Binary feature encoding scheme consumed by the cross-validation scripts) — github.com/NWAFU-LiuLab/csDMA
- **CD-HIT-EST** (Reduces sequence redundancy of the cross-species benchmark at threshold 0.80 before evaluation)

## Examples

```
python classifier_selection.py train_positive_dataset train_negative_dataset matrix_motif1 matrix_motif2
```

## Evaluation signals

- csDMA ExtraTrees cross-validation AUC should reproduce the reported values: ~0.879 on cross-species, ~0.923 on rice, ~0.965 on M. musculus
- iDNA6mA-PseKNC baseline AUC should land near 0.844 (cross-species), 0.896 (rice), 0.974 (M. musculus)
- csDMA AUC must exceed the iDNA6mA-PseKNC baseline on the cross-species and rice datasets to confirm the paper's finding
- Both predictors must be evaluated on identical, CD-HIT-EST-reduced (0.80 threshold) 41nt feature matrices for the comparison to be valid
- All five metrics (AUC, ACC, Sn, Sp, MCC) should be present in the output CSV for each dataset-predictor pair

## Limitations

- csDMA performance was relatively low on the rice and cross-species datasets compared with M. musculus
- i6mA-Pred could not be directly compared because its method was unreachable while the paper was in corrected proof phase; its performance (jackknife ACC 0.831) was known only from the online abstract
- csDMA was implemented in Python 2.7, and genomic/structural feature encodings and species such as human and Arabidopsis thaliana were not yet covered

## Evidence

- [abstract] the ExtraTrees model received the best AUC of 0.878 by using 5-fold cross-validation on the training dataset: "the ExtraTrees model received the best AUC of 0.878 by using 5-fold cross-validation on the training dataset"
- [intro] five classifiers with 5-fold CV: "Thus, we used the ExtraTrees algorithm to train the optimized model."
- [readme] readme: classifier selection usage: "Usage: python classifier_selection.py train_positive_dataset train_negative_dataset matrix_motif1 matrix_motif2"
- [readme] readme: 5-fold cross-validation used for classifier evaluation: "5-fold cross-validation was used to evaluate the performance of different classifier."
- [discussion] relatively low performance on rice and cross-species datasets: "Although our model received a high performance on the M. musculus dataset, the performance on the rice and cross-species datasets were relatively low."
