---
name: motif-similarity-clustering
description: Use when a large pool of predicted motifs (e.g. from DESSO predict.py
  across many ChIP-seq datasets) must be reduced to representative motifs and classified
  as known versus undocumented.
when_to_use_negative:
- Input motifs are already collapsed and database-annotated, so clustering would only
  duplicate prior labels
- Shape motifs (HelT, MGW, ProT, Roll) from DESSO's --feature_format DNAShape mode
  are the target, since JASPAR/TRANSFAC matching of sequence motifs does not directly
  apply to shape motif profiles
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: DESSO
  role: Deep learning (CNN) framework whose predict.py output supplies the predicted
    sequence motifs from the 690 ENCODE ChIP-seq datasets that are clustered and classified
  repo: https://github.com/OSU-BMBL/DESSO
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:82558bbbefb9fb0447241b21fd2558762319efe3d030de8d19eab7516bee93c6
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
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# motif-similarity-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters DNA sequence motifs by TOMTOM all-against-all similarity scores using hierarchical clustering, then labels each cluster as documented (JASPAR/TRANSFAC) or undocumented, as applied to DESSO's motifs from 690 ENCODE ChIP-seq datasets.

## When to use

Use when a large pool of predicted motifs (e.g. from DESSO predict.py across many ChIP-seq datasets) must be reduced to representative motifs and classified as known versus undocumented.

## When NOT to use

- Input motifs are already collapsed and database-annotated, so clustering would only duplicate prior labels
- Shape motifs (HelT, MGW, ProT, Roll) from DESSO's --feature_format DNAShape mode are the target, since JASPAR/TRANSFAC matching of sequence motifs does not directly apply to shape motif profiles

## Inputs

- Predicted sequence motif files from DESSO predict.py (output/encode_101/gc_match/<dataset>/Seq/CNN/)
- JASPAR documented motif database
- TRANSFAC documented motif database

## Outputs

- Motif cluster assignments with known/undocumented labels (e.g. motif_cluster_summary.tsv)
- Counts of known versus undocumented representative motifs (e.g. 435 known / 248 undocumented of 683 representative motifs)

## How to apply

Collect the predicted sequence motifs output by DESSO's predict.py (run with --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03) across the 690 ENCODE ChIP-seq datasets and reduce them to representative motifs. Compute all-against-all motif similarity between representative motifs using TOMTOM. Apply hierarchical clustering to the TOMTOM similarity scores following Supplementary Method S1 to group motifs into clusters. Match each motif cluster against documented motifs from JASPAR and TRANSFAC to label it as known or undocumented, then tabulate the split. In the DESSO study this yielded 683 representative motifs, of which 435 matched documented motifs and 248 were undocumented.

## Related tools

- **DESSO** (Deep learning (CNN) framework whose predict.py output supplies the predicted sequence motifs from the 690 ENCODE ChIP-seq datasets that are clustered and classified) — https://github.com/OSU-BMBL/DESSO

## Examples

```
python predict.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- The number of representative motifs equals the expected total (683 for the full DESSO ENCODE run); known + undocumented counts must sum to the representative total (435 + 248 = 683)
- Every motif cluster receives exactly one label — known (JASPAR/TRANSFAC match) or undocumented — with no unassigned clusters
- TOMTOM similarity matrix is complete (all-against-all pairwise scores computed for every representative motif)
- Cluster assignments are deterministic when hierarchical clustering is re-run on the same TOMTOM scores per Supplementary Method S1
- Output table includes database match metadata (JASPAR/TRANSFAC hit) per cluster

## Limitations

- Motif classes labeled 'undocumented' may simply be absent from JASPAR/TRANSFAC rather than biologically novel; the split depends on the coverage of the reference databases
- Clustering quality depends on TOMTOM similarity scores; DESSO's CNN-based motif detection assumes position-independent contributions, which may underestimate nucleotide dependencies and affect motif fidelity
- Motifs identified previously await experimental validation, so undocumented clusters are putative rather than confirmed novel classes

## Evidence
