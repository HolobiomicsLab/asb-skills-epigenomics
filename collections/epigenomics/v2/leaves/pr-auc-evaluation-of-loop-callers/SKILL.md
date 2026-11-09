---
name: pr-auc-evaluation-of-loop-callers
description: Use when comparing loop-calling models (fused dual-branch vs. single-view
  U-Net or GCN baselines) on downsampled Hi-C contact maps and reporting robustness
  as mean PR-AUC, e.g. at the 1.5% (39 million cis-reads) depth.
when_to_use_negative:
- Input is single-cell Hi-C (scHi-C) data — the article states GILoop is not capable
  of dealing with extremely sparse single-cell Hi-C
- The goal is generic loop detection rather than controlled PR-AUC benchmarking of
  models against a shared downsampled test set
- The model task is not CTCF-mediated loop classification on bulk Hi-C cis-contact
  maps (e.g. non-loop structural variant calling)
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: GILoop
  role: Dual-branch (GCN graph view + U-Net image view) loop-calling model evaluated
    by mean PR-AUC; repository also provides the matrix downsampling function used
    to create the low-depth benchmark
  repo: github.com/fzbio/GILoop
- name: SIP
  role: Existing image-based loop caller cited as prior single-view baseline context
    in the comparison of loop-calling frameworks
  repo: PouletAxel/SIP
- name: Peakachu
  role: Machine-learning-based loop caller that GILoop significantly outperforms,
    contextualizing the PR-AUC benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:d4b1d02200b0fae032231a3572cad4f2e032643ccac15b4e0bcb2513f1d3e3f3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  - Peakachu
  - SIP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# pr-auc-evaluation-of-loop-callers

## Summary

Benchmark CTCF-mediated chromatin loop-calling models by computing mean precision-recall AUC (PR-AUC) per model on sequencing-depth-downsampled Hi-C GM12878 data, quantifying how the dual-branch GILoop (GCN + U-Net) retains performance where single-view branches degrade.

## When to use

Use when comparing loop-calling models (fused dual-branch vs. single-view U-Net or GCN baselines) on downsampled Hi-C contact maps and reporting robustness as mean PR-AUC, e.g. at the 1.5% (39 million cis-reads) depth.

## When NOT to use

- Input is single-cell Hi-C (scHi-C) data — the article states GILoop is not capable of dealing with extremely sparse single-cell Hi-C
- The goal is generic loop detection rather than controlled PR-AUC benchmarking of models against a shared downsampled test set
- The model task is not CTCF-mediated loop classification on bulk Hi-C cis-contact maps (e.g. non-loop structural variant calling)

## Inputs

- High-depth Hi-C cis-contact matrices (GM12878), KR-normalized O/E matrices from Juicer dump or .cool files
- CTCF-mediated loop candidates and non-loop control instances in paired graph and image representations
- Downsampled matrices at 1.5% sequencing depth (39 million cis-reads) produced by the GILoop downsampling function

## Outputs

- Per-model mean PR-AUC values for fused, U-Net, and GCN models (pr_auc_downsample_1.5pct.csv)
- Comparison against reference PR-AUCs from Figure 4 and Tables S4-S10 (fused ~0.208, U-Net ~0.135, GCN ~0.101)

## How to apply

Downsample high-depth Hi-C cis-contact matrices to 1.5% sequencing depth (39 million cis-reads) with the matrix downsampling function provided in the GILoop repository, then convert the downsampled matrices into paired graph and image training instances centered on CTCF-mediated loop candidates and non-loop controls. Train the fused dual-branch GILoop model (GCN graph view + U-Net image view) plus the single-view U-Net and GCN baselines on the same downsampled training instances. Evaluate all three models on the downsampled test set and compute the mean precision-recall AUC (PR-AUC) per model. Judge correctness by comparing against reference values from Figure 4 and Tables S4-S10: fused ~0.208, U-Net ~0.135, GCN ~0.101, with the fused model absorbing the image-view performance loss. Write per-model mean PR-AUC results to pr_auc_downsample_1.5pct.csv.

## Related tools

- **GILoop** (Dual-branch (GCN graph view + U-Net image view) loop-calling model evaluated by mean PR-AUC; repository also provides the matrix downsampling function used to create the low-depth benchmark) — github.com/fzbio/GILoop
- **SIP** (Existing image-based loop caller cited as prior single-view baseline context in the comparison of loop-calling frameworks) — PouletAxel/SIP
- **Peakachu** (Machine-learning-based loop caller that GILoop significantly outperforms, contextualizing the PR-AUC benchmark)

## Examples

```
python demo.py
```

## Evaluation signals

- Fused model mean PR-AUC at the 1.5% downsample (39M cis-reads) approximates the reference ~0.208 from Figure 4 / Tables S4-S10
- U-Net branch mean PR-AUC falls to ~0.135 and GCN branch to ~0.101 at the 1.5% depth
- The fused model's PR-AUC exceeds both single-view branches, confirming the fusion absorbs the image-view performance loss
- Results are written to pr_auc_downsample_1.5pct.csv with one mean PR-AUC per model
- All three models are trained and evaluated on the identical downsampled training/test instances

## Limitations

- GILoop only focuses on bulk Hi-C and is not capable of dealing with extremely sparse single-cell Hi-C (scHi-C) data
- GILoop requires a relatively large amount of pre-processing time before training, mainly from the downsampling procedure (matrix transformations and hard disk reading/writing)
- Reference PR-AUC values are specific to the GM12878 1.5% downsample (39 million cis-reads); results will differ at other depths or cell lines

## Evidence

- [results] At the 1.5% downsample (39 million cis-reads), the fused model maintains a mean PR-AUC of ~0.208 while the U-Net branch drops to 0.135 and the GCN branch averages 0.101: [evidence span withheld — anchor status: not_found]
- [discussion] In the GILoop repository, we provide a function for matrix downsampling at the training stage: "In the GILoop repository, we provide a function for matrix downsampling at the training stage"
- [abstract] Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops.: "Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops."
- [readme] This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.), and GILoop sampling and training.: "This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.), and GILoop sampling and training."
