---
name: image-based-loop-prediction-model-training
description: Use when you need to train or reproduce a loop-calling model on Hi-C
  cis-contact maps represented as image patches (KR-normalized O/E matrices or .cool
  input), including low-depth downsampled runs.
when_to_use_negative:
- Input is extremely sparse single-cell Hi-C (scHi-C) data — the article states GILoop
  'is still not capable enough to deal with' scHi-C
- You only have raw FASTQ reads or unprocessed contact maps and cannot produce KR-normalized
  O/E matrices or .cool files
- The question concerns the graph view alone (GCN branch on the graph representation)
  rather than image-based prediction
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: GILoop
  role: Dual-branch neural network providing the U-Net image branch, the matrix downsampling
    function, and the full training pipeline (demo.py) for CTCF-mediated loop prediction
    on Hi-C contact maps
  repo: github.com/fzbio/GILoop
- name: SIP
  role: Alternative image-based loop caller that treats Hi-C matrices as images; referenced
    as prior single-view approach, not used for training this model
  repo: PouletAxel/SIP
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:a89a8c5263357d0b40f714fedfde37f26e1e29bf2e11abf7dd073bf5190ce1e0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  - SIP
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# image-based-loop-prediction-model-training

## Summary

Trains GILoop's U-Net image branch (or the fused dual-branch model) to predict CTCF-mediated loops from Hi-C contact maps treated as images, using matrix downsampling to control sequencing depth and PR-AUC as the evaluation metric.

## When to use

Use when you need to train or reproduce a loop-calling model on Hi-C cis-contact maps represented as image patches (KR-normalized O/E matrices or .cool input), including low-depth downsampled runs.

## When NOT to use

- Input is extremely sparse single-cell Hi-C (scHi-C) data — the article states GILoop 'is still not capable enough to deal with' scHi-C
- You only have raw FASTQ reads or unprocessed contact maps and cannot produce KR-normalized O/E matrices or .cool files
- The question concerns the graph view alone (GCN branch on the graph representation) rather than image-based prediction

## Inputs

- Hi-C cis-contact matrices as KR-normalized O/E matrices (Juicer dump text format) or .cool files
- Downsampled Hi-C contact maps (e.g. 1.5% sequencing depth / 39 million cis-reads) produced with the GILoop matrix downsampling function
- Paired image training instances centered on CTCF-mediated loop candidates and non-loop controls

## Outputs

- Trained U-Net image-view loop prediction model (and optionally a fused dual-branch GILoop model)
- Per-model mean PR-AUC values on the test set (e.g. written to pr_auc_downsample_1.5pct.csv)

## How to apply

Convert the intra-chromosomal interaction matrix into image patches centered on CTCF-mediated loop candidates and non-loop controls; KR-normalized O/E matrices are required when starting from Juicer dump text files, while .cool input goes through GILoop's full preprocessing (sequencing depth alignment, normalization, expected vector calculation). Optionally apply the repository's matrix downsampling function at the training stage, either to a target depth such as 1.5% (39 million cis-reads) for robustness experiments or to the experimentally obtained optimal density of 260 million cis-contacts. Train the U-Net image branch on the image representation, and optionally train the fused dual-branch model (GCN graph view + U-Net image view) since the article shows the two views are complementary and fusion absorbs image-view performance loss at low depth. Evaluate on a held-downsampled test set by computing mean precision-recall AUC per model and compare against reference values (e.g. fused ~0.208 vs U-Net ~0.135 vs GCN ~0.101 at 1.5% depth).

## Related tools

- **GILoop** (Dual-branch neural network providing the U-Net image branch, the matrix downsampling function, and the full training pipeline (demo.py) for CTCF-mediated loop prediction on Hi-C contact maps) — github.com/fzbio/GILoop
- **SIP** (Alternative image-based loop caller that treats Hi-C matrices as images; referenced as prior single-view approach, not used for training this model) — PouletAxel/SIP

## Examples

```
python demo_from_processed.py
```

## Evaluation signals

- At the 1.5% downsample (39 million cis-reads), the fused model reaches mean PR-AUC ~0.208, U-Net ~0.135, GCN ~0.101; your per-model PR-AUCs should be in comparable ranges
- Trained model should show robustness against low-quality (low sequencing depth) Hi-C libraries relative to single-view baselines
- Downsampled training data should reach the target depth (e.g. 39 million cis-reads for 1.5%, or 260 million cis-contacts for optimal density)
- Output CSV of per-model mean PR-AUCs (e.g. pr_auc_downsample_1.5pct.csv) should match the reference values from Figure 4 and Tables S4-S10 when reproducing

## Limitations

- Preprocessing before training (especially matrix transformations and hard disk reading/writing during downsampling) is relatively slow
- Usability is weak if the targeted cell line's sequencing depth matches none of the pre-trained models in the repository
- GILoop only handles bulk Hi-C and cannot process extremely sparse single-cell Hi-C data

## Evidence

- [abstract] dual-branch architecture rationale: "Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops."
- [intro] two views of the intra-chromosomal interaction matrix: "We exemplify it by developing a dual-branch neural network termed GILoop, which learns from both views of the intra-chromosomal interaction matrix"
- [discussion] matrix downsampling function: "In the GILoop repository, we provide a function for matrix downsampling at the training stage"
- [discussion] optimal downsampling density: "which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts)"
- [readme] GILoop README tool description: "GILoop is a deep learning model for detecting CTCF-mediated loops on Hi-C contact maps."
- [readme] input format requirement: "please make sure the text files are KR-normalized O/E matrices"
