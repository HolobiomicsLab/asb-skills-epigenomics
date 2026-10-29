---
name: gene-expression-prediction-evaluation
description: Use when a trained model predicts tumor gene expression from somatic
  alterations and you must quantify held-out test-set accuracy (predicted vs measured
  GEP) and compare against baselines.
when_to_use_negative:
- The model output pickle lacks the 'labels_test' and 'preds_test' test-set fields
  (only full-dataset 'labels'/'preds' are available), so held-out evaluation cannot
  be performed without retraining with a holdout split
- The goal is interpreting TF activities or attention weights (e.g., in silico knock
  out analysis) rather than quantifying expression-prediction accuracy
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2640
tools:
- name: CITRUS
  role: Model whose held-out test predictions ('preds_test') are evaluated against
    measured expression ('labels_test')
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: SciPy
  role: Computes Spearman rank correlation between predicted and measured gene expression
- name: Pandas
  role: Handles the pickled TCGA matrices and aggregates tuning metrics
- name: PyTorch
  role: Runtime environment in which run_CITRUS.py is trained and test predictions
    are generated
- name: scikit-learn
  role: Supporting package for regression baselines and metrics in the evaluation
    environment
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:5e9c3bb5052a98d1e52672af8ec8cd1193b7f62d92a6cd4023c78c0f16df331a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/osmanbeyoglulab/CITRUS
  tools:
  - Anaconda
  - CITRUS
  - Numpy
  - Pandas
  - PyTorch
  - Python
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# gene-expression-prediction-evaluation

## Summary

Evaluates a somatic-alteration-to-gene-expression prediction model (e.g., CITRUS) by computing Spearman correlation and mean squared error between predicted and measured gene expression on held-out TCGA test tumors, and benchmarking against an affinity regression baseline.

## When to use

Use when a trained model predicts tumor gene expression from somatic alterations and you must quantify held-out test-set accuracy (predicted vs measured GEP) and compare against baselines.

## When NOT to use

- The model output pickle lacks the 'labels_test' and 'preds_test' test-set fields (only full-dataset 'labels'/'preds' are available), so held-out evaluation cannot be performed without retraining with a holdout split
- The goal is interpreting TF activities or attention weights (e.g., in silico knock out analysis) rather than quantifying expression-prediction accuracy

## Inputs

- output_dataset_CITRUS.pkl containing 'labels_test', 'preds_test', 'tmr_test', and 'can_test' fields
- dataset_CITRUS.pkl integrated TCGA dataset (SGA index lists, GEP, TF-target gene matrix)
- Affinity regression (AR) baseline predictions trained per cancer type

## Outputs

- Mean Spearman correlation between predicted and measured gene expression on the held-out test set
- Mean squared error (MSE) on the test set
- Per-configuration correlation metrics for hyperparameter tuning (e.g., tuning_results.csv)

## How to apply

After training CITRUS with run_CITRUS.py on the integrated TCGA dataset (dataset_CITRUS.pkl, 5803 samples across 17 tumor types), read the saved output pickle (output_dataset_CITRUS.pkl) and extract the 'labels_test' (measured expression), 'preds_test' (predicted expression), 'tmr_test' (tumor list), and 'can_test' (cancer type list) fields for the held-out test tumors. Compute mean Spearman rank correlation between predicted and measured gene expression per sample and the mean squared error on the test set, using scipy.stats and NumPy. For benchmarking, train the affinity regression (AR) baseline independently for each cancer type on the same splits and compare test-set Spearman correlation and MSE against CITRUS; the reported result is that CITRUS achieves significantly better prediction performance. When tuning hyperparameters (e.g., dropout_rate 0.1–0.5, attention_size {256,128} × attention_head {32,16,8,4}, learning_rate 1e-3, weight_decay 1e-5), record average sample-level correlation as the selection metric for each configuration.

## Related tools

- **CITRUS** (Model whose held-out test predictions ('preds_test') are evaluated against measured expression ('labels_test')) — https://github.com/osmanbeyoglulab/CITRUS
- **SciPy** (Computes Spearman rank correlation between predicted and measured gene expression)
- **Pandas** (Handles the pickled TCGA matrices and aggregates tuning metrics)
- **PyTorch** (Runtime environment in which run_CITRUS.py is trained and test predictions are generated)
- **scikit-learn** (Supporting package for regression baselines and metrics in the evaluation environment)

## Examples

```
output_data = pickle.load(open("./data/output_dataset_CITRUS.pkl", "rb")); labels_test, preds_test = output_data["labels_test"], output_data["preds_test"]
```

## Evaluation signals

- output_dataset_CITRUS.pkl exists and contains the 'labels_test', 'preds_test', 'tmr_test', and 'can_test' fields
- Spearman correlation and MSE are computed on the held-out test set, not the training set
- CITRUS test-set Spearman correlation exceeds the per-cancer-type affinity regression baseline (reported as significantly better)
- Hyperparameter sweep confirms dropout rates 0.1–0.5 each yield average sample correlation above 0.9, and the mesh grid recovers attention_size 256 with 8 attention heads

## Limitations

- TF motifs cannot disambiguate TFs of the same family sharing similar motifs, so predicted-expression quality reflects pooled TF activities
- Evaluation on tumor samples is confounded by stromal/immune cell presence and cancer cell heterogeneity
- Hyperparameters should be re-tuned when training on datasets other than the provided TCGA pickle

## Evidence

- [full_text] CITRUS achieved significantly better prediction performance than the affinity regression (AR) baseline algorithm that was trained independently for each cancer type.: [evidence span withheld — anchor status: not_found]
- [readme] The output contains many elements including predicted gene expression, extracted TF activities, gene embedding, tumor embedding, attention weight, etc.: "The output contains many elements including "predicted gene expression", "extracted TF activities", "gene embedding", "tumor embedding", "attention weight", etc."
- [readme] First read in the pickle file output_dataset_CITRUS.pkl from ./data: "output_data = pickle.load(open("./data/output_dataset_CITRUS.pkl", "rb"))"
- [readme] The code runs on python 3.7 and above with PyTorch, Pandas, Numpy, scikit-learn, Scipy used.: "The code runs on python 3.7 and above. Besides python 3, some other packages such as PyTorch, Pandas, Numpy, scikit-learn, Scipy are used."
