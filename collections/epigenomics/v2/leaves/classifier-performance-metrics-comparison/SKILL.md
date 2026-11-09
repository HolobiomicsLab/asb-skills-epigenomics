---
name: classifier-performance-metrics-comparison
description: 'Use when a prediction pipeline must choose among several candidate learning
  algorithms: train each on identical encoded features, evaluate with 5-fold cross-validation,
  and tabulate Sn/ACC/MCC/AUC/F1 to justify the final classifier choice.'
when_to_use_negative:
- The goal is to compare different feature encodings rather than classifiers — the
  feature subset selection (Motif+Kmer+Binary, AUC 0.871) is a separate experiment
- The task is final-model evaluation on held-out data — use the independent testing
  workflow (csDMA_indepedent_testing.py), not the classifier selection comparison
- The dataset is too large for repeated full CV of 1,000-tree ensembles and only a
  single algorithm is required
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: csDMA
  role: Python 2.7 pipeline whose classifier_selection.py script trains and cross-validates
    the five candidate classifiers on the encoded feature matrices
  repo: https://github.com/liuze-nwafu/csDMA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-019-49430-4
    title: 'csDMA: an improved bioinformatics tool for identifying DNA 6 mA modifications
      via Chou’s 5-step rule'
  source_content_hash: sha256:c630efd53ab6c34d748d98d7c4a1cd3abaa77701e84c600024c4b712ccd996c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liuze-nwafu/csDMA
  tools:
  - Python
  - csDMA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# classifier-performance-metrics-comparison

## Summary

A systematic procedure for benchmarking multiple classifiers (Random Forest, GradientBoosting, AdaBoost, ExtraTrees, SVM) on the same encoded feature matrix using 5-fold cross-validation and computing Sn, ACC, MCC, AUC and F1, so the best-performing algorithm can be selected for the final model (here, ExtraTrees for csDMA).

## When to use

Use when a prediction pipeline must choose among several candidate learning algorithms: train each on identical encoded features, evaluate with 5-fold cross-validation, and tabulate Sn/ACC/MCC/AUC/F1 to justify the final classifier choice.

## When NOT to use

- The goal is to compare different feature encodings rather than classifiers — the feature subset selection (Motif+Kmer+Binary, AUC 0.871) is a separate experiment
- The task is final-model evaluation on held-out data — use the independent testing workflow (csDMA_indepedent_testing.py), not the classifier selection comparison
- The dataset is too large for repeated full CV of 1,000-tree ensembles and only a single algorithm is required

## Inputs

- training positive and negative sequence datasets (e.g. 80% stratified split of the 2,768 positive / 2,716 negative cross-species 6mA dataset, 41nt sequences)
- encoded motif feature matrices matrix_motif1 and matrix_motif2 produced by feature_selection.py
- Motif+Kmer+Binary feature matrix (k-mer k=2,3,4 giving 336 features, plus binary and motif scores)

## Outputs

- cross-validation metric table (classifier_cv_metrics.csv) with Sn, ACC, MCC, AUC, F1 per classifier
- selected best classifier (ExtraTrees) used to train the final model file csDMA.pkl and normalization file normalization.pkl

## How to apply

Assemble one common feature matrix — for csDMA this is the Motif+Kmer+Binary encoding from feature_selection.py, with k-mer features for k=2,3,4 (42+43+44=336 k-mer patterns) plus motif score matrices (matrix_motif1, matrix_motif2). Train the tree ensembles (Random Forest, GradientBoosting, AdaBoost, ExtraTrees) each with 1,000 trees, and tune the SVM's penalty (C) and kernel coefficient (gamma) by grid search. Run 5-fold cross-validation on the training dataset (e.g. the 80% split of the cross-species 6mA dataset) for every classifier, computing Sn, ACC, MCC, AUC and F1 per model. Select the algorithm with the best cross-validated metrics — in the csDMA study ExtraTrees achieved the highest MCC of 0.603, AUC of 0.878 and F1 of 0.811 — and use it to train the optimized model (csDMA.pkl with normalization.pkl). Save the comparison as a table (classifier_cv_metrics.csv) so the selection is auditable.

## Related tools

- **csDMA** (Python 2.7 pipeline whose classifier_selection.py script trains and cross-validates the five candidate classifiers on the encoded feature matrices) — https://github.com/liuze-nwafu/csDMA

## Examples

```
python classifier_selection.py train_positive_dataset train_negative_dataset matrix_motif1 matrix_motif2
```

## Evaluation signals

- ExtraTrees must reproduce the reported cross-validation values: AUC 0.878, Sn 0.864, ACC 0.799, MCC 0.603, F1 0.811 on the training dataset
- All five classifiers must be evaluated on the identical Motif+Kmer+Binary feature matrix with the same 5-fold CV splits for a fair comparison
- SVM hyperparameters (C, gamma) must be tuned by grid search rather than left at defaults
- The output table contains exactly one row per classifier (Random Forest, GradientBoosting, AdaBoost, ExtraTrees, SVM) with all five metrics filled
- The chosen classifier (ExtraTrees) should also remain the best performer on the independent test set (AUC 0.893), confirming selection consistency

## Limitations

- SVM tuning by grid search over C and gamma is computationally expensive compared to the fixed tree ensembles
- Classifier ranking from the training-dataset CV may not transfer to other species' datasets — csDMA performance was relatively low on the rice and cross-species datasets
- The comparison is implemented in Python 2.7, which may require environment workarounds on modern systems
- Only five algorithm families are covered; genomic and structural feature encodings and other classifiers were not incorporated

## Evidence

- [abstract] ExtraTrees classifier comparison: "the ExtraTrees model received the best AUC of 0.878 by using 5-fold cross-validation on the training dataset"
- [intro] classifier set and CV: [evidence span withheld — anchor status: not_found]
- [intro] feature subset rationale: "the model achieved the best AUC value of 0.871 when three feature subsets Motif, Kmer, and Binary feature subsets were selected"
- [readme] README classifier selection usage: "Usage: python classifier_selection.py train_positive_dataset train_negative_dataset matrix_motif1 matrix_motif2"
- [readme] README evaluation method: "5-fold cross-validation was used to evaluate the performance of different classifier."
