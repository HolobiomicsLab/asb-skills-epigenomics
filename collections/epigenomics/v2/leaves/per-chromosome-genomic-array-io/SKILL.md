---
name: per-chromosome-genomic-array-io
description: Use when a pipeline needs to persist or reload per-chromosome, binned
  genomic annotation arrays (e.g. deepC skeleton tags in 1–10 range) from Hi-C contact
  maps at 5 kb resolution.
when_to_use_negative:
- Input is raw, unnormalized Hi-C contact matrices that have not yet been distance-stratified
  and percentile-normalized into skeleton tags
- The analysis requires base-pair-resolution or continuous-valued interaction scores
  rather than 5 kb-binned categorical skeleton annotations
- Predicting or training with deepC models directly from DNA sequence, which takes
  sequence windows rather than skeleton tag arrays as input
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0102
tools:
- name: deepC
  role: Source framework whose skeleton preprocessing produces the per-chromosome
    tag arrays; used from Python scripts via the deepCregr module
  repo: github.com/rschwess/deepC
- name: pysam
  role: Python library required by the deepC environment for handling genomic data
    files alongside the skeleton array I/O
  repo: https://github.com/pysam-developers/pysam
- name: pybedtools
  role: Python library installed with pysam to support genomic interval handling around
    the deepC preprocessing workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:e18fbb70db515d893f8a4c721a2a7de4fbd6127f7dd3f156972efb70e219f91e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# per-chromosome-genomic-array-io

## Summary

Handles loading and writing of per-chromosome genomic signal arrays — specifically deepC's percentile-normalized Hi-C 'skeleton' tag arrays — as .npy/.h5 files, converting continuous KR-normalized contact frequencies into discrete 1–10 percentile class annotations stored per chromosome.

## When to use

Use when a pipeline needs to persist or reload per-chromosome, binned genomic annotation arrays (e.g. deepC skeleton tags in 1–10 range) from Hi-C contact maps at 5 kb resolution.

## When NOT to use

- Input is raw, unnormalized Hi-C contact matrices that have not yet been distance-stratified and percentile-normalized into skeleton tags
- The analysis requires base-pair-resolution or continuous-valued interaction scores rather than 5 kb-binned categorical skeleton annotations
- Predicting or training with deepC models directly from DNA sequence, which takes sequence windows rather than skeleton tag arrays as input

## Inputs

- KR-normalized Hi-C contact maps from Rao et al. (GEO: GSE63525) at 5 kb bin resolution
- Per-chromosome dense contact frequency matrices
- Existing skeleton tag arrays in .npy or .h5 format (for read-back)

## Outputs

- Per-chromosome skeleton arrays of discrete percentile tags (integers 1–10) written as .npy/.h5 files
- Validated parsed arrays with all tags in range 1–10 and all 10 percentile classes represented

## How to apply

Load the KR-normalized Rao et al. Hi-C contact maps (GEO accession GSE63525) at 5 kb bin resolution as dense per-chromosome contact matrices, stratify contact frequencies by genomic interaction distance, and assign each contact a discrete percentile tag from 1 to 10 using the uneven pyramid percentile scheme (2 bins of 20%, 4 bins of 10%, 4 bins of 5%). Serialize each chromosome's tag array to disk as .npy or .h5. When reading arrays back for deepC training or prediction, parse with numpy (v1.16.4 or above, per the deepC requirements) and verify tags stay in the 1–10 range before feeding downstream models, which are used from Python via `import deepCregr`.

## Related tools

- **deepC** (Source framework whose skeleton preprocessing produces the per-chromosome tag arrays; used from Python scripts via the deepCregr module) — github.com/rschwess/deepC
- **pysam** (Python library required by the deepC environment for handling genomic data files alongside the skeleton array I/O) — https://github.com/pysam-developers/pysam
- **pybedtools** (Python library installed with pysam to support genomic interval handling around the deepC preprocessing workflow)

## Examples

```
import deepCregr; import numpy as np; skeleton = np.load('chr1_skeleton.npy')
```

## Evaluation signals

- Every skeleton file exists and parses successfully in .npy/.h5 format
- All stored percentile tags fall in the range 1–10
- The full 10-class percentile binning (2×20%, 4×10%, 4×5%) is represented in the tag distributions
- Arrays are organized per chromosome and are consistent with 5 kb bin resolution of the source contact maps

## Limitations

- Skeleton tags are categorical (1–10) representations of distance-stratified contact percentiles, so they discard continuous contact magnitudes
- deepC requires a specific software stack (python 3.5+, tensorflow v1.x with GPU for training, numpy >= 1.16.4, pysam ~0.15.2, pybedtools plus bedtools) that must match for array consumption
- The skill is tied to the deepC preprocessing convention (KR-normalized 5 kb Rao maps); other Hi-C normalizations or resolutions are not covered by the documented workflow

## Evidence

- [methods] Write per-chromosome skeleton arrays carrying the tags to disk as .npy/.h5 files: [evidence span withheld — anchor status: not_found]
- [methods] confirm every skeleton file exists and parses in .npy/.h5 format, and that all stored tags fall in the range 1-10: [evidence span withheld — anchor status: not_found]
