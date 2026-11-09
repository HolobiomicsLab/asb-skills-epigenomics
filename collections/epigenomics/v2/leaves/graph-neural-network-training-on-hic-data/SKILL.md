---
name: graph-neural-network-training-on-hic-data
description: Use when calling CTCF-mediated chromatin loops from Hi-C contact maps,
  especially when the library is low-depth or downsampled (e.g. 1.5%, ~39 million
  cis-reads) and single-view loop callers (image- or graph-only) lose PR-AUC.
when_to_use_negative:
- Input is single-cell Hi-C (scHi-C) data — GILoop only focuses on bulk Hi-C and is
  not capable of dealing with extremely sparse single-cell data
- The input matrices are not KR-normalized O/E matrices of similar depth — the Juicer-dump
  path (`demo_from_processed.py`) applies no preprocessing
- The analysis goal is a pure image-based or pure graph-based loop caller — this skill's
  rationale is the complementarity of fusing both views
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: GILoop
  role: Dual-branch neural network (GCN graph branch + U-Net image branch) trained
    on Hi-C cis-contact maps to call CTCF-mediated loops; includes a matrix downsampling
    function for training-stage depth alignment
  repo: github.com/fzbio/GILoop
- name: SIP
  role: Existing image-based loop caller cited as prior art that regards Hi-C matrices
    as images; contextual comparison baseline
  repo: https://github.com/PouletAxel/SIP
- name: Peakachu
  role: Existing machine-learning loop caller that GILoop significantly outperforms;
    comparison baseline for CTCF-associated loop detection
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:df8eb9116dd176c048613e36d7571a6670cbaa259d0192685f2f9aad5836ebe9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  - Peakachu
  - SIP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# graph-neural-network-training-on-hic-data

## Summary

Training GILoop, a dual-branch neural network (GCN graph view + U-Net image view), on Hi-C cis-contact maps to identify genome-wide CTCF-mediated loops, exploiting the graph-image duality of Hi-C data for robustness at low sequencing depths.

## When to use

Use when calling CTCF-mediated chromatin loops from Hi-C contact maps, especially when the library is low-depth or downsampled (e.g. 1.5%, ~39 million cis-reads) and single-view loop callers (image- or graph-only) lose PR-AUC.

## When NOT to use

- Input is single-cell Hi-C (scHi-C) data — GILoop only focuses on bulk Hi-C and is not capable of dealing with extremely sparse single-cell data
- The input matrices are not KR-normalized O/E matrices of similar depth — the Juicer-dump path (`demo_from_processed.py`) applies no preprocessing
- The analysis goal is a pure image-based or pure graph-based loop caller — this skill's rationale is the complementarity of fusing both views

## Inputs

- Hi-C cis-contact matrix as a `.cool` (cooler) file
- Juicer dump text files of KR-normalized O/E matrices with similar read counts in source and target
- Downsampled Hi-C contact maps (e.g. 1.5% sequencing depth, 39 million cis-reads)
- CTCF-mediated loop candidates and non-loop controls converted to paired graph and image training instances

## Outputs

- Trained fused dual-branch GILoop model (GCN + U-Net)
- Trained single-view baselines (U-Net image model, GCN graph model)
- Mean PR-AUC per model on the test set (e.g. written to pr_auc_downsample_1.5pct.csv)
- Predicted CTCF-mediated loops genome-wide

## How to apply

Pre-process KR-normalized O/E cis-contact matrices (cooler `.cool` or Juicer dump text format) via sequencing depth alignment, normalization, and expected vector calculation, then convert them into paired graph and image training instances centered on CTCF-mediated loop candidates and non-loop controls. Train the fused dual-branch GILoop model, in which the GCN branch learns the graph view and the U-Net branch learns the image view; the GILoop repository provides a matrix downsampling function to automatically downsample contact maps to the experimentally obtained optimal density of 260 million cis-contacts at the training stage. For robustness studies, downsample to target depths (e.g. 1.5% downsample, 39 million cis-reads) and compare the fused model's mean PR-AUC against single-view baselines (e.g. fused ~0.208 vs U-Net ~0.135 vs GCN ~0.101 at 1.5%), which demonstrates that fusion absorbs image-view performance loss. Alternatively, use the non-downsampling 'lazy' mode to train the GCN branch directly on the input matrix when preprocessing overhead should be avoided. Judge success by mean precision-recall AUC on the held-out test set and by consistency with reference values from Figure 4 and Tables S4-S10.

## Related tools

- **GILoop** (Dual-branch neural network (GCN graph branch + U-Net image branch) trained on Hi-C cis-contact maps to call CTCF-mediated loops; includes a matrix downsampling function for training-stage depth alignment) — github.com/fzbio/GILoop
- **SIP** (Existing image-based loop caller cited as prior art that regards Hi-C matrices as images; contextual comparison baseline) — https://github.com/PouletAxel/SIP
- **Peakachu** (Existing machine-learning loop caller that GILoop significantly outperforms; comparison baseline for CTCF-associated loop detection)

## Examples

```
conda create -n GIL python=3.8 && conda activate GIL && pip install -r requirements.txt && python demo.py
```

## Evaluation signals

- Mean PR-AUC of the fused model on the downsampled test set matches reference values (fused ~0.208, U-Net ~0.135, GCN ~0.101 at the 1.5% downsample with 39 million cis-reads)
- Fused-model PR-AUC does not degrade below the better single-view branch at low depth, demonstrating that fusion absorbs the image-view performance loss
- Predicted loops are CTCF-associated, consistent with GILoop's reported outperformance of Peakachu in finding CTCF-associated loops
- Training data are aligned to the experimentally obtained optimal density of 260 million cis-contacts when using the default downsampling mode
- Input files pass format checks: KR-normalized O/E matrices for the Juicer-dump path, or valid `.cool` files for the cooler path

## Limitations

- GILoop only focuses on bulk Hi-C and is not capable of dealing with extremely sparse single-cell Hi-C (scHi-C) data
- GILoop requires a relatively large amount of pre-processing time before training, mainly from the downsampling procedure (matrix transformations and hard disk reading/writing)
- Weak usability if the targeted cell line has a sequencing depth different from any of the pre-trained models provided in the repository

## Evidence

- [abstract] dual-branch architecture claim: "Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops."
- [discussion] downsampling function availability: "In the GILoop repository, we provide a function for matrix downsampling at the training stage"
- [discussion] lazy training mode: "users can also choose to use the non-downsampling mode (the “lazy” mode) to train the model, which directly trains the GCN branch on the input matrix"
- [readme] preprocessing pipeline contents: "This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.), and GILoop sampling and training."
- [readme] input format requirement for Juicer dump path: "please make sure the text files are KR-normalized O/E matrices, and the source and the target are of similar number of reads."
