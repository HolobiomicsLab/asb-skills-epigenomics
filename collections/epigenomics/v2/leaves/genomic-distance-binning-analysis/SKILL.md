---
name: genomic-distance-binning-analysis
description: Use when coverage-normalized Hi-C or pcHi-C fragment-pair tables (Rij
  residuals with cov_frag1, cov_frag2, freq, dist columns) need distance-dependent
  background removal before calling significant long-range interactions.
when_to_use_negative:
- Input is raw (unnormalized) fragment-pair data — the bias-removing normCoverage
  step must run first, since the distance model is fit on coverage-normalized residuals
  Rij
- Inter-chromosomal or trans interactions — covNorm's demonstrated workflow focuses
  on cis-interactions only, and the distance background model applies to intra-chromosomal
  genomic distance
- Fragment pairs beyond the max_dist cutoff (e.g. > 2 Mb in the example) — these are
  excluded from distance-dependent background normalization
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: covNorm
  role: R package whose normDistance function fits the distance-dependent background
    model and computes distance-normalized frequencies (dist_res); part of the covNormRpkg
    pipeline after normCoverage
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: R
  role: Language environment in which covNormRpkg is installed via devtools and the
    normalization/plotting workflow is executed
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: HiC-spector
  role: Used to evaluate reproducibility of covNorm-normalized contact maps via the
    HiC-spector reproducibility score, including expected-contact-vs-distance behavior
  repo: https://github.com/gersteinlab/HiC-spector
- name: bedtools
  role: coverageBed computes the 40 kb bin coverage values (cov_frag1, cov_frag2)
    that feed the upstream bias-removal step before distance normalization
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:9f01572b2e7bce8819eddeba0b345dac67442e5eb79a85c8314145af2000159f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kaistcbfg/covNormRpkg
  repo_url: https://github.com/kaistcbfg/covNormRpkg
  tools:
  - HiC-spector
  - R
  - bedtools
  - covNorm
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# genomic-distance-binning-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Performs genomic distance-dependent background normalization of Hi-C/capture Hi-C interaction frequencies, fitting expected ligation frequency as a function of distance and removing this background from coverage-normalized contact data using the covNorm R package's normDistance function.

## When to use

Use when coverage-normalized Hi-C or pcHi-C fragment-pair tables (Rij residuals with cov_frag1, cov_frag2, freq, dist columns) need distance-dependent background removal before calling significant long-range interactions.

## When NOT to use

- Input is raw (unnormalized) fragment-pair data — the bias-removing normCoverage step must run first, since the distance model is fit on coverage-normalized residuals Rij
- Inter-chromosomal or trans interactions — covNorm's demonstrated workflow focuses on cis-interactions only, and the distance background model applies to intra-chromosomal genomic distance
- Fragment pairs beyond the max_dist cutoff (e.g. > 2 Mb in the example) — these are excluded from distance-dependent background normalization

## Inputs

- Coverage-normalized fragment-pair data.frame from covNormRpkg::normCoverage (columns: frag1, frag2, cov_frag1, cov_frag2, freq, dist)
- Distance cutoff parameter (example workflow uses max_dist = 2000000, i.e. 2 Mb)

## Outputs

- Result data.frame with a dist_res column of distance-normalized interaction frequencies for every fragment pair within the distance cutoff
- QC plots from checkFreqDistPCC (frequency–distance PCC) and plotDistNormRes (distance normalization hexmap)

## How to apply

Start from the output of covNormRpkg::normCoverage — a data.frame of bias-removed residuals Rij with frag1, frag2, coverage, frequency, and distance columns (e.g. GM19204/GM19240 chr17 cis fragment pairs). Fit the distance model log(uij) = β0 + β1·Dij to estimate Ed, the expected ligation frequency at genomic distance d, using negative binomial model assumptions. Call covNormRpkg::normDistance with max_dist = 2000000 so only fragment pairs within 2 Mb are normalized against the distance-dependent background. Compute the final signal with the pseudocount-adjusted ratio (Rij + avg(Rij)) / (Ed + avg(Rij)), where avg(Rij) acts as the pseudocount to stabilize low-frequency pairs. Write the resulting data.frame whose dist_res column contains the distance-normalized interaction frequencies, and optionally proceed to covNormRpkg::contactPval for significance calling.

## Related tools

- **covNorm** (R package whose normDistance function fits the distance-dependent background model and computes distance-normalized frequencies (dist_res); part of the covNormRpkg pipeline after normCoverage) — https://github.com/kaistcbfg/covNormRpkg
- **R** (Language environment in which covNormRpkg is installed via devtools and the normalization/plotting workflow is executed) — https://github.com/kaistcbfg/covNormRpkg
- **HiC-spector** (Used to evaluate reproducibility of covNorm-normalized contact maps via the HiC-spector reproducibility score, including expected-contact-vs-distance behavior) — https://github.com/gersteinlab/HiC-spector
- **bedtools** (coverageBed computes the 40 kb bin coverage values (cov_frag1, cov_frag2) that feed the upstream bias-removal step before distance normalization) — https://github.com/arq5x/bedtools2

## Examples

```
library('covNormRpkg'); dist_result <- covNormRpkg::normDistance(cov_df, max_dist=2000000); dist_df <- dist_result$result_df
```

## Evaluation signals

- Output table exists and the dist_res column is non-empty and populated for all fragment pairs within the 2 Mb (max_dist = 2000000) distance cutoff
- dist_res values match the pseudocount-adjusted ratio (Rij + avg(Rij)) / (Ed + avg(Rij)) recomputed from the fitted distance model Ed
- QC plot checkFreqDistPCC shows reasonable Pearson correlation between normalized frequency and distance after normalization
- plotDistNormRes hexmap shows the distance-decay signal is removed (flattened frequency-vs-distance relationship) relative to the coverage-normalized input
- Downstream contactPval runs without error on the dist-normalized data.frame, indicating schema compatibility (dist_res column present)

## Limitations

- Only fragment pairs within the max_dist cutoff (2 Mb in the example workflow) are normalized against the distance-dependent background; pairs beyond this distance are not handled by normDistance
- Zero-valued DNA fragment pairs were removed from the demonstrated input data in this study, so the distance model is fit on non-zero frequencies
- The coverage cut-off threshold default of 200 at 40 kb resolution upstream can affect which fragments enter distance normalization; pcHi-C PO-interaction mode uses different thresholds (promoter 200, other 50)

## Evidence
