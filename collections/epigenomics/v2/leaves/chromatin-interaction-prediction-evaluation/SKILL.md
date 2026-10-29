---
name: chromatin-interaction-prediction-evaluation
description: Use when assessing how well sequence-based 3D genome folding predictions
  (deepC) reproduce observed Hi-C contacts on holdout chromosomes, e.g. computing
  Pearson correlations for raw vs smoothed maps or low-depth training sets.
when_to_use_negative:
- The goal is functional interpretation of deletion effects (in silico deletion screen
  scoring) rather than prediction-accuracy evaluation against observed Hi-C.
- The task is training or fine-tuning a deepC model itself rather than evaluating
  an already-trained model on holdout data.
- Data are not chromatin contact maps (e.g. 1D ChIP-seq/DNase tracks), since the Pearson
  correlation benchmarks are defined on interaction matrices.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0102
tools:
- name: deepC
  role: Sequence-based chromatin interaction prediction package (import as deepCregr)
    whose predictions are evaluated against observed Hi-C
  repo: https://github.com/rschwess/deepC
- name: TensorFlow 2.1
  role: Deep learning backend for running deepC predictions; GPU support recommended
    for chromosome-scale prediction speed
- name: pysam
  role: Python library used with pybedtools to prepare sequence windows and handle
    input data for the deepC pipeline
  repo: https://github.com/pysam-developers/pysam
- name: pybedtools
  role: Python library installed alongside pysam for preparing the deepC prediction
    input data
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:a20c99121efc50823a9a2f6b17b893dd67e722e45f9ad1b0faf7c23434feb22c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rschwess/deepC
  tools:
  - TensorFlow 2.1
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# chromatin-interaction-prediction-evaluation

## Summary

Quantitative evaluation of deepC-predicted chromatin interaction maps against observed Hi-C on held-out chromosomes, using distance-stratified and mean Pearson correlations across raw and smoothed contact maps and across Hi-C sequencing depths.

## When to use

Use when assessing how well sequence-based 3D genome folding predictions (deepC) reproduce observed Hi-C contacts on holdout chromosomes, e.g. computing Pearson correlations for raw vs smoothed maps or low-depth training sets.

## When NOT to use

- The goal is functional interpretation of deletion effects (in silico deletion screen scoring) rather than prediction-accuracy evaluation against observed Hi-C.
- The task is training or fine-tuning a deepC model itself rather than evaluating an already-trained model on holdout data.
- Data are not chromatin contact maps (e.g. 1D ChIP-seq/DNase tracks), since the Pearson correlation benchmarks are defined on interaction matrices.

## Inputs

- Trained deepC model checkpoints (e.g. GM12878 model or chromatin-feature-seeded base model from ./models in the deepC repository)
- Hi-C contact matrices / skeleton arrays for held-out chromosomes (GSE63525 GM12878, including 1B/100M/10M downsampled versions prepared with HiC-Pro)
- Megabase-scale DNA sequence windows from the reference genome (extracted with pysam)
- Smoothed and raw predicted and observed contact maps at 5 kb resolution

## Outputs

- correlation_table.csv with per-cell-line distance-stratified Pearson coefficients (raw and smoothed)
- Per-model mean Pearson correlation values on chr16/17 for downsampled-depth models (CSV)
- Predicted chromatin interaction maps for held-out chromosome windows

## How to apply

Load the trained deepC model checkpoints and the Hi-C skeleton arrays for the held-out chromosomes (e.g. chr16/17 in the GM12878 low-depth experiment), then run sequence-based contact predictions over the chromosome windows via `import deepCregr` (GPU support strongly recommended; chromosome-scale prediction takes ~2h on CPU versus ~5 min on GPU). Generate both raw and smoothed versions of predicted and observed 5 kb-resolution contact maps, then compute Pearson correlation coefficients between predicted and observed contacts: distance-stratified within distance bins per chromosome for GM12878 and K562, or mean correlations per chromosome for the downsampled-depth models. Aggregate per-cell-line or per-model coefficients into a single table (e.g. correlation_table.csv). Judge success against the reported benchmarks: ~0.36 raw / ~0.57 smoothed in GM12878 and ~0.28 / ~0.51 in K562, and 0.46 (1B) vs 0.4 (100M) mean Pearson on chr16/17 with the 10M model failing to learn, and confirm the pattern matches Supplementary Figures 5 and 8.

## Related tools

- **deepC** (Sequence-based chromatin interaction prediction package (import as deepCregr) whose predictions are evaluated against observed Hi-C) — https://github.com/rschwess/deepC
- **TensorFlow 2.1** (Deep learning backend for running deepC predictions; GPU support recommended for chromosome-scale prediction speed)
- **pysam** (Python library used with pybedtools to prepare sequence windows and handle input data for the deepC pipeline) — https://github.com/pysam-developers/pysam
- **pybedtools** (Python library installed alongside pysam for preparing the deepC prediction input data)

## Examples

```
python -c "import deepCregr"  # import deepC under TensorFlow 2.1 to run holdout-chromosome predictions for Pearson evaluation
```

## Evaluation signals

- GM12878 distance-stratified Pearson correlation falls within tolerance of ~0.36 raw / ~0.57 smoothed on holdout chromosomes
- K562 distance-stratified Pearson correlation falls within tolerance of ~0.28 raw / ~0.51 smoothed
- Low-depth training: 1B model reaches mean Pearson ~0.46 and 100M model ~0.4 on chr16/17; the 10M model shows failure to learn chromatin structure
- Distance-stratified correlation pattern matches Supplementary Figure 5; low-depth results match Supplementary Figure 8 (output_matches_reference)
- Correlation CSV exists with required fields and per-cell-line/per-model coefficients within expected value ranges (value_in_range)

## Limitations

- CPU-only prediction is prohibitively slow for chromosome-scale evaluation (~2h vs ~5 min with a GPU), so GPU support is effectively required.
- deepC supports specific TensorFlow versions (1.8, 1.14, 2.1); other versions are untested, so the deepC compatibility branch must match the installed TensorFlow version.
- Models trained on very low-depth Hi-C (10M contacts) may fail to learn chromatin structure, making evaluation meaningless for that condition.
- No direct comparative benchmark between deepC and Akita is presented; the authors defer this to future work.

## Evidence

- [methods] deepC was developed under v1.8 and supports (v1.8, 1.14 and 2.1 other versions have not been tested): "We recommend tensorflow 2.1 but deepC was developed under v1.8 and supports (v1.8, 1.14 and 2.1 other versions have not been tested)"
- [methods] To use from within a python script import as `import deepCregr`: "To use from within a python script import as `import deepCregr`"
- [discussion] By validating the results with NG Capture-C and Tiled-C, each 3C methods capable of extreme depth and sensitivity, we showed that the deepC approach effectively increases the resolution of Hi-C data.: "By validating the results with NG Capture-C and Tiled-C, each 3C methods capable of extreme depth and sensitivity, we showed that the deepC approach effectively increases the resolution of Hi-C data."
- [intro] DeepC uses a transfer learning approach and tissue-specific Hi-C data to train models that predict genome folding from megabase (Mb) windows of DNA sequence: "DeepC uses a transfer learning approach and tissue-specific Hi-C data to train models that predict genome folding from megabase (Mb) windows of DNA sequence"
