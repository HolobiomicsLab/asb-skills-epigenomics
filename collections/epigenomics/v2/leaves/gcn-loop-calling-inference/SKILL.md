---
name: gcn-loop-calling-inference
description: Use when evaluating how a graph-based loop caller's PR-AUC and smoothness
  (MAD) change across Hi-C sequencing depths, or when comparing graph-view vs image-view
  branch robustness on downsampled contact maps.
when_to_use_negative:
- The input is extremely sparse single-cell Hi-C (scHi-C) data — GILoop only focuses
  on bulk Hi-C and is not capable of dealing with it
- The goal is to evaluate the image-based (U-Net) branch, whose PR-AUC falls rather
  than rises as depth decreases
- The contact maps are raw unnormalized matrices fed via demo_from_processed.py, which
  expects KR-normalized O/E matrices and applies no preprocessing
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3912
tools:
- name: GILoop
  role: Dual-branch (GCN + U-Net) loop caller; the GCN branch is run in inference
    mode per downsampling depth, and its repository provides the matrix downsampling
    function used to generate the seven depth levels
  repo: github.com/fzbio/GILoop
- name: SIP
  role: Image-based loop caller referenced as an example of treating Hi-C matrices
    as images (the alternative view to the graph branch); not used in the inference
    workflow itself
  repo: PouletAxel/SIP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:0ad3586a13757644b740e8a964068b0fb88940c246119ba9fbadbc46ff41c6b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  - SIP
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3912
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gcn-loop-calling-inference

## Summary

Run inference with the trained graph-convolutional (GCN) branch of GILoop on downsampled Hi-C contact maps, score candidate CTCF-mediated loops, and quantify depth-dependent behavior via PR-AUC and Mean Average Distance (MAD). This reproduces the paper's finding that GCN performance rises as sequencing depth decreases, peaking around 260 million cis-contacts.

## When to use

Use when evaluating how a graph-based loop caller's PR-AUC and smoothness (MAD) change across Hi-C sequencing depths, or when comparing graph-view vs image-view branch robustness on downsampled contact maps.

## When NOT to use

- The input is extremely sparse single-cell Hi-C (scHi-C) data — GILoop only focuses on bulk Hi-C and is not capable of dealing with it
- The goal is to evaluate the image-based (U-Net) branch, whose PR-AUC falls rather than rises as depth decreases
- The contact maps are raw unnormalized matrices fed via demo_from_processed.py, which expects KR-normalized O/E matrices and applies no preprocessing

## Inputs

- Seven downsampled Hi-C test contact maps generated with the GILoop matrix downsampling function
- Ground-truth CTCF-mediated loop labels for the test set
- Trained GILoop GCN-branch model checkpoint (from models/ assets)

## Outputs

- gcn_depth_performance.csv with per-depth PR-AUC and MAD values
- Plot of PR-AUC and MAD versus sequencing depth reproducing the Figure 5 trend

## How to apply

Generate seven downsampled Hi-C test contact maps using the GILoop repository's matrix downsampling function, spanning a range of sequencing depths down to and including the experimentally optimal density of 260 million cis-contacts. For each depth, run inference with the trained GCN branch to score candidate CTCF-mediated loops against ground-truth loop labels, then compute precision-recall AUC per depth. Additionally compute Mean Average Distance (MAD) following Equations 14-16 of the paper to measure prediction smoothness across depths. Assemble depth, PR-AUC, and MAD into a single table (e.g., gcn_depth_performance.csv) and plot both metrics against depth to reproduce the Figure 5 trend. Validation requires GCN-branch PR-AUC to rise as depth decreases (the image-based U-Net branch shows the opposite, collapsing at low depth), with MAD smoothness tracking the same shift.

## Related tools

- **GILoop** (Dual-branch (GCN + U-Net) loop caller; the GCN branch is run in inference mode per downsampling depth, and its repository provides the matrix downsampling function used to generate the seven depth levels) — github.com/fzbio/GILoop
- **SIP** (Image-based loop caller referenced as an example of treating Hi-C matrices as images (the alternative view to the graph branch); not used in the inference workflow itself) — PouletAxel/SIP

## Examples

```
python demo.py
```

## Evaluation signals

- GCN-branch PR-AUC rises steadily as depth decreases, peaking around 260 million reads
- MAD smoothness measurements across depths track the same performance shift as PR-AUC
- The reproduced trend is directionally opposite to the image-based U-Net branch, whose performance sharply drops at reduced depth
- The results table contains seven (depth, PR-AUC, MAD) rows, one per downsampling level
- MAD is computed strictly per Equations 14-16 of the paper

## Limitations

- GILoop only focuses on bulk Hi-C and is not capable of dealing with extremely sparse single-cell Hi-C (scHi-C) data
- GILoop requires a relatively large amount of pre-processing time before training, mainly from the downsampling procedure (matrix transformations and hard disk reading/writing)
- Usability is weak if the targeted cell line has a sequencing depth different from any of the pre-trained models provided in the repository

## Evidence

- [discussion] which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts): "which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts)"
- [discussion] In the GILoop repository, we provide a function for matrix downsampling at the training stage: "In the GILoop repository, we provide a function for matrix downsampling at the training stage"
- [abstract] We also uncover distinct preferences for matrix density by graph-based and image-based models: "We also uncover distinct preferences for matrix density by graph-based and image-based models"
- [abstract] Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops.: "Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops."
- [readme] please make sure the text files are KR-normalized O/E matrices: "please make sure the text files are KR-normalized O/E matrices, and the source and the target are of similar number of reads."
