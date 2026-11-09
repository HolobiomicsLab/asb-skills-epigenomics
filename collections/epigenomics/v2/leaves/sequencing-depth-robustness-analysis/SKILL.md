---
name: sequencing-depth-robustness-analysis
description: Use when evaluating loop-calling model robustness on downsampled Hi-C
  contact maps, or when comparing graph-view vs image-view branch performance across
  sequencing depths (e.g., 1.5% / 39M reads up to 520M reads).
when_to_use_negative:
- Input is single-cell Hi-C (scHi-C) data — the paper states GILoop only focuses on
  bulk Hi-C and cannot handle extremely sparse scHi-C
- The targeted cell line's sequencing depth does not match any pre-trained model in
  the repository and retraining is out of scope — the paper flags weak usability in
  this case
- Only full-depth loop calling without any depth-variation or robustness question
  — a standard loop caller benchmark suffices
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: GILoop
  role: Dual-branch GCN+U-Net loop-calling model; provides the matrix downsampling
    function and pre-processing pipeline used to generate depth-series inputs and
    branch-level predictions
  repo: github.com/fzbio/GILoop
- name: Peakachu
  role: Machine-learning loop caller used as the baseline against which GILoop's PR-AUC
    is compared at full depth (520M reads), in both clustering and non-clustering
    modes
merged_aliases:
- sequencing-depth-robustness-evaluation
merged_alias_records:
- alias: sequencing-depth-robustness-benchmarking
  slug: sequencing-depth-robustness-benchmarking
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
- alias: sequencing-depth-robustness-evaluation
  slug: sequencing-depth-robustness-evaluation
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:e841774d7ef1f63f62111a4a7018218ee2b45a2251170ae1bc575a8d8e3b012f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  - Peakachu
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# sequencing-depth-robustness-analysis

## Summary

A skill for assessing how CTCF-mediated chromatin loop calling performance degrades or shifts across downsampled Hi-C sequencing depths, using GILoop's dual-branch (GCN graph view + U-Net image view) model and its matrix downsampling function. It reproduces the paper's depth-preference findings by tracking PR-AUC and MAD smoothness as depth decreases.

## When to use

Use when evaluating loop-calling model robustness on downsampled Hi-C contact maps, or when comparing graph-view vs image-view branch performance across sequencing depths (e.g., 1.5% / 39M reads up to 520M reads).

## When NOT to use

- Input is single-cell Hi-C (scHi-C) data — the paper states GILoop only focuses on bulk Hi-C and cannot handle extremely sparse scHi-C
- The targeted cell line's sequencing depth does not match any pre-trained model in the repository and retraining is out of scope — the paper flags weak usability in this case
- Only full-depth loop calling without any depth-variation or robustness question — a standard loop caller benchmark suffices

## Inputs

- High-depth Hi-C contact maps (e.g., 520 million-read GM12878 .cool files or Juicer-dump KR-normalized O/E matrices)
- Ground-truth CTCF-mediated loop labels (intersection of HICCUPS and FitHiC calls)
- GILoop repository matrix downsampling function
- Trained GILoop fused model and single-view GCN / U-Net baselines

## Outputs

- Per-model mean PR-AUC table across downsampling levels (e.g., pr_auc_downsample_1.5pct.csv)
- Depth vs PR-AUC vs MAD results table (e.g., gcn_depth_performance.csv)
- PR-AUC and MAD vs sequencing depth plots reproducing the Figure 4/5 trends

## How to apply

Use the matrix downsampling function provided in the GILoop repository (github.com/fzbio/GILoop) to generate seven downsampled Hi-C test contact maps spanning depths from 1.5% (39 million cis-reads) up to the 520 million-read GM12878 data. At each depth, run inference with the trained fused model and its single-view branches: score candidate CTCF-mediated loops with the GCN branch on the graph representation and the U-Net branch on the image representation. Compute per-model mean precision-recall AUC (PR-AUC) against the ground-truth loop set (intersection of HICCUPS and FitHiC calls), and compute Mean Average Distance (MAD) following Equations 14-16 for smoothness tracking. Expect the image branch (U-Net) to collapse as depth decreases (PR-AUC ~0.135 at 1.5%) while the GCN branch rises steadily with decreasing depth, peaking near the optimal density of 260 million cis-contacts; the fused model absorbs the image-view loss (PR-AUC ~0.208 at 1.5%). Assemble depth, PR-AUC, and MAD values into results tables (e.g., gcn_depth_performance.csv, pr_auc_downsample_1.5pct.csv) and plot PR-AUC/MAD against depth to reproduce the Figure 4/5 trends.

## Related tools

- **GILoop** (Dual-branch GCN+U-Net loop-calling model; provides the matrix downsampling function and pre-processing pipeline used to generate depth-series inputs and branch-level predictions) — github.com/fzbio/GILoop
- **Peakachu** (Machine-learning loop caller used as the baseline against which GILoop's PR-AUC is compared at full depth (520M reads), in both clustering and non-clustering modes)

## Examples

```
python demo.py
```

## Evaluation signals

- At the 1.5% downsample (39 million cis-reads), mean PR-AUC should approximate: fused ~0.208, U-Net ~0.135, GCN ~0.101
- GCN-branch PR-AUC must rise steadily as depth decreases, while the U-Net (image) branch falls — the two trends must move in opposite directions
- GCN MAD smoothness measurements across depths should track the same performance shift as its PR-AUC
- The reproduced PR-AUC/MAD-vs-depth curves must match the Figure 4/5 trend, with the GCN peak near 260 million reads (the paper's optimal downsampling density)
- GILoop's PR curves should lie above Peakachu (full and non-clustering) across score thresholds on the 520M-read test set

## Limitations

- GILoop only handles bulk Hi-C and cannot deal with extremely sparse single-cell Hi-C data
- Pre-processing (including the downsampling procedure: matrix transformations and hard disk reading/writing) requires a relatively large amount of time before training
- Usability is weak if the targeted cell line's sequencing depth differs from any of the pre-trained models in the repository
- Juicer-dump text inputs must be KR-normalized O/E matrices with similar read counts in source and target, and no preprocessing is applied in that mode

## Evidence

- [abstract] distinct matrix-density preferences: "We also uncover distinct preferences for matrix density by graph-based and image-based models"
- [discussion] optimal downsampling density: "which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts)"
- [discussion] downsampling function availability: "In the GILoop repository, we provide a function for matrix downsampling at the training stage"
- [discussion] robustness scope: "Despite the robustness against low sequencing depth (of bulk Hi-C), the proposed tool is still not capable enough to deal with"
- [readme] input format requirement: "please make sure the text files are KR-normalized O/E matrices, and the source and the target are of similar number of reads"
