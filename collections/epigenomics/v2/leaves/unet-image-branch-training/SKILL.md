---
name: unet-image-branch-training
description: Use when training GILoop on Hi-C contact matrices and you need the image
  branch to learn from high-depth data — e.g., when combining branches pre-trained
  at their preferred matrix densities (U-Net at full depth) before mixed-density fine-tuning.
when_to_use_negative:
- Input is single-cell Hi-C (scHi-C) data — GILoop is not capable enough to deal with
  extremely sparse single-cell Hi-C data
- The targeted cell line's sequencing depth differs from any pre-trained models in
  the repository and no transfer-learning strategy is intended
- A single-view loop caller (e.g., Peakachu, SIP, Chromosight) suffices and dual-branch
  training overhead is not warranted
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3420
tools:
- name: GILoop
  role: Dual-branch neural network framework providing the U-Net image branch, training
    pipeline, and matrix downsampling function used for pre-training at full depth
  repo: github.com/fzbio/GILoop
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:060863fd1af9801c2f2a24ff235f85f6ca3dec2a691c5dc950a9956c664eb646
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3420
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# unet-image-branch-training

## Summary

Pre-train GILoop's image branch (U-Net) on full-depth (100%) GM12878 Hi-C contact maps, exploiting image-based models' preference for dense matrices, before joint fine-tuning of the dual-branch model. This leverages the complementary strengths of graph and image views of cis-contact maps for CTCF-mediated loop calling.

## When to use

Use when training GILoop on Hi-C contact matrices and you need the image branch to learn from high-depth data — e.g., when combining branches pre-trained at their preferred matrix densities (U-Net at full depth) before mixed-density fine-tuning.

## When NOT to use

- Input is single-cell Hi-C (scHi-C) data — GILoop is not capable enough to deal with extremely sparse single-cell Hi-C data
- The targeted cell line's sequencing depth differs from any pre-trained models in the repository and no transfer-learning strategy is intended
- A single-view loop caller (e.g., Peakachu, SIP, Chromosight) suffices and dual-branch training overhead is not warranted

## Inputs

- KR-normalized O/E GM12878 Hi-C contact matrices (Juicer dump format or .cool files)
- Full-depth (100%) downsampled GM12878 contact maps generated with the GILoop repository downsampling function
- CTCF-mediated loop labels for training

## Outputs

- Pre-trained U-Net image branch weights for GILoop
- Jointly fine-tuned dual-branch GILoop model
- Mean PR-AUC on held-out test chromosomes (expected ~0.3790 for mixed-density fine-tuned model vs 0.3718 for single-depth baseline)
- pr_auc_comparison.csv with mean PR-AUC values

## How to apply

Load downsampled GM12878 Hi-C contact matrices using GILoop's data loaders, applying the repository-provided matrix downsampling function to generate the 100%-depth training set. Pre-train the image branch (U-Net) on these full-depth GM12878 contact maps, where image-based models are known to perform best. Then jointly fine-tune the dual-branch GILoop model on the mixed-density training set so both branches learn complementary loop-calling patterns across depths. Evaluate the fine-tuned model on held-out test chromosomes and compute mean PR-AUC; the mixed-density fine-tuned model should reach approximately 0.3790 vs 0.3718 for the best single-depth (90%) baseline (Table S11). The rationale is the demonstrated complementarity between graph and image views of Hi-C data, where each view has distinct preferred matrix densities.

## Related tools

- **GILoop** (Dual-branch neural network framework providing the U-Net image branch, training pipeline, and matrix downsampling function used for pre-training at full depth) — github.com/fzbio/GILoop

## Examples

```
python demo.py  # full pipeline including pre-processing, sampling, and dual-branch (U-Net image + GCN graph) training on GM12878 .cool input
```

## Evaluation signals

- Mean PR-AUC of the mixed-density fine-tuned dual-branch model exceeds the best single-depth 90% pre-training baseline (0.3790 vs 0.3718, per Table S11)
- Image branch is pre-trained on 100%-depth GM12878 matrices, consistent with image-based models' distinct preference for denser matrices
- After joint fine-tuning, both branches improve over their individually pre-trained single-depth performance on held-out test chromosomes
- pr_auc_comparison.csv contains the expected mean PR-AUC values and correctly contrasts mixed-density vs single-depth baselines

## Limitations

- GILoop only focuses on bulk Hi-C and is not capable of dealing with extremely sparse single-cell Hi-C (scHi-C) data
- GILoop requires a relatively large amount of pre-processing time before training, mainly from the downsampling procedure (matrix transformations and hard disk reading/writing)
- Weak usability if the targeted cell line has a sequencing depth different from any of the pre-trained models provided in the repository

## Evidence

- [abstract] Image branch full-depth preference and complementarity: "We also uncover distinct preferences for matrix density by graph-based and image-based models"
- [abstract] Complementary graph and image views: "we explore the combined strength of integrating the two view representations of Hi-C data and corroborate the complementary relationship between the views"
- [discussion] Downsampling function in repository: "In the GILoop repository, we provide a function for matrix downsampling at the training stage"
- [readme] Input format requirement: "please make sure the text files are KR-normalized O/E matrices, and the source and the target are of similar number of reads"
- [readme] Pre-processing overhead: "The preprocessing overhead could be heavy and could lead to a large running time."
