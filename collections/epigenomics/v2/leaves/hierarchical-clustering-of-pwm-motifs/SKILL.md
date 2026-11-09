---
name: hierarchical-clustering-of-pwm-motifs
description: Use when a large set of predicted sequence motifs (e.g. DESSO motifs
  from 690 ENCODE ChIP-seq datasets) must be de-duplicated into representative motifs
  and classified as matching documented JASPAR/TRANSFAC motifs or as novel.
when_to_use_negative:
- When motifs are already curated or few in number, making clustering unnecessary
  — match each motif directly against JASPAR/TRANSFAC instead
- When the goal is de novo motif discovery from ChIP-seq peaks (use DESSO's train.py/predict.py
  first; clustering operates on the predicted motif output)
- When clustering DNA shape motifs (HelT, MGW, ProT, Roll) rather than sequence PWMs,
  unless shape-specific similarity is desired
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: DESSO
  role: Deep learning framework whose predict.py (CNN, --feature_format Seq) produces
    the sequence motifs that are clustered
  repo: https://github.com/OSU-BMBL/DESSO
- name: TOMTOM
  role: Computes all-against-all PWM similarity scores used as the distance input
    for hierarchical clustering and for JASPAR/TRANSFAC matching
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:dccd876535b5900c606d9bd41b36c3e7efcf4cb97950cd2b56bead88cf75fe69
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
  - TOMTOM
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# hierarchical-clustering-of-pwm-motifs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Groups a large collection of predicted PWM motifs by computing all-against-all TOMTOM similarity scores and applying hierarchical clustering (Supplementary Method S1), then labels each resulting cluster as known or undocumented by matching against JASPAR and TRANSFAC.

## When to use

Use when a large set of predicted sequence motifs (e.g. DESSO motifs from 690 ENCODE ChIP-seq datasets) must be de-duplicated into representative motifs and classified as matching documented JASPAR/TRANSFAC motifs or as novel.

## When NOT to use

- When motifs are already curated or few in number, making clustering unnecessary — match each motif directly against JASPAR/TRANSFAC instead
- When the goal is de novo motif discovery from ChIP-seq peaks (use DESSO's train.py/predict.py first; clustering operates on the predicted motif output)
- When clustering DNA shape motifs (HelT, MGW, ProT, Roll) rather than sequence PWMs, unless shape-specific similarity is desired

## Inputs

- Predicted sequence motifs (PWMs) output by DESSO's predict.py across 690 ENCODE ChIP-seq datasets
- Documented motif databases: JASPAR and TRANSFAC
- Motif similarity scores computed by TOMTOM (all-against-all)

## Outputs

- Motif cluster assignments with TOMTOM-based similarity groupings
- Known/undocumented labels per representative motif (JASPAR/TRANSFAC matches)
- motif_cluster_summary.tsv containing cluster assignments, database matches, and counts (435 known, 248 undocumented of 683 representatives)

## How to apply

Collect the predicted sequence motifs emitted by DESSO's predict.py (run with --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03) across the 690 ENCODE ChIP-seq datasets, and reduce them to representative motifs. Compute all-against-all motif similarity between representatives using TOMTOM. Apply hierarchical clustering to the TOMTOM similarity scores following Supplementary Method S1 to partition motifs into clusters. Match each cluster against documented motifs from JASPAR and TRANSFAC to label it known or undocumented; in the DESSO study this yielded 683 representative motifs splitting into 435 known and 248 undocumented motifs, with the 88 most frequently observed representatives tabulated for the split.

## Related tools

- **DESSO** (Deep learning framework whose predict.py (CNN, --feature_format Seq) produces the sequence motifs that are clustered) — https://github.com/OSU-BMBL/DESSO
- **TOMTOM** (Computes all-against-all PWM similarity scores used as the distance input for hierarchical clustering and for JASPAR/TRANSFAC matching)

## Examples

```
python predict.py --start_index 0 --end_index 690 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- The number of representative motifs after clustering matches the expected scale (683 in the DESSO study of 690 ENCODE datasets)
- Known + undocumented counts sum to the total representative motifs (435 + 248 = 683)
- Every motif cluster carries a label of either a documented JASPAR/TRANSFAC match or 'undocumented', with no unlabeled clusters
- motif_cluster_summary.tsv contains cluster assignments, database matches, and counts for the 88 most frequently observed representative motifs
- TOMTOM was run all-against-all over the representative motif set, so the similarity matrix is square and symmetric over all representatives

## Limitations

- Cluster-based classification depends on the coverage of JASPAR/TRANSFAC; motifs absent from these databases are labeled undocumented rather than spurious
- Hierarchical clustering of TOMTOM scores follows Supplementary Method S1, whose grouping thresholds determine how motifs are merged; clustering parameters are not exposed as CLI options in the DESSO README
- The pipeline depends on upstream DESSO motif quality: predict.py motif cutoffs (--start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03) affect which motifs enter clustering

## Evidence
