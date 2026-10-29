---
name: precision-recall-curve-analysis
description: Use when comparing loop-calling predictions (e.g., GILoop vs. Peakachu)
  or evaluating a transferred model on K562/HeLa-S3 Hi-C data, and you need threshold-free
  precision-recall comparison against a defined ground-truth loop set.
when_to_use_negative:
- Input is single-cell Hi-C (scHi-C) data — GILoop is explicitly not capable of dealing
  with extremely sparse single-cell Hi-C
- The target cell line's sequencing depth matches none of the repository's pre-trained
  models, making transfer evaluation unreliable without retraining
- No independent ground-truth loop set (e.g., HICCUPS/FitHiC intersection) is available,
  since precision and recall cannot be defined without it
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: GILoop
  role: Dual-branch (graph + image) neural network whose loop probability predictions
    are threshold-swept to build the PR curves being compared
  repo: github.com/fzbio/GILoop
- name: Peakachu
  role: Machine-learning loop-calling baseline run in full (clustering) and non-clustering
    modes to generate comparison PR curves
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:aaefbc8d502563fa29b63cc07908e2aa98b5b6688033508bff3ae8c5be0a5011
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  - Peakachu
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# precision-recall-curve-analysis

## Summary

Builds and compares precision-recall curves (and PR-AUC) for chromatin loop callers such as GILoop and Peakachu by sweeping score thresholds against a ground-truth loop set, enabling both model benchmarking and transfer-learning evaluation across cell lines.

## When to use

Use when comparing loop-calling predictions (e.g., GILoop vs. Peakachu) or evaluating a transferred model on K562/HeLa-S3 Hi-C data, and you need threshold-free precision-recall comparison against a defined ground-truth loop set.

## When NOT to use

- Input is single-cell Hi-C (scHi-C) data — GILoop is explicitly not capable of dealing with extremely sparse single-cell Hi-C
- The target cell line's sequencing depth matches none of the repository's pre-trained models, making transfer evaluation unreliable without retraining
- No independent ground-truth loop set (e.g., HICCUPS/FitHiC intersection) is available, since precision and recall cannot be defined without it

## Inputs

- Hi-C intra-chromosomal contact maps (e.g., 520 million-read GM12878, KR-normalized O/E matrices or .cool files)
- Ground-truth loop set (intersection of HICCUPS and FitHiC calls)
- Loop-caller probability/score outputs from GILoop (fused model) and Peakachu (clustering and non-clustering modes)
- Pre-trained GILoop GM12878 checkpoints (20% and 50% downsampled) for transfer settings
- K562 Hi-C data (GEO GSE63525) and HeLa-S3 Hi-C data (4DN 4DNESCMX7L58)

## Outputs

- Precision-recall curves per caller or transfer setting
- PR-AUC values used to rank GILoop versus Peakachu
- Combined figure panels (e.g., figure3_model_comparison.png with PR curves, APA z-score enrichment, and well-supported-loop recovery; figure6_transfer_pr_curves.png with overlaid source/target PR curves)

## How to apply

Construct a ground-truth loop set — for benchmarking on the 520 million-read GM12878 Hi-C data, use the intersection of HICCUPS and FitHiC calls. Run each caller (GILoop fused model, Peakachu full/clustering mode, and Peakachu non-clustering mode) in inference mode on the same 520M-read contact maps, then sweep each caller's score threshold to compute precision and recall at every point and assemble the PR curve; compare via PR-AUC. For cross-cell-line transfer, run GILoop inference with the 20%-downsampled GM12878 checkpoint on K562 (GSE62525-derived contact maps) and the 50% checkpoint on HeLa-S3 (4DNESCMX7L58), overlaying target PR curves with the GM12878 source baseline to verify minimal performance decay. Complement PR analysis with APA z-score enrichment and well-supported-loop recovery fractions to corroborate that higher PR-AUC reflects more accurate CTCF-mediated loop annotation.

## Related tools

- **GILoop** (Dual-branch (graph + image) neural network whose loop probability predictions are threshold-swept to build the PR curves being compared) — github.com/fzbio/GILoop
- **Peakachu** (Machine-learning loop-calling baseline run in full (clustering) and non-clustering modes to generate comparison PR curves)

## Examples

```
conda activate GIL && python demo.py  # run GILoop on .cool input to produce loop scores, then sweep thresholds to build PR curves vs. HICCUPS∩FitHiC ground truth
```

## Evaluation signals

- GILoop's PR-AUC should exceed Peakachu's (both modes) across all probability thresholds on the 520M-read test sets
- In transfer settings, target-cell-line PR curves (K562 with 20% model, HeLa-S3 with 50% model) should lie near-identically to the GM12878 source curves (minimal performance decay)
- APA z-score enrichment around predicted loops should be positive and consistent with PR ranking for GILoop, Peakachu full, and Peakachu non-clustering
- The fraction of high-confidence (well-supported) ground-truth loops recovered should remain consistent with the PR curve ranking across score thresholds
- Precision and recall must be computed against the same fixed ground-truth loop set (HICCUPS ∩ FitHiC) for every caller and threshold

## Limitations

- Performance depends on the ground-truth definition; calls in the HICCUPS/FitHiC intersection bias evaluation toward callers resembling those pipelines
- Transfer results can weaken if the target cell line's sequencing depth differs from any pre-trained model provided in the repository (weak usability scenario)
- GILoop pre-processing (including downsampling and matrix transformations) is time-heavy before PR evaluation can even start

## Evidence

- [discussion] GILoop significantly outperforms Peakachu in finding CTCF-associated loops – the latter is the most recent machine learning-based chromatin loop detector: "GILoop significantly outperforms Peakachu in finding CTCF-associated loops – the latter is the most recent machine learning-based chromatin loop detector"
- [intro] A trained GILoop model is able to predict CTCF-mediated loops in another cell line with minimal performance decay: "the GILoop framework proposed in this work is transferable across cell lines. A trained GILoop model is able to predict CTCF-mediated loops in another cell line with minimal performance decay"
- [readme] README: input formats and pipeline entry points: "You can either use `.cool` files as input, or use the output format of Juicer dump as input."
- [readme] README: preprocessing caveat: "The preprocessing overhead could be heavy and could lead to a large running time."
