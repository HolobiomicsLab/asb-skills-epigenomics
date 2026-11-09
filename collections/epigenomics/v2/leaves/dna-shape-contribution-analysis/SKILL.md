---
name: dna-shape-contribution-analysis
description: Use when quantifying how much each DNA shape feature (HelT, MGW, ProT,
  Roll) contributes to predicting TF-DNA binding specificity across many ChIP-seq
  datasets, or when comparing shape-feature AUCs against sequence-only baselines.
when_to_use_negative:
- The goal is sequence-motif discovery rather than shape-feature contribution quantification
  — use --feature_format Seq and predict.py for sequence motifs instead
- Only one or a handful of ChIP-seq datasets are available — per-dataset AUCs will
  be noisy and cannot be meaningfully averaged over the 690-dataset benchmark
- Peak regions longer than 101 bp (e.g. peak_flank 500 with GCNN) are required — this
  skill is calibrated for the CNN/101-bp DNAShape setup
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3474
tools:
- name: DESSO
  role: Deep learning framework used to train CNN models on DNA shape features and
    extract max-pooling filter contributions per shape feature
  repo: https://github.com/OSU-BMBL/DESSO
- name: DNAshapeR
  role: Computes the four DNA shape features (HelT, MGW, ProT, Roll) for the 101-bp
    peak regions
- name: TensorFlow
  role: Backend for training the DESSO CNN models on DNAShape feature inputs
- name: scikit-learn
  role: Dependency used for evaluation (test AUC) of trained DESSO models
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:d344733c04f928546934db80d0474d704d0bd8ce2ca4b6bf99d578c7fe60853f
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
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# dna-shape-contribution-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains DESSO CNN models on the four DNA shape features (HelT, MGW, ProT, Roll) individually and combined across the 690 ENCODE ChIP-seq datasets to quantify each feature's predictive power for TF-DNA binding specificity, yielding average AUCs (HelT 0.72, MGW 0.66, ProT 0.69, Roll 0.75, combination 0.81) and per-feature filter contributions (32/9/22/37%).

## When to use

Use when quantifying how much each DNA shape feature (HelT, MGW, ProT, Roll) contributes to predicting TF-DNA binding specificity across many ChIP-seq datasets, or when comparing shape-feature AUCs against sequence-only baselines.

## When NOT to use

- The goal is sequence-motif discovery rather than shape-feature contribution quantification — use --feature_format Seq and predict.py for sequence motifs instead
- Only one or a handful of ChIP-seq datasets are available — per-dataset AUCs will be noisy and cannot be meaningfully averaged over the 690-dataset benchmark
- Peak regions longer than 101 bp (e.g. peak_flank 500 with GCNN) are required — this skill is calibrated for the CNN/101-bp DNAShape setup

## Inputs

- 690 ENCODE ChIP-seq peak datasets (encode_101, 101-bp peak regions, gc-matched background)
- GRCh37.p13.genome.fa reference genome
- Four DNA shape feature tracks (HelT, MGW, ProT, Roll) computed with DNAshapeR
- Trained DESSO CNN models (--feature_format DNAShape)

## Outputs

- Test_result.txt per dataset with test AUC of the trained shape model
- Max-pooling filter contributions per shape feature
- Aggregated shape_auc_summary.csv with mean AUCs and contribution percentages

## How to apply

Compute the four DNA shape features with DNAshapeR for the 101-bp peak regions (peak_flank 50, i.e. 2*50+1 bp) of all 690 ENCODE ChIP-seq datasets against the gc-matched background. Train a DESSO CNN per dataset on each single shape feature and on the combined HelT/MGW/ProT/Roll input using train.py with --peak_flank 50 --network CNN --feature_format DNAShape. Collect test AUCs from the Test_result.txt files under /output/encode_101/gc_match/<dataset>/DNAShape/CNN. Extract max-pooling filter contributions per shape feature from the trained models to quantify each feature's signal contribution, then average AUCs and contributions over the 690 datasets and compare against the reported values (HelT 0.72, MGW 0.66, ProT 0.69, Roll 0.75, combination 0.81; contributions 32/9/22/37%).

## Related tools

- **DESSO** (Deep learning framework used to train CNN models on DNA shape features and extract max-pooling filter contributions per shape feature) — https://github.com/OSU-BMBL/DESSO
- **DNAshapeR** (Computes the four DNA shape features (HelT, MGW, ProT, Roll) for the 101-bp peak regions)
- **TensorFlow** (Backend for training the DESSO CNN models on DNAShape feature inputs)
- **scikit-learn** (Dependency used for evaluation (test AUC) of trained DESSO models)

## Examples

```
python train.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format DNAShape
```

## Evaluation signals

- Average AUCs over the 690 datasets approximate the reported values: HelT 0.72, MGW 0.66, ProT 0.69, Roll 0.75, and combined DNAShape 0.81
- Per-feature max-pooling filter contributions approximate the reported 32% (HelT), 9% (MGW), 22% (ProT), 37% (Roll)
- Test_result.txt exists under /output/encode_101/gc_match/<dataset>/DNAShape/CNN for every dataset and contains a valid AUC
- HelT and Roll should surpass MGW and ProT in per-feature classification performance
- Combined-feature AUC (0.81) should exceed every single-feature AUC

## Limitations

- The role of DNA shape in explaining TF-DNA binding is still in debate, which mainly stems from the fact that sequence and shape features covary
- Inferred shape contributions and indirect TF-motif interaction mechanisms are predictions that await experimental validation
- The repository ships only wgEncodeEH002288-related example data in data/encode_101; the full 690-dataset set (~5.9GB) must be downloaded separately

## Evidence
