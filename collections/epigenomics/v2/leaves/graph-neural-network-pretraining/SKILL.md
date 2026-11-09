---
name: graph-neural-network-pretraining
description: Use when Hi-C contact matrices at multiple sequencing depths (e.g. 10%-downsampled
  and full-depth GM12878 maps) are available and a dual-branch CTCF loop caller should
  be trained to exploit per-branch matrix-density preferences.
when_to_use_negative:
- Input is extremely sparse single-cell Hi-C (scHi-C) data — GILoop only focuses on
  bulk Hi-C and cannot deal with it
- The target cell line's sequencing depth matches one of the pre-trained models already
  provided in the GILoop repository — reuse that model instead of re-running branch
  pre-training
- Only a single sequencing depth is available and no downsampling is intended — single-depth
  90% pre-training is the fallback baseline, not this mixed-density scheme
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: GILoop
  role: Dual-branch neural network whose GCN graph branch and U-Net image branch are
    pre-trained at their preferred matrix densities (10% downsampled vs full-depth
    GM12878) and jointly fine-tuned on the mixed-density set for CTCF-mediated loop
    calling
  repo: github.com/fzbio/GILoop
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:09b3e3b11443018dfa638b7ec102236f76cd4ef366c6a5255b02400716a51afa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# graph-neural-network-pretraining

## Summary

Pre-train the two branches of GILoop's dual-branch CTCF loop caller at their preferred Hi-C matrix densities — the GCN graph branch on 10%-downsampled GM12878 contact maps and the U-Net image branch on full-depth maps — then jointly fine-tune on the mixed-density set, raising mean PR-AUC to 0.3790 versus 0.3718 for the best single-depth 90% training.

## When to use

Use when Hi-C contact matrices at multiple sequencing depths (e.g. 10%-downsampled and full-depth GM12878 maps) are available and a dual-branch CTCF loop caller should be trained to exploit per-branch matrix-density preferences.

## When NOT to use

- Input is extremely sparse single-cell Hi-C (scHi-C) data — GILoop only focuses on bulk Hi-C and cannot deal with it
- The target cell line's sequencing depth matches one of the pre-trained models already provided in the GILoop repository — reuse that model instead of re-running branch pre-training
- Only a single sequencing depth is available and no downsampling is intended — single-depth 90% pre-training is the fallback baseline, not this mixed-density scheme

## Inputs

- GM12878 Hi-C intra-chromosomal cis-contact maps at full (100%) sequencing depth, as .cool files or Juicer-dump-format KR-normalized O/E text files
- 10%-depth downsampled GM12878 contact matrices generated with the repository-provided matrix downsampling function
- Chromosome-level train/test split with held-out test chromosomes

## Outputs

- Pre-trained GCN graph-branch weights from 10%-depth GM12878 training
- Pre-trained U-Net image-branch weights from 100%-depth GM12878 training
- Jointly fine-tuned dual-branch GILoop model weights
- Mean PR-AUC comparison file pr_auc_comparison.csv (0.3790 mixed-density vs 0.3718 single-depth 90%, per Table S11)

## How to apply

Use the GILoop repository's provided matrix downsampling function to generate 10%-depth and 100%-depth training matrices from GM12878 Hi-C contact maps (the paper's optimal training-stage downsampling target is 260 million cis-contacts; inputs may be .cool files or Juicer-dump-format KR-normalized O/E text). Pre-train the graph branch (GCN) on the sparse 10%-depth maps, exploiting the graph view's robustness at low sequencing depth, and pre-train the image branch (U-Net) on the full-depth maps where image-based models perform best. Jointly fine-tune the dual-branch GILoop model on the mixed-density training set so both branches learn complementary loop-calling patterns across depths. Evaluate on held-out test chromosomes and compute mean precision-recall AUC (PR-AUC), comparing against the best single-depth 90% pre-training baseline. Correct application yields mean PR-AUC 0.3790 for the mixed-density fine-tuned model versus 0.3718 for single-depth 90% training (Table S11), recorded in pr_auc_comparison.csv.

## Related tools

- **GILoop** (Dual-branch neural network whose GCN graph branch and U-Net image branch are pre-trained at their preferred matrix densities (10% downsampled vs full-depth GM12878) and jointly fine-tuned on the mixed-density set for CTCF-mediated loop calling) — github.com/fzbio/GILoop

## Examples

```
conda activate GIL && python demo.py
```

## Evaluation signals

- Mixed-density fine-tuned model reaches mean PR-AUC 0.3790, exceeding the 0.3718 of the best single-depth 90% pre-trained models (Table S11)
- Branch-density assignment is correct: GCN branch pre-trained on 10%-depth downsampled matrices, U-Net branch on 100%-depth (full-depth) GM12878 matrices
- Downsampled matrices are generated with the repository-provided matrix downsampling function (optimal density 260 million cis-contacts), not ad-hoc scripts
- Evaluation is restricted to held-out test chromosomes and reported as mean precision-recall AUC (PR-AUC)
- pr_auc_comparison.csv contains both mean PR-AUC values (0.3790 and 0.3718) for the comparison

## Limitations

- GILoop only focuses on bulk Hi-C and is not capable of dealing with extremely sparse single-cell Hi-C (scHi-C) data
- GILoop requires a relatively large amount of pre-processing time before training, mainly from the downsampling procedure (matrix transformations and hard disk reading/writing)
- Weak usability if the targeted cell line has a sequencing depth different from any of the pre-trained models provided in the repository

## Evidence

- [abstract] Dual-branch architecture of GILoop: "Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops."
- [abstract] Distinct matrix-density preferences of graph- vs image-based models: "We also uncover distinct preferences for matrix density by graph-based and image-based models"
- [discussion] Repository-provided matrix downsampling function: "In the GILoop repository, we provide a function for matrix downsampling at the training stage"
- [discussion] Optimal training-stage downsampling density: "which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts)"
- [discussion] Bulk Hi-C scope limitation: "The tool we developed in this study, however, only focuses on bulk Hi-C."
- [readme] Accepted Hi-C input formats: "You can either use `.cool` files as input, or use the output format of Juicer dump as input."
