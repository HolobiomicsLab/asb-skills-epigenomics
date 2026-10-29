---
name: tensorflow-model-shape-verification
description: Use when reimplementing, auditing, or extending the DESSO motif-prediction
  CNN and you must verify that filter kernels, fully connected weights, and output-layer
  weights match the published layer dimensions.
when_to_use_negative:
- The goal is to train the Gated-CNN (GCNN) on long 1001 bp sequences rather than
  verify the base CNN — use the GCNN workflow with --peak_flank 500 instead
- Input is already a trained model checkpoint and the task is motif prediction (predict.py)
  rather than architecture verification
- The framework or TensorFlow version differs substantially (DESSO was built against
  TensorFlow 1.1.0 and Python 2.7), so shape conventions may not transfer directly
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DESSO
  role: Framework whose one-layer CNN architecture (filters, kernels, FC and sigmoid
    output layers) is reconstructed and shape-verified
  repo: https://github.com/OSU-BMBL/DESSO
- name: TensorFlow
  role: Backend in which the CNN layers are implemented and their parameter shapes
    asserted (Tensorflow 1.1.0 per README)
  repo: https://www.tensorflow.org/install/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:1205207a256a287a2c059a5103d350523fca22a33e5d838f30c2cc568f2a92ac
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# tensorflow-model-shape-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the one-layer DESSO CNN architecture in TensorFlow (16 convolutional filters, ReLU, max pooling, a 32-neuron fully connected layer, and a single sigmoid output unit) and asserts that every parameter shape matches the published specification for sequence and DNA shape inputs.

## When to use

Use when reimplementing, auditing, or extending the DESSO motif-prediction CNN and you must verify that filter kernels, fully connected weights, and output-layer weights match the published layer dimensions.

## When NOT to use

- The goal is to train the Gated-CNN (GCNN) on long 1001 bp sequences rather than verify the base CNN — use the GCNN workflow with --peak_flank 500 instead
- Input is already a trained model checkpoint and the task is motif prediction (predict.py) rather than architecture verification
- The framework or TensorFlow version differs substantially (DESSO was built against TensorFlow 1.1.0 and Python 2.7), so shape conventions may not transfer directly

## Inputs

- One-hot encoded DNA sequence matrix for 101 bp ChIP-seq peaks (Seq feature format)
- Per-position DNA shape feature channels: HelT, MGW, ProT, Roll (DNAShape feature format)
- DESSO model specification: 16 filters, 24x4 (sequence) / 24x1 (shape) kernels, 32-neuron FC layer, single sigmoid output

## Outputs

- Verified TensorFlow CNN parameter set whose kernel, FC, and output weight shapes match the DESSO specification
- Sigmoid output scalar in [0,1] from a forward pass on a synthetic one-hot input
- Trained model directory plus Test_result.txt reporting test AUC for TF-DNA binding specificity

## How to apply

Implement the convolutional layer in TensorFlow 1.1.0 with 16 filters: use 24x4 kernels for the one-hot encoded sequence input (peaks are 2*peak_flank+1 = 101 bp long by default with --peak_flank 50) and 24x1 kernels for each of the four DNA shape channels (HelT, MGW, ProT, Roll). Follow the convolution with ReLU activation and max pooling, then a 32-neuron fully connected layer and a single output unit passed through a sigmoid so the prediction lies in [0,1]. Configure the model to accept six input modes: sequence alone (Seq) and individual or combined shape feature sets (DNAShape). Assert all parameter shapes (filter kernels, FC weights, output weights) against these dimensions, then run a forward pass on a synthetic one-hot input and check the sigmoid output falls in [0,1]. The trained model's AUC on test data is reported in Test_result.txt, which serves as an end-to-end sanity check after verification.

## Related tools

- **DESSO** (Framework whose one-layer CNN architecture (filters, kernels, FC and sigmoid output layers) is reconstructed and shape-verified) — https://github.com/OSU-BMBL/DESSO
- **TensorFlow** (Backend in which the CNN layers are implemented and their parameter shapes asserted (Tensorflow 1.1.0 per README)) — https://www.tensorflow.org/install/

## Examples

```
python train.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq
```

## Evaluation signals

- Every parameter shape assertion passes: 16 convolution filters of size 24x4 (sequence) or 24x1 (each shape channel), FC layer weights of 32 neurons, single output weight
- A forward pass on a synthetic one-hot encoded input completes without error
- The sigmoid output prediction falls within [0,1] as expected from formula 3
- The model accepts all six input modes (Seq plus HelT/MGW/ProT/Roll and combined DNAShape) without shape errors
- After training, Test_result.txt reports a valid test AUC for TF-DNA binding specificity prediction

## Limitations

- Existing DL-based methods assume that each position in a motif detector contributes to TF-DNA binding affinity independently, which may underestimate nucleotide dependencies
- Application of deep learning to motif prediction is intrinsically limited by the availability of large-scale sequencing and protein-DNA interaction data such as ChIP-seq
- The repo ships example data only for wgEncodeEH002288 due to file size limits; full datasets (~5.9GB) must be downloaded separately for the 690 ENCODE datasets

## Evidence
