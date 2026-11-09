---
name: hic-contact-map-percentile-normalization
description: Use when raw Hi-C contact frequency matrices (e.g. KR-normalized 5 kb
  bins) must be converted into distance-aware, percentile-binned skeleton labels to
  train or run inference with deepC.
when_to_use_negative:
- Input is not Hi-C contact data (e.g. ChIP-seq tracks, sequence FASTA) — the skeleton
  scheme is defined only over distance-stratified contact frequencies
- The contact maps are not KR-normalized or are at a different bin resolution than
  5 kb — percentile classes would not match the deepC training distribution
- A continuous-valued interaction target is needed (e.g. for regression models like
  Akita) rather than deepC's discrete 10-class skeleton
tools:
- name: deepC
  role: Source framework whose skeleton preprocessing implements the distance-stratified
    percentile normalization of Hi-C contact maps
  repo: github.com/rschwess/deepC
- name: pysam
  role: Python dependency installed alongside deepC for handling genomic sequence/coordinate
    data during preprocessing
  repo: github.com/pysam-developers/pysam
- name: pybedtools
  role: Python dependency installed alongside deepC for genomic interval handling
    during Hi-C preprocessing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:d677c8e8e5cbe41029c9869cea9f87d1b6a542d6606778269667aafcd03359b0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - deepC
  - pybedtools
  - pysam
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# hic-contact-map-percentile-normalization

## Summary

Computes the deepC 'skeleton' by percentile-normalizing KR-normalized 5 kb Hi-C contact maps stratified by genomic distance, yielding discrete 1–10 interaction classes used as training targets for sequence-based chromatin interaction prediction.

## When to use

Use when raw Hi-C contact frequency matrices (e.g. KR-normalized 5 kb bins) must be converted into distance-aware, percentile-binned skeleton labels to train or run inference with deepC.

## When NOT to use

- Input is not Hi-C contact data (e.g. ChIP-seq tracks, sequence FASTA) — the skeleton scheme is defined only over distance-stratified contact frequencies
- The contact maps are not KR-normalized or are at a different bin resolution than 5 kb — percentile classes would not match the deepC training distribution
- A continuous-valued interaction target is needed (e.g. for regression models like Akita) rather than deepC's discrete 10-class skeleton

## Inputs

- KR-normalized Hi-C contact maps (Rao et al., GEO GSE63525) at 5 kb bin resolution
- Per-chromosome dense contact frequency matrices
- Genomic distance strata for contact comparison

## Outputs

- Per-chromosome skeleton arrays with discrete percentile tags 1–10 stored as .npy/.h5 files
- Categorical interaction-class annotations used as deepC training/prediction targets

## How to apply

Load KR-normalized Rao et al. (GSE63525) Hi-C contact maps at 5 kb resolution as dense per-chromosome matrices. Stratify contact frequencies by genomic interaction distance so only contacts of similar distance are compared, which reveals longer-range interactions and enhances contrast at domain boundaries. Apply the uneven pyramid percentile scheme — 2 bins of 20%, then 4 bins of 10% and 4 bins of 5% — giving 10 percentile classes with finer resolution in the distribution tails. Assign each contact a discrete tag from 1 to 10 and write per-chromosome skeleton arrays to .npy/.h5 files. Rationale: converting continuous contact values into categorical skeleton annotations is what deepC trains on and predicts, so binning choices directly shape model targets.

## Related tools

- **deepC** (Source framework whose skeleton preprocessing implements the distance-stratified percentile normalization of Hi-C contact maps) — github.com/rschwess/deepC
- **pysam** (Python dependency installed alongside deepC for handling genomic sequence/coordinate data during preprocessing) — github.com/pysam-developers/pysam
- **pybedtools** (Python dependency installed alongside deepC for genomic interval handling during Hi-C preprocessing)

## Examples

```
import numpy as np; np.save('chr1_skeleton.npy', tags)  # per-chromosome array of discrete percentile tags 1-10
```

## Evaluation signals

- Every skeleton file exists and parses in .npy/.h5 format
- All stored tags fall in the range 1–10
- All 10 percentile classes of the binning scheme are represented (no empty classes)
- Contacts of similar genomic distance are compared within strata (distance stratification verified before normalization)
- Resulting skeletons enhance contrast at domain boundaries and expose longer-range interactions, consistent with deepC's expected training signal

## Limitations

- Skeleton tags are categorical (1–10), so fine-grained quantitative contact strength is lost; binning resolution is fixed by the uneven pyramid scheme
- Training deepC models requires GPU support and can take hours up to days; predictions are significantly faster with a GPU
- deepC was developed under tensorflow v1.8 and only v1.8, 1.14 and 2.1 have been tested

## Evidence

- [methods] Percentile-normalizing contact frequency by genomic distance: [evidence span withheld — anchor status: not_found]
