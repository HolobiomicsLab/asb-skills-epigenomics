---
name: deep-learning-model-training-tuning
description: Use when training or re-tuning CITRUS on a TCGA-style SGA/GEP/TF-target
  pickle dataset and you need to select attention architecture and optimizer settings
  by grid search over prediction performance.
when_to_use_negative:
- Training on a different, non-TCGA dataset without re-tuning — the README explicitly
  warns 'When training dataset other than provided. It is better to re-tune these
  parameters'
- The goal is interpreting TF activities or attention weights from an already-trained
  model rather than selecting hyperparameters
- You need TF–target directionality or co-binding effects; the model does not represent
  activation vs repression or AP-1−IRF-style complexes
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: CITRUS
  role: Model being trained and tuned; run via run_CITRUS.py with command-line hyperparameter
    arguments
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: PyTorch
  role: Deep learning framework implementing the CITRUS encoder-decoder with self-attention
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: Python 3.7
  role: Runtime for run_CITRUS.py and ensemble_TF.py
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: Pandas
  role: Aggregate hyperparameter settings and correlation metrics into tuning_results.csv
- name: Anaconda
  role: Recommended package manager to install PyTorch and dependencies in a conda
    environment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:2951e8530599952f8723571d5b4155504517c16b9441db7be6e1111bc056eec7
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
  - Python 3.7
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.55
n_claims: 20
---

# deep-learning-model-training-tuning

## Summary

Train and hyperparameter-tune the CITRUS PyTorch encoder-decoder model (self-attention over somatic alterations, TF-state hidden layer) by sweeping dropout rates, the attention size/head mesh grid, and Adam learning-rate/weight-decay settings against average sample-level correlation between predicted and measured gene expression.

## When to use

Use when training or re-tuning CITRUS on a TCGA-style SGA/GEP/TF-target pickle dataset and you need to select attention architecture and optimizer settings by grid search over prediction performance.

## When NOT to use

- Training on a different, non-TCGA dataset without re-tuning — the README explicitly warns 'When training dataset other than provided. It is better to re-tune these parameters'
- The goal is interpreting TF activities or attention weights from an already-trained model rather than selecting hyperparameters
- You need TF–target directionality or co-binding effects; the model does not represent activation vs repression or AP-1−IRF-style complexes

## Inputs

- dataset_CITRUS.pkl pickle file (SGA index lists, GEP of 5541 filtered genes, TF-target gene matrix, cancer types, tumor barcodes) placed in ./data
- run_CITRUS.py command-line arguments (embedding_size, hidden_size, attention_size, attention_head, learning_rate, weight_decay, dropout_rate, batch_size, activation, patience, gep_normalization)

## Outputs

- Per-configuration rows in tuning_results.csv (hyperparameter values plus average sample-level correlation)
- Output pickle (e.g. output_dataset_CITRUS.pkl) containing preds, labels, hid_tmr (TF activity), attn_wt, emb_tmr, gene_emb
- Ensembled TF activity from 10 runs via ensemble_TF.py

## How to apply

Load the pickled CITRUS dataset (dataset_CITRUS.pkl, 5803 TCGA samples across 17 tumor types, GEP filtered to 5541 genes) from ./data and run run_CITRUS.py, varying command-line hyperparameters one grid at a time. Sweep --dropout_rate over 0.1-0.5 while keeping defaults (embedding_size 512, hidden_size 400, batch_size 100, activation tanh, patience 30, gep_normalization scaleRow); then run the attention mesh grid crossing --attention_size {256, 128} with --attention_head {32, 16, 8, 4}; then scan --learning_rate and --weight_decay around the Adam defaults 1e-3 and 1e-5. Score each configuration by average sample-level correlation between predicted and measured gene expression (available as preds/labels in the output pickle dictionary) and aggregate parameter values and correlations into tuning_results.csv with one row per setting. For the final model, use an ensemble of 10 training runs (run_count 1..10) and combine TF activities with ensemble_TF.py --runs 10, since early stopping uses patience 30.

## Related tools

- **CITRUS** (Model being trained and tuned; run via run_CITRUS.py with command-line hyperparameter arguments) — https://github.com/osmanbeyoglulab/CITRUS
- **PyTorch** (Deep learning framework implementing the CITRUS encoder-decoder with self-attention) — https://github.com/osmanbeyoglulab/CITRUS
- **Python 3.7** (Runtime for run_CITRUS.py and ensemble_TF.py) — https://github.com/osmanbeyoglulab/CITRUS
- **Pandas** (Aggregate hyperparameter settings and correlation metrics into tuning_results.csv)
- **Anaconda** (Recommended package manager to install PyTorch and dependencies in a conda environment)

## Examples

```
python run_CITRUS.py --dataset_name dataset_CITRUS.pkl --attention_size 256 --attention_head 8 --learning_rate 1e-3 --weight_decay 1e-5 --run_count 1
```

## Evaluation signals

- Attention sweep recovers attention_size 256 with 8 attention heads as the performance-selected configuration
- Learning-rate/weight-decay sweep recovers learning_rate 1e-3 and weight_decay 1e-5
- Every dropout rate in 0.1-0.5 yields average sample-level correlation between predicted and measured gene expression above 0.9
- tuning_results.csv has exactly one row per hyperparameter configuration with matching parameter values and correlation metric
- Ensemble output from 10 runs (ensemble_TF.py --runs 10) produces TF activities consistent with the published ensemble_TF activities hosted at the supplementary site

## Limitations

- Default --attention_head is 32, but the tuned selection is 8 heads; defaults do not equal the tuned optima
- TF motifs cannot disambiguate TFs of the same family, so learned TF states may encompass multiple TFs
- TF–target priors carry no directionality (activation vs repression), and co-binding patterns such as AP-1−IRF complexes are not represented
- Tested torch versions are 1.2.0 (Windows), 1.5.1+cu101 (Linux), and 1.9.0 (Mac); other versions are unverified

## Evidence

- [readme] hyperparameter defaults: "All the demos we showed above trained CITRUS deep learning model with default hyperparameters. There are more than 10 hyperparameters that have been tuned to get optimal results."
- [readme] attention/optimizer defaults: [evidence span withheld — anchor status: not_found]
- [methods] normalization default: ""--gep_normalization", help="how to normalize gep", type=str, default="scaleRow""
- [readme] ensemble procedure: [evidence span withheld — anchor status: not_found]
- [readme] tested torch versions: "We have tested our code on torch version 1.2.0 (Windows), torch version 1.5.1+cu101 (Linux), torch version 1.9.0 (Mac)"
