---
name: chip-seq-peak-motif-classification
description: Use when ChIP-seq peaks for multiple TFs (e.g. the 100 TFs expressed
  in K562) have DESSO-predicted motifs and TFBSs, and you need to separate direct
  from indirect binding to infer tethering.
when_to_use_negative:
- Peaks lacking DESSO-predicted motifs/TFBSs for the TF panel, so no own-motif vs.
  other-motif discrimination is possible
- Studies aiming to discover de novo motifs rather than classify existing peaks —
  DESSO motif prediction (train.py/predict.py) should be run first
- Single-TF ChIP-seq analysis with no second TF's motifs available for cross-matching
  indirect peaks
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
tools:
- name: DESSO
  role: CNN-based framework that predicts sequence motifs and identifies TFBSs used
    to classify peaks into D/I/K sets and to score TF-DNA binding
  repo: https://github.com/OSU-BMBL/DESSO
- name: bedtools
  role: Extracts DNA sequences from peak BED files so motifs can be scanned within
    peaks
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:6fb78668539b82e6f55efee769289189305972df0206eeee263ebd667d3f16d1
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
  - bedtools
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# chip-seq-peak-motif-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies each ChIP-seq peak of a TF into direct-binding (D), indirect-binding (I), or indecipherable (K) sets based on whether the peak contains the TF's own DESSO-predicted motif, another TF's motif, or neither, enabling downstream inference of protein-protein-DNA tethering interactions.

## When to use

Use when ChIP-seq peaks for multiple TFs (e.g. the 100 TFs expressed in K562) have DESSO-predicted motifs and TFBSs, and you need to separate direct from indirect binding to infer tethering.

## When NOT to use

- Peaks lacking DESSO-predicted motifs/TFBSs for the TF panel, so no own-motif vs. other-motif discrimination is possible
- Studies aiming to discover de novo motifs rather than classify existing peaks — DESSO motif prediction (train.py/predict.py) should be run first
- Single-TF ChIP-seq analysis with no second TF's motifs available for cross-matching indirect peaks

## Inputs

- DESSO-predicted sequence motifs and TFBSs for each TF (output of predict.py, e.g. output/encode_101/gc_match/wgEncodeEH002288/Seq/CNN/0)
- ENCODE ChIP-seq peak files (BED format) for the 100 TFs expressed in K562
- Reference genome sequences extracted from peaks via bedtools (e.g. GRCh37.p13.genome.fa)

## Outputs

- Per-TF peak classification into D (direct-binding), I (indirect-binding), and K (indecipherable) sets
- tethering_associations.csv listing significant putative tethering pairs (e.g. 61 interactions including ATF3-USF1 and NFE2-MAX)

## How to apply

Load the DESSO-identified sequence motifs and TFBSs predicted for each TF together with its ENCODE ChIP-seq peak files. For each peak of each TF, classify it as direct-binding (D) if it contains the TF's own DESSO motif, indirect-binding (I) if it contains another TF's motif instead, or indecipherable (K) if it contains neither, following Supplementary Method S5. Cross-match I peaks of each TF against D peaks of every other TF to count co-occurrences between absent-motif peaks and other TFs' motif-bearing peaks. Apply the statistical enrichment test specified in Supplementary Method S5 to each TF-TF pair to call significant putative tethering associations. The rationale is that a ChIP-seq peak lacking the immunoprecipitated TF's own motif but enriched for another TF's motif suggests the TF was recruited to DNA through protein-protein interaction (tethering) rather than direct sequence-specific binding.

## Related tools

- **DESSO** (CNN-based framework that predicts sequence motifs and identifies TFBSs used to classify peaks into D/I/K sets and to score TF-DNA binding) — https://github.com/OSU-BMBL/DESSO
- **bedtools** (Extracts DNA sequences from peak BED files so motifs can be scanned within peaks)

## Examples

```
python predict.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- The recovered association list contains exactly 61 putative tethering interactions among the 100 K562 TFs
- The association list includes the two known tethering mechanisms ATF3-USF1 and NFE2-MAX
- Every peak is assigned to exactly one of D, I, or K with no unclassified peaks
- Significant TF-TF pairs are supported by the statistical enrichment test of Supplementary Method S5 (Table S6 records)
- Trained model quality is confirmed by AUC in Test_result.txt before motifs are used for classification

## Limitations

- Predicted indirect TF-motif interaction mechanisms via tethering activities and co-factor motifs await experimental validation
- DESSO's application is intrinsically limited by the availability of large-scale ChIP-seq data
- K (indecipherable) peaks — those containing neither the TF's own nor another TF's motif — cannot contribute to tethering inference

## Evidence

- [abstract] Cross-matching I peaks of one TF against D peaks of another among the 100 K562 TFs, DESSO predicted 61 putative tethering associations: [evidence span withheld — anchor status: not_found]
- [supplementary] Classify peaks by own motif, another TF's motif, or neither (Supplementary Method S5): [evidence span withheld — anchor status: not_found]
