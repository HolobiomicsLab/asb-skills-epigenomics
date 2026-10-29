---
name: distance-stratified-contact-decile-binning
description: Use when converting raw KR-normalized Hi-C contact matrices (e.g. Rao
  et al. GSE63525 at 5 kb bins) into categorical skeleton annotations to train or
  run predictions with deepC.
when_to_use_negative:
- Input is already a normalized or categorical skeleton file — re-binning would destroy
  the original percentile classes
- Working with plaid/other binning schemes or raw unnormalized contact maps without
  KR normalization, where the percentile classes will not match the deepC-trained
  targets
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: deepC
  role: Framework whose preprocessing pipeline implements the distance-stratified
    percentile ('skeleton') binning of Hi-C contacts used as training/prediction targets
  repo: github.com/rschwess/deepC
- name: pysam
  role: Additional python library installed alongside deepC for handling genomic data
    during preprocessing
  repo: https://github.com/pysam-developers/pysam
- name: pybedtools
  role: Additional python library installed alongside deepC for genomic interval operations
    during preprocessing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:19bfc5819a1728b0e5e06ec55bf2393b7bd9ced239e471bd91b784f6bd14ec05
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - deepC
  - pybedtools
  - pysam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Distance-stratified contact decile binning

## Summary

DeepC skeleton preprocessing that percentile-normalizes raw Hi-C contact frequencies by genomic interaction distance, converting continuous contact values into 10 discrete percentile tags (1-10) using an uneven pyramid binning scheme. This reveals longer-range interactions and enhances contrast at domain boundaries for training and prediction.

## When to use

Use when converting raw KR-normalized Hi-C contact matrices (e.g. Rao et al. GSE63525 at 5 kb bins) into categorical skeleton annotations to train or run predictions with deepC.

## When NOT to use

- Input is already a normalized or categorical skeleton file — re-binning would destroy the original percentile classes
- Working with plaid/other binning schemes or raw unnormalized contact maps without KR normalization, where the percentile classes will not match the deepC-trained targets

## Inputs

- KR-normalized Hi-C contact frequency matrices (Rao et al., GEO accession GSE63525) at 5 kb bin resolution
- Per-chromosome dense contact matrices
- Genomic distance information for each contact pair

## Outputs

- Per-chromosome skeleton arrays of discrete percentile tags 1-10
- Skeleton files written in .npy or .h5 format

## How to apply

Load the KR-normalized Rao et al. Hi-C contact maps (GEO accession GSE63525) at 5 kb bin resolution as dense per-chromosome contact matrices. Stratify contact frequencies by genomic interaction distance so that only contacts of similar distance are compared, removing the strong distance-dependence of raw contact frequency. Apply the uneven pyramid percentile scheme to each distance-stratified distribution — 2 bins of 20%, then 4 bins of 10%, then 4 bins of 5% — yielding 10 percentile classes with finer resolution in the distribution tails. Assign each contact a discrete percentile tag from 1 to 10 based on its class assignment and write per-chromosome skeleton arrays to disk as .npy/.h5 files. The rationale is that distance stratification plus tail-weighted binning reveals informative longer-range interactions and enhances contrast at domain boundaries used as deepC training targets.

## Related tools

- **deepC** (Framework whose preprocessing pipeline implements the distance-stratified percentile ('skeleton') binning of Hi-C contacts used as training/prediction targets) — github.com/rschwess/deepC
- **pysam** (Additional python library installed alongside deepC for handling genomic data during preprocessing) — https://github.com/pysam-developers/pysam
- **pybedtools** (Additional python library installed alongside deepC for genomic interval operations during preprocessing)

## Evaluation signals

- Every skeleton file exists and parses successfully as .npy or .h5
- All stored tags fall in the range 1-10
- The full 10-class binning is represented (all ten percentile classes present in the output)
- Each contact's tag derives from its distance-stratified distribution — contacts are only compared against others of similar genomic distance
- Tag distribution reflects the uneven pyramid scheme: 2 bins of 20%, 4 bins of 10%, 4 bins of 5%

## Limitations

- Binning classes are defined on a specific reference dataset (KR-normalized Rao et al. GSE63525 at 5 kb resolution); applying the scheme to maps with different normalization or resolution may produce non-comparable tags
- Converting continuous contact values into 10 categorical tags discards within-bin amplitude information
- The uneven pyramid scheme emphasizes distribution tails; classes in the mid-distribution have coarser resolution (20% bins)

## Evidence

- [other] DeepC's skeleton preprocessing works by percentile-normalizing the raw contact frequency signal in Hi-C data by genomic distance: [evidence span withheld — anchor status: not_found]
- [other] 2 bins of 20% followed by 4 bins of 10% and 4 bins of 5%: [evidence span withheld — anchor status: not_found]
