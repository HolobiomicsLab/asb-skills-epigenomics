---
name: distance-stratified-correlation-analysis
description: Use when a sequence-derived chromatin contact prediction (e.g. from deepC)
  needs quantitative validation against observed Hi-C on held-out chromosomes, requiring
  distance-stratified Pearson correlations for raw and smoothed maps.
when_to_use_negative:
- The goal is a direct comparative benchmark of deepC against alternative models such
  as Akita — the article explicitly defers such comparison to future work
- Validation target is locus-specific 3C-style data (NG Capture-C, Tiled-C) rather
  than genome-wide distance-stratified Hi-C agreement
- Predictions must be run on CPU only — chromosome-scale prediction is prohibitively
  slow without GPU support
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_0102
tools:
- name: deepC
  role: Sequence-based chromatin contact prediction over held-out chromosome windows;
    imported in Python as deepCregr
  repo: https://github.com/rschwess/deepC
- name: TensorFlow 2.1
  role: Deep learning backend (v1.8, 1.14 and 2.1 supported) with GPU support for
    running predictions
- name: pysam
  role: Additional python library required by deepC for sequence handling
  repo: https://github.com/pysam-developers/pysam
- name: pybedtools
  role: Additional python library required by deepC for genomic interval handling
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:77b32c587d1b3e370511baea589f5a572b974c6f6b503ed6ce6bf605ab72223f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rschwess/deepC
  tools:
  - TensorFlow 2.1
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# distance-stratified-correlation-analysis

## Summary

Validates sequence-based chromatin contact predictions (deepC) against observed Hi-C by computing Pearson correlation coefficients within genomic distance bins on holdout chromosomes, separately for raw and smoothed contact maps. Quantifies prediction accuracy in a cross-validation scheme across cell lines (GM12878 and K562).

## When to use

Use when a sequence-derived chromatin contact prediction (e.g. from deepC) needs quantitative validation against observed Hi-C on held-out chromosomes, requiring distance-stratified Pearson correlations for raw and smoothed maps.

## When NOT to use

- The goal is a direct comparative benchmark of deepC against alternative models such as Akita — the article explicitly defers such comparison to future work
- Validation target is locus-specific 3C-style data (NG Capture-C, Tiled-C) rather than genome-wide distance-stratified Hi-C agreement
- Predictions must be run on CPU only — chromosome-scale prediction is prohibitively slow without GPU support

## Inputs

- Trained deepC models (from prior cross-validation training task)
- Hi-C skeleton arrays for held-out chromosomes (GM12878 and K562)
- Genomic sequence windows from held-out chromosomes

## Outputs

- correlation_table.csv with per-cell-line raw and smoothed distance-stratified Pearson coefficients
- Raw and smoothed predicted and observed contact maps for held-out chromosome windows

## How to apply

Load the trained deepC models and the Hi-C skeleton arrays for the held-out chromosomes, then import the package as `import deepCregr` and run sequence-based contact predictions over held-out chromosome windows, using GPU support since chromosome-scale prediction is otherwise prohibitively slow on CPU. Generate both raw and smoothed versions of the predicted and observed contact maps, since accuracy differs substantially between the two conditions. For each held-out chromosome, compute distance-stratified Pearson correlation coefficients between predicted and observed contacts within genomic distance bins, separately for GM12878 and K562. Aggregate the per-cell-line raw and smoothed coefficients into a single table (correlation_table.csv). In cross-validation, expect average correlations of ~0.36 raw / ~0.57 smoothed in GM12878 and ~0.28 raw / ~0.51 smoothed in K562; compare the distance-stratified pattern against Supplementary Figure 5.

## Related tools

- **deepC** (Sequence-based chromatin contact prediction over held-out chromosome windows; imported in Python as deepCregr) — https://github.com/rschwess/deepC
- **TensorFlow 2.1** (Deep learning backend (v1.8, 1.14 and 2.1 supported) with GPU support for running predictions)
- **pysam** (Additional python library required by deepC for sequence handling) — https://github.com/pysam-developers/pysam
- **pybedtools** (Additional python library required by deepC for genomic interval handling)

## Examples

```
import deepCregr; preds = deepCregr.predict(model_path, seq_windows)  # then compute per-distance-bin Pearson r vs Hi-C skeleton
```

## Evaluation signals

- Each cell-line coefficient falls within tolerance of the reported values: ~0.36 raw / ~0.57 smoothed (GM12878) and ~0.28 raw / ~0.51 smoothed (K562)
- The distance-stratified correlation pattern matches Supplementary Figure 5 (output_matches_reference check)
- correlation_table.csv contains coefficients for both GM12878 and K562, each with raw and smoothed conditions
- Per-chromosome coefficients were computed within distance bins over held-out chromosomes only (not training chromosomes)
- Both predicted and observed maps were smoothed with the identical procedure before the smoothed-condition comparison

## Limitations

- Chromosome-scale prediction is prohibitively slow on CPU; GPU support is required (essential for training, preferable for predictions)
- deepC is only tested with TensorFlow versions v1.8, 1.14 and 2.1; other versions have not been tested
- No direct comparative benchmark between deepC and Akita is presented; the authors defer this to future work
- Smoothed correlations (~0.51–0.57) substantially exceed raw correlations (~0.28–0.36), so the comparison condition must be reported explicitly

## Evidence
