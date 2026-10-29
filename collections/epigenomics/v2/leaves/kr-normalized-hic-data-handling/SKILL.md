---
name: kr-normalized-hic-data-handling
description: Use when raw KR-normalized Hi-C contact frequency matrices must be converted
  into the discrete 1-10 percentile skeleton tags deepC needs for training or prediction
  input.
when_to_use_negative:
- Input is already a discrete skeleton/percentile-tag array — re-normalizing would
  destroy the categorical tags
- Contact maps are not KR-normalized or are not at 5 kb bin resolution, since the
  percentile scheme and bin geometry assume the Rao et al. GSE63525 KR-normalized
  5 kb maps
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: deepC
  role: Provides the skeleton preprocessing pipeline that percentile-normalizes Hi-C
    contacts by genomic distance into 1-10 tags
  repo: https://github.com/rschwess/deepC
- name: pysam
  role: Python module required by the deepC skeleton/Hi-C data handling code
  repo: https://github.com/pysam-developers/pysam
- name: pybedtools
  role: Python library installed alongside pysam to support deepC's Hi-C genomic interval
    handling
  repo: https://github.com/rschwess/deepC
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:0fbe76689509a3c44ceb90d5e283059d35b55e13ac97cb8052b27e84a5d59d53
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rschwess/deepC
  tools:
  - deepC
  - pybedtools
  - pysam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# kr-normalized-hic-data-handling

## Summary

Loading KR-normalized Hi-C contact maps (Rao et al., GEO GSE63525, 5 kb bins) and converting them into deepC's distance-stratified percentile-normalized 'skeleton' representation, which reveals longer-range interactions and enhances contrast at domain boundaries.

## When to use

Use when raw KR-normalized Hi-C contact frequency matrices must be converted into the discrete 1-10 percentile skeleton tags deepC needs for training or prediction input.

## When NOT to use

- Input is already a discrete skeleton/percentile-tag array — re-normalizing would destroy the categorical tags
- Contact maps are not KR-normalized or are not at 5 kb bin resolution, since the percentile scheme and bin geometry assume the Rao et al. GSE63525 KR-normalized 5 kb maps

## Inputs

- KR-normalized Hi-C contact maps from Rao et al. (GEO accession GSE63525) at 5 kb bin resolution
- Per-chromosome dense contact frequency matrices

## Outputs

- Per-chromosome skeleton arrays with discrete percentile tags 1-10
- Skeleton files in .npy/.h5 format

## How to apply

Load the KR-normalized Rao et al. Hi-C contact maps (GEO accession GSE63525) at 5 kb bin resolution into memory as dense per-chromosome contact matrices. Stratify contact frequencies by genomic interaction distance so only contacts of similar distance are compared, which removes the monotonic distance-decay effect. Apply deepC's uneven pyramid percentile scheme to each distance-stratified distribution — 2 bins of 20% followed by 4 bins of 10% and 4 bins of 5% — yielding 10 percentile classes with finer resolution in the distribution tails. Assign each contact a discrete percentile tag from 1 to 10 based on its class assignment, converting continuous contact values into categorical skeleton annotations. Write per-chromosome skeleton arrays carrying the tags to disk as .npy/.h5 files for downstream deepC training and prediction.

## Related tools

- **deepC** (Provides the skeleton preprocessing pipeline that percentile-normalizes Hi-C contacts by genomic distance into 1-10 tags) — https://github.com/rschwess/deepC
- **pysam** (Python module required by the deepC skeleton/Hi-C data handling code) — https://github.com/pysam-developers/pysam
- **pybedtools** (Python library installed alongside pysam to support deepC's Hi-C genomic interval handling) — https://github.com/rschwess/deepC

## Examples

```
pip install pysam; pip install pybedtools
```

## Evaluation signals

- Every skeleton file exists and parses as .npy/.h5
- All stored percentile tags fall in the range 1-10
- The full 10-class binning is represented in the output tags
- Contacts were stratified by genomic interaction distance before percentile assignment (no cross-distance comparisons)
- Skeleton arrays are per-chromosome and preserve 5 kb bin resolution of the KR-normalized maps

## Limitations

- The percentile scheme is a fixed uneven pyramid (2×20%, 4×10%, 4×5%) tied to deepC's expected input format; changing it breaks compatibility with trained models
- The deepC repository requires python 3.5+, tensorflow v1, numpy v1.16.4+, pysam (tested v0.15.2), pybedtools plus a compatible bedtools, and R 3.4.4+ for downstream analysis
- GPU support is preferable for predictions and essential for training deepC models, though loading/skeleton generation itself is CPU work

## Evidence

- [other] DeepC's skeleton preprocessing works by percentile-normalizing the raw contact frequency signal in Hi-C data by genomic distance: [evidence span withheld — anchor status: not_found]
- [full_text] Load the KR-normalized Rao et al. Hi-C contact maps (GEO accession GSE63525) at 5 kb bin resolution: [evidence span withheld — anchor status: not_found]
- [other] Apply the uneven pyramid percentile scheme to each distance-stratified distribution — 2 bins of 20% followed by 4 bins of 10% and 4 bins of 5% — yielding 10 percentile classes: [evidence span withheld — anchor status: not_found]
