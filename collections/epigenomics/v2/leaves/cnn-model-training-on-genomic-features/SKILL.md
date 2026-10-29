---
name: cnn-model-training-on-genomic-features
description: Use when training supervised deep learning classifiers on ENCODE ChIP-seq
  peak data — either DNA sequence or DNA shape feature matrices — to later extract
  motifs and TFBSs, e.g. reproducing DESSO's 690-dataset benchmark.
when_to_use_negative:
- Goal is single-dataset exploratory peak calling or motif scanning with existing
  motif databases (MEME/TOMTOM only) rather than de novo deep learning model training
- Peaks are longer than 1001 bp or the region of interest requires long-range context
  — use DESSO's GCNN with --peak_flank 500 instead of CNN
- No GC-matched background or reference genome is available, since DESSO's training
  relies on gc_match data/encode_101 and GRCh37.p13.genome.fa
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3474
tools:
- name: DESSO
  role: Provides train.py, processing_peaks.py, and the CNN architecture used to train
    per-dataset models on Seq or DNAShape inputs
  repo: https://github.com/OSU-BMBL/DESSO
- name: TensorFlow
  role: Deep learning backend (version 1.1.0) executing the CNN training
- name: Biopython
  role: Dependency (version 1.7.0) supporting sequence handling in the DESSO pipeline
- name: scikit-learn
  role: Dependency used within DESSO's training/evaluation utilities
- name: bedtools
  role: Extracts DNA sequences from the peak file when preparing custom ChIP-seq inputs
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:741ff69e39d0e6560f3fe12df544afd46a68454d03177e328c72fbe306e52985
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
  - Basset
  - Biopython
  - DESSO
  - DeepBind
  - KMAC
  - TOMTOM
  - TensorFlow
  - bedtools
  - gkm-SVM
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# cnn-model-training-on-genomic-features

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train per-dataset convolutional neural network (CNN) models with DESSO on GC-matched ENCODE ChIP-seq peak inputs, using either one-hot DNA sequences (Seq) or the four DNA shape features (HelT, MGW, ProT, Roll) extracted from 101-bp (2*50+1 bp) peak regions around summits.

## When to use

Use when training supervised deep learning classifiers on ENCODE ChIP-seq peak data — either DNA sequence or DNA shape feature matrices — to later extract motifs and TFBSs, e.g. reproducing DESSO's 690-dataset benchmark.

## When NOT to use

- Goal is single-dataset exploratory peak calling or motif scanning with existing motif databases (MEME/TOMTOM only) rather than de novo deep learning model training
- Peaks are longer than 1001 bp or the region of interest requires long-range context — use DESSO's GCNN with --peak_flank 500 instead of CNN
- No GC-matched background or reference genome is available, since DESSO's training relies on gc_match data/encode_101 and GRCh37.p13.genome.fa

## Inputs

- ENCODE ChIP-seq peak dataset (e.g. encode_101, one of 690 human datasets such as wgEncodeEH002288)
- GC-matched background sets (encode_101_background)
- Reference genome FASTA (GRCh37.p13.genome.fa)
- BED file of ChIP-seq peaks with summits (for custom analysis, e.g. fox01_peaks.bed)
- DNA shape feature matrices (HelT, MGW, ProT, Roll) for the 101-bp peak regions (when --feature_format DNAShape)

## Outputs

- Trained CNN model per dataset (e.g. output/encode_101/gc_match/wgEncodeEH002288/Seq/CNN or .../DNAShape/CNN)
- Test_result.txt with the model's test-set ROC AUC for TF-DNA binding specificity

## How to apply

Prepare GC-matched peak/background sets (encode_101 with GRCh37.p13.genome.fa and encode_101_background) where each peak region is 2*50+1 = 101 bp around the summit (peak_flank 50). For each dataset index in the 690 ENCODE ChIP-seq collection, run DESSO's train.py with --network CNN and --feature_format Seq (one-hot sequences) or --feature_format DNAShape (concatenated HelT, MGW, ProT, Roll matrices). If preparing custom peaks, first process a BED file with processing_peaks.py --peak_flank 50 and extract sequences with bedtools. Record each model's test AUC from the generated Test_result.txt to confirm the classifier learned TF-DNA binding specificity before proceeding to motif prediction with predict.py.

## Related tools

- **DESSO** (Provides train.py, processing_peaks.py, and the CNN architecture used to train per-dataset models on Seq or DNAShape inputs) — https://github.com/OSU-BMBL/DESSO
- **TensorFlow** (Deep learning backend (version 1.1.0) executing the CNN training)
- **Biopython** (Dependency (version 1.7.0) supporting sequence handling in the DESSO pipeline)
- **scikit-learn** (Dependency used within DESSO's training/evaluation utilities)
- **bedtools** (Extracts DNA sequences from the peak file when preparing custom ChIP-seq inputs)

## Examples

```
python train.py --start_index 0 --end_index 690 --peak_flank 50 --network CNN --feature_format DNAShape
```

## Evaluation signals

- Test_result.txt reports a test-set AUC indicating the CNN discriminates bound from GC-matched background peaks; a near-0.5 AUC signals training failure
- Output directory structure matches the expected layout: output/encode_101/gc_match/<dataset>/<Seq|DNAShape>/CNN
- Peak length invariant: with --peak_flank 50 each input region is exactly 2*50+1 = 101 bp; with GCNN and --peak_flank 500 it is 1001 bp
- Reproduction check: trained models across the index range 0–690 must cover all 690 ENCODE datasets before downstream motif prediction
- For DNAShape training, the model consumes exactly four features (HelT, MGW, ProT, Roll), and downstream predict.py must yield four corresponding shape-motif directories (CNN/0–CNN/3)

## Limitations

- The README ships only wgEncodeEH002288-related example data due to file size limits; the full ~5.9GB code+data bundle must be downloaded for all 690 datasets
- DESSO training is pinned to an old stack (TensorFlow 1.1.0, CUDA 8.0.44, Python 2.7, Biopython 1.7.0), which may not run on modern environments
- Existing DL-based methods, including DESSO, assume each position in a motif detector contributes independently, which may underestimate nucleotide dependencies
- Applying DL to motif prediction is intrinsically limited by the availability of large-scale sequencing and protein–DNA interaction data

## Evidence
