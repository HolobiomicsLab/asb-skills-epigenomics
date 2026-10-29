---
name: hic-contact-matrix-downsampling
description: Use when benchmarking loop callers across sequencing depths, training
  GILoop branches at their preferred matrix density (GCN at 10%, U-Net at full depth),
  or testing robustness of CTCF loop calling on sparse Hi-C libraries.
when_to_use_negative:
- Input is extremely sparse single-cell Hi-C (scHi-C) — the article states GILoop
  'only focuses on bulk Hi-C' and 'is still not capable enough' for scHi-C
- The targeted cell line's sequencing depth already matches one of the pre-trained
  models in the GILoop repository, in which case downsampling only adds pre-processing
  overhead
- You are using the 'lazy' non-downsampling training mode, which directly trains the
  GCN branch on the input matrix
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: GILoop
  role: Provides the matrix downsampling function used at the training stage, plus
    the demo.py pipeline (sequencing depth alignment, normalization, expected vector
    calculation) that consumes the downsampled maps
  repo: github.com/fzbio/GILoop
- name: FAN-C
  role: Hi-C pre-processing whose slow runtime motivates future mitigation of downsampling-related
    pre-processing cost (major faster FAN-C version announced)
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_004
  - task_007
  - task_002
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:1f5c59b2debb366e0b99ff402d2ab37897e8eeca836fa075190379306ed075df
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FAN-C
  - GILoop
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# hic-contact-matrix-downsampling

## Summary

Downsamples Hi-C intra-chromosomal contact maps to controlled sequencing depths (e.g. 20% = 520M cis-reads, 1.5% = 39M cis-reads) using the matrix downsampling function provided in the GILoop repository, enabling depth-controlled evaluation of loop-calling performance and training at experimentally optimal matrix density.

## When to use

Use when benchmarking loop callers across sequencing depths, training GILoop branches at their preferred matrix density (GCN at 10%, U-Net at full depth), or testing robustness of CTCF loop calling on sparse Hi-C libraries.

## When NOT to use

- Input is extremely sparse single-cell Hi-C (scHi-C) — the article states GILoop 'only focuses on bulk Hi-C' and 'is still not capable enough' for scHi-C
- The targeted cell line's sequencing depth already matches one of the pre-trained models in the GILoop repository, in which case downsampling only adds pre-processing overhead
- You are using the 'lazy' non-downsampling training mode, which directly trains the GCN branch on the input matrix

## Inputs

- High-depth Hi-C intra-chromosomal contact matrix (e.g. full-depth GM12878, .cool file or Juicer dump KR-normalized O/E matrix)
- CTCF ChIA-PET loop labels (ground truth)
- Target downsampling depth specification (e.g. 1.5% / 39M cis-reads, 20% / 520M cis-reads, 260M cis-contacts optimum)

## Outputs

- Downsampled Hi-C contact matrices at each specified depth (seven levels in the paper)
- Per-depth results table (depth, PR-AUC, MAD) e.g. gcn_depth_performance.csv or pr_auc_downsample_1.5pct.csv
- Downsampled training matrices for branch-specific pre-training (10%-depth for GCN, 100%-depth for U-Net)

## How to apply

Apply the matrix downsampling function provided in the GILoop repository (github.com/fzbio/GILoop) to the raw GM12878 (or other cell line) intra-chromosomal contact matrices, generating downsampled libraries at defined depths (the paper spans seven downsampling levels from 1.5% / 39 million cis-reads up to 100%). The article's discussion notes GILoop can automatically downsample the Hi-C contact map to the experimentally obtained optimal density of 260 million cis-contacts, and a non-downsampling 'lazy' mode exists that instead trains the GCN branch directly on the input matrix. Feed the downsampled matrices into the GILoop pipeline via demo.py, which performs data pre-processing (sequencing depth alignment, normalization, and expected vector calculation) before sampling and training; if starting from Juicer dump output, use KR-normalized O/E matrices from sources with similar read counts and skip pre-processing with demo_from_processed.py. Rationale: depth-controlled downsampling is what exposes the paper's key findings — the complementary graph/image views (fused PR-AUC ~0.208 vs U-Net 0.135 and GCN 0.101 at 1.5% depth) and the opposite depth preferences of the GCN and U-Net branches.

## Related tools

- **GILoop** (Provides the matrix downsampling function used at the training stage, plus the demo.py pipeline (sequencing depth alignment, normalization, expected vector calculation) that consumes the downsampled maps) — github.com/fzbio/GILoop
- **FAN-C** (Hi-C pre-processing whose slow runtime motivates future mitigation of downsampling-related pre-processing cost (major faster FAN-C version announced))

## Evaluation signals

- Downsampled library read counts match the nominal depth (e.g. 39 million cis-reads at 1.5%, 520 million cis-reads at 20%)
- At 1.5% depth, fused-model PR-AUC falls near ~0.208 with U-Net ~0.135 and GCN ~0.101 (Figure 4, Tables S4-S10)
- GCN-branch PR-AUC across the seven downsampling levels rises steadily as depth decreases, peaking around 260 million reads, with MAD smoothness tracking the same trend (Figure 5)
- Mixed-density fine-tuned model (GCN at 10%-depth, U-Net at 100%-depth) reaches mean PR-AUC 0.3790 vs 0.3718 for single-depth 90% training (Table S11)
- Downsampling procedure runs without error from the repository-provided function and the outputs load through GILoop data loaders

## Limitations

- GILoop requires relatively large pre-processing time before training, mainly from the downsampling procedure (matrix transformations and hard disk reading/writing)
- Weak usability if the targeted cell line has a sequencing depth different from any of the pre-trained models provided in the repository
- Downsampling is specific to bulk Hi-C; it cannot make the tool adequate for extremely sparse single-cell Hi-C data

## Evidence

- [discussion] Matrix downsampling at the training stage to the optimal density of 260 million cis-contacts: "which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts)"
- [discussion] Repository downsampling function: "In the GILoop repository, we provide a function for matrix downsampling at the training stage"
- [discussion] Non-downsampling lazy mode: "users can also choose to use the non-downsampling mode (the “lazy” mode) to train the model, which directly trains the GCN branch on the input matrix"
- [readme] Pre-processing pipeline in demo.py: "This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.)"
- [readme] KR-normalized O/E input requirement: "please make sure the text files are KR-normalized O/E matrices, and the source and the target are of similar number of reads"
