---
name: dna-shape-feature-generation
description: Use when you need to compute HelT/MGW/ProT/Roll shape features for ChIP-seq
  peak regions (2*peak_flank+1 bp, e.g. peak_flank 50 = 101 bp) and evaluate their
  predictive power for TF-DNA binding versus a gc-matched background.
when_to_use_negative:
- Your goal is sequence motif discovery from DNA sequences only — use --feature_format
  Seq instead of DNAShape.
- Your peak regions are long (e.g. 1001 bp) and require the Gated-CNN (GCNN) architecture
  rather than the 101-bp CNN setup.
- Your input is already a shape feature table — this skill computes shape features
  from peak regions against a genome.
edam_topics:
- http://edamontology.org/topic_0749
tools:
- name: DESSO
  role: CNN framework trained with --feature_format DNAShape on the four shape features;
    produces Test_result.txt AUCs and shape motifs
  repo: https://github.com/OSU-BMBL/DESSO
- name: DNAshapeR
  role: Computes the four DNA shape features (HelT, MGW, ProT, Roll) for the peak
    regions used as DNAShape input
- name: TensorFlow
  role: Backend dependency for training the DESSO CNN models
- name: scikit-learn
  role: Dependency for model evaluation metrics (AUC) in the DESSO pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:fd90ded9c189098a5ed7472160c3765cd9f188c388826e46a0c6e69ab6936f62
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# dna-shape-feature-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates the four DNA shape features (HelT, MGW, ProT, Roll) for 101-bp peak regions and trains DESSO CNN models with the DNAShape feature format to quantify shape-based TF-DNA binding predictive power across the 690 ENCODE ChIP-seq datasets.

## When to use

Use when you need to compute HelT/MGW/ProT/Roll shape features for ChIP-seq peak regions (2*peak_flank+1 bp, e.g. peak_flank 50 = 101 bp) and evaluate their predictive power for TF-DNA binding versus a gc-matched background.

## When NOT to use

- Your goal is sequence motif discovery from DNA sequences only — use --feature_format Seq instead of DNAShape.
- Your peak regions are long (e.g. 1001 bp) and require the Gated-CNN (GCNN) architecture rather than the 101-bp CNN setup.
- Your input is already a shape feature table — this skill computes shape features from peak regions against a genome.

## Inputs

- 690 ENCODE ChIP-seq peak datasets (data/encode_101, e.g. wgEncodeEH002288) with peak summit coordinates
- gc-matched background sequences (encode_101_background)
- GRCh37.p13.genome.fa reference genome
- peak flank parameter (peak_flank 50, yielding 101-bp regions)

## Outputs

- HelT/MGW/ProT/Roll shape feature matrices for each peak region
- Trained DESSO CNN models under /output/encode_101/gc_match/<dataset>/DNAShape/CNN
- Test_result.txt with test AUC per dataset and feature format
- Four shape motifs per dataset (output/.../DNAShape/CNN/0-3 = HelT, MGW, ProT, Roll motifs)
- Aggregated AUC summary (e.g. shape_auc_summary.csv)

## How to apply

Compute HelT, MGW, ProT and Roll with DNAshapeR for the 101-bp peak regions of each of the 690 ENCODE ChIP-seq datasets (peak_flank 50) against the gc-matched background, then run DESSO training with --feature_format DNAShape, which feeds the combination of the four shape features into the CNN (single-feature models can be trained per shape feature for individual AUCs). Collect test AUCs from the Test_result.txt files under /output/encode_101/gc_match/<dataset>/DNAShape/CNN. For shape motifs, run predict.py with --feature_format DNAShape; the model produces four shape motifs, with output subdirectories 0-3 corresponding to HelT, MGW, ProT and Roll motifs respectively. Extract max-pooling filter contributions per shape feature to quantify each feature's signal contribution, then average AUCs over the 690 datasets and compare with reported values (HelT 0.72, MGW 0.66, ProT 0.69, Roll 0.75, combination 0.81; contributions 32/9/22/37%).

## Related tools

- **DESSO** (CNN framework trained with --feature_format DNAShape on the four shape features; produces Test_result.txt AUCs and shape motifs) — https://github.com/OSU-BMBL/DESSO
- **DNAshapeR** (Computes the four DNA shape features (HelT, MGW, ProT, Roll) for the peak regions used as DNAShape input)
- **TensorFlow** (Backend dependency for training the DESSO CNN models)
- **scikit-learn** (Dependency for model evaluation metrics (AUC) in the DESSO pipeline)

## Examples

```
cd code/ && python train.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format DNAShape
```

## Evaluation signals

- Test_result.txt exists under /output/encode_101/gc_match/<dataset>/DNAShape/CNN for every dataset and contains a test AUC.
- Averaged AUCs over 690 datasets match reported values: HelT 0.72, MGW 0.66, ProT 0.69, Roll 0.75, combined shape 0.81, with HelT and Roll surpassing MGW and ProT.
- Shape-motif output directories 0-3 map correctly to HelT, MGW, ProT, Roll motifs respectively.
- Max-pooling filter contributions per shape feature approximate the reported 32/9/22/37% split for HelT/MGW/ProT/Roll.
- Peak length check: peak_flank 50 must yield 2*50+1 = 101-bp input regions.

## Limitations

- The role of DNA shape in explaining TF-DNA binding is still in debate because sequence and shape features covary, so shape AUCs should not be interpreted as fully independent contributions.
- The GitHub repository's data/encode_101 directory only contains wgEncodeEH002288-related data as an example; the full 690-dataset set (~5.9GB) must be downloaded separately.
- Shape-based predictions of indirect TF-motif interactions await experimental validation.

## Evidence
