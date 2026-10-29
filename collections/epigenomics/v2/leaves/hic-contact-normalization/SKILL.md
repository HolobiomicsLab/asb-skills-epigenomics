---
name: hic-contact-normalization
description: Use when raw Hi-C or capture Hi-C fragment-pair tables (frag1, frag2,
  cov_frag1, cov_frag2, freq, dist) need bias removal, distance-background normalization,
  or significance calling of cis long-range contacts.
when_to_use_negative:
- Input is already a normalized contact map or .hic matrix rather than raw fragment-pair
  coverage/frequency tables — use map-balancing methods like ICE/SCN (iced) or HiC-spector
  instead
- Analysis targets inter-chromosomal (trans) interactions — the method filters to
  cis-interactions only
- Promoter-capture Hi-C PO-interactions with unbalanced promoter/other coverage —
  use the dedicated normCoverage cov1_thresh=200, cov2_thresh=50 variant, not the
  symmetric Hi-C default
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: covNorm
  role: Core R package performing negative binomial coverage normalization, distance-dependent
    background removal, and significant interaction calling
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: R
  role: Runtime environment; covNorm is implemented as an R package using MASS::glm.nb,
    propagate::fitDistr, and FAdist::pweibull3
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: BEDTools coverage
  role: Computes 40 kb resolution coverage profiles of individual chromosomes used
    as cov_frag1/cov_frag2 inputs
  repo: https://github.com/arq5x/bedtools2
- name: HiC-spector
  role: Computes reproducibility scores between normalized Hi-C contact maps for benchmarking
    (run_reproducibility_v2.py)
  repo: https://github.com/gersteinlab/HiC-spector
- name: iced
  role: Python package used to run ICE and SCN normalization as baseline normalization
    methods for comparison
  repo: https://github.com/hiclib/iced
- name: Fit-Hi-C
  role: Baseline significant-contact caller run at 40 kb resolution with 2 Mb distance
    parameter; q-values compared against covNorm significance calls
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:12042fc01c0f73656ba8c8f6a71bcfe92b3cd61b0c3187afeb5a0ca728472c39
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
  - HiC-spector
  - R
  - covNorm
  - iced
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# hic-contact-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Coverage-based implicit normalization of Hi-C and capture Hi-C data using covNorm: a negative binomial GLM removes fragment-coverage biases, a three-parameter Weibull fit removes the distance-dependent background, and per-contact p-values (FDR-adjusted) call significant long-range chromatin contacts. Benchmarking against Fit-Hi-C/ICE/SCN uses HiC-spector scores and reproducibility of significant contacts between biological replicates.

## When to use

Use when raw Hi-C or capture Hi-C fragment-pair tables (frag1, frag2, cov_frag1, cov_frag2, freq, dist) need bias removal, distance-background normalization, or significance calling of cis long-range contacts.

## When NOT to use

- Input is already a normalized contact map or .hic matrix rather than raw fragment-pair coverage/frequency tables — use map-balancing methods like ICE/SCN (iced) or HiC-spector instead
- Analysis targets inter-chromosomal (trans) interactions — the method filters to cis-interactions only
- Promoter-capture Hi-C PO-interactions with unbalanced promoter/other coverage — use the dedicated normCoverage cov1_thresh=200, cov2_thresh=50 variant, not the symmetric Hi-C default

## Inputs

- Hi-C/pcHi-C fragment-pair data.frame with columns frag1, frag2, cov_frag1, cov_frag2, freq, dist (e.g. GM19204.chr17.cis.feature.gz, GM19240.chr17.cis.feature.gz)
- 40 kb resolution bin coverage profiles computed with BEDTools coverage
- Preprocessed BAM files aligned to hg38 with BWA-mem, PCR-duplicate-removed via Picard MarkDuplicates

## Outputs

- Coverage-normalized fragment table (result_df from normCoverage)
- Distance-normalized table with removed genomic distance-dependent background (result_df from normDistance)
- Significant interaction calls with per-contact p-values and FDR q-values (final_df from contactPval)
- QC plots: coverage/distance PCC checks, coverage heatmap, distance hexmap, Weibull fit diagnostic (fit.pdf)

## How to apply

Format input as a data.frame with columns frag1, frag2, cov_frag1, cov_frag2, freq, dist at 40 kb resolution (bin coverage computed with BEDTools coverage). Run covNormRpkg::filterInputDF to remove zero-valued fragment pairs and low-coverage fragments (default coverage cut-off 200 at 40 kb; shuffle 50% of coverage values with do_shuffle to avoid sorted-order bias). Normalize coverage via normCoverage, which fits a negative binomial GLM with MASS::glm.nb, then normDistance (max_dist=2000000) to remove the distance-dependent background by fitting a three-parameter Weibull (location, shape, scale) with propagate::fitDistr on values within 2-fold of expectation. Call significant interactions with contactPval, which computes per-contact p-values with FAdist::pweibull3 and FDR with p.adjust ('fdr' method). Validate by measuring overlap of significant contacts between biological replicates (e.g. GM19204 vs GM19240 chromosome 17) and ROC AUC of significance scores for discriminating reproducible interactions, compared against a Fit-Hi-C baseline run at 40 kb resolution with a 2 Mb distance parameter.

## Related tools

- **covNorm** (Core R package performing negative binomial coverage normalization, distance-dependent background removal, and significant interaction calling) — https://github.com/kaistcbfg/covNormRpkg
- **R** (Runtime environment; covNorm is implemented as an R package using MASS::glm.nb, propagate::fitDistr, and FAdist::pweibull3) — https://github.com/kaistcbfg/covNormRpkg
- **BEDTools coverage** (Computes 40 kb resolution coverage profiles of individual chromosomes used as cov_frag1/cov_frag2 inputs) — https://github.com/arq5x/bedtools2
- **HiC-spector** (Computes reproducibility scores between normalized Hi-C contact maps for benchmarking (run_reproducibility_v2.py)) — https://github.com/gersteinlab/HiC-spector
- **iced** (Python package used to run ICE and SCN normalization as baseline normalization methods for comparison) — https://github.com/hiclib/iced
- **Fit-Hi-C** (Baseline significant-contact caller run at 40 kb resolution with 2 Mb distance parameter; q-values compared against covNorm significance calls)

## Examples

```
raw_data <- read.table(gzfile("GM19204.chr17.cis.feature.gz"),head=TRUE); cov_result <- covNormRpkg::normCoverage(covNormRpkg::filterInputDF(raw_data)); dist_df <- covNormRpkg::normDistance(cov_result$result_df, max_dist=2000000)$result_df; final_df <- covNormRpkg::contactPval(dist_df,'fit.pdf')
```

## Evaluation signals

- Over 65% of significant long-range chromatin contacts should be reproducible between biological replicates (GM19204 vs GM19240) for both covNorm and Fit-Hi-C
- ROC AUC of each method's significance scores for discriminating reproducible interactions should be approximately 0.85, with covNorm and Fit-Hi-C AUCs differing by less than 1%
- HiC-spector scores of covNorm-normalized maps should change significantly against raw data's scores (paired t-test p-value = 0.019 in the paper)
- QC check: checkFreqCovPCC and checkFreqDistPCC plots should show expected frequency-coverage and frequency-distance correlations after each normalization stage
- Default coverage cut-off of 200 at 40 kb resolution applied and zero-valued DNA fragment pairs removed before normalization

## Limitations

- No changelog available for the software repository and no explicit version numbering beyond the 1.1.0 tarball name in the README
- No explicit limitations of covNorm are discussed in the discussion section of the article
- Coverage cut-off (default 200 at 40 kb) and Weibull fitting (values within 2-fold of expectation) may need user adjustment for other resolutions or derivative 'C' protocols

## Evidence
