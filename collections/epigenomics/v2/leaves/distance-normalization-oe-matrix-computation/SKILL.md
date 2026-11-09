---
name: distance-normalization-oe-matrix-computation
description: Use when a KR-normalized intra-chromosomal Hi-C contact map must be converted
  to an O/E matrix before tiling into 64x64 kb patches for dual-branch (image + graph)
  loop-calling model training or prediction.
when_to_use_negative:
- Input is already a KR-normalized O/E matrix from Juicer dump output — GILoop's demo_from_processed.py
  path applies patching/training only, so re-normalization would be redundant
- Data are extremely sparse single-cell Hi-C (scHi-C) — the paper states GILoop is
  'not capable enough' to deal with scHi-C despite robustness on low-depth bulk Hi-C
- Only raw (unnormalized) contact counts are available and no expected-vector calculation
  or normalization step (e.g., KR, sequencing-depth alignment) will be run — O/E computation
  on unnormalized counts is out of scope here
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: GILoop
  role: Executes the preprocessing (normalization, expected vector calculation, O/E
    transformation) and subsequent patch sampling of the O/E matrix into paired image
    and graph training inputs
  repo: https://github.com/fzbio/GILoop
- name: SIP
  role: Comparative image-view loop caller context; treats Hi-C matrices as images,
    contrasted with GILoop's dual graph-image view
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:add38f99928f5fa956fdbee236f1920a9f945e345a5f0a51b9c4b8d62ed28bbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fzbio/GILoop
  tools:
  - GILoop
  - SIP
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# distance-normalization-oe-matrix-computation

## Summary

Converts KR-normalized intra-chromosomal Hi-C contact matrices into distance-normalized observed/expected (O/E) matrices by dividing each observed contact count by the expected contact frequency at the corresponding genomic distance, yielding inputs suitable for paired image and graph sampling in GILoop.

## When to use

Use when a KR-normalized intra-chromosomal Hi-C contact map must be converted to an O/E matrix before tiling into 64x64 kb patches for dual-branch (image + graph) loop-calling model training or prediction.

## When NOT to use

- Input is already a KR-normalized O/E matrix from Juicer dump output — GILoop's demo_from_processed.py path applies patching/training only, so re-normalization would be redundant
- Data are extremely sparse single-cell Hi-C (scHi-C) — the paper states GILoop is 'not capable enough' to deal with scHi-C despite robustness on low-depth bulk Hi-C
- Only raw (unnormalized) contact counts are available and no expected-vector calculation or normalization step (e.g., KR, sequencing-depth alignment) will be run — O/E computation on unnormalized counts is out of scope here

## Inputs

- KR-normalized intra-chromosomal Hi-C contact matrix (from a .cool file or Juicer dump text format)
- Expected contact frequency vector by genomic distance (computed during GILoop preprocessing from the .cool input)

## Outputs

- Distance-normalized O/E matrix (numpy/scipy array) ready for 64x64 kb patch sampling
- Paired training samples: image patches (e.g., image_patches.npz) and graph adjacency samples (e.g., graph_samples.npz) derived from the O/E matrix

## How to apply

Load the KR-normalized intra-chromosomal Hi-C contact map into a numpy/scipy dense or sparse matrix representation. Compute the distance-normalized O/E matrix per the paper's Equation 1 by dividing each observed contact count by the expected contact frequency at the corresponding genomic distance, which removes the strong distance-decay signal inherent to cis-contact maps. If starting from Juicer dump text files, the inputs must already be KR-normalized O/E matrices and the source and target must have a similar number of reads; otherwise run the full GILoop preprocessing pipeline (sequencing depth alignment, normalization, expected vector calculation) from a .cool file. The resulting O/E matrix is then tiled into symmetric diagonal and asymmetric off-diagonal 64x64 kb patches, and the sampled patches are transformed to generate symmetric adjacency matrices for the graph branch. Note that the preprocessing overhead (including downsampling toward ~260 million cis-contacts) can be substantial, which is a documented cost of this transformation chain.

## Related tools

- **GILoop** (Executes the preprocessing (normalization, expected vector calculation, O/E transformation) and subsequent patch sampling of the O/E matrix into paired image and graph training inputs) — https://github.com/fzbio/GILoop
- **SIP** (Comparative image-view loop caller context; treats Hi-C matrices as images, contrasted with GILoop's dual graph-image view)

## Examples

```
python demo.py  # full GILoop pipeline: preprocessing (sequencing depth alignment, normalization, expected vector calculation) on .cool input, then O/E sampling and training
```

## Evaluation signals

- The O/E matrix removes the diagonal distance-decay gradient: contact values at equivalent genomic distances should have comparable magnitudes after division by the expected vector
- For Juicer dump inputs, verify files are KR-normalized O/E matrices and that source and target have a similar number of reads before proceeding
- Both output files (image_patches.npz and graph_samples.npz) exist, load correctly as NumPy archives, and contain an equal number of paired image and graph samples
- Sampled patches tile the O/E matrix as non-overlapping 64x64 kb blocks, and both symmetric diagonal and asymmetric off-diagonal patch types transform into symmetric adjacency matrices
- If downsampling mode is used, the contact map is aligned toward the experimentally optimal density of 260 million cis-contacts before sampling

## Limitations

- GILoop preprocessing requires a relatively large amount of time, mainly from downsampling procedures (matrix transformations and hard disk reading/writing)
- Only bulk Hi-C is supported; the tool is not capable enough to deal with extremely sparse single-cell Hi-C data
- Juicer dump text inputs must already be KR-normalized O/E matrices with source and target of similar read numbers — no preprocessing is applied on that path

## Evidence

- [abstract] Graph and image are two common representations of Hi-C cis-contact maps.: "Graph and image are two common representations of Hi-C cis-contact maps."
- [intro] We exemplify it by developing a dual-branch neural network termed GILoop, which learns from both views of the intra-chromosomal interaction matrix: "We exemplify it by developing a dual-branch neural network termed GILoop, which learns from both views of the intra-chromosomal interaction matrix"
- [discussion] GILoop requires a relatively large amount of time to pre-process the data before sending the data into the computational pipeline to train the model.: "GILoop requires a relatively large amount of time to pre-process the data before sending the data into the computational pipeline to train the model."
- [discussion] which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts): "which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts)"
- [readme] please make sure the text files are KR-normalized O/E matrices, and the source and the target are of similar number of reads: "please make sure the text files are KR-normalized O/E matrices, and the source and the target are of similar number of reads."
- [readme] This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.): "This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.), and GILoop sampling and training."
