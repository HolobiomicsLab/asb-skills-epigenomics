---
name: deep-learning-motif-tool-benchmark-comparison
description: Use when reproducing or extending motif-prediction benchmarks over ChIP-seq
  peak sets, especially when comparing a CNN-based tool's significance scores (E/P/q-values)
  against multiple existing motif finders.
when_to_use_negative:
- Input is not ChIP-seq peak data aligned to the human GRCh37.p13 genome (e.g. bulk
  RNA-seq or single-cell matrices)
- Only one motif-finding tool is of interest and no cross-tool significance comparison
  is needed
- Non-human TF binding studies where the ENCODE hg19-based encode_101/encode_101_background
  index sets do not apply
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: DESSO
  role: CNN-based motif prediction framework under benchmark; trains per-dataset models
    (train.py) and predicts motifs (predict.py)
  repo: https://github.com/OSU-BMBL/DESSO
- name: DeepBind
  role: Comparison method whose predicted motifs are benchmarked against DESSO's
- name: Basset
  role: Comparison method for motif prediction on the same 690 datasets
- name: MEME-ChIP
  role: Comparison method for motif prediction on the same 690 datasets
- name: KMAC
  role: Comparison method for motif prediction on the same 690 datasets
- name: gkm-SVM
  role: Comparison method for motif prediction on the same 690 datasets
- name: TOMTOM
  role: Motif similarity search used to score predicted motifs against documented
    JASPAR/TRANSFAC motifs
- name: TensorFlow
  role: Deep learning backend for training and predicting with DESSO models
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:4481c1f502ae347f4bb197d967df049035ea9b955015cf40e63d27ae41e3bb86
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
  - DeepBind
  - KMAC
  - MEME-ChIP
  - TOMTOM
  - TensorFlow
  - gkm-SVM
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# deep-learning-motif-tool-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks deep learning-based motif prediction (DESSO CNN with a binomial-distribution TFBS model) against DeepBind, Basset, MEME-ChIP, KMAC and gkm-SVM across the 690 human ENCODE ChIP-seq datasets, scoring predicted motifs against JASPAR/TRANSFAC using TOMTOM E-value, P-value and q-value significance.

## When to use

Use when reproducing or extending motif-prediction benchmarks over ChIP-seq peak sets, especially when comparing a CNN-based tool's significance scores (E/P/q-values) against multiple existing motif finders.

## When NOT to use

- Input is not ChIP-seq peak data aligned to the human GRCh37.p13 genome (e.g. bulk RNA-seq or single-cell matrices)
- Only one motif-finding tool is of interest and no cross-tool significance comparison is needed
- Non-human TF binding studies where the ENCODE hg19-based encode_101/encode_101_background index sets do not apply

## Inputs

- 690 human ENCODE ChIP-seq datasets (encode_101, 101-bp peaks around summits)
- GC-matched background sequences (encode_101_background)
- GRCh37.p13 reference genome (GRCh37.p13.genome.fa)
- Documented motif databases (JASPAR, TRANSFAC)

## Outputs

- Trained per-dataset CNN models with test AUCs in Test_result.txt
- Predicted sequence motifs per dataset (output/encode_101/gc_match/<dataset>/Seq/CNN/0)
- TOMTOM similarity scores (E-value, P-value, q-value) per tool vs JASPAR/TRANSFAC
- Aggregated benchmark comparison table reproducing Figure 2A / Supplementary Figure S2A

## How to apply

Extract 101-bp peak sequences (peak_flank 50, i.e. 2*50+1 bp around each summit) from the 690 ENCODE ChIP-seq datasets using GRCh37.p13 and the GC-matched encode_101_background sets. Train per-dataset CNN models with DESSO train.py (--network CNN --feature_format Seq) over the dataset index range 0-690, recording test AUCs in Test_result.txt. Predict motifs with predict.py using a cutoff scan --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03, and run equivalent pipelines for DeepBind, Basset, MEME-ChIP, KMAC and gkm-SVM on the same datasets. Evaluate all predicted motifs against documented JASPAR/TRANSFAC motifs via TOMTOM, then aggregate -log2(E-value), -log2(P-value) and -log2(q-value) per tool and test differences with a Wilcoxon test (DESSO showed significantly larger values, P < 5 x 10^-4).

## Related tools

- **DESSO** (CNN-based motif prediction framework under benchmark; trains per-dataset models (train.py) and predicts motifs (predict.py)) — https://github.com/OSU-BMBL/DESSO
- **DeepBind** (Comparison method whose predicted motifs are benchmarked against DESSO's)
- **Basset** (Comparison method for motif prediction on the same 690 datasets)
- **MEME-ChIP** (Comparison method for motif prediction on the same 690 datasets)
- **KMAC** (Comparison method for motif prediction on the same 690 datasets)
- **gkm-SVM** (Comparison method for motif prediction on the same 690 datasets)
- **TOMTOM** (Motif similarity search used to score predicted motifs against documented JASPAR/TRANSFAC motifs)
- **TensorFlow** (Deep learning backend for training and predicting with DESSO models)

## Examples

```
python train.py --start_index 0 --end_index 690 --peak_flank 50 --network CNN --feature_format Seq; python predict.py --start_index 0 --end_index 690 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- Test_result.txt per dataset reports AUC of the trained CNN on held-out test data; AUCs should be present for all 690 indexed datasets
- TOMTOM outputs E-value, P-value and q-value per predicted motif against JASPAR/TRANSFAC; DESSO should show significantly larger -log2(E-value), -log2(P-value) and -log2(q-value) than DeepBind, Basset, MEME-ChIP, KMAC and gkm-SVM
- Wilcoxon test P-values comparing DESSO vs each tool should be < 5 x 10^-4, matching Figure 2A
- Reproduced aggregate table should match Figure 2A and Supplementary Figure S2A trends
- Motif output directories (e.g. output/encode_101/gc_match/wgEncodeEH002288/Seq/CNN/0) must exist for each processed dataset

## Limitations

- DESSO's benchmark depends on large-scale sequencing/protein-DNA interaction data availability; such experiments are expensive
- Existing DL-based methods, including DESSO, assume each position in a motif detector contributes independently, potentially underestimating nucleotide dependencies
- Comparison with DeepBind is one-sided for motif prediction; other performance dimensions are not covered here

## Evidence
