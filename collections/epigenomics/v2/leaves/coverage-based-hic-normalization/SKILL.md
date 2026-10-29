---
name: coverage-based-hic-normalization
description: Use when Hi-C or capture Hi-C replicate contact maps must be bias-corrected
  before reproducibility assessment, or when comparing how normalization methods hold
  up as sequencing depth is downsampled to 15%-50%.
when_to_use_negative:
- Input is already normalized or is a matrix-format .hic contact map rather than a
  fragment-pair feature table with coverage columns
- The target is promoter-centered capture Hi-C PO-interaction normalization with distinct
  thresholds (promoter side 200, other side 50) and 15 kb-2 Mb interaction selection,
  which requires the pcHi-C variant of the workflow
- No replicate data exist to compute a reproducibility score against
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: covNorm
  role: Negative binomial GLM-based coverage-bias removal via normCoverage; R package
    covNormRpkg
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: HiC-spector
  role: Computes reproducibility scores between normalized maps via run_reproducibility_v2.py
    and get_reproducibility
  repo: https://github.com/gersteinlab/HiC-spector
- name: R
  role: Runtime environment for covNormRpkg::normCoverage and downstream subsampling
    scripts
- name: BEDTools coverage
  role: Computes 40 kb resolution DNA fragment bin coverage used as cov_frag1/cov_frag2
    inputs
  repo: https://github.com/arq5x/bedtools2
- name: iced
  role: Alternative ICE and SCN normalization used as baseline for reproducibility
    comparison
  repo: https://github.com/hiclib/iced
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:ff35f85b8014dabf647be985d157ed7d22ce1b5003292e2c53eeb6cc40cf726a
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
  - BEDTools coverage
  - HiC-spector
  - R
  - covNorm
  - iced
  edam_operation: http://edamontology.org/operation_3675
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

# coverage-based-hic-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalize Hi-C contact maps for fragment-coverage biases using covNorm's negative binomial GLM (log(u) = beta0 + beta1*C_frag1 + beta2*C_frag2), then quantify reproducibility across biological replicates with the HiC-spector score, including robustness under downsampling to 15%-50% of sequencing depth.

## When to use

Use when Hi-C or capture Hi-C replicate contact maps must be bias-corrected before reproducibility assessment, or when comparing how normalization methods hold up as sequencing depth is downsampled to 15%-50%.

## When NOT to use

- Input is already normalized or is a matrix-format .hic contact map rather than a fragment-pair feature table with coverage columns
- The target is promoter-centered capture Hi-C PO-interaction normalization with distinct thresholds (promoter side 200, other side 50) and 15 kb-2 Mb interaction selection, which requires the pcHi-C variant of the workflow
- No replicate data exist to compute a reproducibility score against

## Inputs

- Hi-C cis-interaction feature table (frag1, frag2, cov_frag1, cov_frag2, freq, dist; e.g. GM19204.chr17.cis.feature.gz at 40 kb resolution)
- Replicate contact map for reproducibility scoring (e.g. GM19240 chr17)
- Randomly downsampled fragment-pair subsets at 15%-50% sequencing depth (5% intervals)

## Outputs

- Coverage-normalized contact map (normCoverage result_df)
- HiC-spector reproducibility scores per subsampling ratio (num_evec = 20)
- subsample_reproducibility_reduction.csv with reduction rate per subsampling ratio

## How to apply

Load the cis-interaction feature table (e.g. GM19204.chr17.cis.feature.gz with frag1/frag2, cov_frag1, cov_frag2, freq, dist columns at 40 kb resolution) and remove zero-valued DNA fragment pairs plus fragments with coverage below the 200 threshold at 40 kb. Apply covNormRpkg::normCoverage, which fits the negative binomial GLM log(u) = beta0 + beta1*C_frag1 + beta2*C_frag2 and returns coverage-bias-removed residuals; for downsampling robustness, randomly subsample fragment pairs at 15%, 20%, 25%, 30%, 35%, 40%, 45%, and 50% of the data before normalization. Score each normalized map against the replicate map (e.g. GM19240 chr17) with HiC-spector's run_reproducibility_v2.py using get_reproducibility and num_evec = 20 (default). Compute each subsample's score reduction rate relative to the full (100%) dataset score to compare robustness across normalization methods.

## Related tools

- **covNorm** (Negative binomial GLM-based coverage-bias removal via normCoverage; R package covNormRpkg) — https://github.com/kaistcbfg/covNormRpkg
- **HiC-spector** (Computes reproducibility scores between normalized maps via run_reproducibility_v2.py and get_reproducibility) — https://github.com/gersteinlab/HiC-spector
- **R** (Runtime environment for covNormRpkg::normCoverage and downstream subsampling scripts)
- **BEDTools coverage** (Computes 40 kb resolution DNA fragment bin coverage used as cov_frag1/cov_frag2 inputs) — https://github.com/arq5x/bedtools2
- **iced** (Alternative ICE and SCN normalization used as baseline for reproducibility comparison) — https://github.com/hiclib/iced

## Examples

```
cov_result <- covNormRpkg::normCoverage(covNormRpkg::filterInputDF(read.table(gzfile('GM19204.chr17.cis.feature.gz'), head=TRUE)))
```

## Evaluation signals

- subsample_reproducibility_reduction.csv exists and contains a reduction-rate row for each of the 8 subsampling ratios (15%-50%)
- Every reported reduction rate lies within [0, 1]
- Coverage-normalized output retains the input schema plus normCoverage residual columns (result_df from cov_result)
- HiC-spector scores are computed with num_evec = 20 (default) against the matched replicate map
- covNorm achieves the highest average HiC-spector score (lowest reduction in reproducibility) across all downsampling ratios compared with baseline methods

## Limitations

- None of the examined methods showed significantly higher HiC-spector scores than the others (paired t-test, p-values > 0.05), though covNorm recorded the highest median value
- No version number or changelog is stated for covNorm in the repository documentation
- No explicit license terms are stated; only copyright registration at the Korea Copyright Commission (C-2021-022800) and contact-the-authors terms for commercial use

## Evidence

- [methods] negative binomial GLM with log(u) = beta0 + beta1*C_frag1 + beta2*C_frag2: [evidence span withheld — anchor status: not_found]
- [results] covNorm robustness under downsampling: [evidence span withheld — anchor status: not_found]
- [methods] HiC-spector scoring parameters: [evidence span withheld — anchor status: not_found]
- [methods] Preprocessing thresholds: [evidence span withheld — anchor status: not_found]
