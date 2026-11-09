---
name: cell-line-specific-model-training
description: Use when you have fasta-formatted SE/TE sequences for one cell line (e.g.
  mESC) and want a cell-line-specific sequence-only classifier with user-set learning
  rate and epochs.
when_to_use_negative:
- You need a generalizing model across multiple cell lines or species — use DeepSE-integrative
  (train_integrative1/2/3.py) instead of the cell-line-specific model.
- Input is already an encoded feature matrix from another featurization scheme — the
  CNN expects dna2vec k-mer embedding matrices produced by get_datavec.py.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_3474
tools:
- name: DeepSE
  role: Two-layer CNN model whose cell-line-specific variant is trained via train_specific.py
    to distinguish SEs from TEs in one cell line
  repo: https://github.com/QiaoyingJi/DeepSE
- name: dna2vec
  role: Provides the pre-trained k-mer feature embeddings (mm9/hg19) used to encode
    SE/TE fasta sequences into feature matrices before training
  repo: https://ngpatrick.com/dna2vec/
- name: Keras
  role: Framework implementing the two-layer CNN defined in model.py that train_specific.py
    trains
- name: Python
  role: Runtime environment (>=3.6) in which get_datavec.py, train_specific.py, and
    LogUtils.py are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ygeno.2021.10.007
    title: 'DeepSE: Detecting super-enhancers among typical enhancers using only sequence
      feature embeddings'
  source_content_hash: sha256:e2647b7b97cc3023569d3bc52d0188eba29861e4c8ca2f788de022566f476359
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# cell-line-specific-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a DeepSE-specific two-layer CNN (Keras) that distinguishes super-enhancers from typical enhancers in a single chosen cell line, using only dna2vec k-mer feature embeddings of the SE/TE DNA sequences — no ChIP-seq data required.

## When to use

Use when you have fasta-formatted SE/TE sequences for one cell line (e.g. mESC) and want a cell-line-specific sequence-only classifier with user-set learning rate and epochs.

## When NOT to use

- You need a generalizing model across multiple cell lines or species — use DeepSE-integrative (train_integrative1/2/3.py) instead of the cell-line-specific model.
- Input is already an encoded feature matrix from another featurization scheme — the CNN expects dna2vec k-mer embedding matrices produced by get_datavec.py.

## Inputs

- Fasta-formatted DNA sequences of super-enhancers (SEs) and typical enhancers (TEs) for one cell line (e.g. mESC)
- Pre-trained dna2vec k-mer embedding matrix (mm9 for mouse, hg19 for human)
- Encoded feature matrices from get_datavec.py located in ./data/<cell_name>

## Outputs

- Trained cell-line-specific two-layer CNN (Keras model)
- Encoded k-mer feature matrices written to ./data/<cell_name>
- Training log files generated via LogUtils.py
- Evaluation metrics (e.g. accuracy/AUC) on held-out SE/TE data for that cell line

## How to apply

Set up a Python >=3.6 environment and install the modules listed in requirements.txt. Encode the cell line's SE/TE fasta sequences into feature matrices by running get_datavec.py with the cell name and a pre-trained genome embedding (mouse = mm9, human = hg19), writing encoded matrices to ./data/<cell_name>. Instantiate the two-layer CNN defined in model.py and train the cell-line specific model via train_specific.py passing cell_name, learning_rate, and epochs (e.g. 0.001 and 80 per the paper's mESC demo). LogUtils.py records training information to log files while progress also prints to console. Evaluate on held-out SE/TE data for that cell line and save metrics (e.g. accuracy/AUC) alongside the trained Keras model.

## Related tools

- **DeepSE** (Two-layer CNN model whose cell-line-specific variant is trained via train_specific.py to distinguish SEs from TEs in one cell line) — https://github.com/QiaoyingJi/DeepSE
- **dna2vec** (Provides the pre-trained k-mer feature embeddings (mm9/hg19) used to encode SE/TE fasta sequences into feature matrices before training) — https://ngpatrick.com/dna2vec/
- **Keras** (Framework implementing the two-layer CNN defined in model.py that train_specific.py trains)
- **Python** (Runtime environment (>=3.6) in which get_datavec.py, train_specific.py, and LogUtils.py are executed)

## Examples

```
python get_datavec.py mESC mm9; python train_specific.py mESC 0.001 80
```

## Evaluation signals

- Encoded matrices for the chosen cell line exist under ./data/<cell_name> after running get_datavec.py (e.g. ./data/mESC).
- train_specific.py is invoked with the correct three arguments: cell_name, learning_rate, and epochs (demo: mESC 0.001 80).
- Training console output is mirrored in log files generated by LogUtils.py.
- Evaluation on the held-out SE/TE data yields recorded metrics (e.g. accuracy/AUC) alongside the saved Keras model.
- On mESC with the paper's settings, classification performance should match or approach the paper's reported DeepSE-specific results, which outperformed all state-of-the-art sequence-only methods.

## Limitations

- The model is cell-line specific: it is trained and evaluated within one cell line and does not pool data across lines — cross-cell-line or cross-species generalization requires the DeepSE-integrative scripts (train_integrative1/2/3.py).
- Requires Python >=3.6 and third-party packages from requirements.txt; embeddings are only pre-provided for hg19 (human) and mm9 (mouse), so other genomes need training your own dna2vec embedding.
- The README shows inconsistent learning-rate text (one line says 0.01 while the demo command uses 0.001); follow the demo command values.
- No discussion-section content is available, so caveats about the approach beyond the README are not documented.

## Evidence

- [abstract] Cell-line-specific CNN training on encoded sequence features: "With only the DNA sequence information, DeepSE outperformed all state-of-the-art methods"
- [methods] Encoding step produces per-cell-line matrices: [evidence span withheld — anchor status: not_found]
- [methods] Cell-line specific training command signature: [evidence span withheld — anchor status: not_found]
- [readme] Demo invocation with exact hyperparameters: "To run a demo of DeepSE-specific model on mESC dataset with epochs = 80 and learning_rate = 0.001, you can type"
- [methods] Model architecture implemented in Keras: "model.py A two-layer CNN model was implemented by Keras."
