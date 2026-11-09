---
name: transcription-factor-motif-database-matching
description: Use when a set of de novo TF sequence motifs (e.g., predicted by DESSO
  from ENCODE ChIP-seq peaks) needs to be labeled as matching documented JASPAR/TRANSFAC
  motifs or flagged as novel/undocumented.
when_to_use_negative:
- Input motifs are already annotated with database IDs — matching is only needed for
  de novo, unannotated motifs
- Analyzing DNA shape motifs (HelT, MGW, ProT, Roll) from --feature_format DNAShape
  — these are not directly matchable to JASPAR/TRANSFAC sequence logos
- The goal is TF-DNA binding prediction accuracy (AUC) rather than motif annotation/classification
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: DESSO
  role: Generates the de novo sequence motifs from the 690 ENCODE ChIP-seq datasets
    that are subsequently clustered and matched against JASPAR/TRANSFAC
  repo: https://github.com/OSU-BMBL/DESSO
- name: JASPAR
  role: Reference database of documented motifs used to label motif clusters as known
- name: TRANSFAC
  role: Reference database of documented motifs used to label motif clusters as known
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:e526a7040f67fb03916692edfea250a609ef5dd1058c6592e7ca51a5dc24038a
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
  - JASPAR
  - TRANSFAC
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# transcription-factor-motif-database-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies de novo DNA motifs discovered by DESSO's CNN framework on ENCODE ChIP-seq data into known versus undocumented classes by matching them against documented motifs in the JASPAR and TRANSFAC databases, using TOMTOM similarity scores and hierarchical clustering of representative motifs.

## When to use

Use when a set of de novo TF sequence motifs (e.g., predicted by DESSO from ENCODE ChIP-seq peaks) needs to be labeled as matching documented JASPAR/TRANSFAC motifs or flagged as novel/undocumented.

## When NOT to use

- Input motifs are already annotated with database IDs — matching is only needed for de novo, unannotated motifs
- Analyzing DNA shape motifs (HelT, MGW, ProT, Roll) from --feature_format DNAShape — these are not directly matchable to JASPAR/TRANSFAC sequence logos
- The goal is TF-DNA binding prediction accuracy (AUC) rather than motif annotation/classification

## Inputs

- DESSO-predicted sequence motif files from predict.py across 690 ENCODE ChIP-seq datasets (e.g., output/encode_101/gc_match/wgEncodeEH002288/Seq/CNN/0)
- Documented motif databases: JASPAR and TRANSFAC motif collections

## Outputs

- Cluster assignments of representative motifs with known/undocumented labels
- Motif cluster summary table (motif_cluster_summary.tsv) with counts: 435 known vs 248 undocumented of 683 representative motifs

## How to apply

Collect the predicted sequence motifs output by DESSO's predict.py (run with --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03) across the 690 ENCODE ChIP-seq datasets, then reduce them to representative motifs. Compute all-against-all motif similarity between representative motifs using TOMTOM, and apply hierarchical clustering to the TOMTOM similarity scores following Supplementary Method S1 to group motifs into clusters. Match each motif cluster against documented motifs from JASPAR and TRANSFAC to label it as known or undocumented; in the reference application this yielded 683 representative motifs, of which 435 were known and 248 were undocumented. The rationale is that clusters matching JASPAR/TRANSFAC entries validate the deep learning predictions against established TF binding profiles, while unmatched clusters represent candidate novel motifs for further study.

## Related tools

- **DESSO** (Generates the de novo sequence motifs from the 690 ENCODE ChIP-seq datasets that are subsequently clustered and matched against JASPAR/TRANSFAC) — https://github.com/OSU-BMBL/DESSO
- **JASPAR** (Reference database of documented motifs used to label motif clusters as known)
- **TRANSFAC** (Reference database of documented motifs used to label motif clusters as known)

## Examples

```
python predict.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- The representative motif count reconciles: known + undocumented = total (435 + 248 = 683)
- Each motif cluster has a definitive JASPAR or TRANSFAC match decision, not an empty label
- TOMTOM similarity scores form a complete all-against-all matrix over representative motifs before clustering
- Cluster assignments reproduce the paper's split when applied to the 88 most frequently observed representative motifs

## Limitations

- DL-based motif prediction is limited by the availability of large-scale sequencing and protein-DNA interaction data including ChIP-seq, and such experiments are expensive
- Existing DL-based methods assume each position in a motif detector contributes to binding affinity independently, which may underestimate nucleotide dependencies
- Motifs documented only in JASPAR/TRANSFAC are labeled known; genuinely novel TF motifs remain 'undocumented' pending experimental validation

## Evidence
