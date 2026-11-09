---
name: roc-auc-reproducibility-evaluation
description: Use when comparing two or more Hi-C significance-calling or normalization
  methods on replicate datasets, and you need quantitative reproducibility (replicate
  overlap fractions) and ROC AUC of significance scores.
when_to_use_negative:
- Input is raw FASTQ/BAM Hi-C data that has not yet been processed into binned fragment-pair
  coverage tables (.feature format)
- The goal is map-level reproducibility (HiC-spector score) or compartment A/B profile
  correlation rather than significant-interaction overlap and ROC AUC
- Only a single replicate is available, so reproducible-interaction labels cannot
  be defined across replicates
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: covNorm
  role: 'Primary normalization and significance-calling method: negative binomial
    coverage normalization, distance-dependent background removal, Weibull3 p-values
    and FDR significance calls'
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: Fit-Hi-C
  role: Baseline significance-calling method run at 40 kb resolution with 2 Mb distance
    parameter; q-values used as significance threshold for comparison
- name: HiC-spector
  role: Used for matrix-level reproducibility scoring (run_reproducibility_v2.py)
    in the broader performance evaluation this ROC AUC analysis complements
  repo: https://github.com/gersteinlab/HiC-spector
- name: R packages (MASS, propagate, FAdist, p.adjust)
  role: GLM fitting (MASS::glm.nb), Weibull3 distribution fitting (propagate::fitDistr),
    per-contact p-value computation (FAdist::pweibull3), and FDR correction (p.adjust,
    'fdr' method)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:81fd44f5d421022d0c74552509d05b7751f89c1b270ab84738f6687a716a50aa
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
  - Fit-Hi-C
  - HiC-spector
  - R
  - R packages (MASS, propagate, FAdist, p.adjust)
  - covNorm
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# roc-auc-reproducibility-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that evaluates Hi-C normalization/significance-calling methods (covNorm vs Fit-Hi-C) by the fraction of significant long-range chromatin contacts reproducible across biological replicates and by ROC AUC of significance scores for discriminating reproducible interactions.

## When to use

Use when comparing two or more Hi-C significance-calling or normalization methods on replicate datasets, and you need quantitative reproducibility (replicate overlap fractions) and ROC AUC of significance scores.

## When NOT to use

- Input is raw FASTQ/BAM Hi-C data that has not yet been processed into binned fragment-pair coverage tables (.feature format)
- The goal is map-level reproducibility (HiC-spector score) or compartment A/B profile correlation rather than significant-interaction overlap and ROC AUC
- Only a single replicate is available, so reproducible-interaction labels cannot be defined across replicates

## Inputs

- Replicate Hi-C cis-interaction tables in covNorm .feature.gz format (frag1, frag2, cov_frag1, cov_frag2, freq, dist), e.g. GM19204.chr17.cis.feature.gz and GM19240.chr17.cis.feature.gz
- Fit-Hi-C-compatible 40 kb-resolution contact/input files with 2 Mb distance parameter
- Per-contact significance scores (covNorm FDR-adjusted p-values; Fit-Hi-C q-values) for each replicate

## Outputs

- Per-method fraction of significant long-range chromatin contacts reproducible between Hi-C biological replicates (>65% expected)
- Per-method ROC AUC of significance scores for discriminating reproducibly called interactions (~0.85 expected, methods within 1%)
- Binary labels of interactions reproducibly called across replicates

## How to apply

Load replicate .feature.gz inputs (e.g. GM19204 and GM19240 chromosome 17) formatted as frag1/frag2/cov_frag1/cov_frag2/freq/dist data.frames, remove zero-valued DNA fragment pairs, and filter fragments below the default coverage cut-off of 200 at 40 kb resolution; shuffle 50% of coverage values to avoid sorted-order bias. Normalize each replicate with covNorm (negative binomial GLM via MASS::glm.nb, distance-dependent background removal, Weibull3 background fit with R propagate::fitDistr within 2-fold of expectation), then compute per-contact p-values with FAdist::pweibull3 and FDR via p.adjust('fdr') to obtain significance calls. Run Fit-Hi-C as the baseline at 40 kb resolution with 2 Mb distance parameters, using its q-value output as the significance threshold. Label interactions reproducibly called across the two biological replicates, and compute the ROC AUC of each method's significance scores for discriminating reproducible long-range interactions. Accept the result if reproduced ROC AUCs (~0.85 expected) differ between covNorm and Fit-Hi-C by less than ~1%, consistent with the paper's reported values.

## Related tools

- **covNorm** (Primary normalization and significance-calling method: negative binomial coverage normalization, distance-dependent background removal, Weibull3 p-values and FDR significance calls) — https://github.com/kaistcbfg/covNormRpkg
- **Fit-Hi-C** (Baseline significance-calling method run at 40 kb resolution with 2 Mb distance parameter; q-values used as significance threshold for comparison)
- **HiC-spector** (Used for matrix-level reproducibility scoring (run_reproducibility_v2.py) in the broader performance evaluation this ROC AUC analysis complements) — https://github.com/gersteinlab/HiC-spector
- **R packages (MASS, propagate, FAdist, p.adjust)** (GLM fitting (MASS::glm.nb), Weibull3 distribution fitting (propagate::fitDistr), per-contact p-value computation (FAdist::pweibull3), and FDR correction (p.adjust, 'fdr' method))

## Examples

```
raw_data <- read.table(gzfile("GM19204.chr17.cis.feature.gz"),head=TRUE); raw_data_filter <- covNormRpkg::filterInputDF(raw_data); cov_result <- covNormRpkg::normCoverage(raw_data_filter); dist_df <- covNormRpkg::normDistance(cov_result$result_df, max_dist=2000000)$result_df; final_df <- covNormRpkg::contactPval(dist_df, 'fit.pdf')
```

## Evaluation signals

- Both covNorm and Fit-Hi-C yield >65% of significant long-range chromatin contacts reproducible between Hi-C replicates
- Reproduced ROC AUC values are close to the paper's reported ~0.85 and differ between covNorm and Fit-Hi-C by less than 1%
- Filtering step applied: zero-valued fragment pairs removed and fragments below the 200-coverage cut-off at 40 kb resolution excluded; 50% of coverage values shuffled
- Significance calls defined consistently: covNorm p-values FDR-corrected with p.adjust('fdr'); Fit-Hi-C q-values taken as the significance threshold
- covNorm result changes significantly versus raw data at the map level (paired t-test p-value = 0.019 on HiC-spector scores), confirming the normalization pipeline ran

## Limitations

- Evaluation is grounded on chromosome 17 sample data from two lymphoblastoid cell lines (GM19204, GM19240); conclusions may not generalize to all cell types or chromosomes
- No version number stated for covNorm in the discussion; only copyright registration is mentioned and no explicit license terms are stated
- The coverage cut-off of 200 is a default at 40 kb resolution and must be user-defined for other resolutions

## Evidence

- [readme] HiC-spector reproducibility tooling: [evidence span withheld — anchor status: external_doc]
