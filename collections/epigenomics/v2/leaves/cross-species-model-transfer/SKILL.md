---
name: cross-species-model-transfer
description: Use when a trained sequence-classification model must be evaluated for
  cross-species transfer, e.g. training DeepSE-integrative on human hg19 cell lines
  (H2171, MM1.S, U87) and testing on mouse mm9 cell lines (mESC, myotube, macrophage,
  pro-B, Th-cell), or vice versa.
when_to_use_negative:
- The goal is a cell-line specific model or a within-species integrative model — use
  train_specific.py or train_integrative1.py/train_integrative2.py instead, since
  train_integrative3.py is specific to training on one species and testing on another.
- The target species has no pre-trained dna2vec embedding matrix — only human (hg19)
  and mouse (mm9) embeddings are supplied; a custom embedding matrix would have to
  be trained via dna2vec first.
- Input data is already encoded as feature vectors or comes from ChIP-seq-based SE/TE
  predictors — this skill operates on raw DNA fasta sequences of TEs and SEs only.
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: DeepSE
  role: Two-layer CNN framework whose train_integrative3.py script performs the cross-species
    training/testing
  repo: https://github.com/QiaoyingJi/DeepSE
- name: dna2vec
  role: Supplies the pre-trained k-mer embedding matrices for human (hg19) and mouse
    (mm9) used to encode sequences before transfer
  repo: https://ngpatrick.com/dna2vec/
- name: Keras
  role: Implements the two-layer CNN model that is trained on one species and tested
    on the other
- name: Python
  role: Runtime environment (>=3.6) in which the encoding and transfer scripts are
    executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ygeno.2021.10.007
    title: 'DeepSE: Detecting super-enhancers among typical enhancers using only sequence
      feature embeddings'
  source_content_hash: sha256:b6bc6057db5e1b5b92def9a4510c1834720954262910cfe29890c8f0c3b2fd97
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
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cross-species-model-transfer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains the DeepSE-integrative two-layer CNN on all cell-line datasets of one species (human hg19 or mouse mm9) and evaluates it on the other species' datasets, testing whether sequence-based super-enhancer vs typical-enhancer patterns transfer across species.

## When to use

Use when a trained sequence-classification model must be evaluated for cross-species transfer, e.g. training DeepSE-integrative on human hg19 cell lines (H2171, MM1.S, U87) and testing on mouse mm9 cell lines (mESC, myotube, macrophage, pro-B, Th-cell), or vice versa.

## When NOT to use

- The goal is a cell-line specific model or a within-species integrative model — use train_specific.py or train_integrative1.py/train_integrative2.py instead, since train_integrative3.py is specific to training on one species and testing on another.
- The target species has no pre-trained dna2vec embedding matrix — only human (hg19) and mouse (mm9) embeddings are supplied; a custom embedding matrix would have to be trained via dna2vec first.
- Input data is already encoded as feature vectors or comes from ChIP-seq-based SE/TE predictors — this skill operates on raw DNA fasta sequences of TEs and SEs only.

## Inputs

- SE/TE DNA sequences in fasta format for each cell-line dataset (e.g. mESC, myotube, macrophage, pro-B cell, Th-cell for mouse; H2171, MM1.S, U87 for human), organized by cell name
- Pre-trained dna2vec k-mer embedding matrices for human (hg19) and mouse (mm9)
- Hyperparameters: species ('hg' or 'mm'), learning_rate (0.001), epochs (80)

## Outputs

- Encoded feature matrices per cell line stored under './data/<cell_name>'
- Training/test log files produced by LogUtils.py
- Per-cell-line test metrics for the held-out species in both transfer directions (hg->mm and mm->hg)

## How to apply

Install the DeepSE dependencies via pip from requirements.txt (Python >=3.6) and load the pre-trained dna2vec k-mer embedding matrices supplied for human (hg19) and mouse (mm9). Encode each cell line's SE/TE fasta sequences with get_datavec.py specifying the correct genome (e.g. 'python get_datavec.py mESC mm9'), writing encoded matrices under './data/<cell_name>'. Run 'python train_integrative3.py hg 0.001 80' to train the Keras two-layer CNN on all human cell-line datasets and test on mouse datasets (learning_rate=0.001, epochs=80), then repeat with 'mm' as the training species to transfer in the opposite direction. LogUtils.py records the training/test process in log files during training. The rationale is that DeepSE generalizes well across cell lines, implying cell-type specific super-enhancers share hidden sequence patterns that may also cross species.

## Related tools

- **DeepSE** (Two-layer CNN framework whose train_integrative3.py script performs the cross-species training/testing) — https://github.com/QiaoyingJi/DeepSE
- **dna2vec** (Supplies the pre-trained k-mer embedding matrices for human (hg19) and mouse (mm9) used to encode sequences before transfer) — https://ngpatrick.com/dna2vec/
- **Keras** (Implements the two-layer CNN model that is trained on one species and tested on the other)
- **Python** (Runtime environment (>=3.6) in which the encoding and transfer scripts are executed)

## Examples

```
python train_integrative3.py hg 0.001 80
```

## Evaluation signals

- Encoded matrices exist under './data/<cell_name>' for every cell line of both species after running get_datavec.py with the correct genome argument
- Training completes in both directions (train_integrative3.py with 'hg' and with 'mm') and LogUtils.py log files record the corresponding test results
- Per-cell-line test metrics are produced for every cell line of the held-out species in both transfer directions
- All metric values fall within the plausible [0, 1] range

## Limitations

- Only pre-trained dna2vec embedding matrices for human (hg19) and mouse (mm9) are provided; other species require training a custom dna2vec model to obtain embedding matrices.
- Cross-species generalization is inferred from cross-cell-line generalization; the underlying assumption is that cell-type specific SEs share hidden sequence patterns, which may not hold for distant species or novel cell types.
- The transfer script depends on pre-uploaded encoded cell data (mESC_constituent, mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87); new cell lines must first be encoded with get_datavec.py.

## Evidence

- [methods] cross-species script invocation: [evidence span withheld — anchor status: not_found]
- [readme] training configuration parameters: [evidence span withheld — anchor status: not_found]
