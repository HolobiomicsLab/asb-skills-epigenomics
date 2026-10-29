---
name: cnn-model-training-for-tf-dna-binding
description: Use when you have GC-matched ChIP-seq peak/background data (e.g. the
  690 ENCODE datasets, 101-bp or 1001-bp windows) and need to train a CNN classifier
  of TF-DNA binding on sequence or DNAShape features.
when_to_use_negative:
- Your goal is motif discovery from custom peaks without a labeled GC-matched background
  for supervised training — use the processing_peaks.py + predict.py custom-peak path
  instead.
- You need to compare against non-deep-learning motif finders (MEME-ChIP, KMAC, gkm-SVM)
  on the same peaks rather than train a binding classifier.
- Your input is already a trained model or feature table; this skill covers training
  from raw peak/sequence or shape-feature inputs only.
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3474
tools:
- name: DESSO
  role: Deep learning framework whose train.py trains the CNN/GCNN TF-DNA binding
    classifier on the ENCODE ChIP-seq datasets
  repo: https://github.com/OSU-BMBL/DESSO
- name: TensorFlow
  role: Deep learning backend (version 1.1.0, with CUDA 8.0.44) required to run DESSO
    model training
- name: scikit-learn
  role: Python dependency of DESSO used in the training/evaluation pipeline
- name: DNAshapeR
  role: Computes the four DNA shape features (HelT, MGW, ProT, Roll) used as DNAShape
    input to the CNN
- name: bedtools
  role: Extracts DNA sequences from peak BED files when training on custom ChIP-seq
    peaks
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:3a57c6f40464fb28cbc6c358eb5dfee954aef8c604aa4170b7d512875d5bcb60
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
  - DNAshapeR
  - TensorFlow
  - bedtools
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cnn-model-training-for-tf-dna-binding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a DESSO convolutional neural network (CNN) on ENCODE ChIP-seq peak regions to predict TF-DNA binding specificity, using either one-hot DNA sequence or the four DNA shape features (HelT, MGW, ProT, Roll) as input, and record test AUCs for downstream motif and shape-motif prediction.

## When to use

Use when you have GC-matched ChIP-seq peak/background data (e.g. the 690 ENCODE datasets, 101-bp or 1001-bp windows) and need to train a CNN classifier of TF-DNA binding on sequence or DNAShape features.

## When NOT to use

- Your goal is motif discovery from custom peaks without a labeled GC-matched background for supervised training — use the processing_peaks.py + predict.py custom-peak path instead.
- You need to compare against non-deep-learning motif finders (MEME-ChIP, KMAC, gkm-SVM) on the same peaks rather than train a binding classifier.
- Your input is already a trained model or feature table; this skill covers training from raw peak/sequence or shape-feature inputs only.

## Inputs

- GC-matched ENCODE ChIP-seq peak dataset (data/encode_101 for 101-bp or data/encode_1001 for 1001-bp windows)
- Reference genome FASTA (GRCh37.p13.genome.fa) and encode_101_background placed in data/
- Dataset index range selecting among the 690 ENCODE ChIP-seq datasets
- DNA shape features (HelT, MGW, ProT, Roll) computed with DNAshapeR when --feature_format DNAShape is used

## Outputs

- Trained CNN/GCNN model under /output/encode_101/gc_match/<dataset>/<feature_format>/CNN (or /output/encode_1001/gc_match/<dataset>/Seq/GCNN)
- Test_result.txt with the area under the ROC curve (AUC) of the trained model on held-out test data
- Per-feature shape motif detectors in output/encode_101/gc_match/<dataset>/DNAShape/CNN/0-3 (HelT, MGW, ProT, Roll)

## How to apply

Prepare GC-matched peak data under data/ (e.g. encode_101 or encode_1001 with GRCh37.p13.genome.fa reference), then invoke DESSO's train.py with --start_index/--end_index to select among the 690 ENCODE ChIP-seq datasets. Use --peak_flank 50 for 101-bp windows (2*50+1) with the one-layer CNN baseline, or --peak_flank 500 for 1001-bp windows when running the gated GCNN network. Choose --feature_format Seq for one-hot DNA sequence input or DNAShape for the combined HelT/MGW/ProT/Roll shape features computed with DNAshapeR. Trained models and the test AUC (Test_result.txt) are written to /output/encode_101/gc_match/<dataset>/<feature_format>/CNN (or encode_1001/.../GCNN); for shape runs, per-feature motif detectors appear in subfolders 0-3 corresponding to HelT, MGW, ProT and Roll. Repeat training across the dataset index range and aggregate per-dataset AUCs to benchmark against published averages (e.g. shape-feature AUCs HelT 0.72, MGW 0.66, ProT 0.69, Roll 0.75, combination 0.81).

## Related tools

- **DESSO** (Deep learning framework whose train.py trains the CNN/GCNN TF-DNA binding classifier on the ENCODE ChIP-seq datasets) — https://github.com/OSU-BMBL/DESSO
- **TensorFlow** (Deep learning backend (version 1.1.0, with CUDA 8.0.44) required to run DESSO model training)
- **scikit-learn** (Python dependency of DESSO used in the training/evaluation pipeline)
- **DNAshapeR** (Computes the four DNA shape features (HelT, MGW, ProT, Roll) used as DNAShape input to the CNN)
- **bedtools** (Extracts DNA sequences from peak BED files when training on custom ChIP-seq peaks)

## Examples

```
cd code/ && python train.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq
```

## Evaluation signals

- Test_result.txt is produced in the expected output directory (/output/encode_101/gc_match/<dataset>/<feature_format>/CNN or /output/encode_1001/gc_match/<dataset>/Seq/GCNN) and contains a test AUC.
- With --peak_flank 50 the effective peak length must be 101 bp (2*50+1); with --peak_flank 500 it must be 1001 bp (2*500+1).
- Averaged AUCs over the 690 datasets should be compared to reported baselines (e.g. shape-feature AUCs HelT 0.72, MGW 0.66, ProT 0.69, Roll 0.75, combination 0.81).
- For DNAShape runs, four shape-motif output folders (0-3) must appear, mapping to HelT, MGW, ProT and Roll respectively.
- Index bounds --start_index/--end_index must map to the intended ENCODE datasets (e.g. 0-1 is wgEncodeEH002288; 1-3 are the second and third datasets).

## Limitations

- DESSO depends on specific legacy versions: TensorFlow 1.1.0, CUDA 8.0.44, Python 2.7, Biopython 1.7.0.
- The GitHub repository ships only wgEncodeEH002288 example data for encode_101/encode_1001/TfbsUniform_hg19_ENCODE; the full datasets (~5.9 GB) require a separate download.
- The application of deep learning to motif prediction is intrinsically limited by the availability of large-scale sequencing and protein-DNA interaction data including ChIP-seq.
- Existing DL-based methods assume each position in a motif detector contributes independently, which may underestimate nucleotide dependencies.
- Extending the one-layer CNN baseline with more convolutional layers improved shape-feature performance (Wilcoxon P < 5e-6) but showed little or negative effect on DNA sequence input.

## Evidence

- [results] GCNN improvement finding: [evidence span withheld — anchor status: not_found]
