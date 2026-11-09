---
name: motif-similarity-evaluation-against-jaspar-transfac
description: Use when predicted motifs from a motif-discovery or deep-learning pipeline
  must be validated or benchmarked against curated motif databases (JASPAR/TRANSFAC)
  via TOMTOM similarity statistics.
when_to_use_negative:
- The input is raw ChIP-seq peaks or sequences rather than already-predicted motif
  matrices — use the motif prediction skill (train.py/predict.py) first
- Evaluating DNA shape motifs (HelT, MGW, ProT, Roll), which are not directly comparable
  to JASPAR/TRANSFAC sequence motifs via TOMTOM
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: DESSO
  role: CNN-based motif prediction framework whose predicted motifs are the evaluation
    input
  repo: https://github.com/OSU-BMBL/DESSO
- name: TOMTOM
  role: Similarity search tool scoring predicted motifs against documented JASPAR/TRANSFAC
    motifs
- name: DeepBind
  role: Comparison tool whose predicted motifs are benchmarked alongside DESSO
- name: Basset
  role: Comparison tool whose predicted motifs are benchmarked alongside DESSO
- name: MEME-ChIP
  role: Comparison tool whose predicted motifs are benchmarked alongside DESSO
- name: KMAC
  role: Comparison tool whose predicted motifs are benchmarked alongside DESSO
- name: gkm-SVM
  role: Comparison tool whose predicted motifs are benchmarked alongside DESSO
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:03ffc071d0cee51d61785a764a6e8f197acbe8c2d6c7b06edbb0db1614df691e
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
  - gkm-SVM
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# motif-similarity-evaluation-against-jaspar-transfac

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates predicted sequence motifs (e.g. from DESSO CNN models) by TOMTOM similarity search against documented JASPAR and TRANSFAC motifs, using E-value, P-value and q-value significance measures to benchmark motif prediction quality across the 690 human ENCODE ChIP-seq datasets.

## When to use

Use when predicted motifs from a motif-discovery or deep-learning pipeline must be validated or benchmarked against curated motif databases (JASPAR/TRANSFAC) via TOMTOM similarity statistics.

## When NOT to use

- The input is raw ChIP-seq peaks or sequences rather than already-predicted motif matrices — use the motif prediction skill (train.py/predict.py) first
- Evaluating DNA shape motifs (HelT, MGW, ProT, Roll), which are not directly comparable to JASPAR/TRANSFAC sequence motifs via TOMTOM

## Inputs

- Predicted motif PWMs from DESSO CNN models (output/encode_101/gc_match/<dataset>/Seq/CNN/0)
- Predicted motifs from comparison tools (DeepBind, Basset, MEME-ChIP, KMAC, gkm-SVM) on the same 690 ENCODE ChIP-seq datasets
- Documented motif databases: JASPAR and TRANSFAC

## Outputs

- TOMTOM similarity results per predicted motif (E-value, P-value, q-value)
- Aggregated per-dataset, per-tool significance comparison table reproducing Figure 2A and Supplementary Figure S2A

## How to apply

After predicting motifs per dataset (e.g. DESSO predict.py with --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03 on 101-bp peak sequences, peak_flank 50), run the equivalent motif-prediction pipelines for comparison tools (DeepBind, Basset, MEME-ChIP, KMAC, gkm-SVM) on the same 690 ENCODE ChIP-seq datasets. Evaluate every predicted motif against documented JASPAR and TRANSFAC motifs using the TOMTOM similarity search, recording E-value, P-value and q-value per comparison. Aggregate per-dataset and per-tool significance scores (e.g. -log2(E-value), -log2(P-value), -log2(q-value)) into a summary comparison table and test differences between tools with a Wilcoxon test; DESSO showed significantly larger values than the other five methods (Wilcoxon P-values < 5 × 10−4, Figure 2A).

## Related tools

- **DESSO** (CNN-based motif prediction framework whose predicted motifs are the evaluation input) — https://github.com/OSU-BMBL/DESSO
- **TOMTOM** (Similarity search tool scoring predicted motifs against documented JASPAR/TRANSFAC motifs)
- **DeepBind** (Comparison tool whose predicted motifs are benchmarked alongside DESSO)
- **Basset** (Comparison tool whose predicted motifs are benchmarked alongside DESSO)
- **MEME-ChIP** (Comparison tool whose predicted motifs are benchmarked alongside DESSO)
- **KMAC** (Comparison tool whose predicted motifs are benchmarked alongside DESSO)
- **gkm-SVM** (Comparison tool whose predicted motifs are benchmarked alongside DESSO)

## Examples

```
python predict.py --start_index 0 --end_index 690 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- Each predicted motif has a TOMTOM match record with E-value, P-value and q-value against JASPAR/TRANSFAC
- Wilcoxon test comparing -log2(E-value), -log2(P-value) and -log2(q-value) distributions across tools yields P-values < 5 × 10−4 when reproducing Figure 2A
- Per-dataset results cover all 690 ENCODE ChIP-seq datasets (161 TFs in 91 cell lines)
- Benchmark table matches Supplementary Figure S2A per-tool significance pattern

## Limitations

- Existing DL-based methods assume each position in a motif detector contributes to binding affinity independently, which may underestimate nucleotide dependencies
- Application of DL to motif prediction is limited by availability of large-scale sequencing and protein–DNA interaction data

## Evidence

- [results] Wilcoxon test P-values < 5 × 10−4 showing larger -log2(E-value), -log2(P-value), -log2(q-value): [evidence span withheld — anchor status: not_found]
