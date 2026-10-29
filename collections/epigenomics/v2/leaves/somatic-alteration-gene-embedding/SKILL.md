---
name: somatic-alteration-gene-embedding
description: Use when training or adapting CITRUS on TCGA pan-cancer data where the
  sparse binary SGA matrix (11998 genes) must be converted into dense gene/tumor embeddings
  before attention-based modeling.
when_to_use_negative:
- You need TF activity readouts rather than embeddings — run the ensemble pipeline
  (10 runs + ensemble_TF.py) instead of reading embedding layers
- Your input is already a dense feature matrix rather than a sparse binary SGA matrix;
  the index-list transformation assumes 1/0 alteration encoding
- Training on a custom dataset without re-tuning hyperparameters — the README states
  'It is better to re-tune these parameters'
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: CITRUS
  role: Model whose encoder learns the 512-d gene/tumor embeddings from SGA index
    lists via self-attention
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: PyTorch
  role: Framework implementing the CITRUS encoder-decoder and embedding layers (tested
    on torch 1.2.0, 1.5.1+cu101, 1.9.0)
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: Anaconda
  role: Recommended package manager to install PyTorch and dependencies (Pandas, Numpy,
    scikit-learn, SciPy) for the Python 3.7+ environment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:33dd39233493cbf9d89c6777f0673457921d2dc533d3d313ca70540dc1779543
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# somatic-alteration-gene-embedding

## Summary

Embeds somatic alteration (SGA) gene features via CITRUS's gene embedding layer, transforming the sparse binary mutation/CNV matrix into dense 512-dimensional gene and tumor representations used by the self-attention encoder-decoder.

## When to use

Use when training or adapting CITRUS on TCGA pan-cancer data where the sparse binary SGA matrix (11998 genes) must be converted into dense gene/tumor embeddings before attention-based modeling.

## When NOT to use

- You need TF activity readouts rather than embeddings — run the ensemble pipeline (10 runs + ensemble_TF.py) instead of reading embedding layers
- Your input is already a dense feature matrix rather than a sparse binary SGA matrix; the index-list transformation assumes 1/0 alteration encoding
- Training on a custom dataset without re-tuning hyperparameters — the README states 'It is better to re-tune these parameters'

## Inputs

- dataset_CITRUS.pkl — integrated TCGA pickle with SGA index lists (11998 altered genes), GEP (5541 genes), TF-target gene matrix, cancer types, and tumor barcodes
- Binary somatic alteration gene matrix (mutations + GISTIC copy number alterations) for 5803 TCGA samples across 17 tumor types

## Outputs

- 512-dimensional gene embedding (gene_emb) and tumor embedding (emb_tmr)
- Stratified tumor embedding (emb_sga) and self-attention weights (attn_wt)
- Trained tf_gene decoder weight matrix constrained by the TF-target gene prior
- output_dataset_CITRUS.pkl pickle containing all embeddings and predictions

## How to apply

Start from the integrated TCGA pickle (dataset_CITRUS.pkl) containing the SGA index lists, GEP (5541 filtered genes), and TF-target gene matrix for 5803 samples across 17 tumor types. CITRUS transforms the sparse binary SGA — where 1 marks a distinct mutation or copy number alteration and 0 none — into index lists to facilitate gene embedding, then learns gene and tumor embeddings of dimension embedding_size=512 inside the encoder's self-attention module (attention_size=256, attention_head=32). Train with the default hyperparameters (learning_rate=1e-3, weight_decay=1e-5, batch_size=100, dropout_rate=0.2, tanh activation, patience=30, gep_normalization=scaleRow) by running python run_CITRUS.py. The learned embedding weights (gene_emb, emb_tmr, emb_sga) are returned in the output pickle alongside predictions, and gene–TF relationships in the decoder are constrained by the TF-target gene matrix derived from ATAC-seq motif analysis.

## Related tools

- **CITRUS** (Model whose encoder learns the 512-d gene/tumor embeddings from SGA index lists via self-attention) — https://github.com/osmanbeyoglulab/CITRUS
- **PyTorch** (Framework implementing the CITRUS encoder-decoder and embedding layers (tested on torch 1.2.0, 1.5.1+cu101, 1.9.0)) — https://github.com/osmanbeyoglulab/CITRUS
- **Anaconda** (Recommended package manager to install PyTorch and dependencies (Pandas, Numpy, scikit-learn, SciPy) for the Python 3.7+ environment)

## Examples

```
python run_CITRUS.py --embedding_size 512 --attention_size 256 --attention_head 32 --gep_normalization scaleRow
```

## Evaluation signals

- SGA must be transformed from a 11998-gene binary matrix into index lists before embedding — verify the loaded dataset provides index lists, not a dense 0/1 matrix
- Output pickle contains 'gene_emb', 'emb_tmr', 'emb_sga', and 'attn_wt' keys with embedding dimension matching embedding_size=512
- GEP carries exactly 5541 genes filtered by top variant genes per cancer type and gene availability in the TF-target gene matrix
- Trained tf_gene weight matrix respects the ATAC-seq-derived TF-target gene prior as a hard constraint
- All 5803 samples across 17 tumor types are represented across train/validation/test splits

## Limitations

- TF motifs cannot disambiguate TFs of the same family sharing similar motifs, so TF-target priors may conflate multiple TFs
- No directionality (activation vs repression) is represented in the TF–target gene priors used to constrain the decoder
- Co-binding TF patterns (e.g. AP-1−IRF complexes) are not represented in the model
- Tumor regulatory network analysis is confounded by stromal/immune cells and cancer cell heterogeneity

## Evidence

- [readme] SGA is originally a binary matrix containing 11998 genes with 1 as having a distinct mutation or copy number alteration and 0 as none of them. We transformed the sparse binary SGA into index lists to facilitate gene embedding.: "SGA is originally a binary matrix containing 11998 genes with 1 as having a distinct mutation or copy number alteration and 0 as none of them. We transformed the sparse binary SGA into index lists to"
- [readme] GEP contains continuous values of gene expression with filtered 5541 genes by considering top variant genes in each cancer type as well as gene availability in TF-target gene matrix.: "GEP contains continuous values of gene expression with filtered 5541 genes by considering top variant genes in each cancer type as well as gene availability in TF-target gene matrix."
- [abstract] CITRUS uses a layer of hidden nodes to explicitly represent the state of transcription factors (TFs) to learn the relationships between TFs and their target genes based on TF binding motifs: "CITRUS uses a layer of hidden nodes to explicitly represent the state of transcription factors (TFs) to learn the relationships between TFs and their target genes based on TF binding motifs in the"
- [discussion] we do not represent directionality in the TF–target gene priors (i.e. whether a gene is activated or repressed by a TF): "we do not represent directionality in the TF–target gene priors (i.e. whether a gene is activated or repressed by a TF)"
- [readme] gene_emb": gene_emb,     # gene embedding: ""gene_emb": gene_emb,     # gene embedding"
