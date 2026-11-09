---
name: gcn-graph-based-loop-classifier-training
description: Use when a Hi-C cis-contact map must be encoded as a graph (nodes/edges
  weighted by contact counts) to train or evaluate a graph-only CTCF loop-calling
  classifier, e.g. as a uni-modal baseline for view-complementarity tests.
when_to_use_negative:
- Input is extremely sparse single-cell Hi-C (scHi-C) data — GILoop only focuses on
  bulk Hi-C and is not capable of dealing with it
- The analysis goal is image-view loop calling — the article reports distinct preferences
  for matrix density by graph-based vs image-based models, so the GCN branch alone
  is not a substitute for the U-Net image branch
- Applying a pre-trained model to a cell line whose sequencing depth differs from
  any pre-trained model in the repository, without retraining or downsampling alignment
tools:
- name: GILoop
  role: Dual-branch framework providing the GCN graph branch, training configuration,
    and downsampling/'lazy' training modes for the graph-only loop classifier
  repo: github.com/fzbio/GILoop
- name: FAN-C
  role: Hi-C pre-processing upstream of GILoop training; its speed is a noted bottleneck
    deferred to a future faster version
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:e9d9bf459981944f61110be52f07d752635dc042bad845aea50031148aaea688
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FAN-C
  - GILoop
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# gcn-graph-based-loop-classifier-training

## Summary

Trains a GCN-only branch of GILoop that treats the Hi-C intra-chromosomal contact matrix as a graph to call CTCF-mediated loops, serving as the graph-view baseline against the fused dual-branch model.

## When to use

Use when a Hi-C cis-contact map must be encoded as a graph (nodes/edges weighted by contact counts) to train or evaluate a graph-only CTCF loop-calling classifier, e.g. as a uni-modal baseline for view-complementarity tests.

## When NOT to use

- Input is extremely sparse single-cell Hi-C (scHi-C) data — GILoop only focuses on bulk Hi-C and is not capable of dealing with it
- The analysis goal is image-view loop calling — the article reports distinct preferences for matrix density by graph-based vs image-based models, so the GCN branch alone is not a substitute for the U-Net image branch
- Applying a pre-trained model to a cell line whose sequencing depth differs from any pre-trained model in the repository, without retraining or downsampling alignment

## Inputs

- Hi-C intra-chromosomal contact matrix (KR-normalized O/E), e.g. .cool file or Juicer dump text format
- CTCF ChIA-PET loop labels (binary loop/no-loop) matched to the contact regions
- GILoop training configuration from github.com/fzbio/GILoop

## Outputs

- Trained GCN-only loop-calling model
- Per-chromosome precision-recall curves and PR-AUC values for the graph branch
- Test statistics (e.g. Wilcoxon signed-rank) comparing graph-only PR-AUC to fused and U-Net branches, writable to pr_auc_comparison.csv

## How to apply

Convert each intra-chromosomal contact region of the GM12878 Hi-C matrix (e.g. the 520-million-read 20% downsample) into a graph with nodes/edges weighted by contact counts, and attach binary loop/no-loop labels from matched CTCF ChIA-PET data. Train the GCN-only model on the graph representations using the GILoop training configuration from github.com/fzbio/GILoop, keeping the same labels and train/test splits as any image-branch or fused model so results are directly comparable. Optionally use the 'lazy' non-downsampling mode, which directly trains the GCN branch on the input matrix, or the standard mode that downsamples to the optimal density of 260 million cis-contacts. Evaluate on held-out chromosomes by computing precision-recall curves and per-chromosome PR-AUC, then compare against the fused and U-Net models (e.g. Wilcoxon signed-rank test, p < 0.05) to quantify the graph view's contribution relative to the complementary image view.

## Related tools

- **GILoop** (Dual-branch framework providing the GCN graph branch, training configuration, and downsampling/'lazy' training modes for the graph-only loop classifier) — github.com/fzbio/GILoop
- **FAN-C** (Hi-C pre-processing upstream of GILoop training; its speed is a noted bottleneck deferred to a future faster version)

## Examples

```
conda create -n GIL python=3.8 && conda activate GIL && pip install -r requirements.txt && python demo.py
```

## Evaluation signals

- Per-chromosome PR-AUC values computed on held-out chromosomes are finite and reported for all models being compared
- Wilcoxon signed-rank test across chromosomes yields p < 0.05 for the expected fused > graph-only ordering reproduced from Figure 2C-D
- Across 11 replicate experiments the fused model consistently outperforms the GCN-only branch; if the graph-only branch wins, the fusion/comparison setup is misconfigured
- Training and evaluation use identical labels and chromosome splits across GCN-only, U-Net-only, and fused models
- Input matrices are KR-normalized O/E with source and target of similar read counts when using the Juicer dump text-file path

## Limitations

- GILoop only focuses on bulk Hi-C and is not capable of dealing with extremely sparse single-cell Hi-C data
- GILoop requires a relatively large amount of pre-processing time before training, mainly from the downsampling procedure (matrix transformations and hard disk reading/writing)
- Weak usability if the targeted cell line has a sequencing depth different from any of the pre-trained models provided in the repository
- The graph-only branch alone underperforms the fused model; combining both views is needed for the state-of-the-art result

## Evidence

- [abstract] dual-branch learning from graph and image views: "Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops."
- [abstract] view complementarity: "we explore the combined strength of integrating the two view representations of Hi-C data and corroborate the complementary relationship between the views"
- [abstract] graph vs image matrix density preferences: "We also uncover distinct preferences for matrix density by graph-based and image-based models"
- [discussion] lazy non-downsampling mode directly trains GCN branch: "users can also choose to use the non-downsampling mode (the “lazy” mode) to train the model, which directly trains the GCN branch on the input matrix"
- [readme] README: full pipeline with pre-processing: "This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.), and GILoop sampling and training."
- [readme] README: KR-normalized O/E input requirement: "please make sure the text files are KR-normalized O/E matrices, and the source and the target are of similar number of reads."
