---
name: motif-frequency-summarization
description: Use when you have a set of representative motifs predicted by DESSO from
  ChIP-seq data and need to determine how many correspond to known JASPAR/TRANSFAC
  motifs versus undocumented classes.
when_to_use_negative:
- You need to identify motifs from raw ChIP-seq peaks — use DESSO's train.py/predict.py
  motif prediction workflow first; this skill only summarizes and classifies already-predicted
  representative motifs.
- Your motifs are DNA shape motifs (HelT, MGW, ProT, Roll) rather than sequence motifs;
  the known/undocumented JASPAR/TRANSFAC labeling described here targets sequence
  motifs.
- You want to compare motif prediction performance against DeepBind, Basset, MEME-ChIP,
  KMAC or gkm-SVM rather than summarize motif class frequencies.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: DESSO
  role: Source of the predicted sequence motifs and representative motifs that are
    clustered and classified; invoked via predict.py with cutoff interval 0.01–1 and
    step 0.03
  repo: https://github.com/OSU-BMBL/DESSO
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:d575f58591e444e276cf6a8b679d6f56915c1e16877d450e5427eef6b433d114
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# motif-frequency-summarization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters DESSO-predicted representative sequence motifs by TOMTOM similarity using hierarchical clustering, then labels each cluster as documented (JASPAR/TRANSFAC) or undocumented and tabulates motif frequencies and class counts.

## When to use

Use when you have a set of representative motifs predicted by DESSO from ChIP-seq data and need to determine how many correspond to known JASPAR/TRANSFAC motifs versus undocumented classes.

## When NOT to use

- You need to identify motifs from raw ChIP-seq peaks — use DESSO's train.py/predict.py motif prediction workflow first; this skill only summarizes and classifies already-predicted representative motifs.
- Your motifs are DNA shape motifs (HelT, MGW, ProT, Roll) rather than sequence motifs; the known/undocumented JASPAR/TRANSFAC labeling described here targets sequence motifs.
- You want to compare motif prediction performance against DeepBind, Basset, MEME-ChIP, KMAC or gkm-SVM rather than summarize motif class frequencies.

## Inputs

- Representative sequence motifs predicted by DESSO (predict.py output, e.g. output/encode_101/gc_match/wgEncodeEH002288/Seq/CNN/0) from the 690 ENCODE ChIP-seq datasets
- Documented motif databases: JASPAR and TRANSFAC motif collections

## Outputs

- Motif cluster assignments based on TOMTOM similarity with hierarchical clustering (Supplementary Method S1)
- Known/undocumented labels per motif cluster via JASPAR/TRANSFAC matching
- Motif frequency tabulation (e.g. the 88 most frequently observed representative motifs) and counts, written to motif_cluster_summary.tsv

## How to apply

Collect the predicted sequence motifs output by DESSO's predict.py (run with --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03) across the 690 human ENCODE ChIP-seq datasets. Compute all-against-all motif similarity between representative motifs with TOMTOM. Apply hierarchical clustering to the TOMTOM similarity scores following the paper's Supplementary Method S1 to group motifs into clusters. Match each motif cluster against documented motifs from JASPAR and TRANSFAC to label it as known or undocumented. Identify the most frequently observed representative motifs (e.g. the 88 highlighted in the paper) and write cluster assignments, database matches, and counts to a summary table such as motif_cluster_summary.tsv. Correct application reproduces the paper's split of 683 representative motifs into 435 known and 248 undocumented.

## Related tools

- **DESSO** (Source of the predicted sequence motifs and representative motifs that are clustered and classified; invoked via predict.py with cutoff interval 0.01–1 and step 0.03) — https://github.com/OSU-BMBL/DESSO

## Evaluation signals

- The total number of representative motifs matches the expected scale (683 representative motifs from the 690 ENCODE ChIP-seq datasets in the paper).
- The known/undocumented split sums correctly: 435 known + 248 undocumented = 683 representative motifs.
- Each cluster labeled 'known' has an explicit JASPAR or TRANSFAC database match recorded; undocumented clusters have none.
- The summary table (motif_cluster_summary.tsv) contains cluster assignments, database matches, and frequency counts for all representative motifs, including the 88 most frequently observed.
- Hierarchical clustering of TOMTOM similarity scores follows Supplementary Method S1 and yields stable, reproducible cluster memberships.

## Limitations

- Undocumented motifs labeled as 'new' may simply be absent from JASPAR/TRANSFAC rather than biologically novel; classification depends entirely on the coverage of these two databases.
- Results depend on DESSO's cutoff parameters (--start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03); different cutoffs change which motifs are predicted and hence the counts.
- The article notes DL-based motif prediction is intrinsically limited by the availability of large-scale sequencing and protein–DNA interaction data, so summarization inherits limited-training-data failure modes.

## Evidence
