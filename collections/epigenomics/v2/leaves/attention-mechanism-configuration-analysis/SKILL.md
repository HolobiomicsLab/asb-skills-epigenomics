---
name: attention-mechanism-configuration-analysis
description: Use when tuning or reproducing the attention hyperparameters of the CITRUS
  PyTorch model on the TCGA 5803-sample/17-tumor-type pickled dataset, where attention
  size and head count must be grid-searched against prediction correlation.
when_to_use_negative:
- Training CITRUS on a newly supplied dataset other than the provided TCGA pickled
  dataset — the README advises re-tuning all hyperparameters rather than reusing the
  pan-cancer grid results
- Tuning hyperparameters outside the attention module (dropout rate 0.1-0.5 or learning-rate/weight-decay
  grids), which are separate sweeps
- Generating ensembled TF activities, which requires running the model 10 times with
  different --run_count and ensemble_TF.py instead of varying attention parameters
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: CITRUS
  role: Model whose self-attention module (attention parameter beta_j, attention heads)
    is configured and grid-searched
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: PyTorch
  role: Framework implementing run_CITRUS.py training used for each grid point
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: Pandas
  role: Aggregating per-configuration parameter values and correlation metrics into
    tuning_results.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:f34bf1fa43aa65407a3cfa1a48460eb646e6211c284c44f51b3816d7d4db3001
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/osmanbeyoglulab/CITRUS
  tools:
  - CITRUS
  - Numpy
  - Pandas
  - PyTorch
  - Python
  - SciPy
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# attention-mechanism-configuration-analysis

## Summary

Reproduces CITRUS's mesh-grid search over the self-attention module's --attention_size ({256, 128}) and --attention_head ({32, 16, 8, 4}) settings, selecting the configuration that maximizes average sample-level correlation between predicted and measured TCGA gene expression.

## When to use

Use when tuning or reproducing the attention hyperparameters of the CITRUS PyTorch model on the TCGA 5803-sample/17-tumor-type pickled dataset, where attention size and head count must be grid-searched against prediction correlation.

## When NOT to use

- Training CITRUS on a newly supplied dataset other than the provided TCGA pickled dataset — the README advises re-tuning all hyperparameters rather than reusing the pan-cancer grid results
- Tuning hyperparameters outside the attention module (dropout rate 0.1-0.5 or learning-rate/weight-decay grids), which are separate sweeps
- Generating ensembled TF activities, which requires running the model 10 times with different --run_count and ensemble_TF.py instead of varying attention parameters

## Inputs

- dataset_CITRUS.pkl pickle file (SGA index lists, 5541-gene GEP matrix, TF-target gene matrix, cancer types, tumor barcodes) placed in ./data
- run_CITRUS.py command-line argument parser (--attention_size, --attention_head and remaining hyperparameter defaults)

## Outputs

- tuning_results.csv with one row per attention configuration and its parameter values and correlation metrics
- Selected attention configuration (attention_size 256 with 8 attention heads by prediction performance)
- Trained CITRUS output pickle (predicted gene expression, extracted TF activities, attention weight) per configuration

## How to apply

Load dataset_CITRUS.pkl (5803 TCGA samples, 17 tumor types, 5541 filtered GEP genes, sparse SGA index lists) from ./data and parse run_CITRUS.py arguments. Train CITRUS repeatedly over the attention mesh grid, crossing --attention_size in {256, 128} with --attention_head in {32, 16, 8, 4}, keeping remaining defaults (embedding_size 512, hidden_size 400, batch_size 100, activation tanh, patience 30, gep_normalization scaleRow, learning_rate 1e-3, weight_decay 1e-5). Score each configuration by the average sample-level correlation between predicted and measured gene expression, and aggregate values and metrics into tuning_results.csv with one row per setting. The rationale is that the self-attention encoder models the contextual functional impact of somatic alterations, so its capacity (parameter beta_j size) and head count directly determine how tumor-specific TF activity states are inferred before the decoder's TF hidden layer.

## Related tools

- **CITRUS** (Model whose self-attention module (attention parameter beta_j, attention heads) is configured and grid-searched) — https://github.com/osmanbeyoglulab/CITRUS
- **PyTorch** (Framework implementing run_CITRUS.py training used for each grid point) — https://github.com/osmanbeyoglulab/CITRUS
- **Pandas** (Aggregating per-configuration parameter values and correlation metrics into tuning_results.csv)

## Examples

```
python run_CITRUS.py --attention_size 256 --attention_head 8 --tag attn_256_h8
```

## Evaluation signals

- The sweep reproduces the reported optimal selection: attention_size 256 with 8 attention heads chosen by prediction performance
- Each configuration's average sample-level correlation between predicted and measured gene expression is computed and exceeds the comparable sweep's quality bar (dropout sweep correlation > 0.9 for all rates 0.1-0.5)
- tuning_results.csv contains one row for every cell of the {256,128} x {32,16,8,4} mesh grid (8 rows)
- Non-selected hyperparameters stay at README defaults (embedding_size 512, hidden_size 400, batch_size 100, activation tanh, patience 30, gep_normalization scaleRow, learning_rate 1e-3, weight_decay 1e-5) across the sweep

## Limitations

- The README's argument defaults list attention_head default 32, so the grid optimum of 8 heads differs from the shipped default and must be passed explicitly via --attention_head
- There are more than 10 hyperparameters tuned in CITRUS; attention grid results depend on holding the others at their defaults and may not transfer to other datasets
- The README provides no software version, license, or changelog details, so exact code provenance for reproduced sweeps should be verified against the git history

## Evidence

- [readme] attention size argument: "help="size of attention parameter beta_j", type=int, default=256"
- [readme] attention head argument: "help="number of attention heads", type=int, default=32"
- [readme] hyperparameter tuning scope: "There are more than 10 hyperparameters that have been tuned to get optimal results."
- [abstract] self-attention role in model: "Our approach employs a self-attention mechanism to model the contextual impact of somatic alterations."
- [methods] software dependencies: "some other packages such as PyTorch, Pandas, Numpy, scikit-learn, Scipy are used"
- [methods] GEP normalization default: "help="how to normalize gep", type=str, default="scaleRow""
