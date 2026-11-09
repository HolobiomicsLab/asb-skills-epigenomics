---
name: graph-adjacency-construction-from-contact-matrix
description: Use when a Hi-C intra-chromosomal contact map (KR-normalized, O/E transformed)
  must be converted into graph-view training samples — e.g., paired adjacency matrices
  and image patches for training a dual-branch model like GILoop.
when_to_use_negative:
- The data are extremely sparse single-cell Hi-C (scHi-C) — GILoop is not capable
  enough to deal with scHi-C despite bulk-Hi-C robustness
- Input is raw, unnormalized contact data for the Juicer-dump path — the demo_from_processed.py
  script applies no preprocessing and requires KR-normalized O/E matrices
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: GILoop
  role: Provides the patch-sampling and O/E-to-adjacency preprocessing pipeline that
    builds the paired graph-view and image-view training inputs
  repo: github.com/fzbio/GILoop
- name: FAN-C
  role: Mentioned as the source of preprocessing (matrix transformations and hard
    disk reading/writing) whose runtime cost a future faster version will address
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:403672666e44bfd4c1d2537841878de4186df0ce2ec59473ae0c72c6a8251dfa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FAN-C
  - GILoop
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# graph-adjacency-construction-from-contact-matrix

## Summary

Converts KR-normalized Hi-C cis-contact matrices (O/E transformed) into paired graph-view adjacency structures and image-view patches, as used by GILoop's dual-branch GCN/U-Net loop caller. Exploits the graph-image duality of Hi-C data so that a GCN branch can learn from contact graphs alongside the image branch.

## When to use

Use when a Hi-C intra-chromosomal contact map (KR-normalized, O/E transformed) must be converted into graph-view training samples — e.g., paired adjacency matrices and image patches for training a dual-branch model like GILoop.

## When NOT to use

- The data are extremely sparse single-cell Hi-C (scHi-C) — GILoop is not capable enough to deal with scHi-C despite bulk-Hi-C robustness
- Input is raw, unnormalized contact data for the Juicer-dump path — the demo_from_processed.py script applies no preprocessing and requires KR-normalized O/E matrices

## Inputs

- KR-normalized intra-chromosomal Hi-C contact matrix (dense numpy or sparse scipy representation)
- Juicer dump text files of KR-normalized O/E matrices (no further preprocessing applied)
- .cool files (supported by GILoop with sequencing-depth alignment, normalization, and expected-vector preprocessing)

## Outputs

- Paired image-view training samples: 64x64 kb O/E matrix patches serialized as numpy arrays (e.g., image_patches.npz)
- Paired graph-view training samples: symmetric adjacency structures built from patch contact values (e.g., graph_samples.npz)

## How to apply

Load the KR-normalized intra-chromosomal Hi-C contact map with numpy/scipy and compute the distance-normalized O/E matrix (Equation 1), dividing each observed contact count by the expected contact frequency at the corresponding genomic distance. Tile the O/E matrix into non-overlapping 64x64 kb patches following GILoop's patch sampling strategy, in which both symmetric diagonal and asymmetric off-diagonal patch types are transformed to generate symmetric adjacency matrices. For each patch, build the graph-view adjacency structure using the Equation 2 construction rule applied to the patch's contact values, and serialize each patch's 2D matrix as an image-view sample, writing paired graph and image samples to NPZ archives. If input is a Juicer dump text file, it must already be a KR-normalized O/E matrix with source and target of similar read counts, since the Juicer-input path performs no preprocessing. GILoop also supports a 'lazy' training mode in which the GCN branch is trained directly on the input matrix without downsampling.

## Related tools

- **GILoop** (Provides the patch-sampling and O/E-to-adjacency preprocessing pipeline that builds the paired graph-view and image-view training inputs) — github.com/fzbio/GILoop
- **FAN-C** (Mentioned as the source of preprocessing (matrix transformations and hard disk reading/writing) whose runtime cost a future faster version will address)

## Examples

```
python demo_from_processed.py  # Juicer dump input: KR-normalized O/E matrices, GILoop patching + training without preprocessing
```

## Evaluation signals

- Both output archives (image_patches.npz and graph_samples.npz) exist, load as NumPy archives, and contain an equal number of paired image and graph samples
- Every constructed adjacency matrix is symmetric, covering both symmetric diagonal and asymmetric off-diagonal patch types after transformation
- Patches are non-overlapping 64x64 kb tiles of the distance-normalized O/E matrix, with each contact value divided by the expected frequency at its genomic distance
- For Juicer-dump input, the source and target matrices have similar numbers of reads; otherwise the read-count mismatch invalidates the paired training set

## Limitations

- GILoop requires relatively large preprocessing time before training, mainly from the downsampling procedure (matrix transformations and hard disk reading/writing)
- Weak usability if the targeted cell line's sequencing depth differs from any pre-trained model provided in the repository
- Only applicable to bulk Hi-C; not capable of dealing with extremely sparse single-cell Hi-C data

## Evidence

- [abstract] dual-branch design rationale: "Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops."
- [abstract] complementary views: "we explore the combined strength of integrating the two view representations of Hi-C data and corroborate the complementary relationship between the views"
- [discussion] lazy mode trains GCN directly on the matrix: "users can also choose to use the non-downsampling mode (the “lazy” mode) to train the model, which directly trains the GCN branch on the input matrix"
- [readme] Juicer input format requirement: "In this case, please make sure the text files are KR-normalized O/E matrices, and the source and the target are of similar number of reads."
- [results] patch sampling produces both views: [evidence span withheld — anchor status: not_found]
