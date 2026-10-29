---
name: unet-image-based-loop-classifier-training
description: Use when Hi-C contact matrices (KR-normalized O/E, .cool or Juicer dump
  format) must be converted to image-view patches and a U-Net classifier trained with
  binary loop/no-loop labels, e.g. as the image branch or baseline in GILoop experiments.
when_to_use_negative:
- Input is extremely sparse single-cell Hi-C (scHi-C) data — the article states the
  proposed tool 'only focuses on bulk Hi-C' and is not capable of dealing with scHi-C
- The goal is a standalone production loop caller rather than a branch/baseline —
  the fused dual-branch GILoop outperforms the U-Net branch alone, so the single view
  is mainly for complementarity testing
- Juicer dump text inputs are not KR-normalized O/E matrices — the README requires
  'KR-normalized O/E matrices' for the processed-input path, which skips preprocessing
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: GILoop
  role: Provides the dual-branch framework whose image view is the U-Net; its repository
    supplies the training configuration, downsampling function, and demo pipeline
  repo: github.com/fzbio/GILoop
- name: FAN-C
  role: Hi-C preprocessing (normalization, matrix transformations) whose runtime the
    article notes is dominated by downsampling-related I/O
- name: SIP
  role: Prior image-based loop caller referenced as treating matrices as images; used
    as a non-ML baseline context rather than for training
  repo: https://github.com/PouletAxel/SIP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:b75f42fe4312e220d953b1a8010e71bfebd3ed6fdf967133616ff09a91923aeb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FAN-C
  - GILoop
  - SIP
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# U-Net image-based loop classifier training

## Summary

Trains the U-Net-only (image-view) branch of the GILoop framework on 2D Hi-C intra-chromosomal contact matrix patches to classify CTCF-mediated loops, serving both as a baseline for view-complementarity tests and as a component of the fused dual-branch model.

## When to use

Use when Hi-C contact matrices (KR-normalized O/E, .cool or Juicer dump format) must be converted to image-view patches and a U-Net classifier trained with binary loop/no-loop labels, e.g. as the image branch or baseline in GILoop experiments.

## When NOT to use

- Input is extremely sparse single-cell Hi-C (scHi-C) data — the article states the proposed tool 'only focuses on bulk Hi-C' and is not capable of dealing with scHi-C
- The goal is a standalone production loop caller rather than a branch/baseline — the fused dual-branch GILoop outperforms the U-Net branch alone, so the single view is mainly for complementarity testing
- Juicer dump text inputs are not KR-normalized O/E matrices — the README requires 'KR-normalized O/E matrices' for the processed-input path, which skips preprocessing

## Inputs

- GM12878 (or other cell line) Hi-C intra-chromosomal contact matrices in .cool or KR-normalized O/E Juicer dump text format
- Binary loop/no-loop labels from matched CTCF ChIA-PET data
- Per-chromosome train/test split (held-out evaluation chromosomes)

## Outputs

- Trained U-Net image-branch model checkpoint
- Per-chromosome precision-recall curves and PR-AUC values for the image-only model
- pr_auc_comparison.csv rows for the image branch in fused vs. single-branch comparisons

## How to apply

Convert each Hi-C intra-chromosomal contact region into a 2D contact matrix patch (the image view) from KR-normalized O/E matrices, and attach binary loop/no-loop labels derived from matched CTCF ChIA-PET data. Downsample the Hi-C contact map to the optimal density (the paper experimentally obtained 260 million cis-contacts; for the 20% GM12878 experiment this is the 520-million-read matrix) using the matrix downsampling function provided in the GILoop repository, or choose the non-downsampling 'lazy' mode. Train the U-Net on these image representations following the GILoop training configuration in github.com/fzbio/GILoop, evaluating on held-out chromosomes with precision-recall curves and PR-AUC per chromosome. When compared against the GCN graph branch and the fused dual-branch model, test PR-AUC ordering with a Wilcoxon signed-rank test across chromosomes at p < 0.05, as in the Figure 2C-D analysis.

## Related tools

- **GILoop** (Provides the dual-branch framework whose image view is the U-Net; its repository supplies the training configuration, downsampling function, and demo pipeline) — github.com/fzbio/GILoop
- **FAN-C** (Hi-C preprocessing (normalization, matrix transformations) whose runtime the article notes is dominated by downsampling-related I/O)
- **SIP** (Prior image-based loop caller referenced as treating matrices as images; used as a non-ML baseline context rather than for training) — https://github.com/PouletAxel/SIP

## Examples

```
conda activate GIL; python demo.py
```

## Evaluation signals

- PR-AUC per held-out chromosome is computed and the image-only branch underperforms the fused dual-branch model across 11 replicate experiments, as in Figure 2C-D
- Wilcoxon signed-rank test across chromosomes yields p < 0.05 for fused > image-branch PR-AUC
- Trained image branch shows a distinct matrix-density preference relative to the graph branch (graph- and image-based models 'show distinct preferences for matrix density')
- Training matrices are KR-normalized O/E and downsampled to ~260 million cis-contacts (or a matched sequencing-depth alignment) before patching
- Model checkpoints reproduce the transfer-learning behavior with minimal performance decay when applied to another cell line

## Limitations

- GILoop (including the U-Net image branch) only supports bulk Hi-C and cannot handle extremely sparse single-cell Hi-C data
- Relatively large preprocessing time before training, mainly from the downsampling procedure (matrix transformations and hard disk reading/writing)
- Weak usability if the target cell line's sequencing depth matches none of the pre-trained models in the repository, necessitating retraining or downsampling

## Evidence

- [abstract] GILoop integrates graph and image representations of Hi-C data and the two views are complementary: "we explore the combined strength of integrating the two view representations of Hi-C data and corroborate the complementary relationship between the views"
- [abstract] Image and graph branches show distinct density preferences: "We also uncover distinct preferences for matrix density by graph-based and image-based models"
- [discussion] Optimal downsampling density for training: "which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts)"
- [discussion] Repository provides downsampling function for training: "In the GILoop repository, we provide a function for matrix downsampling at the training stage"
- [readme] Juicer-dump input must be KR-normalized O/E matrices: "please make sure the text files are KR-normalized O/E matrices, and the source and the target are of similar number of reads"
