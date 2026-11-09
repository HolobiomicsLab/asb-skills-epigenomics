---
name: chromosome-holdout-cross-validation
description: Use when evaluating sequence-based chromatin contact predictors under
  a chromosome-holdout cross-validation scheme, where predicted and observed Hi-C
  contact maps on held-out chromosomes are compared via distance-stratified Pearson
  correlations across cell lines.
when_to_use_negative:
- You need a head-to-head benchmark between deepC and Akita — the article explicitly
  presents no direct comparative benchmark and defers it to future work
- You are training deepC models rather than evaluating predictions on held-out chromosomes
  — training requires a separate multi-hour/days GPU workflow
- Your goal is variant-effect prediction (e.g. CTCF, enhancer, or promoter deletion
  scoring), which uses a different deepC workflow
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: deepC
  role: Sequence-based chromatin contact prediction model imported as deepCregr to
    predict contacts on held-out chromosome windows
  repo: https://github.com/rschwess/deepC
- name: TensorFlow 2.1
  role: Backend used to run the deepC models for chromosome-scale predictions (v1.8,
    1.14 and 2.1 also supported)
- name: pysam
  role: Python library used for reading genome sequence during prediction over held-out
    chromosome windows
  repo: https://github.com/pysam-developers/pysam
- name: pybedtools
  role: Python library used alongside pysam for genomic interval handling in the prediction
    workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:dd23db0a4f1677a7abfe551213cd4b0bb02cd9ea53a4f652d05405de439f9807
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rschwess/deepC
  tools:
  - TensorFlow 2.1
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# chromosome-holdout-cross-validation

## Summary

A chromosome-holdout cross-validation scheme for deepC sequence-based chromatin contact predictions, scoring predicted Hi-C skeletons against observed Hi-C via distance-stratified Pearson correlations in GM12878 and K562.

## When to use

Use when evaluating sequence-based chromatin contact predictors under a chromosome-holdout cross-validation scheme, where predicted and observed Hi-C contact maps on held-out chromosomes are compared via distance-stratified Pearson correlations across cell lines.

## When NOT to use

- You need a head-to-head benchmark between deepC and Akita — the article explicitly presents no direct comparative benchmark and defers it to future work
- You are training deepC models rather than evaluating predictions on held-out chromosomes — training requires a separate multi-hour/days GPU workflow
- Your goal is variant-effect prediction (e.g. CTCF, enhancer, or promoter deletion scoring), which uses a different deepC workflow

## Inputs

- Trained deepC TensorFlow models from the transfer-learning training step
- Hi-C skeleton arrays for held-out chromosomes from task_002
- DNA sequence windows spanning the held-out chromosomes (FASTA, accessed via pysam/pybedtools)
- Observed Hi-C contact maps for GM12878 and K562

## Outputs

- correlation_table.csv with per-cell-line raw and smoothed distance-stratified Pearson correlation coefficients
- Smoothed and raw predicted contact maps for held-out chromosomes
- Distance-stratified correlation pattern comparable to Supplementary Figure 5

## How to apply

Load the trained deepC models (imported in Python as `import deepCregr`) and the skeleton arrays for the held-out chromosomes, then run sequence-based contact predictions over the held-out chromosome windows; GPU support is required since chromosome-scale prediction is otherwise prohibitively slow. Generate both raw and smoothed versions of the predicted and observed contact maps. For each held-out chromosome and each cell line (GM12878 and K562), compute Pearson correlation coefficients between predicted and observed contacts stratified by genomic distance bins. Aggregate raw and smoothed coefficients per cell line into correlation_table.csv, then check each coefficient against the reported tolerances: ~0.36 raw / ~0.57 smoothed for GM12878 and ~0.28 raw / ~0.51 smoothed for K562, and confirm the distance-stratified pattern matches Supplementary Figure 5.

## Related tools

- **deepC** (Sequence-based chromatin contact prediction model imported as deepCregr to predict contacts on held-out chromosome windows) — https://github.com/rschwess/deepC
- **TensorFlow 2.1** (Backend used to run the deepC models for chromosome-scale predictions (v1.8, 1.14 and 2.1 also supported))
- **pysam** (Python library used for reading genome sequence during prediction over held-out chromosome windows) — https://github.com/pysam-developers/pysam
- **pybedtools** (Python library used alongside pysam for genomic interval handling in the prediction workflow)

## Examples

```
import deepCregr
```

## Evaluation signals

- GM12878 average distance-stratified Pearson correlation falls within tolerance of ~0.36 (raw) and ~0.57 (smoothed)
- K562 average distance-stratified Pearson correlation falls within tolerance of ~0.28 (raw) and ~0.51 (smoothed)
- Distance-stratified correlation pattern matches Supplementary Figure 5 (output_matches_reference)
- correlation_table.csv contains per-cell-line coefficients for both raw and smoothed conditions across all held-out chromosomes
- Predictions over all held-out chromosome windows complete without CPU fallback errors, confirming GPU execution

## Limitations

- Chromosome-scale prediction is prohibitively slow on CPU; GPU support is essential, so this evaluation cannot be run without GPU-backed TensorFlow
- No direct comparative benchmark between deepC and Akita is presented; the authors defer this to future work
- Smoothed-condition scores are systematically higher than raw ones, so raw and smoothed results must not be mixed when comparing to the reported values

## Evidence
