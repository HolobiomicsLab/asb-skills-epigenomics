---
name: dna-shape-feature-input-handling
description: Use when training or evaluating DESSO CNN models on ChIP-seq peaks with
  --feature_format DNAShape, or when reconstructing the shape-kernel CNN that uses
  24x1 kernels per shape channel instead of 24x4 sequence kernels.
when_to_use_negative:
- When the goal is pure sequence motif discovery only — use --feature_format Seq with
  the 24x4 kernel architecture instead of shape channels
- When working with very long input regions (e.g., 1001-bp peaks with peak_flank 500),
  where the Gated-CNN (GCNN) architecture, not this plain CNN shape input, is the
  specified model
- When data is insufficient for deep learning — the paper notes DL-based motif prediction
  is intrinsically limited by availability of large-scale ChIP-seq data
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0622
tools:
- name: DESSO
  role: Framework implementing the CNN/GCNN that consumes sequence and DNA shape (HelT,
    MGW, ProT, Roll) inputs to predict sequence and shape motifs
  repo: https://github.com/OSU-BMBL/DESSO
- name: TensorFlow
  role: Backend for constructing the one-layer CNN (16 filters, 24x4 sequence kernels
    / 24x1 shape kernels, ReLU, max pooling, 32-neuron FC, single sigmoid output)
- name: bedtools
  role: Extracts DNA sequences from the user's peak file before feature encoding
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:a688ea385225237d2bf06f49cfdf4232ce061d9d8d04e04f2228c5670254bb89
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
  - bedtools
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# dna-shape-feature-input-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes four DNA shape features (HelT, MGW, ProT, Roll) from ChIP-seq peak regions as input channels for the DESSO one-layer CNN, alongside or instead of one-hot sequence input, to quantitatively assess shape contributions to TF–DNA binding.

## When to use

Use when training or evaluating DESSO CNN models on ChIP-seq peaks with --feature_format DNAShape, or when reconstructing the shape-kernel CNN that uses 24x1 kernels per shape channel instead of 24x4 sequence kernels.

## When NOT to use

- When the goal is pure sequence motif discovery only — use --feature_format Seq with the 24x4 kernel architecture instead of shape channels
- When working with very long input regions (e.g., 1001-bp peaks with peak_flank 500), where the Gated-CNN (GCNN) architecture, not this plain CNN shape input, is the specified model
- When data is insufficient for deep learning — the paper notes DL-based motif prediction is intrinsically limited by availability of large-scale ChIP-seq data

## Inputs

- ENCODE ChIP-seq peak BED files (e.g., TfbsUniform_hg19_ENCODE) with peak summits
- Genome reference (GRCh37.p13.genome.fa or hg38.fa) for extracting peak sequences
- Four DNA shape feature channels per peak: HelT, MGW, ProT, Roll (regional shape features)
- One-hot encoded DNA sequence input (101 bp peaks by default with peak_flank 50)

## Outputs

- Trained CNN model saved under output/encode_101/gc_match/<dataset>/DNAShape/CNN
- Test_result.txt reporting the AUC of TF–DNA binding specificity prediction on test data
- Four shape motif files: HelT motif (CNN/0), MGW motif (CNN/1), ProT motif (CNN/2), Roll motif (CNN/3)
- Sigmoid prediction values in [0,1] per peak

## How to apply

Prepare peak inputs centered on the summit with a --peak_flank (default 50, giving 101-bp peaks) and encode the four regional DNA shape features — HelT, MGW, ProT, Roll — rather than one-hot nucleotide sequences. In the TensorFlow CNN, feed each shape channel through a convolutional layer with 16 filters using 24x1 kernels (versus 24x4 kernels for sequence input), followed by ReLU activation, max pooling, a 32-neuron fully connected layer, and a single sigmoid output unit predicting binding specificity in [0,1]. Select the input mode via --feature_format: 'Seq' for sequences, 'DNAShape' for the combined [H,M,P,R] shape set; the framework also supports individual shape features as separate inputs. After training, run predict.py with cutoff stepping (default --start_cutoff 0.01, --end_cutoff 1, --step_cutoff 0.03) to extract shape motifs, which are written separately for HelT (CNN/0), MGW (CNN/1), ProT (CNN/2), and Roll (CNN/3).

## Related tools

- **DESSO** (Framework implementing the CNN/GCNN that consumes sequence and DNA shape (HelT, MGW, ProT, Roll) inputs to predict sequence and shape motifs) — https://github.com/OSU-BMBL/DESSO
- **TensorFlow** (Backend for constructing the one-layer CNN (16 filters, 24x4 sequence kernels / 24x1 shape kernels, ReLU, max pooling, 32-neuron FC, single sigmoid output))
- **bedtools** (Extracts DNA sequences from the user's peak file before feature encoding)

## Examples

```
python train.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format DNAShape
```

## Evaluation signals

- All parameter shapes match the specification: 16 filters, 24x4 kernels for one-hot sequence input and 24x1 kernels per HelT/MGW/ProT/Roll shape channel, 32-neuron FC layer, single sigmoid output unit
- A forward pass on a synthetic encoded input completes without error and the sigmoid prediction falls within [0,1]
- Test_result.txt reports a finite AUC for the trained model on held-out test data
- With --feature_format DNAShape, exactly four motif outputs are produced, one per shape channel, in the expected CNN/0–CNN/3 directories
- Model artifacts appear at output/encode_101/gc_match/<dataset>/DNAShape/CNN when shape mode was used (Seq path otherwise)

## Limitations

- Sequence and shape features covary, so the role of DNA shape in explaining TF–DNA binding remains debated and shape contributions may be confounded with sequence effects
- Existing DL-based methods, including this CNN, assume each position in a motif detector contributes to TF–DNA binding affinity independently, which may underestimate nucleotide dependencies
- Application of DL to motif prediction is limited by availability of large-scale sequencing and protein–DNA interaction data such as ChIP-seq, since experiments are expensive
- Predicted indirect TF–motif interactions and shape contributions await experimental validation

## Evidence
