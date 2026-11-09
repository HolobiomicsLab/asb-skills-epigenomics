---
name: holdout-chromosome-pearson-correlation-analysis
description: Use when a retrained deepC model (e.g. trained on 1B/100M/10M downsampled
  GSE63525 GM12878 Hi-C) must be quantitatively evaluated on held-out chromosomes
  16 and 17 via predicted-vs-observed Pearson correlation.
when_to_use_negative:
- The model being evaluated was trained on data that includes chromosomes 16/17, so
  the holdout evaluation would leak into training
- The goal is a head-to-head accuracy benchmark between deepC and Akita — the article
  presents no such comparative benchmark and defers it to future work
- Input is not Hi-C-derived chromatin interaction data (e.g. bulk RNA-seq or ChIP-seq
  prediction tasks), where Pearson correlation of contact maps is meaningless
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: deepC
  role: Sequence-based predictor imported as `import deepCregr` to generate predicted
    chromatin interaction maps over holdout chr16/17 windows
  repo: https://github.com/rschwess/deepC
- name: TensorFlow 2.1
  role: Runtime for the deepC model; recommended version with GPU support for fast
    prediction (v1.8, 1.14 and 2.1 supported)
- name: pybedtools
  role: Additional python library installed (with pysam) to support deepC genomic
    interval handling during prediction
- name: pysam
  role: Additional python library required by deepC for reading sequence/alignment
    data
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:9f016282a7de861a71cca93931800f0ef115bc398cd198359101182518e4477e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rschwess/deepC
  tools:
  - TensorFlow 2.1
  - deepC
  - pybedtools
  - pysam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# holdout-chromosome-pearson-correlation-analysis

## Summary

Evaluates retrained deepC chromatin-structure models by running sequence-based predictions over holdout chromosomes (chr16/17) and computing mean Pearson correlations between predicted and observed Hi-C contact maps, as used to assess models trained on downsampled GM12878 data.

## When to use

Use when a retrained deepC model (e.g. trained on 1B/100M/10M downsampled GSE63525 GM12878 Hi-C) must be quantitatively evaluated on held-out chromosomes 16 and 17 via predicted-vs-observed Pearson correlation.

## When NOT to use

- The model being evaluated was trained on data that includes chromosomes 16/17, so the holdout evaluation would leak into training
- The goal is a head-to-head accuracy benchmark between deepC and Akita — the article presents no such comparative benchmark and defers it to future work
- Input is not Hi-C-derived chromatin interaction data (e.g. bulk RNA-seq or ChIP-seq prediction tasks), where Pearson correlation of contact maps is meaningless

## Inputs

- Retrained deepC model (from ./models pre-trained chromatin-feature-seeded base model, retrained per contact depth)
- Observed Hi-C contact maps on holdout chromosomes 16 and 17 (HiC-Pro processed GM12878, GSE63525)
- DNA sequence windows for chr16/17 at 5 kb resolution
- Installed TensorFlow environment with GPU support (recommended v2.1 with CUDA 10.1 and matching deepC compatibility branch)

## Outputs

- CSV of mean Pearson correlation between predicted and observed interaction maps per model and per holdout chromosome
- Predicted chromatin interaction maps for chr16/17 windows
- Comparison against Supplementary Figure 8 reference values (1B ~0.46, 100M ~0.4, 10M failure to learn)

## How to apply

Split training from evaluation by chromosome: train the 5 kb-resolution deepC model on the downsampled GM12878 contact matrices (HiC-Pro prepared, GSE63525) and reserve chromosomes 16 and 17 as holdout. Import the retrained model via `import deepCregr` and run sequence-based predictions over the chr16/17 windows (GPU strongly recommended; CPU-only prediction takes ~2h versus ~5 min on GPU). Compute the mean Pearson correlation between predicted and observed chromatin interaction maps across the chr16/17 windows for each sequencing-depth model. Judge correctness against the published benchmarks: the 1B-contact model reaches ~0.46 and the 100M model ~0.4 on chr16/17, while the 10M model fails to learn chromatin structure, consistent with Supplementary Figure 8; write the per-model values to a CSV and verify with a value_in_range check plus comparison to the reference figure.

## Related tools

- **deepC** (Sequence-based predictor imported as `import deepCregr` to generate predicted chromatin interaction maps over holdout chr16/17 windows) — https://github.com/rschwess/deepC
- **TensorFlow 2.1** (Runtime for the deepC model; recommended version with GPU support for fast prediction (v1.8, 1.14 and 2.1 supported))
- **pybedtools** (Additional python library installed (with pysam) to support deepC genomic interval handling during prediction)
- **pysam** (Additional python library required by deepC for reading sequence/alignment data) — https://github.com/pysam-developers/pysam

## Examples

```
import deepCregr
```

## Evaluation signals

- 1B-contact model chr16/17 mean Pearson correlation falls in the range around 0.46 (value_in_range)
- 100M-contact model chr16/17 mean Pearson correlation falls around 0.4 (value_in_range)
- 10M-contact model shows failure to learn chromatin structure (no meaningful correlation)
- Per-model CSV results match Supplementary Figure 8 (output_matches_reference)
- Predictions cover all chr16/17 holdout windows at 5 kb resolution without missing regions

## Limitations

- CPU-only prediction is impractically slow (~2h versus ~5 min with a GPU), so evaluation without GPU support is a bottleneck
- deepC was developed under TensorFlow v1.8 and only supports v1.8, 1.14 and 2.1; other versions have not been tested, so the deepC compatibility branch must match the installed TensorFlow
- The 10M-contact training depth is below the limit at which deepC learns chromatin structure, so evaluation at that depth yields no usable model
- Training deepC models requires GPU support and takes several hours up to days depending on dataset and epochs

## Evidence
