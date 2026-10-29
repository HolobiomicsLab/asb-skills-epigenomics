---
name: downsampling-benchmark-analysis
description: Use when comparing how robustly normalization methods preserve inter-replicate
  contact map reproducibility as sequencing depth is reduced, or when needing reduction-rate
  curves of a reproducibility metric across sampling ratios.
when_to_use_negative:
- Input is not a Hi-C or capture Hi-C fragment-pair coverage table (covNorm's input
  requires frag1/frag2/cov/freq/dist columns); raw BAM/FASTQ must be preprocessed
  first
- The comparison targets inter-chromosomal interactions — the article filters inter-chromosomal
  interactions and focuses on cis-interactions only
- Full-depth replicate data are available for all methods, so no depth-robustness
  question exists — a single-shot reproducibility comparison suffices
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: covNorm
  role: Normalizes each downsampled subsample via negative binomial GLM-based coverage-bias
    removal (normCoverage)
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: HiC-spector
  role: Computes the HiC-spector reproducibility score between normalized subsample
    maps and the reference map via run_reproducibility_v2.py (get_reproducibility,
    num_evec=20)
  repo: https://github.com/gersteinlab/HiC-spector
- name: R
  role: Runtime environment for covNormRpkg subsampling, filtering, and normalization
    steps
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: iced
  role: Provides ICE and SCN baseline normalization methods for comparison against
    covNorm in the benchmark
  repo: https://github.com/hiclib/iced
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:dcb015f5d0c7becb62deb56c2534493af1e099d4eacba578cce6a342db8f87a6
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
  - covNorm
  - iced
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# downsampling-benchmark-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking protocol that randomly downsamples sequencing-depth-limited Hi-C data (15%-50% in 5% intervals), normalizes each subsample (e.g., with covNorm's negative binomial coverage normalization), and quantifies reproducibility loss via the HiC-spector score to compare normalization method robustness.

## When to use

Use when comparing how robustly normalization methods preserve inter-replicate contact map reproducibility as sequencing depth is reduced, or when needing reduction-rate curves of a reproducibility metric across sampling ratios.

## When NOT to use

- Input is not a Hi-C or capture Hi-C fragment-pair coverage table (covNorm's input requires frag1/frag2/cov/freq/dist columns); raw BAM/FASTQ must be preprocessed first
- The comparison targets inter-chromosomal interactions — the article filters inter-chromosomal interactions and focuses on cis-interactions only
- Full-depth replicate data are available for all methods, so no depth-robustness question exists — a single-shot reproducibility comparison suffices

## Inputs

- Hi-C cis-interaction feature table (frag1, frag2, cov_frag1, cov_frag2, freq, dist) such as GM19204.chr17.cis.feature.gz
- Reference replicate contact map (e.g., GM19240 chr17) for reproducibility scoring
- Sampling ratios (15%, 20%, 25%, 30%, 35%, 40%, 45%, 50%)
- Coverage cut-off threshold (200 at 40 kb resolution)

## Outputs

- Normalized subsample contact maps (one per downsampling ratio per method)
- HiC-spector reproducibility scores per subsampling ratio
- subsample_reproducibility_reduction.csv containing each ratio's score and reduction rate relative to the full-dataset score

## How to apply

Start from a cis-interaction feature table (e.g., GM19204.chr17.cis.feature.gz with frag1, frag2, cov_frag1, cov_frag2, freq, dist columns at 40 kb resolution) and generate random subsamples retaining 15%, 20%, 25%, 30%, 35%, 40%, 45%, and 50% of fragment pairs. Preprocess each subsample by removing zero-valued DNA fragment pairs and filtering fragments with coverage below the 200 threshold at 40 kb resolution. Normalize each subsample with covNormRpkg::normCoverage, which fits the negative binomial GLM log(u) = beta0 + beta1*C_frag1 + beta2*C_frag2 and returns coverage-bias-removed residuals. Score each normalized subsample against the reference replicate's contact map (e.g., GM19240 chr17) using HiC-spector's run_reproducibility_v2.py with num_evec = 20. Finally compute each subsample's score reduction rate relative to the full (100%) dataset score and record all ratios and scores in a single CSV for cross-method comparison.

## Related tools

- **covNorm** (Normalizes each downsampled subsample via negative binomial GLM-based coverage-bias removal (normCoverage)) — https://github.com/kaistcbfg/covNormRpkg
- **HiC-spector** (Computes the HiC-spector reproducibility score between normalized subsample maps and the reference map via run_reproducibility_v2.py (get_reproducibility, num_evec=20)) — https://github.com/gersteinlab/HiC-spector
- **R** (Runtime environment for covNormRpkg subsampling, filtering, and normalization steps) — https://github.com/kaistcbfg/covNormRpkg
- **iced** (Provides ICE and SCN baseline normalization methods for comparison against covNorm in the benchmark) — https://github.com/hiclib/iced

## Examples

```
cov_result <- covNormRpkg::normCoverage(raw_data_filter); python run_reproducibility_v2.py covNorm_subsample_30pct.txt GM19240.chr17.txt --num_evec 20
```

## Evaluation signals

- subsample_reproducibility_reduction.csv exists and contains one reduction-rate row for each of the 8 subsampling ratios (15%-50% in 5% steps)
- Every reported reduction rate lies within [0, 1]
- Each subsample was filtered to remove zero-valued fragment pairs and fragments with coverage below 200 at 40 kb resolution before normalization
- HiC-spector scores decline monotonically (or near-monotonically) with decreasing sampling ratio for all methods, matching the expected behavior of all methods' scores declining as sampling ratio decreases
- A correct covNorm run yields the highest average HiC-spector score at all sampling ratios and the lowest reduction in reproducibility

## Limitations

- The article reports that none of the examined methods showed significantly higher HiC-spector scores than the others (paired t-test, p-values > 0.05), so robustness differences may not be statistically significant
- The coverage cut-off threshold of 200 at 40 kb resolution is a default; it is user-definable, and the choice affects filtering of subsamples
- No version number is stated for covNorm, and no changelog is available for the software repository, complicating exact reproduction

## Evidence
