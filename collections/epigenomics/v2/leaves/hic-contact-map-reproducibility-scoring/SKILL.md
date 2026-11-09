---
name: hic-contact-map-reproducibility-scoring
description: Use when you have two 40 kb-resolution Hi-C contact maps (e.g., biological
  replicates, normalized vs reference, or downsampled vs full data) and need a single
  reproducibility score to compare normalization methods or sequencing-depth robustness.
when_to_use_negative:
- Input is a single contact map with no second replicate or reference map to compare
  against — reproducibility scoring requires a map pair.
- Contact maps are at a resolution other than the 40 kb used in the covNorm benchmarks
  without re-validating the coverage cut-off of 200, or data still contain zero-valued
  fragment pairs and unfiltered low-coverage fragments.
- The goal is differential interaction calling or compartment analysis rather than
  a between-map reproducibility metric.
edam_operation: http://edamontology.org/operation_3351
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: HiC-spector
  role: Provides get_reproducibility and run_reproducibility_v2.py to compute the
    eigenvector-based reproducibility score between two contact maps with num_evec
    = 20
  repo: https://github.com/gersteinlab/HiC-spector
- name: covNorm
  role: R package producing coverage-normalized contact maps (via covNormRpkg::normCoverage)
    whose reproducibility is scored; the method under evaluation
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: bedtools (coverageBed)
  role: Computes the 40 kb-resolution bin coverage values (cov_frag1, cov_frag2) used
    in the feature-table inputs
  repo: https://github.com/arq5x/bedtools2
- name: iced
  role: Python package used in the paper to run ICE and SCN normalization baselines
    for reproducibility comparison
  repo: https://github.com/hiclib/iced
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:7bac12a422e45186ab6b235152d8b9817fef72f7341120423c9c129995ff4f6a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kaistcbfg/covNormRpkg
  repo_url: https://github.com/gersteinlab/HiC-spector
  tools:
  - HiC-spector
  - R
  - bedtools (coverageBed)
  - covNorm
  - iced
  edam_operation: http://edamontology.org/operation_3351
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# hic-contact-map-reproducibility-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify reproducibility between two Hi-C/capture Hi-C contact maps using HiC-spector's eigenvector-decomposition-based score (get_reproducibility / run_reproducibility_v2.py with num_evec = 20), the metric covNorm's normalization performance was benchmarked against (e.g., GM19204 vs GM19240 chr17 at 40 kb resolution).

## When to use

Use when you have two 40 kb-resolution Hi-C contact maps (e.g., biological replicates, normalized vs reference, or downsampled vs full data) and need a single reproducibility score to compare normalization methods or sequencing-depth robustness.

## When NOT to use

- Input is a single contact map with no second replicate or reference map to compare against — reproducibility scoring requires a map pair.
- Contact maps are at a resolution other than the 40 kb used in the covNorm benchmarks without re-validating the coverage cut-off of 200, or data still contain zero-valued fragment pairs and unfiltered low-coverage fragments.
- The goal is differential interaction calling or compartment analysis rather than a between-map reproducibility metric.

## Inputs

- Two 40 kb-resolution cis-interaction contact maps as tab-delimited feature tables (frag1, frag2, cov_frag1, cov_frag2, freq, dist), e.g., GM19204.chr17.cis.feature.gz and GM19240.chr17.cis.feature.gz
- Optionally, randomly downsampled contact maps at 15%-50% of fragment pairs in 5% intervals
- .hic format files or simple text-delimited matrix files (no header: bin_i, bin_j, count) accepted by run_reproducibility_v2.py via straw.py

## Outputs

- HiC-spector reproducibility score (0-1 range) for each map pair
- Reduction-rate table (e.g., subsample_reproducibility_reduction.csv) with one row per downsampling ratio giving the HiC-spector score and its reduction relative to the full dataset

## How to apply

Prepare each contact map as a cis-interaction feature table (frag1, frag2, cov_frag1, cov_frag2, freq, dist) at 40 kb resolution, after removing zero-valued DNA fragment pairs and filtering fragments with coverage below the default cut-off of 200 at 40 kb resolution. Score map pairs with HiC-spector's run_reproducibility_v2.py script using the get_reproducibility function and num_evec = 20 (default), which compares the top eigenvectors of the two maps' Strang-score-style decomposition. For downsampling robustness analyses (15%-50% of fragment pairs in 5% intervals), compute each subsample's score reduction rate relative to the full (100%) dataset score and write all ratios and scores to a CSV such as subsample_reproducibility_reduction.csv. Lower score reduction with decreasing sampling depth indicates higher reproducibility robustness of the normalization method being evaluated.

## Related tools

- **HiC-spector** (Provides get_reproducibility and run_reproducibility_v2.py to compute the eigenvector-based reproducibility score between two contact maps with num_evec = 20) — https://github.com/gersteinlab/HiC-spector
- **covNorm** (R package producing coverage-normalized contact maps (via covNormRpkg::normCoverage) whose reproducibility is scored; the method under evaluation) — https://github.com/kaistcbfg/covNormRpkg
- **bedtools (coverageBed)** (Computes the 40 kb-resolution bin coverage values (cov_frag1, cov_frag2) used in the feature-table inputs) — https://github.com/arq5x/bedtools2
- **iced** (Python package used in the paper to run ICE and SCN normalization baselines for reproducibility comparison) — https://github.com/hiclib/iced

## Examples

```
python run_reproducibility_v2.py GM19204.chr17.cis.normalized.txt GM19240.chr17.cis.normalized.txt --num_evec 20
```

## Evaluation signals

- subsample_reproducibility_reduction.csv exists and contains a reduction-rate row for each of the 8 subsampling ratios (15%-50% in 5% intervals)
- Every reported reduction rate lies within [0, 1]
- Scores are computed with num_evec = 20 (the default) via get_reproducibility in run_reproducibility_v2.py
- All scores were calculated only after removing zero-valued DNA fragment pairs and filtering fragments with coverage below 200 at 40 kb resolution
- Normalized methods should show equal or higher HiC-spector scores than raw data (covNorm's score change was significant, paired t-test p-value = 0.019)

## Limitations

- The Python reproducibility script requires the Aiden lab straw.py to read .hic files; text input must be a simple no-header bin_i, bin_j, count format
- None of the examined normalization methods showed significantly higher HiC-spector scores than the others (paired t-test, p-values > 0.05); covNorm only recorded the highest median value, so score differences may not be statistically significant
- The Julia core of HiC-spector was tested only in Julia 0.4 and 0.5, which may complicate installation
- No version number is stated for covNorm in the discussion; only version covNormRpkg_1.1.0 appears in the README build example

## Evidence
