---
name: ctcf-mediated-loop-calling
description: Use when you need to identify CTCF-mediated chromatin loops from Hi-C
  data — especially at reduced sequencing depth or when transferring a loop-calling
  model trained on one cell line (e.g., GM12878) to another (K562, HeLa-S3) — rather
  than threshold-based callers like SIP or Chromosight.
when_to_use_negative:
- 'The input is single-cell Hi-C (scHi-C) data: GILoop focuses on bulk Hi-C and is
  not capable of dealing with extremely sparse scHi-C.'
- You need a single-view image-based caller on raw hic files without a trained model
  — SIP or Chromosight treat matrices as images and may be simpler, though less robust.
- The targeted cell line's sequencing depth differs from any pre-trained model in
  the repository and you cannot retrain or downsample.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: GILoop
  role: Dual-branch (graph + image) neural network for CTCF-mediated loop calling;
    includes pre-trained checkpoints and a matrix downsampling function for transfer
    across cell lines and sequencing depths
  repo: github.com/fzbio/GILoop
- name: SIP
  role: Alternative image-based loop caller for Hi-C maps, referenced as a comparator;
    not used within the GILoop transfer workflow itself
  repo: PouletAxel/SIP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:97debea2b3f77cbbb452266fdf77eb466ea0b3c445c058b5b123010465c85d7e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  - SIP
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# ctcf-mediated-loop-calling

## Summary

Call genome-wide CTCF-mediated chromatin loops from Hi-C contact maps using GILoop, a dual-branch neural network that learns from both graph (GCN) and image views of the intra-chromosomal interaction matrix, including transfer of pre-trained models across cell lines.

## When to use

Use when you need to identify CTCF-mediated chromatin loops from Hi-C data — especially at reduced sequencing depth or when transferring a loop-calling model trained on one cell line (e.g., GM12878) to another (K562, HeLa-S3) — rather than threshold-based callers like SIP or Chromosight.

## When NOT to use

- The input is single-cell Hi-C (scHi-C) data: GILoop focuses on bulk Hi-C and is not capable of dealing with extremely sparse scHi-C.
- You need a single-view image-based caller on raw hic files without a trained model — SIP or Chromosight treat matrices as images and may be simpler, though less robust.
- The targeted cell line's sequencing depth differs from any pre-trained model in the repository and you cannot retrain or downsample.

## Inputs

- Hi-C contact maps as .cool / .mcool files
- Juicer-dump text files of KR-normalized O/E intra-chromosomal matrices
- Pre-trained GILoop model checkpoints (e.g., 20% and 50% downsampled GM12878 models) for cross-cell-line transfer

## Outputs

- Genome-wide CTCF-mediated loop predictions per cell line
- Precision-recall curves for source and transferred (target-cell-line) settings, e.g. overlaid as figure6_transfer_pr_curves.png

## How to apply

Preprocess each contact map into intra-chromosomal interaction matrices and the graph structures required by GILoop's dual-branch input format (supported inputs are .cool files or Juicer-dump KR-normalized O/E matrices with a similar number of reads between source and target). Run the full pipeline via demo.py, which handles sequencing-depth alignment, normalization, and expected vector calculation before GILoop patching, training, and prediction; for pre-processed Juicer text files use demo_from_processed.py to skip preprocessing. For low-depth libraries, use the repository's built-in downsampling function, which automatically resamples the contact map to the experimentally obtained optimal density of 260 million cis-contacts, or select the non-downsampling 'lazy' mode that trains the GCN branch directly on the input matrix. For cross-cell-line transfer, apply the appropriate pre-trained checkpoint (e.g., the 20%-downsampled GM12878 model to K562 and the 50% model to HeLa-S3) and compute precision-recall curves of target-cell-line predictions against the source-cell-line baseline; acceptance is near-identical target versus source PR curves, indicating minimal performance decay.

## Related tools

- **GILoop** (Dual-branch (graph + image) neural network for CTCF-mediated loop calling; includes pre-trained checkpoints and a matrix downsampling function for transfer across cell lines and sequencing depths) — github.com/fzbio/GILoop
- **SIP** (Alternative image-based loop caller for Hi-C maps, referenced as a comparator; not used within the GILoop transfer workflow itself) — PouletAxel/SIP

## Examples

```
python demo.py  # full pipeline on .cool input: depth alignment, normalization, expected vector calc, then GILoop patching and training
```

## Evaluation signals

- Target-cell-line PR curves (e.g., K562 with the 20%-GM12878 model, HeLa-S3 with the 50% model) lie near-identically to the source-cell-line GM12878 baseline curves — minimal performance decay.
- GILoop predictions outperform baseline loop callers such as Peakachu on CTCF-associated loop detection.
- Downsampled (low-depth) runs remain robust: performance should not collapse when the contact map is resampled to ~260 million cis-contacts.
- Input validation for demo_from_processed.py: text files must be KR-normalized O/E matrices, and source and target must have a similar number of reads.
- Preprocessing invariants for the .cool path: sequencing depth alignment, normalization, and expected vector calculation must complete before GILoop sampling and training.

## Limitations

- GILoop only focuses on bulk Hi-C and is not capable of dealing with extremely sparse single-cell Hi-C (scHi-C) data.
- GILoop requires a relatively large amount of pre-processing time before training, mainly from the downsampling procedure (matrix transformations and hard disk reading/writing).
- Weak usability if the targeted cell line has a sequencing depth different from any of the pre-trained models provided in the repository.
- A faster pre-processing implementation is deferred to a future faster version of FAN-C (not yet available).

## Evidence

- [abstract] Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops.: "Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops."
- [intro] A trained GILoop model is able to predict CTCF-mediated loops in another cell line with minimal performance decay: "the GILoop framework proposed in this work is transferable across cell lines. A trained GILoop model is able to predict CTCF-mediated loops in another cell line with minimal performance decay"
- [discussion] which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts): "which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts)"
- [readme] please make sure the text files are KR-normalized O/E matrices, and the source and the target are of similar number of reads: "In this case, please make sure the text files are KR-normalized O/E matrices, and the source and the target are of similar number of reads."
- [readme] This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.), and GILoop sampling and training.: "This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.), and GILoop sampling and training."
