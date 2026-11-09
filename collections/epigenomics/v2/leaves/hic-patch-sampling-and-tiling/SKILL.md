---
name: hic-patch-sampling-and-tiling
description: Use when preparing Hi-C cis-contact matrices as training inputs for dual-branch
  loop callers like GILoop, i.e. when a contact map must be converted into paired
  image-view patches and graph-view adjacency samples.
when_to_use_negative:
- Input is single-cell Hi-C (scHi-C) data — GILoop is stated to be 'still not capable
  enough' to deal with extremely sparse scHi-C
- Input matrices are raw or non-KR-normalized and only the demo_from_processed.py
  path is used, since that script applies no preprocessing and expects KR-normalized
  O/E matrices
- The goal is loop calling with a single-view tool (e.g. SIP, Chromosight, Peakachu)
  that does not require paired image/graph sample construction
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3922
tools:
- name: GILoop
  role: Provides the patch sampling pipeline (sequencing depth alignment, normalization,
    expected vector calculation, patching, training) that converts contact maps into
    dual-branch training inputs
  repo: github.com/fzbio/GILoop
- name: SIP
  role: Contrasting single-view (image-only) loop caller referenced for how matrices
    can be regarded as images
  repo: PouletAxel/SIP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:1ffc90f9859d60f342695ba03f9e4f7fa489238dca8c3b1301dd532330a319fb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  - SIP
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3922
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# hic-patch-sampling-and-tiling

## Summary

Transforms KR-normalized intra-chromosomal Hi-C cis-contact maps into paired image patches and graph adjacency samples for GILoop's dual-branch (GCN + image) model, using O/E distance normalization and diagonal/off-diagonal patch sampling.

## When to use

Use when preparing Hi-C cis-contact matrices as training inputs for dual-branch loop callers like GILoop, i.e. when a contact map must be converted into paired image-view patches and graph-view adjacency samples.

## When NOT to use

- Input is single-cell Hi-C (scHi-C) data — GILoop is stated to be 'still not capable enough' to deal with extremely sparse scHi-C
- Input matrices are raw or non-KR-normalized and only the demo_from_processed.py path is used, since that script applies no preprocessing and expects KR-normalized O/E matrices
- The goal is loop calling with a single-view tool (e.g. SIP, Chromosight, Peakachu) that does not require paired image/graph sample construction

## Inputs

- KR-normalized intra-chromosomal Hi-C contact map (dense or sparse numpy/scipy matrix)
- Juicer-dump text file of KR-normalized O/E matrices (for demo_from_processed.py path)
- .cool / .mcool cooler file (supported GILoop input format)

## Outputs

- Paired image-view training samples (2D numpy array patches, e.g. image_patches.npz)
- Paired graph-view adjacency samples built from the same patches (e.g. graph_samples.npz)

## How to apply

Load the KR-normalized intra-chromosomal Hi-C contact map with numpy/scipy, then compute the distance-normalized O/E matrix by dividing each observed contact count by the expected contact frequency at the corresponding genomic distance (Equation 1). Tile the O/E matrix into non-overlapping 64x64 kb patches following GILoop's patch sampling strategy, sampling both symmetric diagonal and asymmetric off-diagonal patch types. Serialize each patch's 2D matrix as an image-view training sample and, for the same patch, build the graph-view adjacency structure using the Equation 2 construction rule applied to the patch's contact values, since both patch types are transformed to generate the symmetric adjacency matrices for the graph branch. Write the paired samples to training files (e.g. image_patches.npz and graph_samples.npz). Optionally, downsample the contact map to the experimentally determined optimal density of 260 million cis-contacts before sampling, or use the non-downsampling 'lazy' mode which trains the GCN branch directly on the input matrix.

## Related tools

- **GILoop** (Provides the patch sampling pipeline (sequencing depth alignment, normalization, expected vector calculation, patching, training) that converts contact maps into dual-branch training inputs) — github.com/fzbio/GILoop
- **SIP** (Contrasting single-view (image-only) loop caller referenced for how matrices can be regarded as images) — PouletAxel/SIP

## Examples

```
python demo.py
```

## Evaluation signals

- Both output files (image_patches.npz and graph_samples.npz) exist and load correctly as NumPy archives
- The image and graph sample files contain an equal number of paired samples
- Graph adjacency matrices derived from both symmetric diagonal and asymmetric off-diagonal patch types are symmetric
- Patch values correspond to O/E-normalized contacts (observed divided by expected frequency at the matching genomic distance)
- For downsampled training, the contact map density matches the optimal 260 million cis-contacts

## Limitations

- Pre-processing is heavy and slow: downsampling (matrix transformations and hard disk reading/writing) dominates preprocessing time before training
- Non-downsampling ('lazy') mode trains the GCN branch directly on the input matrix, trading normalization for speed
- The tool only handles bulk Hi-C and is not capable of dealing with extremely sparse single-cell Hi-C data
- Usability is weak if the target cell line's sequencing depth differs from any pre-trained model in the repository

## Evidence

- [methods] Sampled patches together make up the image set, and both patch types are transformed to generate the symmetric adjacency matrices for the graph input: [evidence span withheld — anchor status: not_found]
- [abstract] dual-branch neural network learns from both views: "Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops."
- [discussion] downsamples to optimal density of 260 million cis-contacts: "which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts)"
- [readme] demo.py includes preprocessing; from_processed expects KR-normalized O/E matrices: "This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.), and GILoop sampling and training."
- [readme] please make sure the text files are KR-normalized O/E matrices: "In this case, please make sure the text files are KR-normalized O/E matrices, and the source and the target are of similar number of reads."
