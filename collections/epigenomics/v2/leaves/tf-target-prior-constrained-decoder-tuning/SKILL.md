---
name: tf-target-prior-constrained-decoder-tuning
description: Use when training or re-tuning CITRUS on TCGA pan-cancer SGA/GEP data
  and the decoder's TF layer must remain constrained by the TF-target gene matrix
  (tf_gene weights) while balancing embedding size, attention, and regularization.
when_to_use_negative:
- Your input gene set is not filtered to top-variant genes that overlap the TF-target
  gene matrix — the tf_gene constraint requires GEP genes to be present in the prior
  matrix.
- You need directionality (activation vs. repression) or co-binding TF complexes (e.g.
  AP-1–IRF) in the prior — CITRUS TF–target priors do not represent these.
- You need to disambiguate TFs of the same family sharing similar motifs — the motif-based
  prior cannot distinguish them.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: CITRUS
  role: PyTorch encoder-decoder model whose hidden layer represents TF states constrained
    by the TF-target gene matrix; tuned via run_CITRUS.py arguments
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: PyTorch
  role: Deep learning framework implementing the constrained decoder and self-attention
    encoder
- name: Anaconda
  role: Recommended package manager to install PyTorch and pre-installed Pandas, Numpy,
    scikit-learn, SciPy dependencies
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:145e8f49f6c6b1b49db4e1329ed276b07dd91e0ecb852f530c38bc121755a79a
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# TF-target prior-constrained decoder tuning

## Summary

Configures and tunes the CITRUS decoder whose hidden nodes explicitly represent transcription factor (TF) states, constrained by a TF-target gene matrix derived from ATAC-seq motif analysis, so that gene-expression prediction and extracted TF activities are biologically interpretable.

## When to use

Use when training or re-tuning CITRUS on TCGA pan-cancer SGA/GEP data and the decoder's TF layer must remain constrained by the TF-target gene matrix (tf_gene weights) while balancing embedding size, attention, and regularization.

## When NOT to use

- Your input gene set is not filtered to top-variant genes that overlap the TF-target gene matrix — the tf_gene constraint requires GEP genes to be present in the prior matrix.
- You need directionality (activation vs. repression) or co-binding TF complexes (e.g. AP-1–IRF) in the prior — CITRUS TF–target priors do not represent these.
- You need to disambiguate TFs of the same family sharing similar motifs — the motif-based prior cannot distinguish them.

## Inputs

- dataset_CITRUS.pkl — integrated TCGA pickle with SGA index lists, GEP (5541 filtered genes), TF-target gene matrix, cancer types, tumor barcodes
- TF-target gene matrix derived from TF binding motifs in open chromatin (promoter, intronic, intergenic enhancers) from TCGA ATAC-seq profiles

## Outputs

- output_dataset_CITRUS.pkl — dictionary with 'labels' (measured exp), 'preds' (predicted exp), 'hid_tmr' (TF activity), 'emb_tmr' (tumor embedding), 'attn_wt' (attention weight), 'gene_emb', 'tf_gene' (trained tf_gene-constrained weight), plus 'labels_test', 'preds_test', 'tmr_test', 'can_test'
- ensemble_TF activities from 10 runs (via ensemble_TF.py)

## How to apply

Train CITRUS with the default hyperparameters from the README: embedding_size=512, hidden_size=400 (MLP decoder dimension), attention_size=256, attention_head=32, learning_rate=1e-3 for Adam, weight_decay=1e-5, batch_size=100, dropout_rate=0.2, activation='tanh', patience=30 for early stopping, and gep_normalization='scaleRow'. The GEP input must be pre-filtered to 5541 top-variant genes that are also available in the TF-target gene matrix so the decoder's tf_gene weight matrix stays dimensionally consistent with the ATAC-seq-derived TF-target prior. To stabilize TF activities (the hidden layer before the output gene-expression layer), run the model 10 times with distinct --run_count values and ensemble with ensemble_TF.py --runs 10. When training on a dataset other than the provided 5803-sample/17-cancer-type TCGA pickle, re-tune the hyperparameters (e.g. --learning_rate 1e-4 --activation 'relu'). Decision point: verify the output pickle contains 'labels_test', 'preds_test', 'tmr_test', and 'can_test' before interpreting TF states on held-out tumors.

## Related tools

- **CITRUS** (PyTorch encoder-decoder model whose hidden layer represents TF states constrained by the TF-target gene matrix; tuned via run_CITRUS.py arguments) — https://github.com/osmanbeyoglulab/CITRUS
- **PyTorch** (Deep learning framework implementing the constrained decoder and self-attention encoder)
- **Anaconda** (Recommended package manager to install PyTorch and pre-installed Pandas, Numpy, scikit-learn, SciPy dependencies)

## Examples

```
for (( i = 1; i <= 10; i++ )); do python run_CITRUS.py --run_count $i; done; python ensemble_TF.py --runs 10
```

## Evaluation signals

- output_dataset_CITRUS.pkl exists and contains the 'labels_test', 'preds_test', 'tmr_test', and 'can_test' fields
- The trained 'tf_gene' weight (model.layer_w_2.weight) has shape consistent with the TF-target gene matrix constraint
- GEP matrix contains exactly the filtered 5541 genes selected by top variance and TF-target gene matrix availability
- Spearman correlation and MSE computed on held-out test set ('labels_test' vs 'preds_test') match or exceed per-cancer affinity regression baselines
- Ensembled TF activity from 10 runs is more stable than single-run hid_tmr outputs

## Limitations

- TF motifs cannot disambiguate TFs of the same family (shared similar motifs), so motifs may encompass activities of multiple TFs.
- Co-binding TF binding patterns (e.g. AP-1–IRF complexes) are not represented in the model.
- No directionality (activation vs. repression) is represented in the TF–target gene priors.
- Hyperparameters were tuned for the provided 5803-sample/17-tumor-type dataset; re-tuning is advised for other datasets.

## Evidence

- [abstract] CITRUS uses a layer of hidden nodes to explicitly represent the state of transcription factors (TFs) to learn the relationships between TFs and their target genes based on TF binding motifs in the: "CITRUS uses a layer of hidden nodes to explicitly represent the state of transcription factors (TFs) to learn the relationships between TFs and their target genes based on TF binding motifs in the"
- [readme] GEP contains continuous values of gene expression with filtered 5541 genes by considering top variant genes in each cancer type as well as gene availability in TF-target gene matrix.: "GEP contains continuous values of gene expression with filtered 5541 genes by considering top variant genes in each cancer type as well as gene availability in TF-target gene matrix."
- [readme] To generate ensembled TF activity, First run the model 10 times: "To generate ensembled TF activity, First run the model 10 times"
- [readme] When training dataset other than provided. It is better to re-tune these parameters.: "When training dataset other than provided. It is better to re-tune these parameters."
- [discussion] we do not represent directionality in the TF–target gene priors (i.e. whether a gene is activated or repressed by a TF): "we do not represent directionality in the TF–target gene priors (i.e. whether a gene is activated or repressed by a TF)"
