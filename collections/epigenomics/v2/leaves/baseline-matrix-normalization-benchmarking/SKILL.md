---
name: baseline-matrix-normalization-benchmarking
description: Use when a new Hi-C or capture Hi-C normalization method must be evaluated
  against established baselines (ICE, SCN, KR, raw) using replicate contact map reproducibility
  rather than a single qualitative comparison.
when_to_use_negative:
- When only a single Hi-C sample/replicate is available — the reproducibility benchmark
  requires a biological replicate pair (e.g., GM19204 vs GM19240).
- When the goal is compartment A/B calling or significant interaction detection rather
  than normalization-method benchmarking — those need different evaluation metrics.
- When contact maps are already normalized or produced by matrix balancing; re-running
  this benchmark on balanced maps would not evaluate the normalization step.
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: covNorm
  role: Candidate negative binomial GLM-based implicit normalization method being
    benchmarked
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: iced
  role: Python package used to run ICE and SCN baseline normalizations on the same
    contact maps
  repo: https://github.com/hiclib/iced
- name: HiC-spector
  role: Provides run_reproducibility_v2.py / get_reproducibility (num_evec = 20) used
    to score replicate map reproducibility for every method
  repo: https://github.com/gersteinlab/HiC-spector
- name: HiCcompare
  role: R package whose KRnorm function provides the Knight-Ruiz baseline normalization
- name: R
  role: Environment in which covNorm and MASS::glm.nb fitting are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:af267489070a588fc4eaf7747ccd6724599d5e570b1ddfe4dedc7a7f990261d5
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
  - HiCcompare
  - R
  - covNorm
  - iced
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# baseline-matrix-normalization-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a candidate Hi-C contact map normalization method (e.g., covNorm) against baseline matrix-balancing methods (ICE, SCN, KR) and raw data by scoring replicate reproducibility with the HiC-spector reproducibility metric, and report per-method median scores with paired t-tests.

## When to use

Use when a new Hi-C or capture Hi-C normalization method must be evaluated against established baselines (ICE, SCN, KR, raw) using replicate contact map reproducibility rather than a single qualitative comparison.

## When NOT to use

- When only a single Hi-C sample/replicate is available — the reproducibility benchmark requires a biological replicate pair (e.g., GM19204 vs GM19240).
- When the goal is compartment A/B calling or significant interaction detection rather than normalization-method benchmarking — those need different evaluation metrics.
- When contact maps are already normalized or produced by matrix balancing; re-running this benchmark on balanced maps would not evaluate the normalization step.

## Inputs

- Preprocessed chr17 cis-interaction contact maps of GM19204 and GM19240 at 40 kb resolution in covNorm format (frag1, frag2, cov_frag1, cov_frag2, freq, dist)
- Raw (unnormalized) contact maps for baseline comparison

## Outputs

- Per-method HiC-spector reproducibility scores for covNorm, ICE, KR, SCN, and raw
- Median HiC-spector score per normalization method (e.g., hic_spector_median_scores.csv)
- Paired t-test p-values comparing score sets across methods

## How to apply

Load preprocessed cis-interaction contact maps (e.g., chr17 at 40 kb resolution) in the covNorm input format (frag1, frag2, cov_frag1, cov_frag2, freq, dist). Normalize each replicate with the candidate method — for covNorm, remove zero-valued fragment pairs, filter fragments with coverage below the default 200 cut-off at 40 kb resolution, shuffle 50% of coverage values, fit the negative binomial GLM log(u_ij) = b0 + b1*Ci + b2*Cj via MASS::glm.nb to remove coverage bias, then remove distance-dependent background with pseudocount avg(R_ij). Run ICE and SCN on the same maps with the Python iced package and KR with HiCcompare::KRnorm. For each method (including raw), apply the HiC-spector run_reproducibility_v2.py get_reproducibility function with num_evec = 20 (default) to the replicate pair, collect per-method scores, compute median HiC-spector scores, and compare methods with paired t-tests. The rationale is that replicate reproducibility (GM19204 vs GM19240) is an unbiased, normalization-independent quality metric for implicit normalization.

## Related tools

- **covNorm** (Candidate negative binomial GLM-based implicit normalization method being benchmarked) — https://github.com/kaistcbfg/covNormRpkg
- **iced** (Python package used to run ICE and SCN baseline normalizations on the same contact maps) — https://github.com/hiclib/iced
- **HiC-spector** (Provides run_reproducibility_v2.py / get_reproducibility (num_evec = 20) used to score replicate map reproducibility for every method) — https://github.com/gersteinlab/HiC-spector
- **HiCcompare** (R package whose KRnorm function provides the Knight-Ruiz baseline normalization)
- **R** (Environment in which covNorm and MASS::glm.nb fitting are executed)

## Examples

```
python run_reproducibility_v2.py covNorm_GM19204_chr17.txt covNorm_GM19240_chr17.txt
```

## Evaluation signals

- The output CSV contains HiC-spector scores for all five methods (covNorm, ICE, KR, SCN, raw).
- covNorm achieves the highest (or comparable-best) median HiC-spector score across the replicate pair.
- Paired t-tests between normalization methods yield p-values > 0.05, while covNorm vs raw is significant (p-value = 0.019 reported in the article).
- All methods were applied to the identical preprocessed contact map input so score differences are attributable to normalization only.
- num_evec = 20 (default) was used consistently for every method's get_reproducibility call.

## Limitations

- None of the examined normalization methods showed significantly higher HiC-spector scores than the others (paired t-test, p-values > 0.05); only covNorm vs raw was significant — so HiC-spector score alone may not discriminate well-tuned normalizers.
- The covNorm coverage cut-off default of 200 applies at 40 kb resolution; other resolutions require a user-defined threshold.
- The benchmark used specific lymphoblastoid cell line data (GM19204, GM19240) on chr17; generalization to other cell types, chromosomes, or derivative Hi-C protocols requires re-validation.

## Evidence

- [results] While none of the examined methods showed significantly high HiC-spector scores than the other methods (paired t-test, p-values > 0.05), covNorm recorded the highest median value.: "While none of the examined methods showed significantly high HiC-spector scores than the other methods (paired t-test, p-values > 0.05), covNorm recorded the highest median value."
- [results] The HiC-spector score sets of covNorm significantly changed against the raw data's score set (paired t-test, p-value = 0.019): [evidence span withheld — anchor status: not_found]
- [methods] The 'run_reproducibility_v2.py' script was downloaded from the software repository (https://github.com/gersteinlab/HiC-spector): "The “run_reproducibility_v2.py” script was downloaded from the software repository (https://github.com/gersteinlab/HiC-spector)"
- [methods] Python 'iced' package was used to run ICE and SCN normalization: "Python “iced” package (https://github.com/hiclib/iced) was used to run ICE and SCN normalization"
- [readme] A script is provided in the tool straw for reading the headers (including chr id and the available resolutions) in .hic file.: [evidence span withheld — anchor status: external_doc]
