---
name: model-performance-evaluation
description: Use when a DeepSE model trained on pooled cell-line datasets must be
  scored on held-out cell lines or on another species' datasets (e.g., hg->mm or mm->hg
  transfer), and per-cell-line test metrics are needed.
when_to_use_negative:
- The goal is a cell-line specific model rather than transfer evaluation — use train_specific.py
  instead of train_integrative3.py
- Sequences lack the required 'fasta' format or no embedding matrix exists for the
  genome (only hg19 and mm9 are pre-trained), making dna2vec encoding impossible without
  training a custom matrix
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: DeepSE
  role: Two-layer CNN model (DeepSE-integrative) trained on one species' pooled cell-line
    datasets and evaluated on the other species' datasets
  repo: https://github.com/QiaoyingJi/DeepSE
- name: dna2vec
  role: Provides pre-trained k-mer feature embedding matrices (hg19 and mm9) used
    to encode SE/TE DNA sequences into feature vectors for testing
  repo: https://ngpatrick.com/dna2vec/
- name: Keras
  role: Framework in which the two-layer CNN model.py is implemented for training
    and testing
- name: Python
  role: Runtime environment (>=3.6) in which all pipeline scripts (get_datavec.py,
    train_integrative3.py, LogUtils.py) execute
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ygeno.2021.10.007
    title: 'DeepSE: Detecting super-enhancers among typical enhancers using only sequence
      feature embeddings'
  source_content_hash: sha256:558074ad4a8612bc859209382887d1c630e6fcae7f45a3ff01717504e8137095
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# model-performance-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate a trained DeepSE-integrative two-layer CNN by testing it on held-out cell-line datasets, including cross-species transfer (train on human hg19, test on mouse mm9, and vice versa), aggregating test metrics per cell line.

## When to use

Use when a DeepSE model trained on pooled cell-line datasets must be scored on held-out cell lines or on another species' datasets (e.g., hg->mm or mm->hg transfer), and per-cell-line test metrics are needed.

## When NOT to use

- The goal is a cell-line specific model rather than transfer evaluation — use train_specific.py instead of train_integrative3.py
- Sequences lack the required 'fasta' format or no embedding matrix exists for the genome (only hg19 and mm9 are pre-trained), making dna2vec encoding impossible without training a custom matrix

## Inputs

- Encoded dna2vec feature-vector matrices for SE/TE FASTA sequences per cell-line dataset (mESC_constituent, mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87), stored under ./data/<cell_name>
- Pre-trained dna2vec k-mer embedding matrices for human (hg19) and mouse (mm9)
- Training hyperparameters: species (hg or mm), learning_rate (e.g., 0.001), epochs (e.g., 80)

## Outputs

- cross_species_performance.csv containing test metrics for every cell line of the held-out species in both transfer directions (hg->mm and mm->hg)
- Log files recording the training and testing process (via LogUtils.py)
- Console-printed per-cell-line test results during training

## How to apply

After encoding SE/TE FASTA sequences with get_datavec.py using the species-appropriate pre-trained dna2vec embedding matrix (human hg19 or mouse mm9), run 'python train_integrative3.py species learning_rate epochs' (e.g., hg 0.001 80) to pool all training-species cell-line datasets into the Keras two-layer CNN and test on every cell line of the other species. Capture the training process with LogUtils.py, which records process information in log files, then aggregate the test metrics for each tested cell line of the target species into cross_species_performance.csv. Repeat with the opposite species as training set to cover both transfer directions. Confidence in the evaluation rests on DeepSE's reported ability to generalize well across different cell lines, implying shared hidden sequence patterns in cell-type specific SEs.

## Related tools

- **DeepSE** (Two-layer CNN model (DeepSE-integrative) trained on one species' pooled cell-line datasets and evaluated on the other species' datasets) — https://github.com/QiaoyingJi/DeepSE
- **dna2vec** (Provides pre-trained k-mer feature embedding matrices (hg19 and mm9) used to encode SE/TE DNA sequences into feature vectors for testing) — https://ngpatrick.com/dna2vec/
- **Keras** (Framework in which the two-layer CNN model.py is implemented for training and testing)
- **Python** (Runtime environment (>=3.6) in which all pipeline scripts (get_datavec.py, train_integrative3.py, LogUtils.py) execute)

## Examples

```
python train_integrative3.py hg 0.001 80
```

## Evaluation signals

- cross_species_performance.csv exists and contains performance entries for every cell line of the held-out species in both transfer directions (hg->mm and mm->hg)
- Each metric value in cross_species_performance.csv falls within the plausible [0, 1] range
- Training log files (via LogUtils.py) record the corresponding test results for each evaluated cell line
- Encoded data matrices for each cell line are present under './data/<cell_name>' before running the training script

## Limitations

- Only two pre-trained embedding matrices are provided (human hg19 and mouse mm9); other genomes require training a custom dna2vec model
- Generalization relies on the assumption that cell-type specific SEs share hidden sequence patterns across cell lines and species; cross-species transfer is justified by this interpretation rather than guaranteed
- The README's install instruction 'pip install requirements.txt' is not the standard pip syntax for installing from a requirements file, which may cause setup friction

## Evidence

- [methods] train_integrative3.py cross-species: [evidence span withheld — anchor status: not_found]
- [readme] example cross-species invocation: [evidence span withheld — anchor status: not_found]
