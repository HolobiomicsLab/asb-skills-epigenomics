---
name: ground-truth-loop-set-intersection
description: Use when benchmarking a chromatin loop caller (e.g., GILoop vs Peakachu)
  on deep Hi-C data and an unbiased, stringent ground-truth loop set must be derived
  from consensus of independent callers.
when_to_use_negative:
- The targeted data are extremely sparse single-cell Hi-C (scHi-C); the GILoop framework
  explicitly does not deal with scHi-C
- Only one loop caller's output is available — the intersection definition requires
  independent HICCUPS and FitHiC calls on the same contact maps
- The goal is de novo loop discovery rather than evaluation of loop-caller accuracy
  against a benchmark
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: GILoop
  role: Dual-branch (graph/image) neural network run in inference mode to produce
    genome-wide CTCF-mediated loop probability maps evaluated against the ground-truth
    set
  repo: github.com/fzbio/GILoop
- name: Peakachu
  role: Baseline machine-learning loop caller run in full (clustering) and non-clustering
    modes on the same 520M-read matrices for comparison
- name: SIP
  role: Alternative significant-interaction-peak loop caller referenced among image-based
    Hi-C matrix approaches in the article's intro
  repo: https://github.com/PouletAxel/SIP
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:7ec7f1620e3a4926ade09f1180758c4a743a32c3ca14941db402e3b62a77e512
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  - Peakachu
  - SIP
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ground-truth-loop-set-intersection

## Summary

Constructs a high-confidence benchmark loop set by taking the intersection of HICCUPS and FitHiC calls on 520 million-read GM12878 Hi-C matrices, then evaluates GILoop vs Peakachu (full clustering and non-clustering modes) against this set using PR-AUC, APA z-score enrichment, and well-supported-loop recovery.

## When to use

Use when benchmarking a chromatin loop caller (e.g., GILoop vs Peakachu) on deep Hi-C data and an unbiased, stringent ground-truth loop set must be derived from consensus of independent callers.

## When NOT to use

- The targeted data are extremely sparse single-cell Hi-C (scHi-C); the GILoop framework explicitly does not deal with scHi-C
- Only one loop caller's output is available — the intersection definition requires independent HICCUPS and FitHiC calls on the same contact maps
- The goal is de novo loop discovery rather than evaluation of loop-caller accuracy against a benchmark

## Inputs

- 520 million-read GM12878 Hi-C contact maps
- HICCUPS loop calls on the same matrices
- FitHiC loop calls on the same matrices
- GILoop genome-wide loop probability predictions (fused model)
- Peakachu predictions, full (clustering) mode
- Peakachu predictions, non-clustering mode

## Outputs

- High-confidence ground-truth loop set (HICCUPS ∩ FitHiC intersection)
- Precision-recall curves per caller across score thresholds
- APA z-score enrichment values for GILoop, Peakachu full, and Peakachu non-clustering
- Well-supported-loop recovery fractions across thresholds
- Combined comparison figure (figure3_model_comparison.png)

## How to apply

Load the 520M-read Hi-C contact maps and call loops independently with HICCUPS and FitHiC; define the ground-truth loop set as only those loops recovered by both callers, giving a high-confidence CTCF-mediated loop annotation. Run the fused GILoop model in inference mode and Peakachu in both its full (clustering) and non-clustering modes on the same 520M-read matrices. Sweep each caller's score threshold to build precision-recall curves against the intersection set, then complement PR analysis with aggregate peak analysis (APA) z-score enrichment and quantification of the fraction of high-confidence ground-truth loops recovered at each score threshold.

## Related tools

- **GILoop** (Dual-branch (graph/image) neural network run in inference mode to produce genome-wide CTCF-mediated loop probability maps evaluated against the ground-truth set) — github.com/fzbio/GILoop
- **Peakachu** (Baseline machine-learning loop caller run in full (clustering) and non-clustering modes on the same 520M-read matrices for comparison)
- **SIP** (Alternative significant-interaction-peak loop caller referenced among image-based Hi-C matrix approaches in the article's intro) — https://github.com/PouletAxel/SIP

## Examples

```
python demo.py  # cooler-input GILoop pipeline (pre-processing, sampling, training); predictions then scored vs the HICCUPS∩FitHiC intersection set
```

## Evaluation signals

- GILoop achieves significantly higher PR-AUC than Peakachu (both modes) on the 520M-read test sets
- GILoop maintains higher precision and recall than Peakachu across all probability thresholds
- APA z-score enrichment around GILoop-predicted loops exceeds that of Peakachu full and non-clustering predictions
- Recovery fraction of high-confidence ground-truth loops is consistently higher for GILoop across score thresholds
- The intersection set contains only loops independently supported by both HICCUPS and FitHiC, so set size should be smaller than either caller's output

## Limitations

- Ground truth depends on consensus of HICCUPS and FitHiC; loops missed by both callers cannot enter the benchmark, biasing toward caller-shared biology
- GILoop requires heavy pre-processing (sequencing-depth alignment, normalization, expected-vector calculation, downsampling to ~260 million cis-contacts) and only focuses on bulk Hi-C
- Weak usability if the cell line's sequencing depth differs from any pre-trained model provided in the repository

## Evidence

- [methods] Ground-truth set via caller intersection: [evidence span withheld — anchor status: not_found]
- [abstract] GILoop vs Peakachu benchmark: "significantly outperforming the existing machine learning-based loop calling algorithm Peakachu,27"
- [abstract] Dual-branch model description: "Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops."
- [discussion] Optimal matrix density for training/inference: "which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts)"
- [readme] Repository README grounding: "GILoop is a deep learning model for detecting CTCF-mediated loops on Hi-C contact maps."
