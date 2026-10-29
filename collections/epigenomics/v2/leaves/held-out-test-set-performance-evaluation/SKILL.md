---
name: held-out-test-set-performance-evaluation
description: 'Use when replicating CITRUS''s performance comparison: you have output_dataset_CITRUS.pkl
  with test-set predictions and need Spearman/MSE metrics versus the affinity regression
  baseline.'
when_to_use_negative:
- You need ensembled TF activities (hid_tmr) rather than gene-expression prediction
  accuracy — that requires 10 runs plus ensemble_TF.py, not test-set correlation metrics
- Training on a custom dataset rather than the provided TCGA pan-cancer pickle — the
  README advises re-tuning hyperparameters, so default-parameter performance metrics
  will not transfer
- Input is already a set of performance metrics without the raw 'labels_test'/'preds_test'
  arrays needed to recompute correlations
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: CITRUS
  role: Deep learning model whose held-out test predictions ('preds_test') are evaluated
    against measured expression
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: SciPy
  role: Used to compute Spearman correlation between predicted and measured gene expression
    on the test set
- name: PyTorch
  role: Runs the CITRUS training that produces the output pickle with test-set predictions
- name: Anaconda
  role: Recommended package manager to install PyTorch and pre-install Pandas, Numpy,
    scikit-learn, Scipy for the evaluation environment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:66ef985310f505b4cbc723497492348b90179a101a467190f2581ad9affebec9
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
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# held-out-test-set-performance-evaluation

## Summary

Evaluates the CITRUS pan-cancer model by computing mean Spearman correlation and mean squared error between predicted ('preds_test') and measured ('labels_test') gene expression on held-out test tumors, and comparing against per-cancer-type affinity regression baselines.

## When to use

Use when replicating CITRUS's performance comparison: you have output_dataset_CITRUS.pkl with test-set predictions and need Spearman/MSE metrics versus the affinity regression baseline.

## When NOT to use

- You need ensembled TF activities (hid_tmr) rather than gene-expression prediction accuracy — that requires 10 runs plus ensemble_TF.py, not test-set correlation metrics
- Training on a custom dataset rather than the provided TCGA pan-cancer pickle — the README advises re-tuning hyperparameters, so default-parameter performance metrics will not transfer
- Input is already a set of performance metrics without the raw 'labels_test'/'preds_test' arrays needed to recompute correlations

## Inputs

- dataset_CITRUS.pkl (integrated TCGA dataset with SGA index lists, 5541-gene GEP, TF-target gene matrix, 5803 samples across 17 tumor types)
- output_dataset_CITRUS.pkl (CITRUS output dictionary containing 'labels_test', 'preds_test', 'tmr_test', 'can_test')
- Python 3.7+ environment with PyTorch, Pandas, Numpy, scikit-learn, SciPy

## Outputs

- Mean Spearman correlation between predicted and measured gene expression on the held-out test set
- Mean squared error (MSE) on the held-out test set
- Per-cancer-type comparison of CITRUS metrics against affinity regression baselines

## How to apply

After running 'python run_CITRUS.py' with default hyperparameters (embedding_size=512, hidden_size=400, attention_size=256, attention_head=32, learning_rate=1e-3, weight_decay=1e-5, batch_size=100, dropout_rate=0.2, activation=tanh, patience=30, gep_normalization=scaleRow), load the saved output pickle and extract the 'labels_test' (measured expression), 'preds_test' (predicted expression), 'tmr_test' (tumor list), and 'can_test' (cancer type list) fields for held-out test tumors. Compute the mean Spearman rank correlation between predicted and measured GEP across the 5541 filtered genes per tumor, and the mean squared error on the test set, optionally stratified by 'can_test' across the 17 TCGA cancer types. Compare these metrics against affinity regression (AR) baselines trained independently for each cancer type; the paper reports CITRUS achieved significantly better prediction performance. Validation requires confirming the output pickle exists and contains all four test fields before computing metrics.

## Related tools

- **CITRUS** (Deep learning model whose held-out test predictions ('preds_test') are evaluated against measured expression) — https://github.com/osmanbeyoglulab/CITRUS
- **SciPy** (Used to compute Spearman correlation between predicted and measured gene expression on the test set)
- **PyTorch** (Runs the CITRUS training that produces the output pickle with test-set predictions)
- **Anaconda** (Recommended package manager to install PyTorch and pre-install Pandas, Numpy, scikit-learn, Scipy for the evaluation environment)

## Examples

```
output_data = pickle.load(open("./data/output_dataset_CITRUS.pkl", "rb")); sp = spearmanr(output_data["preds_test"], output_data["labels_test"], axis=1)
```

## Evaluation signals

- Confirm output_dataset_CITRUS.pkl exists in ./data and loads as a Python dictionary containing the four keys 'labels_test', 'preds_test', 'tmr_test', and 'can_test'
- Per-tumor Spearman correlation between 'preds_test' and 'labels_test' across the 5541 GEP genes should be high (CITRUS reportedly outperforms per-cancer-type AR baselines); report the mean over test tumors
- MSE computed on 'preds_test' vs 'labels_test' should be lower than the per-cancer-type affinity regression baseline MSE
- Tumor barcodes in 'tmr_test' and cancer labels in 'can_test' must align row-wise with 'labels_test' and 'preds_test' so metrics can be stratified by the 17 TCGA tumor types
- Stratify metrics by 'can_test' to verify no single cancer type (of the 17) dominates the aggregate performance claim

## Limitations

- TF motifs cannot disambiguate TFs of the same family, and co-binding patterns (e.g. AP-1−IRF complexes) and directionality of TF–target priors are not modeled, which limits what gene-expression prediction accuracy implies biologically
- Regulatory network analysis is confounded by stromal/immune cells within tumors and heterogeneity of cancer cells, which affects interpretation of test-set performance
- The README notes 'When training dataset other than provided. It is better to re-tune these parameters', so default-hyperparameter test metrics only apply to the provided TCGA dataset

## Evidence

- [readme] test-set fields documented in output schema: "'labels_test':labels_test,      # measured exp on test set
'preds_test':preds_test,        # predicted exp on test set"
- [readme] output pickle loading: "output_data = pickle.load(open("./data/output_dataset_CITRUS.pkl", "rb"))"
- [supplementary] supplementary data availability: "Processed input and output files have been made available at the supplementary website for the paper: https://sites.pitt.edu/∼xim33/CITRUS."
- [results] CITRUS vs AR baseline finding: [evidence span withheld — anchor status: not_found]
- [supplementary] GEP gene filtering context: "GEP contains continuous values of gene expression with filtered 5541 genes by considering top variant genes in each cancer type as well as gene availability in TF-target gene matrix."
