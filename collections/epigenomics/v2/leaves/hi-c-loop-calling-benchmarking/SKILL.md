---
name: hi-c-loop-calling-benchmarking
description: Use when comparing a Hi-C loop caller (e.g., GILoop) against machine-learning
  baselines like Peakachu on matched contact maps, requiring PR-AUC, APA enrichment,
  and loop-recovery evaluation against a ground-truth loop set.
when_to_use_negative:
- Input is extremely sparse single-cell Hi-C (scHi-C) data — GILoop only focuses on
  bulk Hi-C and is not capable of dealing with scHi-C
- The goal is de novo loop calling rather than benchmarking caller agreement — use
  a single caller pipeline (e.g., GILoop demo.py) instead of the comparative PR/APA
  framework
- The ground-truth loop set cannot be constructed (no overlapping HICCUPS and FitHiC
  calls available), making precision/recall and recovery evaluation undefined
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3940
tools:
- name: GILoop
  role: Dual-branch (graph + image) neural network loop caller whose fused model supplies
    the genome-wide CTCF-mediated loop probability maps being benchmarked
  repo: https://github.com/fzbio/GILoop
- name: Peakachu
  role: Machine-learning baseline loop caller, run in both full (clustering) and non-clustering
    modes for head-to-head PR-AUC, APA, and recovery comparison
- name: SIP
  role: Alternative image-based loop caller available as a comparator in the broader
    loop-calling tool landscape
  repo: https://github.com/PouletAxel/SIP
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:37d138d9081cc33cd310edfdf6c0800c4e4707c5981de78866f842ea48d3e27d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fzbio/GILoop
  tools:
  - GILoop
  - Peakachu
  - SIP
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3940
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# hi-c-loop-calling-benchmarking

## Summary

Benchmark CTCF-mediated chromatin loop callers by comparing GILoop's dual-branch (graph+image) predictions against the Peakachu framework (full clustering and non-clustering modes) on deep Hi-C data, using precision-recall curves, APA z-score enrichment, and well-supported-loop recovery as evaluation metrics.

## When to use

Use when comparing a Hi-C loop caller (e.g., GILoop) against machine-learning baselines like Peakachu on matched contact maps, requiring PR-AUC, APA enrichment, and loop-recovery evaluation against a ground-truth loop set.

## When NOT to use

- Input is extremely sparse single-cell Hi-C (scHi-C) data — GILoop only focuses on bulk Hi-C and is not capable of dealing with scHi-C
- The goal is de novo loop calling rather than benchmarking caller agreement — use a single caller pipeline (e.g., GILoop demo.py) instead of the comparative PR/APA framework
- The ground-truth loop set cannot be constructed (no overlapping HICCUPS and FitHiC calls available), making precision/recall and recovery evaluation undefined

## Inputs

- 520 million-read GM12878 Hi-C contact maps (KR-normalized O/E matrices, e.g., Juicer dump text or .cool files)
- Ground-truth loop set defined as the intersection of HICCUPS and FitHiC calls
- GILoop fused model predictions (genome-wide loop probability maps)
- Peakachu loop predictions from both full (clustering) and non-clustering modes

## Outputs

- Precision-recall curves and PR-AUC values for GILoop, Peakachu full, and Peakachu non-clustering
- APA z-score enrichment values around each caller's predicted loops
- Well-supported-loop recovery fractions across score thresholds
- Combined comparison figure (e.g., figure3_model_comparison.png)

## How to apply

Construct the ground-truth loop set as the intersection of HICCUPS and FitHiC calls on the deep (520 million-read) GM12878 Hi-C contact maps. Run the fused GILoop model in inference mode on the same 520M-read matrices, and run Peakachu in both its full (clustering) mode and its non-clustering mode to obtain comparable predictions. Sweep the score threshold for each caller's predictions and compute precision and recall against the ground-truth loop set to build precision-recall curves. Perform aggregate peak analysis (APA) around each caller's predicted loops to compute APA z-score enrichment, and quantify the fraction of high-confidence ground-truth loops recovered across thresholds. Combine the PR curves, APA z-score panels, and recovery panels into a single comparison figure.

## Related tools

- **GILoop** (Dual-branch (graph + image) neural network loop caller whose fused model supplies the genome-wide CTCF-mediated loop probability maps being benchmarked) — https://github.com/fzbio/GILoop
- **Peakachu** (Machine-learning baseline loop caller, run in both full (clustering) and non-clustering modes for head-to-head PR-AUC, APA, and recovery comparison)
- **SIP** (Alternative image-based loop caller available as a comparator in the broader loop-calling tool landscape) — https://github.com/PouletAxel/SIP

## Examples

```
python demo.py  # GILoop full pipeline on .cool input; then sweep score thresholds vs Peakachu full/non-clustering, compute PR-AUC, APA z-score, and loop recovery
```

## Evaluation signals

- GILoop's PR-AUC exceeds that of both Peakachu full and Peakachu non-clustering across all probability thresholds on the 520M-read test sets
- APA z-score enrichment at GILoop-predicted loops is comparable to or higher than at Peakachu-predicted loops
- Well-supported-loop recovery fraction (fraction of high-confidence HICCUPS∩FitHiC loops recovered) is higher for GILoop across score thresholds
- Precision-recall curves are computed from a consistent threshold sweep applied identically to all three callers
- All three callers are evaluated on the same 520 million-read contact maps and the same ground-truth loop set

## Limitations

- GILoop only focuses on bulk Hi-C and is not capable of dealing with extremely sparse single-cell Hi-C (scHi-C) data
- GILoop requires a relatively large amount of pre-processing time before training, mainly from the downsampling procedure (matrix transformations and hard disk reading/writing)
- Weak usability if the targeted cell line has a sequencing depth different from any of the pre-trained models provided in the repository

## Evidence

- [discussion] GILoop significantly outperforms Peakachu in finding CTCF-associated loops: "GILoop significantly outperforms Peakachu in finding CTCF-associated loops – the latter is the most recent machine learning-based chromatin loop detector"
- [intro] Peakachu is the machine-learning baseline: "significantly outperforming the existing machine learning-based loop calling algorithm Peakachu,27"
- [readme] GILoop accepts .cool or Juicer dump inputs for loop prediction: "You can either use `.cool` files as input, or use the output format of Juicer dump as input."
- [readme] demo.py is the full GILoop pipeline used to produce predictions: "This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.), and GILoop sampling and training."
