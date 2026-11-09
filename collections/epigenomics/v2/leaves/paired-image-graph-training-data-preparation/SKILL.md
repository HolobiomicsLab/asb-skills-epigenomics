---
name: paired-image-graph-training-data-preparation
description: 'Use when preparing training data for GILoop: the input is a KR-normalized
  intra-chromosomal Hi-C contact map (or Juicer-dump O/E matrix) and paired image
  and graph samples are needed for dual-branch loop-calling training.'
when_to_use_negative:
- The target is extremely sparse single-cell Hi-C (scHi-C) data — GILoop is stated
  to be 'not capable enough' for this
- The pipeline only needs loop prediction with a pre-trained model and no new training
  set is being built
- The input is a raw (non-normalized) contact map for the processed-input path — KR-normalized
  O/E matrices are required there
tools:
- name: GILoop
  role: Dual-branch network whose patch sampling and graph-construction preprocessing
    this skill implements; repo provides demo.py covering preprocessing, sampling
    and training
  repo: github.com/fzbio/GILoop
- name: FAN-C
  role: External preprocessing dependency whose runtime dominates the preprocessing
    overhead (expected vector computation); a faster version was announced but not
    yet available
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:8971fff9d3a922c2cadbf915b8f44260175c22619e9c244f0397c3ddbd21e550
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
yes_rate: 0.7692307692307693
n_claims: 13
---

# paired-image-graph-training-data-preparation

## Summary

Transform a KR-normalized Hi-C cis-contact map into distance-normalized O/E matrices, tile them into 64x64 kb patches, and serialize each patch as both an image-view matrix and a graph-view symmetric adjacency matrix so the dual-branch GILoop network can train on paired representations.

## When to use

Use when preparing training data for GILoop: the input is a KR-normalized intra-chromosomal Hi-C contact map (or Juicer-dump O/E matrix) and paired image and graph samples are needed for dual-branch loop-calling training.

## When NOT to use

- The target is extremely sparse single-cell Hi-C (scHi-C) data — GILoop is stated to be 'not capable enough' for this
- The pipeline only needs loop prediction with a pre-trained model and no new training set is being built
- The input is a raw (non-normalized) contact map for the processed-input path — KR-normalized O/E matrices are required there

## Inputs

- KR-normalized intra-chromosomal Hi-C cis-contact matrix (numpy/scipy dense or sparse, from .cool file or Juicer dump)
- KR-normalized O/E matrix text file in Juicer dump format (for the processed-input path)
- Expected contact frequency vector per genomic distance (for O/E normalization)

## Outputs

- image_patches.npz — numpy archive of 64x64 kb image-view patch matrices
- graph_samples.npz — numpy archive of graph-view symmetric adjacency matrices, one per image patch
- Paired image/graph training samples in equal numbers

## How to apply

Load the KR-normalized intra-chromosomal Hi-C contact map into a numpy/scipy matrix representation, then compute the distance-normalized O/E matrix by dividing each observed contact count by the expected contact frequency at the corresponding genomic distance (Equation 1). If starting from Juicer dump text files, the matrices must already be KR-normalized O/E matrices and pre-processing (sequencing depth alignment, normalization, expected vector calculation) can be skipped. Tile the O/E matrix into non-overlapping 64x64 kb patches following GILoop's patch-sampling strategy, treating symmetric diagonal and asymmetric off-diagonal patches as image samples. For every patch, build the graph-view adjacency structure via the Equation 2 construction rule, transforming both patch types into symmetric adjacency matrices, then write paired outputs as numpy archives (e.g. image_patches.npz and graph_samples.npz). Optionally downsample the contact map to the optimal density of 260 million cis-contacts, or use the non-downsampling 'lazy' mode that trains the GCN branch directly on the input matrix.

## Related tools

- **GILoop** (Dual-branch network whose patch sampling and graph-construction preprocessing this skill implements; repo provides demo.py covering preprocessing, sampling and training) — github.com/fzbio/GILoop
- **FAN-C** (External preprocessing dependency whose runtime dominates the preprocessing overhead (expected vector computation); a faster version was announced but not yet available)

## Examples

```
python demo.py  # full pipeline: sequencing depth alignment, normalization, expected vector calculation, GILoop sampling and training
```

## Evaluation signals

- Both output files (image_patches.npz and graph_samples.npz) exist, load correctly as NumPy archives, and contain an equal number of paired image and graph samples
- Adjacency matrices are symmetric, confirming both diagonal and off-diagonal patch types were transformed per Equation 2
- Image patch values are O/E-normalized: each observed contact equals observed/expected at its genomic distance, not the raw contact count
- Patches are non-overlapping 64x64 kb tiles covering the cis-contact map
- If downsampling is used, the map is aligned to ~260 million cis-contacts (or the 'lazy' mode flag was set)

## Limitations

- GILoop requires a relatively large amount of pre-processing time before training, mainly from the downsampling procedure (matrix transformations and hard disk reading/writing)
- Only bulk Hi-C is supported; extremely sparse single-cell Hi-C data cannot be handled
- The processed-input path assumes the number of reads of source and target are similar — no re-normalization is applied
- Weak usability if the targeted cell line's sequencing depth differs from any provided pre-trained model, requiring re-training with this skill

## Evidence

- [abstract] dual-branch neural network learns from both Hi-C representations: "Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops."
- [readme] README instructs KR-normalized O/E matrices as input: "In this case, please make sure the text files are KR-normalized O/E matrices, and the source and the target are of similar number of reads."
- [discussion] preprocessing cost dominated by downsampling transformations: [evidence span withheld — anchor status: not_found]
- [discussion] optimal downsampling density of 260 million cis-contacts: "which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts)"
- [intro] graph-image duality motivates paired representations: "we examine the graph-image duality (i.e., the property that Hi-C data can be represented as either a graph or an image) of Hi-C cis-contact map"
