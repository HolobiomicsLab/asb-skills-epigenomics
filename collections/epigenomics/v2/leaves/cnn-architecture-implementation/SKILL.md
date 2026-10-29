---
name: cnn-architecture-implementation
description: Use when implementing or verifying the DESSO CNN in TensorFlow 1.1.0/Python
  2.7 to predict TF–DNA binding from 690 ENCODE ChIP-seq datasets, or when validating
  that layer/parameter shapes match the published 16-filter, 32-neuron FC, single-sigmoid-output
  architecture.
when_to_use_negative:
- Long DNA sequences (1001 bp) are required — use the Gated-CNN (GCNN) network with
  --peak_flank 500 instead of the plain CNN
- The goal is motif discovery from user-supplied peaks without first training a model
  — the CNN must be trained (train.py) before predict.py can extract motifs
- Input is already a feature table or motif matrix rather than raw sequences/shape
  channels — this skill only defines the network, not downstream motif-profile construction
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: DESSO
  role: Framework providing the CNN architecture (16 convolution filters, ReLU + max
    pooling, 32-neuron FC layer, single sigmoid output) trained via train.py and used
    for motif prediction via predict.py
  repo: https://github.com/OSU-BMBL/DESSO
- name: TensorFlow
  role: Deep learning backend (TensorFlow 1.1.0) in which the convolutional, pooling,
    fully connected, and sigmoid output layers are implemented
  repo: https://www.tensorflow.org/install/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:5ea539ac326e975421d03a35ee867f9c2c8cc669e1e9e27a6fdbb63c3e21ad56
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/OSU-BMBL/DESSO
  repo_url: https://github.com/OSU-BMBL/DESSO
  tools:
  - DESSO
  - Python
  - TensorFlow
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# cnn-architecture-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct the DESSO one-layer CNN that predicts TF–DNA binding specificity from 101-bp one-hot encoded sequences and/or DNA shape channels (HelT, MGW, ProT, Roll), ending in a single sigmoid output neuron. This is the core architecture needed before motif prediction (predict.py) can extract sequence or shape motifs.

## When to use

Use when implementing or verifying the DESSO CNN in TensorFlow 1.1.0/Python 2.7 to predict TF–DNA binding from 690 ENCODE ChIP-seq datasets, or when validating that layer/parameter shapes match the published 16-filter, 32-neuron FC, single-sigmoid-output architecture.

## When NOT to use

- Long DNA sequences (1001 bp) are required — use the Gated-CNN (GCNN) network with --peak_flank 500 instead of the plain CNN
- The goal is motif discovery from user-supplied peaks without first training a model — the CNN must be trained (train.py) before predict.py can extract motifs
- Input is already a feature table or motif matrix rather than raw sequences/shape channels — this skill only defines the network, not downstream motif-profile construction

## Inputs

- One-hot encoded 101-bp ChIP-seq peak sequences (2 * 50 + 1 bp around peak summit)
- Per-nucleotide DNA shape channels: HelT, MGW, ProT, Roll (24x1 each)
- ENCODE ChIP-seq peak files (e.g. data/encode_101, wgEncodeEH002288) with GC-matched background

## Outputs

- Trained TensorFlow CNN model under output/encode_101/gc_match/<dataset>/Seq/CNN or .../DNAShape/CNN
- Test_result.txt with AUC of TF–DNA binding prediction on test data
- Binding-specificity predictions in [0,1] from the sigmoid output neuron, feeding motif/TFBS identification

## How to apply

Implement the convolutional layer in TensorFlow with 16 filters: use 24x4 kernels for the one-hot sequence input (101 bp peaks, i.e. --peak_flank 50) and 24x1 kernels for each of the four HelT, MGW, ProT, and Roll shape channels. Follow the convolution with ReLU activation and max pooling, then a 32-neuron fully connected layer and a single sigmoid output unit whose value (formula 3) is the predicted TF–DNA binding specificity. Configure the model to accept the six input modes: sequence alone (S), each individual shape feature, and the combined [H,M,P,R] set, selected via the --feature_format argument (Seq or DNAShape) of train.py. Train per dataset with train.py and record AUC in Test_result.txt; the trained model enables predict.py to generate motif cutoffs (start 0.01, end 1, step 0.03).

## Related tools

- **DESSO** (Framework providing the CNN architecture (16 convolution filters, ReLU + max pooling, 32-neuron FC layer, single sigmoid output) trained via train.py and used for motif prediction via predict.py) — https://github.com/OSU-BMBL/DESSO
- **TensorFlow** (Deep learning backend (TensorFlow 1.1.0) in which the convolutional, pooling, fully connected, and sigmoid output layers are implemented) — https://www.tensorflow.org/install/

## Examples

```
python train.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq
```

## Evaluation signals

- All parameter shapes match the specification: 16 convolution kernels of 24x4 (Seq) or 24x1 (each shape channel), 32 FC neuron weights, single output weight
- A forward pass on synthetic one-hot encoded input completes without error and the sigmoid output falls in [0,1]
- train.py produces Test_result.txt containing a test-set AUC for the trained model on the wgEncodeEH002288-style dataset
- All six input modes (S, each individual shape feature, and [H,M,P,R]) are accepted and train independently
- With --feature_format DNAShape, predict.py yields four separate shape motifs (HelT, MGW, ProT, Roll) in output/encode_101/gc_match/<dataset>/DNAShape/CNN/0-3

## Limitations

- The plain CNN is designed for short 101-bp peaks; for long DNA sequences the Gated-CNN (GCNN) variant with peak_flank 500 must be used instead
- The architecture assumes each motif detector position contributes independently to binding affinity, which may underestimate nucleotide dependencies
- Environment is pinned to legacy dependencies (TensorFlow 1.1.0, CUDA 8.0.44, Python 2.7, Biopython 1.7.0), so reimplementation on modern stacks must re-validate shapes and outputs
- DL-based motif prediction is intrinsically limited by the availability of large-scale sequencing and protein–DNA interaction data including ChIP-seq

## Evidence
