---
name: mean-average-distance-evaluation
description: Use when evaluating loop-anchor localization accuracy of a GILoop GCN-branch
  model on downsampled Hi-C contact maps, or when reproducing the Figure 5 trend showing
  MAD tracking GCN PR-AUC as depth decreases toward the 260-million-read optimum.
when_to_use_negative:
- 'Input is single-cell Hi-C (scHi-C): the article states GILoop ''only focuses on
  bulk Hi-C'' and is ''not capable of dealing with extremely sparse single-cell Hi-C
  data'''
- The matrix has not been sequencing-depth aligned or normalized (e.g., KR-normalized
  O/E matrices are assumed when using Juicer dump input), so MAD values would not
  be comparable across depths
- Only the image-based branch is of interest, since the depth-preference trend being
  evaluated (PR-AUC rising as depth decreases) applies to the graph-based GCN branch
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: GILoop
  role: Dual-branch neural network whose GCN branch is run at each downsampling level
    to produce loop predictions scored by PR-AUC and MAD; its repository provides
    the matrix downsampling function used to generate the depth series
  repo: https://github.com/fzbio/GILoop
- name: SIP
  role: Referenced image-based loop caller (treats matrices as images) contextualizing
    the graph-versus-image density preference being evaluated
  repo: https://github.com/PouletAxel/SIP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:4884b27ef82e16f5b186e3c40ef632bad3b82fd84d73f28b7a0ee99a5e520002
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fzbio/GILoop
  tools:
  - GILoop
  - SIP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mean-average-distance-evaluation

## Summary

Computes Mean Average Distance (MAD, Equations 14-16 of GILoop) to quantify how accurately the GCN branch of GILoop localizes CTCF-mediated loop anchors on Hi-C contact maps, and how this localization metric shifts across sequencing depths. It is the paper's counterpart to PR-AUC for reproducing the distinct matrix-density preferences of graph-based versus image-based loop callers.

## When to use

Use when evaluating loop-anchor localization accuracy of a GILoop GCN-branch model on downsampled Hi-C contact maps, or when reproducing the Figure 5 trend showing MAD tracking GCN PR-AUC as depth decreases toward the 260-million-read optimum.

## When NOT to use

- Input is single-cell Hi-C (scHi-C): the article states GILoop 'only focuses on bulk Hi-C' and is 'not capable of dealing with extremely sparse single-cell Hi-C data'
- The matrix has not been sequencing-depth aligned or normalized (e.g., KR-normalized O/E matrices are assumed when using Juicer dump input), so MAD values would not be comparable across depths
- Only the image-based branch is of interest, since the depth-preference trend being evaluated (PR-AUC rising as depth decreases) applies to the graph-based GCN branch

## Inputs

- Seven downsampled Hi-C test contact maps spanning the downsampling levels (generated via the GILoop repository's matrix downsampling function)
- Ground-truth loop labels (CTCF-mediated loop annotations) for the test Hi-C data
- Trained GILoop dual-branch model (GCN branch and image-based U-Net branch checkpoints from the GILoop 'models' assets)

## Outputs

- gcn_depth_performance.csv: table of sequencing depth, GCN-branch PR-AUC, and MAD at each of the seven downsampling levels
- Plot of PR-AUC and MAD versus sequencing depth reproducing the Figure 5 trend

## How to apply

For each of the seven downsampled Hi-C test contact maps (generated with the GILoop repository's matrix downsampling function), run inference with the trained GCN branch to score candidate CTCF-mediated loops against ground-truth loop labels. At each downsampling level, compute precision-recall AUC for the GCN predictions and then compute Mean Average Distance following Equations 14-16 of the paper. Assemble depth, PR-AUC, and MAD into a single results table (gcn_depth_performance.csv) and plot both metrics against sequencing depth to reproduce the Figure 5 trend. Decision point: the expected trend is GCN-branch PR-AUC and MAD improving as depth decreases, peaking around the experimentally obtained optimum of 260 million cis-contacts, while the image-based U-Net branch shows the opposite trend — if the image branch does not fall and the GCN branch does not rise, the reproduction should be treated as failed.

## Related tools

- **GILoop** (Dual-branch neural network whose GCN branch is run at each downsampling level to produce loop predictions scored by PR-AUC and MAD; its repository provides the matrix downsampling function used to generate the depth series) — https://github.com/fzbio/GILoop
- **SIP** (Referenced image-based loop caller (treats matrices as images) contextualizing the graph-versus-image density preference being evaluated) — https://github.com/PouletAxel/SIP

## Examples

```
conda activate GIL && python demo.py  # full GILoop pipeline on .cool input; then score GCN predictions per downsampling level and compute MAD per Equations 14-16
```

## Evaluation signals

- Reproduced GCN-branch PR-AUC must rise steadily as sequencing depth decreases, peaking around 260 million reads (the paper's experimentally obtained optimal density of cis-contacts)
- MAD smoothness measurements across depths must track the same performance shift as PR-AUC (Figure 5 trend)
- The image-based U-Net branch must show the opposite trend (performance collapsing at reduced depth) to confirm the distinct matrix-density preferences of graph-based and image-based models
- The results table gcn_depth_performance.csv must contain exactly seven rows, one per downsampling level, with depth, PR-AUC, and MAD columns populated
- Input matrices must be KR-normalized O/E matrices with source and target of similar read count if using the Juicer dump text-file path

## Limitations

- GILoop is not applicable to extremely sparse single-cell Hi-C data despite its robustness to low bulk Hi-C sequencing depth
- GILoop requires a relatively large amount of pre-processing time before training, mainly from the downsampling procedure (matrix transformations and hard disk reading/writing)
- Usability is weak if the targeted cell line's sequencing depth differs from any pre-trained model provided in the repository
- The low-depth GCN branch can be 'very low and unstable at high depths', so a single depth level may not give a stable MAD estimate

## Evidence

- [abstract] Graph-based and image-based models show distinct preferences for matrix density: "We also uncover distinct preferences for matrix density by graph-based and image-based models"
- [discussion] Optimal downsampling density of 260 million cis-contacts: "which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts)"
- [discussion] Matrix downsampling function in the GILoop repository: "In the GILoop repository, we provide a function for matrix downsampling at the training stage"
- [readme] GILoop demo pipeline with sequencing depth alignment and normalization: "This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.)"
- [readme] GILoop purpose on Hi-C contact maps: "GILoop is a deep learning model for detecting CTCF-mediated loops on Hi-C contact maps."
