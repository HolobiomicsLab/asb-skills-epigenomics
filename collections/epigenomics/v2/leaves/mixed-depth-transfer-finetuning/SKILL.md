---
name: mixed-depth-transfer-finetuning
description: 'Use when training GILoop on GM12878 Hi-C and marginal PR-AUC matters:
  pre-train each branch at its preferred matrix density (GCN on 10% downsampled, U-Net
  on full-depth), then jointly fine-tune on the mixed-density set to beat the single-depth
  90% baseline (0.3790 vs 0.3718).'
when_to_use_negative:
- 'Input is single-cell Hi-C (scHi-C): the paper states that despite robustness against
  low bulk sequencing depth, GILoop is still not capable enough to deal with extremely
  sparse scHi-C data.'
- The goal is off-the-shelf loop calling with a repository pre-trained model whose
  sequencing depth already matches the target cell line — mixed-density re-training
  adds heavy pre-processing overhead for no gain.
- Input to demo_from_processed.py is not a KR-normalized O/E matrix, or source and
  target differ substantially in read depth.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: GILoop
  role: Dual-branch (GCN graph + U-Net image) CTCF-mediated loop caller whose branches
    are pre-trained at 10% and 100% GM12878 matrix densities and jointly fine-tuned;
    its repository supplies the matrix downsampling function used to build the mixed-density
    training set
  repo: github.com/fzbio/GILoop
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:b09823f30b709013b378f112a669e60cbf076cec8ddebbb99b679db6da2b8212
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mixed-depth-transfer-finetuning

## Summary

Fine-tunes the dual-branch GILoop CTCF loop caller by pre-training its GCN graph branch on sparse (10%-downsampled) GM12878 Hi-C maps and its U-Net image branch on full-depth maps, then jointly fine-tuning on a mixed-density set — exploiting the branches' distinct matrix-density preferences to lift mean PR-AUC from 0.3718 (best single-depth 90% training) to 0.3790.

## When to use

Use when training GILoop on GM12878 Hi-C and marginal PR-AUC matters: pre-train each branch at its preferred matrix density (GCN on 10% downsampled, U-Net on full-depth), then jointly fine-tune on the mixed-density set to beat the single-depth 90% baseline (0.3790 vs 0.3718).

## When NOT to use

- Input is single-cell Hi-C (scHi-C): the paper states that despite robustness against low bulk sequencing depth, GILoop is still not capable enough to deal with extremely sparse scHi-C data.
- The goal is off-the-shelf loop calling with a repository pre-trained model whose sequencing depth already matches the target cell line — mixed-density re-training adds heavy pre-processing overhead for no gain.
- Input to demo_from_processed.py is not a KR-normalized O/E matrix, or source and target differ substantially in read depth.

## Inputs

- GM12878 Hi-C contact matrices at full depth (100%) in .cool format or as Juicer-dump KR-normalized O/E matrices
- 10%-downsampled GM12878 contact matrices generated with the GILoop repository matrix downsampling function
- Mixed-density training set combining 10%- and 100%-depth matrices with CTCF loop labels
- Held-out test chromosomes with ground-truth CTCF-mediated loop annotations

## Outputs

- Pre-trained GCN branch weights (sparse-matrix expert) and U-Net branch weights (full-depth expert)
- Jointly fine-tuned dual-branch GILoop model
- Mean PR-AUC on held-out test chromosomes written to pr_auc_comparison.csv (target: 0.3790 mixed-depth vs 0.3718 single-depth 90% baseline, per Table S11)

## How to apply

Generate the 10%-depth and 100%-depth GM12878 training matrices with the repository-provided matrix downsampling function, then pre-train each branch at its preferred density: the GCN graph branch on the sparse 10% maps (graph views stay robust at low depth) and the U-Net image branch on full-depth maps (image-based models prefer dense matrices). Jointly fine-tune the dual-branch GILoop on a training set that mixes both densities so each branch learns complementary CTCF-mediated loop-calling patterns. Use demo.py for the full pipeline (sequencing depth alignment, normalization, expected vector calculation, GILoop sampling and training), or demo_from_processed.py with KR-normalized O/E matrices when pre-processing is already done. Evaluate on held-out test chromosomes and report mean PR-AUC against the single-depth 90% pre-training baseline; Table S11 expects 0.3790 (mixed) vs 0.3718 (single-depth). The paper's downsampled mode targets the experimentally optimal density of 260 million cis-contacts, with a non-downsampling ('lazy') mode available that trains the GCN branch directly on the input matrix.

## Related tools

- **GILoop** (Dual-branch (GCN graph + U-Net image) CTCF-mediated loop caller whose branches are pre-trained at 10% and 100% GM12878 matrix densities and jointly fine-tuned; its repository supplies the matrix downsampling function used to build the mixed-density training set) — github.com/fzbio/GILoop

## Examples

```
conda create -n GIL python=3.8; conda activate GIL; pip install -r requirements.txt; python demo.py
```

## Evaluation signals

- Mean PR-AUC of the mixed-density fine-tuned model exceeds the single-depth 90% baseline (~0.3790 vs ~0.3718, matching Table S11)
- Branch-density assignment is correct: GCN pre-trains on 10% downsampled matrices, U-Net on 100%-depth matrices — not swapped
- Test chromosomes are strictly held out from both pre-training and joint fine-tuning (no chromosome overlap)
- pr_auc_comparison.csv contains both compared mean PR-AUC values and identifies the mixed-depth model as the winner
- Predicted loops are CTCF-mediated and intra-chromosomal, consistent with GILoop's target class

## Limitations

- Only bulk Hi-C is supported; GILoop is not capable of dealing with extremely sparse single-cell Hi-C (scHi-C) data.
- Downsampling-heavy pre-processing (matrix transformations and hard disk reading/writing) requires a relatively large amount of time before training.
- Weak usability if the targeted cell line's sequencing depth differs from any of the pre-trained models provided in the repository.
- The 'models' and 'data' assets must be downloaded separately and replace the ones in the local directory before running.

## Evidence

- [abstract] We also uncover distinct preferences for matrix density by graph-based and image-based models: "We also uncover distinct preferences for matrix density by graph-based and image-based models"
- [abstract] Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops.: "Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops."
- [discussion] In the GILoop repository, we provide a function for matrix downsampling at the training stage: "In the GILoop repository, we provide a function for matrix downsampling at the training stage"
- [discussion] which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts): "which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts)"
- [readme] This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.), and GILoop sampling and training.: "This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.), and GILoop sampling and training."
