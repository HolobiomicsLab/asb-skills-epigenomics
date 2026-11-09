---
name: tf-dna-binding-auc-evaluation
description: Use when quantifying how well individual or combined DNA shape features
  (HelT, MGW, ProT, Roll) or sequences predict TF-DNA binding specificity across ENCODE
  ChIP-seq datasets, using test AUC from a trained DESSO CNN.
when_to_use_negative:
- Input is already a trained model and the goal is motif discovery rather than binding-specificity
  evaluation (use DESSO predict.py with cutoff scanning instead)
- Peak regions longer than 101 bp are needed — GCNN with peak_flank 500 (1001-bp peaks)
  is the appropriate DESSO mode, not the 101-bp CNN AUC setup
- The task is motif similarity assessment against JASPAR/TRANSFAC via TOMTOM rather
  than classification performance (AUC) measurement
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3474
tools:
- name: DESSO
  role: Framework used to train CNN models per dataset with --feature_format Seq or
    DNAShape and report test AUC in Test_result.txt
  repo: https://github.com/OSU-BMBL/DESSO
- name: DNAshapeR
  role: Computes the four DNA shape features (HelT, MGW, ProT, Roll) for the 101-bp
    peak regions used as DNAShape input
- name: TensorFlow
  role: Deep learning backend for DESSO CNN model training (version 1.1.0)
- name: scikit-learn
  role: Dependency used by DESSO for model evaluation including AUC computation
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:f82cfb813b5496c3b313854b9cd23ac80388d16a2b98d0d7fc670bccb74ac320
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
  - DESSO
  - DNAshapeR
  - DeepBind
  - KMAC
  - TOMTOM
  - TensorFlow
  - gkm-SVM
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# tf-dna-binding-auc-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate TF-DNA binding specificity prediction by training DESSO CNN models on 101-bp ENCODE ChIP-seq peak regions (peak_flank 50) with either sequence (Seq) or DNA shape (HelT/MGW/ProT/Roll, DNAShape) inputs, and reading test-set AUCs from Test_result.txt as the performance metric.

## When to use

Use when quantifying how well individual or combined DNA shape features (HelT, MGW, ProT, Roll) or sequences predict TF-DNA binding specificity across ENCODE ChIP-seq datasets, using test AUC from a trained DESSO CNN.

## When NOT to use

- Input is already a trained model and the goal is motif discovery rather than binding-specificity evaluation (use DESSO predict.py with cutoff scanning instead)
- Peak regions longer than 101 bp are needed — GCNN with peak_flank 500 (1001-bp peaks) is the appropriate DESSO mode, not the 101-bp CNN AUC setup
- The task is motif similarity assessment against JASPAR/TRANSFAC via TOMTOM rather than classification performance (AUC) measurement

## Inputs

- 690 human ENCODE ChIP-seq peak datasets (encode_101, 101-bp peaks around summits, peak_flank 50)
- GC-matched background sets (encode_101_background)
- Reference genome GRCh37.p13.genome.fa
- DNA shape features (HelT, MGW, ProT, Roll) computed with DNAshapeR for peak regions

## Outputs

- Per-dataset test AUC values in Test_result.txt under /output/encode_101/gc_match/<dataset>/<Seq|DNAShape>/CNN
- Trained DESSO CNN models per dataset and feature format
- Aggregated shape AUC summary (e.g. shape_auc_summary.csv) with average AUCs per shape feature and combination
- Per-shape-feature max-pooling filter contribution percentages

## How to apply

Extract 101-bp peak regions (2*50+1 bp around each summit, --peak_flank 50) from the 690 ENCODE ChIP-seq datasets (encode_101) against the gc-matched encode_101_background using GRCh37.p13. Train a per-dataset CNN with DESSO train.py, setting --feature_format to Seq for sequence input or DNAShape for the combined four shape features (HelT, MGW, ProT, Roll); for single-feature evaluations train one model per shape feature. Collect the test AUC from the Test_result.txt file under output/encode_101/gc_match/<dataset>/<Seq|DNAShape>/CNN for each dataset and feature format, then average over datasets. Compare against the reported values (HelT 0.72, MGW 0.66, ProT 0.69, Roll 0.75, combined shape 0.81) and extract max-pooling filter contributions per shape feature (reported 32/9/22/37% for HelT/MGW/ProT/Roll) to quantify each feature's signal contribution.

## Related tools

- **DESSO** (Framework used to train CNN models per dataset with --feature_format Seq or DNAShape and report test AUC in Test_result.txt) — https://github.com/OSU-BMBL/DESSO
- **DNAshapeR** (Computes the four DNA shape features (HelT, MGW, ProT, Roll) for the 101-bp peak regions used as DNAShape input)
- **TensorFlow** (Deep learning backend for DESSO CNN model training (version 1.1.0))
- **scikit-learn** (Dependency used by DESSO for model evaluation including AUC computation)

## Examples

```
python train.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format DNAShape
```

## Evaluation signals

- Test_result.txt exists for every dataset under /output/encode_101/gc_match/<dataset>/DNAShape/CNN and contains a test AUC
- Averaged AUCs over the 690 datasets approximate the reported values: HelT 0.72, MGW 0.66, ProT 0.69, Roll 0.75, and combined shape 0.81
- Relative ordering holds: HelT and Roll surpass MGW and ProT in classification performance
- Max-pooling filter contributions per shape feature approximate the reported 32/9/22/37% (HelT/MGW/ProT/Roll)
- Peak length is confirmed as (2 * 50 + 1) = 101 base pairs, matching the encode_101 dataset definition

## Limitations

- The role of DNA shape in explaining TF-DNA binding is still in debate because sequence and shape features covary
- Repo ships only wgEncodeEH002288 example data in data/encode_101, encode_1001, and TfbsUniform_hg19_ENCODE; full datasets (~5.9GB) require a separate download
- Requires legacy environment: Python 2.7, TensorFlow 1.1.0, CUDA 8.0.44, Biopython 1.7.0

## Evidence
