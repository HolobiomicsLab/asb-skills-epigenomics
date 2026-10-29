---
name: hi-c-reproducibility-scoring-hic-spector
description: Use when you have cis-interaction contact maps of two biological replicates
  (e.g., 40 kb-resolution chr17 maps) and need to quantify and compare how well normalization
  methods (covNorm, ICE, KR, SCN) improve replicate reproducibility.
when_to_use_negative:
- The contact maps are inter-chromosomal — the underlying study focuses on cis-interactions
  only and filters inter-chromosomal interactions
- You need detection of reproducible significant long-range contacts or compartment
  A/B profiles rather than the spectral eigenvector-based reproducibility score —
  those were measured with separate metrics in the paper
- Only one replicate is available — the HiC-spector score requires two contact maps
  to compare
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3673
tools:
- name: HiC-spector
  role: Provides the run_reproducibility_v2.py script and get_reproducibility function
    (num_evec = 20) used to compute replicate reproducibility scores of the contact
    maps
  repo: github.com/gersteinlab/HiC-spector
- name: covNorm
  role: Negative binomial model-based implicit normalization producing the normalized
    contact maps scored for reproducibility
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: iced
  role: Python package used to run the ICE and SCN baseline normalizations on the
    contact maps
  repo: github.com/hiclib/iced
- name: HiCcompare
  role: R package whose KRnorm function performs the KR baseline normalization
- name: R
  role: Environment in which covNorm and HiCcompare::KRnorm normalization steps are
    executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:cb1efb9c7ac43256d36361ce001aab2822bdd78afdd1fb343edb7fdc6a9f1dc3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kaistcbfg/covNormRpkg
  tools:
  - HiC-spector
  - HiCcompare
  - R
  - covNorm
  - iced
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# hi-c-reproducibility-scoring-hic-spector

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark Hi-C contact map normalization methods by scoring replicate reproducibility (e.g., GM19204 vs GM19240 chr17 maps at 40 kb resolution) with the HiC-spector run_reproducibility_v2.py script, comparing covNorm against ICE, KR, SCN, and raw data.

## When to use

Use when you have cis-interaction contact maps of two biological replicates (e.g., 40 kb-resolution chr17 maps) and need to quantify and compare how well normalization methods (covNorm, ICE, KR, SCN) improve replicate reproducibility.

## When NOT to use

- The contact maps are inter-chromosomal — the underlying study focuses on cis-interactions only and filters inter-chromosomal interactions
- You need detection of reproducible significant long-range contacts or compartment A/B profiles rather than the spectral eigenvector-based reproducibility score — those were measured with separate metrics in the paper
- Only one replicate is available — the HiC-spector score requires two contact maps to compare

## Inputs

- Preprocessed chr17 cis-interaction contact maps of GM19204 and GM19240 replicates at 40 kb resolution in covNorm input format (frag1, frag2, cov_frag1, cov_frag2, freq, dist)
- Raw (unnormalized) replicate contact maps as baseline
- Normalized contact maps from ICE and SCN (Python iced package) and KR (HiCcompare::KRnorm R function)

## Outputs

- Per-method HiC-spector reproducibility scores for the replicate pair
- Median HiC-spector score per normalization method (e.g., hic_spector_median_scores.csv)
- Paired t-test p-values comparing score sets between methods (e.g., covNorm vs raw: p = 0.019; covNorm vs other methods: p > 0.05)

## How to apply

Load preprocessed chr17 cis-interaction contact maps of the GM19204 and GM19240 replicates at 40 kb resolution in the covNorm input format (frag1, frag2, cov_frag1, cov_frag2, freq, dist). Normalize each replicate map with the candidate methods: covNorm (remove zero-valued fragment pairs, filter fragments with coverage below 200, shuffle 50% of coverage values, fit the negative binomial GLM log(u_ij) = b0 + b1*Ci + b2*Cj via MASS::glm.nb, then remove the distance-dependent background with pseudocount avg(R_ij)), ICE and SCN via the Python iced package, and KR via HiCcompare::KRnorm in R; keep the raw maps as a baseline. For each method, run the HiC-spector run_reproducibility_v2.py script on the replicate pair using the get_reproducibility function with num_evec = 20 (default) to obtain per-chromosome reproducibility scores. Collect per-method scores, compute the median HiC-spector score per method (e.g., writing hic_spector_median_scores.csv), and assess statistical differences between methods with paired t-tests. Validation: the output must contain scores for all five methods, and covNorm should achieve the highest (or comparable-best) median score, significantly improving over raw data (paired t-test p-value = 0.019) while differences between normalization methods are non-significant (p-values > 0.05).

## Related tools

- **HiC-spector** (Provides the run_reproducibility_v2.py script and get_reproducibility function (num_evec = 20) used to compute replicate reproducibility scores of the contact maps) — github.com/gersteinlab/HiC-spector
- **covNorm** (Negative binomial model-based implicit normalization producing the normalized contact maps scored for reproducibility) — https://github.com/kaistcbfg/covNormRpkg
- **iced** (Python package used to run the ICE and SCN baseline normalizations on the contact maps) — github.com/hiclib/iced
- **HiCcompare** (R package whose KRnorm function performs the KR baseline normalization)
- **R** (Environment in which covNorm and HiCcompare::KRnorm normalization steps are executed)

## Examples

```
python run_reproducibility.py -F covNorm_GM19204_chr17.txt covNorm_GM19240_chr17.txt  # get_reproducibility, num_evec = 20 (default)
```

## Evaluation signals

- The results table (e.g., hic_spector_median_scores.csv) contains scores for all five methods: covNorm, ICE, KR, SCN, and raw
- covNorm achieves the highest median HiC-spector score across the replicate pair
- covNorm's score set differs significantly from raw data (paired t-test, p-value = 0.019)
- Differences between covNorm and the other normalization methods are non-significant (paired t-test p-values > 0.05)
- The get_reproducibility function is invoked with num_evec = 20 (default) on the same chr17 bin size (40 kb) for every method so scores are comparable

## Limitations

- None of the examined normalization methods showed significantly higher HiC-spector scores than the others (paired t-test, p-values > 0.05), so the score alone cannot declare a single best method
- The benchmark relied on chr17 at 40 kb resolution for the GM19204/GM19240 pair; generalization to other chromosomes, resolutions, or cell types is not established by this result
- The reproducibility score is one of several evaluation axes — compartment A/B correlation and reproducible significant long-range contact detection were measured separately
- The HiC-spector Python reproducibility script requires Julia-alternative dependencies (e.g., straw.py for .hic input) and memory on the order of two human chr1 10 kb maps fitting in a 16 GB laptop

## Evidence

- [results] The HiC-spector score sets of covNorm significantly changed against the raw data's score set (paired t-test, p-value = 0.019): [evidence span withheld — anchor status: not_found]
- [readme] get_reproducibility
  - to calculate the reproducibility metric between 2 HiC contact maps: "get_reproducibility
  - to calculate the reproducibility metric between 2 HiC contact maps"
