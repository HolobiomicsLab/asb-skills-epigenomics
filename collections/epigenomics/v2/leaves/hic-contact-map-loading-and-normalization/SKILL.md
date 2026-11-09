---
name: hic-contact-map-loading-and-normalization
description: Use when raw Hi-C contact maps (.cool files or Juicer dump text) must
  be loaded, KR-normalized, converted to O/E matrices, and sequencedepth-aligned before
  patch sampling or loop calling.
when_to_use_negative:
- Input is already a KR-normalized O/E matrix from a Juicer dump — use the processed-data
  path (demo_from_processed.py) and skip preprocessing entirely
- Data is extremely sparse single-cell Hi-C (scHi-C) — the paper states the tool 'is
  still not capable enough to deal with' it
- Only the downstream model branch is needed and the input matrix will be consumed
  directly by the GCN in 'lazy' non-downsampling mode
tools:
- name: GILoop
  role: Provides the full preprocessing pipeline (sequencing depth alignment, KR normalization,
    expected vector calculation) plus optional matrix downsampling before patch sampling
    and training
  repo: github.com/fzbio/GILoop
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:bc182ca173210a4b57c8b6abb2501fedcfb674933dd02571dc2b2ac710c6972c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# hic-contact-map-loading-and-normalization

## Summary

Loads Hi-C intra-chromosomal (cis) contact maps from .cool files or Juicer dump text, aligns sequencing depth, and produces KR-normalized O/E matrices that GILoop then tiles into paired image and graph training patches.

## When to use

Use when raw Hi-C contact maps (.cool files or Juicer dump text) must be loaded, KR-normalized, converted to O/E matrices, and sequencedepth-aligned before patch sampling or loop calling.

## When NOT to use

- Input is already a KR-normalized O/E matrix from a Juicer dump — use the processed-data path (demo_from_processed.py) and skip preprocessing entirely
- Data is extremely sparse single-cell Hi-C (scHi-C) — the paper states the tool 'is still not capable enough to deal with' it
- Only the downstream model branch is needed and the input matrix will be consumed directly by the GCN in 'lazy' non-downsampling mode

## Inputs

- .cool files (cooler format) of Hi-C contact maps
- Juicer dump text files (KR-normalized O/E matrices)
- KR-normalized intra-chromosomal Hi-C contact maps

## Outputs

- Sequencing-depth-aligned, KR-normalized O/E cis-contact matrices
- Downsampled (or full) contact matrices at ~260 million cis-contacts density
- Patched image and graph training samples derived from the normalized matrix

## How to apply

Load the intra-chromosomal contact map via GILoop's preprocessing pipeline, which handles sequencing depth alignment, KR normalization, and expected vector calculation; the newer GILoop supports .cool input directly, while Juicer dump text must already be a KR-normalized O/E matrix with source and target of similar read counts. Optionally downsample the contact map to the experimentally determined optimal density of 260 million cis-contacts, or use the non-downsampling 'lazy' mode that trains the GCN branch directly on the input matrix. Once the O/E matrix is obtained, it is tiled into patches to generate the paired image-view and graph-view training inputs for the dual-branch model.

## Related tools

- **GILoop** (Provides the full preprocessing pipeline (sequencing depth alignment, KR normalization, expected vector calculation) plus optional matrix downsampling before patch sampling and training) — github.com/fzbio/GILoop

## Examples

```
python demo.py
```

## Evaluation signals

- Matrices are KR-normalized O/E values — verify no raw counts remain in the loaded map
- For Juicer dump inputs, source and target should have a similar number of reads before patching
- When downsampling mode is used, the resulting contact map should be near the optimal density of 260 million cis-contacts
- Preprocessing must complete before patch sampling — the README notes the overhead 'could be heavy and could lead to a large running time'
- Patched outputs should form paired image and graph samples derived from the same symmetric O/E matrix

## Limitations

- GILoop's preprocessing is slow: 'This pre-processing time mainly comes from the' downsampling procedure (matrix transformations and hard disk reading/writing)
- The tool only focuses on bulk Hi-C and cannot handle extremely sparse single-cell Hi-C data
- Juicer dump text inputs receive no preprocessing — the user must supply KR-normalized O/E matrices with similar read counts in source and target
- Downsampling to the optimal density is only required when sequencing depth differs substantially from the pre-trained models

## Evidence

- [readme] readme: "please make sure the text files are KR-normalized O/E matrices, and the source and the target are of similar number of reads."
- [discussion] discussion: "which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts)"
- [discussion] discussion2: "GILoop requires a relatively large amount of time to pre-process the data before sending the data into the computational pipeline to train the model."
- [abstract] abstract: "Graph and image are two common representations of Hi-C cis-contact maps."
