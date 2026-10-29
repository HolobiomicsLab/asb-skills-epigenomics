---
name: chromatin-contact-significance-calling
description: Use when normalized Hi-C or capture Hi-C fragment-pair data are available
  and you need per-contact significance scores, reproducibility fractions between
  biological replicates, or a ROC AUC comparison against Fit-Hi-C.
when_to_use_negative:
- Input is raw FASTQ or unnormalized BAM files — coverage computation (BEDTools coverage)
  and normalization must precede significance calling
- Analysis targets promoter-centered pcHi-C interactions with different coverage thresholds
  (promoter 200, other 50) rather than symmetric Hi-C fragment coverages
- Contact maps are needed only for reproducibility scoring (HiC-spector) or compartment
  A/B correlation, not for per-contact significance calls
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3366
tools:
- name: covNorm
  role: R package performing negative binomial coverage normalization, Weibull distance-background
    removal, and per-contact significance scoring
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: Fit-Hi-C
  role: Baseline method whose q-values at 40 kb resolution and 2 Mb distance parameter
    provide comparison significance calls
- name: R
  role: Runtime environment; uses MASS::glm.nb, propagate::fitDistr, FAdist::pweibull3,
    and p.adjust('fdr') within the pipeline
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: BEDTools coverage
  role: Computes 40 kb resolution bin coverage profiles that feed the cov_frag1/cov_frag2
    columns of the input table
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:ba5c53e6a72378b05501fb1c9ab6bee48038530fed9c0c3672e77c26caa119df
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
  - Fit-Hi-C
  - R
  - covNorm
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# chromatin-contact-significance-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A workflow for calling significant long-range chromatin contacts from normalized Hi-C data using covNorm's Weibull background p-values with FDR correction, and benchmarking significance calls against Fit-Hi-C by replicate overlap and ROC AUC (~0.85).

## When to use

Use when normalized Hi-C or capture Hi-C fragment-pair data are available and you need per-contact significance scores, reproducibility fractions between biological replicates, or a ROC AUC comparison against Fit-Hi-C.

## When NOT to use

- Input is raw FASTQ or unnormalized BAM files — coverage computation (BEDTools coverage) and normalization must precede significance calling
- Analysis targets promoter-centered pcHi-C interactions with different coverage thresholds (promoter 200, other 50) rather than symmetric Hi-C fragment coverages
- Contact maps are needed only for reproducibility scoring (HiC-spector) or compartment A/B correlation, not for per-contact significance calls

## Inputs

- Chromosome-level Hi-C cis feature table (.feature.gz): frag1, frag2, cov_frag1, cov_frag2, freq, dist at 40 kb resolution (e.g. GM19204.chr17.cis.feature.gz, GM19240.chr17.cis.feature.gz)
- 40 kb bin coverage profiles computed with BEDTools coverage from hg38-aligned, duplicate-filtered BAM files
- Fit-Hi-C q-value output as baseline significance calls

## Outputs

- Per-contact p-values and FDR-adjusted significance calls from covNormRpkg::contactPval
- Distance-normalized fragment-pair table with significance scores
- Reproducible-interaction overlap fractions between biological replicates
- Per-method ROC AUC values for discriminating reproducible long-range contacts

## How to apply

Load chromosome-level .feature.gz tables (columns frag1, frag2, cov_frag1, cov_frag2, freq, dist), remove zero-valued DNA fragment pairs, and filter fragments with coverage below the default cut-off of 200 at 40 kb resolution, shuffling 50% of coverage values to avoid sorted-order bias. Normalize coverages with covNormRpkg::normCoverage (negative binomial GLM via MASS::glm.nb), then remove the distance-dependent background with normDistance (max_dist=2000000) by fitting a three-parameter Weibull (location, shape, scale) with propagate::fitDistr on normalized values within 2-fold of expectation. Call per-contact p-values with FAdist::pweibull3 and convert to FDR with p.adjust('fdr'); run Fit-Hi-C in parallel as a baseline at 40 kb resolution with a 2 Mb distance parameter and use its q-value as the significance threshold. Label interactions reproducibly called across the two replicates (e.g. GM19204 vs GM19240 chr17) and compute ROC AUC of each method's significance scores for discriminating reproducible long-range contacts; accept when reproduced AUCs match the paper's ~0.85 and reproducibility fractions exceed 65% with <1% difference between methods.

## Related tools

- **covNorm** (R package performing negative binomial coverage normalization, Weibull distance-background removal, and per-contact significance scoring) — https://github.com/kaistcbfg/covNormRpkg
- **Fit-Hi-C** (Baseline method whose q-values at 40 kb resolution and 2 Mb distance parameter provide comparison significance calls)
- **R** (Runtime environment; uses MASS::glm.nb, propagate::fitDistr, FAdist::pweibull3, and p.adjust('fdr') within the pipeline) — https://github.com/kaistcbfg/covNormRpkg
- **BEDTools coverage** (Computes 40 kb resolution bin coverage profiles that feed the cov_frag1/cov_frag2 columns of the input table) — https://github.com/arq5x/bedtools2

## Examples

```
cov_result <- covNormRpkg::normCoverage(raw_data_filter); dist_result <- covNormRpkg::normDistance(cov_result$result_df, max_dist=2000000); final_df <- covNormRpkg::contactPval(dist_result$result_df, 'fit.pdf')
```

## Evaluation signals

- Over 65% of significant long-range chromatin contacts are reproducible between Hi-C biological replicates for both covNorm and Fit-Hi-C
- Per-method ROC AUC values are approximately 0.85, with <1% difference between covNorm and Fit-Hi-C
- QC plots (checkFreqCovPCC, checkFreqDistPCC) show the expected coverage-frequency and distance-frequency correlations after each normalization step
- Weibull fit diagnostics ('fit.pdf' from contactPval) show a valid three-parameter background fit on normalized values within 2-fold of expectation
- Filtered input retains only fragments with coverage ≥ 200 (default cut-off at 40 kb resolution) and no zero-valued fragment pairs

## Limitations

- No version number stated for covNorm beyond the 1.1.0 tarball in the README installation example; software is copyright-registered with no explicit open license terms stated
- Distance-background removal and significance calling applied with max_dist=2000000 (2 Mb); interactions beyond this range are not scored
- Coverage cut-off of 200 at 40 kb resolution is a default that the user must redefine for other resolutions or protocols
- 50% coverage-value shuffling is applied to prevent sorted-order bias, adding stochasticity that should be considered when comparing runs

## Evidence

- [abstract] removal of genomic distance-dependent background and calling of the significant interactions can be applied sequentially: "removal of genomic distance-dependent background and calling of the significant interactions can be applied sequentially"
- [methods] default coverage cut-off threshold: "The default coverage cut-off threshold was selected as 200 at 40 kb resolution, but the user can define this threshold"
- [methods] 50% of coverage values shuffled: "50% of the fragment 1 and 2′s coverage values were shuffled to prevent potential biases caused by the sorted order of the input data"
- [results] Both methods over 65% reproducible, ROC AUC within 1%: [evidence span withheld — anchor status: not_found]
- [readme] contactPval step calls significant interactions: [evidence span withheld — anchor status: not_found]
