---
name: pan-cancer-multi-omics-neural-network-training
description: Use when training or reproducing the CITRUS pan-cancer model from somatic
  alteration matrices, gene expression, and TF-target gene priors, or extracting its
  held-out test predictions and TF activities.
when_to_use_negative:
- You are training on a dataset other than the provided dataset_CITRUS.pkl without
  re-tuning hyperparameters — the README warns the >10 hyperparameters were tuned
  for the provided pan-cancer data.
- You need TF directionality (activation vs repression), co-binding TF complexes (e.g.
  AP-1−IRF), or disambiguation of same-family TFs sharing motifs — the model's TF-target
  priors do not represent these.
- You only need evaluation metrics on precomputed predictions — training can be skipped
  by loading the published outputs from the supplementary website instead.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CITRUS
  role: Chromatin-informed Inference of Transcriptional Regulators Using Self-attention
    mechanism — the partially interpretable encoder-decoder model being trained
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: PyTorch
  role: Framework implementing the CITRUS model training and inference (tested on
    torch 1.2.0 Windows, 1.5.1+cu101 Linux, 1.9.0 Mac)
- name: Anaconda
  role: Package manager used to install PyTorch and create the Python 3.7+ conda environment
    with all dependencies
- name: scikit-learn
  role: Package used alongside SciPy for evaluation (Spearman correlation, MSE) of
    predicted versus measured gene expression on the test set
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:573200546219c4e42e88aa388c05ef21f46a0012439ed6937c68a03b7b422462
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6842105263157895
n_claims: 19
---

# pan-cancer-multi-omics-neural-network-training

## Summary

Train the CITRUS partially interpretable encoder-decoder neural network (PyTorch) on pan-cancer TCGA multi-omics data (SGA index lists, filtered GEP of 5541 genes, TF-target gene matrix from ATAC-seq motifs) to predict gene expression and patient-specific TF activities, with a self-attention encoder over somatic alterations.

## When to use

Use when training or reproducing the CITRUS pan-cancer model from somatic alteration matrices, gene expression, and TF-target gene priors, or extracting its held-out test predictions and TF activities.

## When NOT to use

- You are training on a dataset other than the provided dataset_CITRUS.pkl without re-tuning hyperparameters — the README warns the >10 hyperparameters were tuned for the provided pan-cancer data.
- You need TF directionality (activation vs repression), co-binding TF complexes (e.g. AP-1−IRF), or disambiguation of same-family TFs sharing motifs — the model's TF-target priors do not represent these.
- You only need evaluation metrics on precomputed predictions — training can be skipped by loading the published outputs from the supplementary website instead.

## Inputs

- dataset_CITRUS.pkl — integrated TCGA pickle containing SGA index lists (11998 genes, binary mutation/CNA transformed to index lists), GEP (5541 filtered genes), TF-target gene matrix, cancer type list, and tumor barcodes
- TF-target gene matrix derived from TF binding motifs in open chromatin (ATAC-seq tumor profiles from TCGA/Genomic Data Commons)

## Outputs

- output_dataset_CITRUS.pkl — dictionary with 'labels' (measured exp), 'preds' (predicted exp), 'hid_tmr' (TF activity), 'emb_tmr' (tumor embedding), 'attn_wt' (attention weight), 'gene_emb', 'tf_gene' weights
- Held-out test fields: 'labels_test', 'preds_test', 'tmr_test', 'can_test'
- ensemble_TF activities from 10 runs (via ensemble_TF.py)

## How to apply

Download the integrated TCGA dataset (5803 samples, 17 tumor types; SGA index lists, GEP filtered to 5541 top-variant genes, TF-target gene matrix) as dataset_CITRUS.pkl from https://sites.pitt.edu/~xim33/CITRUS and place it in ./data. Set up a Python 3.7+ Anaconda environment with PyTorch, Pandas, Numpy, scikit-learn, and SciPy, then train with default hyperparameters (embedding_size=512, hidden_size=400, attention_size=256, attention_head=32, learning_rate=1e-3, weight_decay=1e-5, batch_size=100, dropout_rate=0.2, activation=tanh, patience=30, gep_normalization=scaleRow) via 'python run_CITRUS.py'. For ensembled TF activity, run the model 10 times with --run_count and ensemble via ensemble_TF.py --runs 10. Read ./data/output_dataset_CITRUS.pkl and extract 'labels_test' (measured expression), 'preds_test' (predicted expression), 'tmr_test', and 'can_test' to evaluate on held-out test tumors (e.g. mean Spearman correlation and MSE, compared against per-cancer-type affinity regression baselines). If training on a custom dataset rather than the provided pickle, re-tune the hyperparameters.

## Related tools

- **CITRUS** (Chromatin-informed Inference of Transcriptional Regulators Using Self-attention mechanism — the partially interpretable encoder-decoder model being trained) — https://github.com/osmanbeyoglulab/CITRUS
- **PyTorch** (Framework implementing the CITRUS model training and inference (tested on torch 1.2.0 Windows, 1.5.1+cu101 Linux, 1.9.0 Mac))
- **Anaconda** (Package manager used to install PyTorch and create the Python 3.7+ conda environment with all dependencies)
- **scikit-learn** (Package used alongside SciPy for evaluation (Spearman correlation, MSE) of predicted versus measured gene expression on the test set)

## Examples

```
for (( i = 1; i <= 10; i++ )); do python run_CITRUS.py --run_count $i; done; python ensemble_TF.py --runs 10
```

## Evaluation signals

- output_dataset_CITRUS.pkl exists and contains the 'labels_test', 'preds_test', 'tmr_test', and 'can_test' fields
- Mean Spearman correlation between predicted and measured gene expression and MSE are computed on the held-out test set and compared against per-cancer-type affinity regression (AR) baselines; CITRUS should achieve significantly better performance
- Ensemble stability check: TF activities ('hid_tmr') from 10 independent runs are combined via ensemble_TF.py --runs 10
- GEP preprocessing invariant: expression is normalized row-wise with the default gep_normalization='scaleRow' and restricted to the 5541 filtered genes available in the TF-target gene matrix
- Dataset integrity: pickle loaded via pickle.load contains SGA index lists, GEP, TF-target matrix, cancer types, and tumor barcodes for 5803 samples across 17 tumor types

## Limitations

- TFs of the same family often share a similar motif and thus are difficult to disambiguate, so TF motifs may encompass the activities of multiple TFs
- Co-binding TF binding patterns (e.g. AP-1−IRF complexes) can be biologically meaningful but are not currently represented in the model
- No directionality is represented in the TF–target gene priors (whether a gene is activated or repressed by a TF)
- Tumor regulatory network analysis is complicated by stromal/immune cells within the tumor and heterogeneity of the cancer cells themselves
- Default hyperparameters were tuned for the provided dataset; when training other datasets they should be re-tuned

## Evidence

- [readme] Download input dataset "dataset_CITRUS.pkl" from https://sites.pitt.edu/~xim33/CITRUS and put it into ./data directory.: "Download input dataset "dataset_CITRUS.pkl" from https://sites.pitt.edu/~xim33/CITRUS and put it into ./data directory."
- [readme] GEP contains continuous values of gene expression with filtered 5541 genes by considering top variant genes in each cancer type as well as gene availability in TF-target gene matrix.: "GEP contains continuous values of gene expression with filtered 5541 genes by considering top variant genes in each cancer type as well as gene availability in TF-target gene matrix."
- [readme] output_data = pickle.load(open("./data/output_dataset_CITRUS.pkl", "rb")): "output_data = pickle.load(open("./data/output_dataset_CITRUS.pkl", "rb"))"
- [methods] The code runs on python 3.7 and above. Besides python 3, some other packages such as PyTorch, Pandas, Numpy, scikit-learn, Scipy are used.: "The code runs on python 3.7 and above. Besides python 3, some other packages such as PyTorch, Pandas, Numpy, scikit-learn, Scipy are used."
- [readme] When training dataset other than provided. It is better to re-tune these parameters.: "When training dataset other than provided. It is better to re-tune these parameters."
