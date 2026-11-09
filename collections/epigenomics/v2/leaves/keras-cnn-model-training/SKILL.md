---
name: keras-cnn-model-training
description: Use when the input is a dna2vec-encoded SE/TE feature matrix (e.g. ./data/mESC)
  and a trained super-enhancer vs typical-enhancer classifier is needed, using learning_rate=0.001
  and epochs=80.
when_to_use_negative:
- Input is still raw FASTA DNA sequences — run get_datavec.py to obtain dna2vec k-mer
  feature matrices before training.
- The task requires ChIP-seq or other genomic signals rather than DNA sequence only
  — DeepSE is explicitly designed for sequence-only discrimination of SEs from TEs.
- No pre-trained dna2vec embedding exists for the organism (only hg19 and mm9 are
  provided, unless you train your own dna2vec model).
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: DeepSE
  role: The overall model whose two-layer CNN (DeepSE-specific and DeepSE-integrative
    variants) is trained by this skill
  repo: https://github.com/QiaoyingJi/DeepSE
- name: Keras
  role: Framework used in model.py to implement the two-layer CNN that is trained
- name: Python
  role: Runtime environment (>=3.6) in which the training scripts run
- name: dna2vec
  role: Provides the pre-trained k-mer feature embeddings used as input to the CNN;
    upstream of training
  repo: https://ngpatrick.com/dna2vec/
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1016/j.ygeno.2021.10.007
    title: 'DeepSE: Detecting super-enhancers among typical enhancers using only sequence
      feature embeddings'
  source_content_hash: sha256:aa3aaccb4fee19696efb1cdcb66561cfd76aafd0b52199e88ca60d69081f1f23
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/QiaoyingJi/DeepSE
  repo_url: https://github.com/QiaoyingJi/DeepSE
  tools:
  - DeepSE
  - Keras
  - Python
  - dna2vec
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# keras-cnn-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train the DeepSE two-layer convolutional neural network (implemented in Keras, model.py) to distinguish super-enhancers from typical enhancers using only dna2vec-encoded DNA sequence features, via cell-line specific or integrative training scripts.

## When to use

Use when the input is a dna2vec-encoded SE/TE feature matrix (e.g. ./data/mESC) and a trained super-enhancer vs typical-enhancer classifier is needed, using learning_rate=0.001 and epochs=80.

## When NOT to use

- Input is still raw FASTA DNA sequences — run get_datavec.py to obtain dna2vec k-mer feature matrices before training.
- The task requires ChIP-seq or other genomic signals rather than DNA sequence only — DeepSE is explicitly designed for sequence-only discrimination of SEs from TEs.
- No pre-trained dna2vec embedding exists for the organism (only hg19 and mm9 are provided, unless you train your own dna2vec model).

## Inputs

- dna2vec-encoded SE/TE feature matrices for a cell line (e.g. ./data/mESC produced by get_datavec.py)
- SE/TE DNA sequences in FASTA format (if encoding has not been done yet)
- Pre-trained dna2vec embedding matrix (human hg19 or mouse mm9)
- Hyperparameters: cell_name, species (hg|mm), learning_rate (0.001), epochs (80)

## Outputs

- Trained Keras two-layer CNN model (DeepSE-specific or DeepSE-integrative)
- Training log files created via LogUtils.py
- Held-out SE/TE classification metrics (accuracy/AUC) per cell line

## How to apply

First ensure the DNA sequences (FASTA format per cell line) have been converted into feature vectors with the pre-trained k-mer dna2vec embeddings via get_datavec.py (e.g. 'python get_datavec.py mESC mm9'), storing encoded matrices under ./data/<cell_name>. Then instantiate the two-layer CNN from model.py in Keras within a Python >=3.6 environment with requirements.txt installed. For a cell-line specific model, run train_specific.py with cell_name, learning_rate=0.001 and epochs=80; for DeepSE-integrative variants, use train_integrative1.py/2.py/3.py with a species setting (hg for human or mm for mouse). Record all training output through LogUtils.py so console information is written to log files, then evaluate the trained model on the held-out SE/TE data and save accuracy/AUC metrics alongside the trained Keras model.

## Related tools

- **DeepSE** (The overall model whose two-layer CNN (DeepSE-specific and DeepSE-integrative variants) is trained by this skill) — https://github.com/QiaoyingJi/DeepSE
- **Keras** (Framework used in model.py to implement the two-layer CNN that is trained)
- **Python** (Runtime environment (>=3.6) in which the training scripts run)
- **dna2vec** (Provides the pre-trained k-mer feature embeddings used as input to the CNN; upstream of training) — https://ngpatrick.com/dna2vec/

## Examples

```
python get_datavec.py mESC mm9; python train_specific.py mESC 0.001 80
```

## Evaluation signals

- The training run completes the full 80 epochs at learning_rate=0.001 and LogUtils.py writes the console output to log files.
- Encoded input matrices exist under './data/<cell_name>' (e.g. ./data/mESC) before training starts.
- The trained model achieves classification performance on the held-out SE/TE test set consistent with the paper's finding that DeepSE outperformed all state-of-the-art methods using only DNA sequence information.
- The integrative model (train_integrative1.py mm 0.001 80) generalizes across cell lines, consistent with the paper's claim that DeepSE can be generalized well across different cell lines.
- The trained Keras model and its saved metrics record (accuracy/AUC) are present alongside the log files after evaluation.

## Limitations

- Requires a runtime environment with Python >=3.6 and the third-party packages from requirements.txt.
- Only two pre-trained dna2vec embedding matrices are provided (human hg19 and mouse mm9); other genomes require training your own dna2vec model.
- The README demo text is internally inconsistent (states epochs = 80 and learning_rate = 0.01 but the command uses 0.001); the paper's demo command uses learning_rate 0.001.
- Training output must be captured via LogUtils.py; otherwise only console output is produced.

## Evidence
