---
name: hi-c-coverage-bias-normalization
description: Use when cis-interaction fragment-pair tables (frag1, frag2, cov_frag1,
  cov_frag2, freq, dist) at ~40 kb resolution show coverage-dependent ligation bias
  and you need coverage-normalized interaction frequencies or reproducible replicates.
when_to_use_negative:
- Input lacks per-fragment coverage values (cov_frag1/cov_frag2) — the negative binomial
  model is fit on coverage, so matrix balancing methods (ICE, KR, SCN) are the appropriate
  choice instead
- Data are inter-chromosomal interactions — the study filters these out and demonstrates
  covNorm on cis-interactions only
- Input fragment pairs include zero-valued frequencies or fragments below the coverage
  cut-off that have not been pre-filtered, since these must be removed before model
  fitting
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: covNorm
  role: R package implementing negative binomial model-based coverage normalization
    (filterInputDF, normCoverage, normDistance, contactPval)
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: R
  role: Runtime environment; MASS::glm.nb fits the negative binomial GLM used by normCoverage
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: HiC-spector
  role: run_reproducibility_v2.py / get_reproducibility (num_evec=20) scores replicate
    contact-map reproducibility to evaluate normalization quality
  repo: https://github.com/gersteinlab/HiC-spector
- name: iced
  role: Runs ICE and SCN baseline normalizations for benchmark comparison against
    covNorm
  repo: https://github.com/hiclib/iced
- name: HiCcompare
  role: KRnorm function provides the Knight-Ruiz baseline normalization for benchmark
    comparison
- name: bedtools
  role: coverageBed computes the per-bin coverage values (cov_frag1/cov_frag2) required
    as model input
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:5ab1aed17a7efe0477f4beee0920a7a845788fc0c66349f601201747b4714953
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
  - bedtools
  - covNorm
  - iced
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# hi-c-coverage-bias-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Coverage-based implicit normalization of Hi-C/capture Hi-C interaction tables using covNorm's negative binomial GLM (MASS::glm.nb), which removes coverage bias from raw ligated-fragment frequencies and can be followed by genomic distance-dependent background removal and significance calling.

## When to use

Use when cis-interaction fragment-pair tables (frag1, frag2, cov_frag1, cov_frag2, freq, dist) at ~40 kb resolution show coverage-dependent ligation bias and you need coverage-normalized interaction frequencies or reproducible replicates.

## When NOT to use

- Input lacks per-fragment coverage values (cov_frag1/cov_frag2) — the negative binomial model is fit on coverage, so matrix balancing methods (ICE, KR, SCN) are the appropriate choice instead
- Data are inter-chromosomal interactions — the study filters these out and demonstrates covNorm on cis-interactions only
- Input fragment pairs include zero-valued frequencies or fragments below the coverage cut-off that have not been pre-filtered, since these must be removed before model fitting

## Inputs

- Tabular fragment-pair data.frame in covNorm format: frag1, frag2 (dot-spliced chr.start.end coordinates), cov_frag1, cov_frag2 (bedtools coverage values), freq (raw interaction frequency), dist (genomic distance between start coordinates)
- Sample Hi-C replicate files (e.g. GM19204.chr17.cis.feature.gz, GM19240.chr17.cis.feature.gz from http://junglab.kaist.ac.kr/Dataset/)

## Outputs

- Coverage-normalized result_df/data.table with added capture_res column (coverage-normalized interaction frequency between DNA fragments)
- Fitted negative binomial GLM coverage bias coefficients (coeff_cov1, coeff_cov2)
- QC plots (coverage-PCC plot, coverage-normalization heatmap) and, after downstream steps, distance-normalized tables with significant interaction p-values

## How to apply

Load the fragment-pair data.frame (frag1, frag2, cov_frag1, cov_frag2, freq, dist) — e.g. GM19204.chr17.cis.feature.gz / GM19240.chr17.cis.feature.gz at 40 kb resolution — then run covNormRpkg::filterInputDF to remove zero-valued fragment pairs and filter fragments below the default coverage cut-off of 200 (user-definable; for pcHi-C PO-interactions use cov1_thresh=200, cov2_thresh=50 and keep 15 kb–2 Mb distances). Call covNormRpkg::normCoverage, which by default shuffles 50% of both fragments' coverage values to prevent sorted-order bias, then fits the negative binomial GLM log(uij) = β0 + β1*Ci + β2*Cj via MASS::glm.nb and removes the fitted expectation to produce bias-removed residuals in result_df (capture_res column). Optionally inspect coeff_cov1/coeff_cov2 and generate QC plots (checkFreqCovPCC, plotCovNormRes). For distance normalization, feed the coverage-normalized data.frame into covNormRpkg::normDistance (max_dist=2,000,000) and contactPval for significant interactions.

## Related tools

- **covNorm** (R package implementing negative binomial model-based coverage normalization (filterInputDF, normCoverage, normDistance, contactPval)) — https://github.com/kaistcbfg/covNormRpkg
- **R** (Runtime environment; MASS::glm.nb fits the negative binomial GLM used by normCoverage) — https://github.com/kaistcbfg/covNormRpkg
- **HiC-spector** (run_reproducibility_v2.py / get_reproducibility (num_evec=20) scores replicate contact-map reproducibility to evaluate normalization quality) — https://github.com/gersteinlab/HiC-spector
- **iced** (Runs ICE and SCN baseline normalizations for benchmark comparison against covNorm) — https://github.com/hiclib/iced
- **HiCcompare** (KRnorm function provides the Knight-Ruiz baseline normalization for benchmark comparison)
- **bedtools** (coverageBed computes the per-bin coverage values (cov_frag1/cov_frag2) required as model input) — https://github.com/arq5x/bedtools2

## Examples

```
raw_data_filter <- covNormRpkg::filterInputDF(raw_data); cov_result <- covNormRpkg::normCoverage(raw_data_filter); cov_df <- cov_result$result_df
```

## Evaluation signals

- Each sample yields a normalized output table containing a populated capture_res column of coverage-normalized interaction frequencies
- Normalized frequencies equal raw frequencies divided by the fitted GLM expectation, Rij = Yij / exp(β̂0 + β̂1*Ci + β̂2*Cj), within numerical tolerance
- coeff_cov1 and coeff_cov2 are returned by normCoverage without error, confirming the MASS::glm.nb fit converged
- checkFreqCovPCC QC plot shows reduced dependence of frequency on fragment coverage after normalization
- HiC-spector scores of normalized GM19204 vs GM19240 maps (num_evec=20) exceed the raw-data score set (reported paired t-test p = 0.019)

## Limitations

- On chr17 GM19204/GM19240 replicates, none of covNorm, ICE, KR, or SCN showed significantly higher HiC-spector scores than the others (paired t-test p > 0.05); covNorm only recorded the highest median value and significantly improved over raw data
- Coverage cut-off default of 200 is calibrated for 40 kb resolution and must be user-adjusted at other resolutions; 50% of coverage values are shuffled by default to counteract sorted-order bias (disable with do_shuffle=FALSE for pcHi-C PO-interactions)
- Zero-valued fragment pairs and inter-chromosomal interactions must be removed beforehand; the coverage-normalization step alone does not remove the genomic distance-dependent background — normDistance must be run sequentially

## Evidence

- [results] While none of the examined methods showed significantly high HiC-spector scores than the other methods (paired t-test p-values > 0.05), covNorm recorded the highest median value.: [evidence span withheld — anchor status: not_found]
