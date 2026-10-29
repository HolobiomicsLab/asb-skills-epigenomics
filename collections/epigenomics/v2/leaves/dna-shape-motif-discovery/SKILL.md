---
name: dna-shape-motif-discovery
description: Use when ChIP-seq peak regions and precomputed DNA shape feature matrices
  (HelT, MGW, ProT, Roll) are available and the goal is to discover shape-based regulatory
  motifs and TFBSs with DESSO.
when_to_use_negative:
- The goal is sequence motif discovery from DNA sequence input — use --feature_format
  Seq instead of DNAShape.
- Peaks are from a user's own ChIP-seq experiment without precomputed DNA shape feature
  matrices; the custom-peak workflow (processing_peaks.py with hg38.fa) targets sequence
  motifs.
- Only motif ranking/comparison against JASPAR/TRANSFAC documented motifs is needed
  without training shape-based CNN models.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0654
tools:
- name: DESSO
  role: Deep learning framework that trains CNN classifiers on DNA shape features
    and predicts shape motifs and TFBSs using the binomial distribution model
  repo: https://github.com/OSU-BMBL/DESSO
- name: TensorFlow
  role: Dependency used to train the CNN models on the DNAShape feature inputs
- name: Biopython
  role: Dependency for DESSO's sequence and feature handling
- name: scikit-learn
  role: Dependency for model training and evaluation in DESSO
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:5ef856996988014582f99fd1acae0e5879ee56227b292409f3a6b02e6fc9f668
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
  - Biopython
  - DESSO
  - TensorFlow
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# dna-shape-motif-discovery

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce DESSO's DNA shape motif discovery across the 690 human ENCODE ChIP-seq datasets by training CNN classifiers on the four DNA shape features (HelT, MGW, ProT, Roll) of 101-bp peak regions and identifying shape motifs with a binomial-distribution-based cutoff scan.

## When to use

Use when ChIP-seq peak regions and precomputed DNA shape feature matrices (HelT, MGW, ProT, Roll) are available and the goal is to discover shape-based regulatory motifs and TFBSs with DESSO.

## When NOT to use

- The goal is sequence motif discovery from DNA sequence input — use --feature_format Seq instead of DNAShape.
- Peaks are from a user's own ChIP-seq experiment without precomputed DNA shape feature matrices; the custom-peak workflow (processing_peaks.py with hg38.fa) targets sequence motifs.
- Only motif ranking/comparison against JASPAR/TRANSFAC documented motifs is needed without training shape-based CNN models.

## Inputs

- ENCODE ChIP-seq peak data for the 690 human datasets (e.g., data/encode_101, data/TfbsUniform_hg19_ENCODE)
- DNA shape feature matrices (HelT, MGW, ProT, Roll) computed for the 101-bp peak regions (peak_flank 50)
- GRCh37.p13.genome.fa and encode_101_background placed in data/

## Outputs

- Predicted shape motifs and TFBSs per dataset in output/encode_101/gc_match/<dataset>/DNAShape/CNN/{0,1,2,3} (HelT, MGW, ProT, Roll)
- shape_motif_counts.csv with per-shape-type motif counts and the number of datasets with at least one shape motif

## How to apply

Load the ENCODE ChIP-seq peak data together with the DNA shape feature matrices computed for the 101-bp peak regions (peak_flank 50, i.e., 2*50+1 base pairs around the peak summit). For each of the 690 ENCODE datasets, train a CNN classifier on the shape features via DESSO's train.py with --network CNN --feature_format DNAShape, then run predict.py with --feature_format DNAShape and the binomial-distribution-based cutoff scan (--start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03) to identify shape motifs and TFBSs. Collect predicted shape motifs from the four per-feature output directories (CNN/0 = HelT, CNN/1 = MGW, CNN/2 = ProT, CNN/3 = Roll). Tally per dataset and per shape type whether at least one motif was predicted, and aggregate total motif counts across all 690 datasets. The rationale is that shape information has strong predictive power for TF-DNA binding and DESSO's binomial distribution model replaces a subjective motif cutoff.

## Related tools

- **DESSO** (Deep learning framework that trains CNN classifiers on DNA shape features and predicts shape motifs and TFBSs using the binomial distribution model) — https://github.com/OSU-BMBL/DESSO
- **TensorFlow** (Dependency used to train the CNN models on the DNAShape feature inputs)
- **Biopython** (Dependency for DESSO's sequence and feature handling)
- **scikit-learn** (Dependency for model training and evaluation in DESSO)

## Examples

```
python predict.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format DNAShape --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- Reproduced total motif counts match the reported 1257 HelT, 84 MGW, 885 ProT, and 478 Roll motifs.
- 598 out of the 690 ENCODE datasets have at least one predicted shape motif.
- Shape motifs appear in the expected per-feature output directories: CNN/0 = HelT, CNN/1 = MGW, CNN/2 = ProT, CNN/3 = Roll.
- Each trained model's Test_result.txt reports an AUC for predicting TF-DNA binding specificity on test data.
- The cutoff scan spans 0.01 to 1 in steps of 0.03 as specified by the predict.py defaults.

## Limitations

- The role of DNA shape in explaining TF-DNA binding is still in debate because sequence and shape features covary.
- DL-based motif prediction is intrinsically limited by the availability of large-scale sequencing and protein-DNA interaction data including ChIP-seq.
- Existing DL-based methods assume that each position in a motif detector contributes to TF-DNA binding affinity independently, which may underestimate nucleotide dependencies.
- The GitHub repository ships only wgEncodeEH002288 example data due to file size limits; the whole ~5.9GB dataset bundle must be downloaded separately.

## Evidence
