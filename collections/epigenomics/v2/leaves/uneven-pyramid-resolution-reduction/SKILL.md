---
name: uneven-pyramid-resolution-reduction
description: Use when raw Hi-C contact maps (e.g. KR-normalized Rao et al. GSE63525
  at 5 kb resolution) must be converted into percentile-tagged skeleton annotations
  as ground-truth targets for deepC training or prediction.
when_to_use_negative:
- Input is already a percentile-normalized skeleton or categorical contact annotation,
  not raw continuous contact frequencies
- The downstream model expects continuous contact values rather than deepC's discrete
  10-class skeleton targets
- Hi-C data is not distance-normalized and you need raw contact intensities for visualization
  rather than domain-contrast annotations
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: deepC
  role: Framework containing the skeleton preprocessing that implements the percentile-normalization
    by genomic distance used for training and prediction
  repo: https://github.com/rschwess/deepC
- name: pysam
  role: Python library installed to support deepC's Hi-C data handling in the preprocessing
    workflow
  repo: https://github.com/pysam-developers/pysam
- name: pybedtools
  role: Python library installed to support genomic interval handling in the deepC
    preprocessing workflow
  repo: https://github.com/daler/pybedtools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:73c24100001714aabb1e5a7566c081cf37f8dab7447cd2777300849c2ac4bac2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rschwess/deepC
  tools:
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Uneven pyramid resolution reduction

## Summary

Converts raw KR-normalized Hi-C contact frequencies into a categorical 'skeleton' by percentile-normalizing contacts stratified by genomic distance, using an uneven pyramid binning scheme (2 bins of 20%, 4 of 10%, 4 of 5%) yielding 10 discrete percentile classes that serve as deepC training/prediction targets.

## When to use

Use when raw Hi-C contact maps (e.g. KR-normalized Rao et al. GSE63525 at 5 kb resolution) must be converted into percentile-tagged skeleton annotations as ground-truth targets for deepC training or prediction.

## When NOT to use

- Input is already a percentile-normalized skeleton or categorical contact annotation, not raw continuous contact frequencies
- The downstream model expects continuous contact values rather than deepC's discrete 10-class skeleton targets
- Hi-C data is not distance-normalized and you need raw contact intensities for visualization rather than domain-contrast annotations

## Inputs

- KR-normalized Hi-C contact maps (GSE63525) at 5 kb bin resolution
- Per-chromosome dense contact frequency matrices

## Outputs

- Per-chromosome skeleton arrays with discrete percentile tags 1-10
- Skeleton files stored in .npy/.h5 format

## How to apply

Load KR-normalized Hi-C contact maps (GEO GSE63525, 5 kb bins) as dense per-chromosome contact matrices. Stratify contact frequencies by genomic interaction distance so only contacts of similar distance are compared, which controls for the well-known distance-decay of Hi-C signal. Apply the uneven pyramid percentile scheme to each distance-stratified distribution: 2 bins of 20% followed by 4 bins of 10% and 4 bins of 5%, producing 10 percentile classes with finer resolution in the distribution tails. Assign each contact a discrete tag from 1 to 10 based on its class, then write per-chromosome skeleton arrays to disk as .npy/.h5 files. The finer tail resolution is the rationale for the uneven scheme: it enhances contrast at domain boundaries and reveals informative longer-range interactions.

## Related tools

- **deepC** (Framework containing the skeleton preprocessing that implements the percentile-normalization by genomic distance used for training and prediction) — https://github.com/rschwess/deepC
- **pysam** (Python library installed to support deepC's Hi-C data handling in the preprocessing workflow) — https://github.com/pysam-developers/pysam
- **pybedtools** (Python library installed to support genomic interval handling in the deepC preprocessing workflow) — https://github.com/daler/pybedtools

## Examples

```
import deepCregr
```

## Evaluation signals

- Every output skeleton file exists and parses in .npy/.h5 format
- All stored tags fall in the range 1-10
- The full 10-class binning (2x20%, 4x10%, 4x5%) is represented in the distribution of tags
- Skeletons are stratified by genomic distance such that percentile assignment is only computed among contacts of similar distance

## Limitations

- The tag scheme is tied to deepC's expected input format (10 discrete classes); other frameworks may require different normalization
- Validation depends on the full 10-class binning being represented; sparse distance strata may fail to populate all classes

## Evidence

- [other] skeleton preprocessing definition: [evidence span withheld — anchor status: not_found]
- [other] skeleton purpose: [evidence span withheld — anchor status: not_found]
