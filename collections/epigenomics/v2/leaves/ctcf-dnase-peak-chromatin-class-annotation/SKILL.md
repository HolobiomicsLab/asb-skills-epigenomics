---
name: ctcf-dnase-peak-chromatin-class-annotation
description: Use when a BED file of regulatory elements (CTF/DNase peaks, promoters,
  enhancers) needs chromatin class labels before scoring deletion effects on predicted
  chromatin interactions with deepC.
when_to_use_negative:
- The goal is predicting interactions from sequence directly — use deepC prediction
  without element-level annotation instead
- Peak sets lack promoter/enhancer reference intervals, so a defensible promoter-vs-enhancer
  class assignment cannot be made
- Analyzing Hi-C contact maps themselves (e.g. TAD calling with TopDom) rather than
  scoring element-level deletion effects
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: pybedtools
  role: Merges and intersects the CTCF, DNase, promoter/enhancer and shuffled-background
    BED intervals to assign chromatin classes
  repo: https://github.com/pybedtools/pybedtools
- name: pysam
  role: Used alongside pybedtools in the deepC workflow to extract reference sequence
    windows for the annotated elements
  repo: https://github.com/pysam-developers/pysam
- name: deepC
  role: 'Downstream consumer: annotated elements are deleted in silico and scored
    by the mean absolute difference between reference and deletion predicted interaction
    maps'
  repo: https://github.com/rschwess/deepC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:b8776d89cb210ec9f85c65984844150f0ea8bdf2661b98e0e248f47e526cd5c4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pybedtools/pybedtools
  tools:
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# ctcf-dnase-peak-chromatin-class-annotation

## Summary

Annotates merged ENCODE CTCF and DNase peak sets (plus shuffled background sites) with chromatin class labels — CTCF site, promoter, or enhancer — so that genome-wide in silico deletion effects scored with deepC can be stratified by element type and activity.

## When to use

Use when a BED file of regulatory elements (CTF/DNase peaks, promoters, enhancers) needs chromatin class labels before scoring deletion effects on predicted chromatin interactions with deepC.

## When NOT to use

- The goal is predicting interactions from sequence directly — use deepC prediction without element-level annotation instead
- Peak sets lack promoter/enhancer reference intervals, so a defensible promoter-vs-enhancer class assignment cannot be made
- Analyzing Hi-C contact maps themselves (e.g. TAD calling with TopDom) rather than scoring element-level deletion effects

## Inputs

- ENCODE GM12878 CTCF ChIP-seq peak BED file (n=45,635)
- ENCODE GM12878 DNase-seq peak BED file (n=47,320)
- BED file of 3,850 shuffled background control sites
- Genomic interval sets defining promoters and enhancers for class assignment

## Outputs

- Merged BED file of regulatory elements with per-element chromatin class annotation (promoter, enhancer, CTCF) and background flag
- Per-element annotation table joining coordinates, chromatin class, and later the mean absolute interaction difference from the deepC deletion screen (deletion_screen_results.csv, ~96,805 rows)

## How to apply

Merge the ENCODE GM12878 CTCF peaks (n=45,635) and DNase peaks (n=47,320) with 3,850 shuffled background sites into a single BED file using pybedtools. Assign each element a chromatin class: elements with CTCF binding are labeled CTCF; remaining DNase peaks overlapping promoters are labeled promoter, and the rest enhancer. Retain activity information where available, since deepC's deletion screen found that enhancers and promoters associated with active chromatin marks have higher predicted impact than those without. The annotated table (~96,805 rows) feeds the deletion screen, where CTCF sites and elements with proximal CTCF binding are expected to show the strongest mean absolute interaction differences between reference and deletion deepC predictions.

## Related tools

- **pybedtools** (Merges and intersects the CTCF, DNase, promoter/enhancer and shuffled-background BED intervals to assign chromatin classes) — https://github.com/pybedtools/pybedtools
- **pysam** (Used alongside pybedtools in the deepC workflow to extract reference sequence windows for the annotated elements) — https://github.com/pysam-developers/pysam
- **deepC** (Downstream consumer: annotated elements are deleted in silico and scored by the mean absolute difference between reference and deletion predicted interaction maps) — https://github.com/rschwess/deepC

## Examples

```
import pybedtools; merged = pybedtools.BedTool('ctcf.bed').cat(pybedtools.BedTool('dnase.bed')); annotated = merged.intersect(pybedtools.BedTool('promoters.bed'), wa=True, v=True)
```

## Evaluation signals

- Merged annotation file contains the expected element counts: CTCF n=45,635, DNase n=47,320, plus 3,850 shuffled background sites (~96,805 rows downstream)
- Every element carries exactly one chromatin class label (promoter, enhancer, or CTCF) and background flag
- Downstream deletion_screen_results.csv contains required fields: coordinates, chromatin class, and mean absolute interaction difference
- Stratified scores reproduce Figure 3b ordering: CTCF sites and elements with proximal CTCF binding show the strongest average predicted deletion impact; promoters > enhancers; active-mark-associated elements > inactive

## Limitations

- Class labels depend on the quality and overlap rules of the promoter/enhancer reference sets; misassignment propagates into the deletion-screen stratification
- The screen scores predicted (deepC) impact, not experimentally measured effects of deletion
- deepC supports specific TensorFlow versions (1.8, 1.14, 2.1); the compatibility version of deepC must match the installed TensorFlow
- Background shuffled sites control for genomic context but do not constitute functional elements

## Evidence

- [methods] Merged CTCF and DNase peak sets with shuffled background: [evidence span withheld — anchor status: not_found]
