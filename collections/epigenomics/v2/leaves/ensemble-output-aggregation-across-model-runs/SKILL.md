---
name: ensemble-output-aggregation-across-model-runs
description: Use when CITRUS has been trained multiple times (--run_count 1..10) and
  a single-run 'hid_tmr' TF activity matrix is too unstable for downstream clustering
  or TF–tumor-type association testing.
when_to_use_negative:
- A single CITRUS run's 'hid_tmr' output is already sufficient and run-to-run variance
  is not a concern
- The model was trained on a custom dataset without multiple runs — first regenerate
  per-run outputs before ensembling
- The quantity of interest is a deterministic output such as attention weights ('attn_wt')
  or trained weights ('tf_gene') rather than the stochastic hidden-layer TF activity
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: CITRUS
  role: Deep learning model whose hidden-layer TF activities ('hid_tmr') are trained
    repeatedly and ensembled
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: PyTorch
  role: Framework in which CITRUS is implemented and trained 10 times per ensemble
- name: ensemble_TF.py
  role: Repository script invoked with --runs 10 to average TF activities across the
    10 trained model outputs
  repo: https://github.com/osmanbeyoglulab/CITRUS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:5a2da4ee41905b819ab7882784cdeb588827ff09cd6473cac00affb4071f71c6
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
  - ensemble_TF.py
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# Ensemble output aggregation across model runs

## Summary

Stabilizes CITRUS-inferred patient-specific transcription factor (TF) activities by running the PyTorch model 10 times and averaging the hidden-layer TF activity ('hid_tmr') across runs via ensemble_TF.py, so downstream analyses (e.g., TF-activity clustering of 5803 TCGA tumors across 17 cancer types) are not driven by the variance of a single training run.

## When to use

Use when CITRUS has been trained multiple times (--run_count 1..10) and a single-run 'hid_tmr' TF activity matrix is too unstable for downstream clustering or TF–tumor-type association testing.

## When NOT to use

- A single CITRUS run's 'hid_tmr' output is already sufficient and run-to-run variance is not a concern
- The model was trained on a custom dataset without multiple runs — first regenerate per-run outputs before ensembling
- The quantity of interest is a deterministic output such as attention weights ('attn_wt') or trained weights ('tf_gene') rather than the stochastic hidden-layer TF activity

## Inputs

- Output pickle files from 10 CITRUS training runs (e.g., output_dataset_CITRUS.pkl per run), each containing 'hid_tmr' TF activity, 'tmr' tumor barcode list, and 'can' cancer type list
- Input dataset pickle 'dataset_CITRUS.pkl' (SGA index lists, GEP with 5541 filtered genes, TF-target gene matrix) from https://sites.pitt.edu/~xim33/CITRUS

## Outputs

- Ensembled TF activity matrix aggregated over 10 CITRUS runs (per-tumor, per-TF, for 5803 samples across 17 tumor types)
- Ensembled output pickle analogous to output_dataset_CITRUS.pkl for downstream clustering (tf_activity_cluster_figure.png) and TF–tumor-type association tables (tf_tumor_type_associations.csv)

## How to apply

Train the CITRUS model 10 times with distinct run counts (e.g., looping 'python run_CITRUS.py --run_count $i' for i=1..10), each saving its output dictionary (containing 'hid_tmr' TF activity, 'tmr' tumor barcodes, and 'can' cancer types) as a pickle. Then run 'python ensemble_TF.py --runs 10' to aggregate TF activities — the hidden layer before the output gene expression layer — across the 10 runs into an ensembled TF activity matrix. Rationale: TF activity is a latent hidden-layer quantity whose values vary across training runs, so ensembling stabilizes it; the paper's downstream tasks (k-means/hierarchical clustering of 17 TCGA tumor types and per-TF SciPy t-tests versus other tumors) used these ensembled activities. Use the pre-computed ensembled TF activities from sites.pitt.edu/~xim33/CITRUS if retraining is not desired.

## Related tools

- **CITRUS** (Deep learning model whose hidden-layer TF activities ('hid_tmr') are trained repeatedly and ensembled) — https://github.com/osmanbeyoglulab/CITRUS
- **PyTorch** (Framework in which CITRUS is implemented and trained 10 times per ensemble)
- **ensemble_TF.py** (Repository script invoked with --runs 10 to average TF activities across the 10 trained model outputs) — https://github.com/osmanbeyoglulab/CITRUS

## Examples

```
for (( i = 1; i <= 10; i++ )); do python run_CITRUS.py --run_count $i; done; python ensemble_TF.py --runs 10
```

## Evaluation signals

- Exactly 10 per-run output pickles exist before aggregation (loop counter i = 1..10)
- ensemble_TF.py completes with --runs 10 and produces an ensembled TF activity matrix with the same dimensions (tumors x TFs) as a single 'hid_tmr' matrix
- Tumor barcode list ('tmr') and cancer type labels ('can') are identical across the 10 runs being ensembled
- Downstream k-means/hierarchical clustering of the ensembled activities recovers the reported tumor-type groupings (squamous-morphology BLCA/CESC/ESCA/HNSC/LUSC; renal KIRC/KIRP; gastrointestinal COAD/STAD; BRCA/UCEC)
- Optionally verify against the pre-computed ensembled TF activities published at sites.pitt.edu/~xim33/CITRUS

## Limitations

- Ensembling requires retraining the full CITRUS model 10 times, which is computationally costly; the ensemble only stabilizes TF activity and does not remove model limitations such as inability to disambiguate TFs of the same family or represent TF–target directionality
- Only TF activity (the hidden layer before the output gene expression layer) is ensembled per the paper; other outputs (attention weights, embeddings) are not covered by ensemble_TF.py
- The README's default hyperparameters were tuned for the provided TCGA 17-tumor-type dataset; new datasets require parameter re-tuning before ensembling is meaningful

## Evidence

- [readme] We use the ensemble method to further stabilize TF activity which is a hidden layer before the output gene expression layer.: "We use the ensemble method to further stabilize TF activity which is a hidden layer before the output gene expression layer."
- [readme] After getting the outputs for all runs, execute the ensemble_TF.py to ensemble TF activities: "After getting the outputs for all runs, execute the ensemble_TF.py to ensemble TF activities"
- [methods] To generate ensembled TF activity, First run the model 10 times: "To generate ensembled TF activity, First run the model 10 times"
- [abstract] CITRUS predicts patient-specific TF activities and reveals transcriptional program variations between and within tumor types.: "CITRUS predicts patient-specific TF activities and reveals transcriptional program variations between and within tumor types."
- [readme] You can find the generated ensemble_TF activities from 10 runs of CITURS model at https://sites.pitt.edu/~xim33/CITRUS: "You can find the generated ensemble_TF activities from 10 runs of CITURS model at https://sites.pitt.edu/~xim33/CITRUS"
