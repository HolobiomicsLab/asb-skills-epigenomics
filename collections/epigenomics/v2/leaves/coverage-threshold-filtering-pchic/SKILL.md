---
name: coverage-threshold-filtering-pchic
description: Use when normalizing pcHi-C promoter-other (P-O) interaction tables with
  covNorm's normCoverage, where promoter-anchored fragments need stricter coverage
  filtering (cov1_thresh=200) than the interacting 'other' fragments (cov2_thresh=50).
when_to_use_negative:
- For whole-genome in situ Hi-C maps where both fragments are non-promoter bins —
  use the default symmetric 200-coverage cutoff instead of the pcHi-C promoter/other
  asymmetric thresholds.
- When the input already contains normalized interaction frequencies rather than raw
  fragment coverage values and ligation frequencies — re-filtering on cov_frag1/cov_frag2
  would not apply.
- For promoter-promoter (P-P) interactions, which covNorm processes and filters separately
  from P-O interactions.
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: covNorm
  role: R package whose normCoverage function applies do_shuffle, cov1_thresh=200
    and cov2_thresh=50 filtering during PO-interaction normalization of pcHi-C data
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: BEDTools coverage
  role: Computes the 40 kb-resolution coverage profiles (cov_frag1/cov_frag2) that
    the thresholds are applied to
  repo: https://github.com/arq5x/bedtools2
- name: R
  role: Environment in which covNorm is implemented and the filtering/normalization
    pipeline is executed
  repo: https://github.com/kaistcbfg/covNormRpkg
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:eb70c8bcdf18a5cc8292e5356014ecfce0961a32ece3d12ef08954cd5f17f44d
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
  - CHiCAGO
  - R
  - covNorm
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# coverage-threshold-filtering-pcHiC

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A covNorm-specific filtering step for promoter-capture Hi-C (pcHi-C) PO-interaction data that removes zero-valued DNA fragment pairs and low-coverage fragments using asymmetric coverage thresholds (200 for the promoter side, 50 for the 'other' side) before negative-binomial coverage normalization.

## When to use

Use when normalizing pcHi-C promoter-other (P-O) interaction tables with covNorm's normCoverage, where promoter-anchored fragments need stricter coverage filtering (cov1_thresh=200) than the interacting 'other' fragments (cov2_thresh=50).

## When NOT to use

- For whole-genome in situ Hi-C maps where both fragments are non-promoter bins — use the default symmetric 200-coverage cutoff instead of the pcHi-C promoter/other asymmetric thresholds.
- When the input already contains normalized interaction frequencies rather than raw fragment coverage values and ligation frequencies — re-filtering on cov_frag1/cov_frag2 would not apply.
- For promoter-promoter (P-P) interactions, which covNorm processes and filters separately from P-O interactions.

## Inputs

- Preprocessed pcHi-C PO-interaction table (data.frame with frag1, frag2, cov_frag1, cov_frag2, freq, dist columns), e.g. GM12878.po.no0.feature.gz or GM19240.po.no0.feature.gz
- 40 kb-resolution DNA fragment coverage profiles computed with bedtools coverage

## Outputs

- Filtered data.frame restricted to 15 kb–2 Mb promoter-centered cis-interactions passing cov1_thresh=200 and cov2_thresh=50
- Coverage-normalized interaction data.frame (result_df from normCoverage) suitable for normDistance and contactPval

## How to apply

Start from a preprocessed PO-interaction data.frame with columns frag1, frag2, cov_frag1, cov_frag2, freq, and dist, where frag1 is the promoter site and zero-valued fragment pairs have already been removed. Restrict to promoter-centered long-range cis-interactions longer than 15 kb (self-ligation removal) and within 2 Mb (filter dist<2000000), and drop inter-chromosomal interactions. Then call normCoverage with do_shuffle=FALSE, cov1_thresh=200 (promoter side), and cov2_thresh=50, since pcHi-C 'other'-side fragments have naturally lower coverage than promoter-baited fragments, justifying the asymmetric thresholds. Interactions failing either threshold are discarded prior to the negative-binomial GLM fitting so that low-coverage fragments do not bias the expected ligation frequency estimates. The filtered, coverage-normalized data.frame then feeds the distance-dependent background removal (normDistance) and significant interaction calling (contactPval).

## Related tools

- **covNorm** (R package whose normCoverage function applies do_shuffle, cov1_thresh=200 and cov2_thresh=50 filtering during PO-interaction normalization of pcHi-C data) — https://github.com/kaistcbfg/covNormRpkg
- **BEDTools coverage** (Computes the 40 kb-resolution coverage profiles (cov_frag1/cov_frag2) that the thresholds are applied to) — https://github.com/arq5x/bedtools2
- **R** (Environment in which covNorm is implemented and the filtering/normalization pipeline is executed) — https://github.com/kaistcbfg/covNormRpkg

## Examples

```
raw_data_filter <- raw_data_filter[which(raw_data_filter$dist<2000000),]; cov_result <- covNormRpkg::normCoverage(raw_data_filter, do_shuffle=FALSE, cov1_thresh=200, cov2_thresh=50)
```

## Evaluation signals

- No rows remain with cov_frag1 < 200 or cov_frag2 < 50 after filtering (promoter-side and other-side coverage thresholds enforced).
- All retained interactions have dist between 15 kb and 2 Mb and are intra-chromosomal (cis only).
- normCoverage QC diagnostics (checkFreqCovPCC, plotCovNormRes) show a reasonable coverage-frequency correlation after normalization.
- Downstream hypergeometric overlap of reproducible significant interactions between replicates yields p-values ~0, as in the paper's GM12878 vs GM19240 pcHi-C benchmark.
- Input tables match the documented format (dot-spliced frag coordinates, raw freq, dist between bin starts with pcHi-C midpoints).

## Limitations

- The default coverage cut-off of 200 was selected for 40 kb resolution; the article notes the user must define thresholds appropriate to their resolution (e.g. 50 for the pcHi-C 'other' side).
- Zero-valued DNA fragment pairs must be removed from the input beforehand, as covNorm's demonstrated input data excludes them.
- 50% of coverage values are normally shuffled to prevent order-related bias, but pcHi-C PO normalization uses do_shuffle=FALSE, so input ordering should be considered.

## Evidence
