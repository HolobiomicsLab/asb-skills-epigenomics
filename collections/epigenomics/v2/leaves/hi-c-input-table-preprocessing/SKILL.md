---
name: hi-c-input-table-preprocessing
description: Use when raw Hi-C or pcHi-C feature tables (e.g. GM19204.chr17.cis.feature.gz
  with columns frag1, frag2, cov_frag1, cov_frag2, freq, dist) must be cleaned before
  normalization or reproducibility scoring.
when_to_use_negative:
- The input is already a filtered/normalized covNorm result table (e.g. output of
  normCoverage), not a raw feature table with freq and coverage columns
- The analysis targets inter-chromosomal (trans) interactions, which the pipeline
  explicitly filters out because it focuses on cis-interactions only
- The data are already at lower resolution or from a protocol where the 40 kb bin
  and 200 coverage cut-off do not apply and a different threshold is needed without
  adaptation
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: covNorm
  role: R package whose filterInputDF function removes zero-valued DNA fragment pairs
    and applies the default 200 coverage cut-off at 40 kb resolution
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: R
  role: Runtime environment in which the covNormRpkg filtering and normalization functions
    are executed on the input data.frame
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: BEDTools coverage
  role: Computes the 40 kb-resolution bin coverage profiles (cov_frag1, cov_frag2)
    used by the filtering step
  repo: https://github.com/arq5x/bedtools2
- name: Picard MarkDuplicates
  role: Upstream removal of PCR duplicates from the filtered BAM before coverage profiles
    and fragment pairs are computed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:f5a3accaa2a453d5ef4608517a055b8f61eba38f5a11e78ccc35a2ee8d2e2bf1
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
  - Picard MarkDuplicates
  - R
  - covNorm
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# hi-c-input-table-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess Hi-C/capture Hi-C input feature tables (frag1, frag2, cov_frag1, cov_frag2, freq, dist) by removing zero-valued DNA fragment pairs and filtering fragments below a coverage cut-off (default 200 at 40 kb resolution) so that covNorm normalization operates on a clean, comparable cis-interaction table.

## When to use

Use when raw Hi-C or pcHi-C feature tables (e.g. GM19204.chr17.cis.feature.gz with columns frag1, frag2, cov_frag1, cov_frag2, freq, dist) must be cleaned before normalization or reproducibility scoring.

## When NOT to use

- The input is already a filtered/normalized covNorm result table (e.g. output of normCoverage), not a raw feature table with freq and coverage columns
- The analysis targets inter-chromosomal (trans) interactions, which the pipeline explicitly filters out because it focuses on cis-interactions only
- The data are already at lower resolution or from a protocol where the 40 kb bin and 200 coverage cut-off do not apply and a different threshold is needed without adaptation

## Inputs

- gzipped Hi-C cis-interaction feature table (e.g. GM19204.chr17.cis.feature.gz, 783588 rows; GM19240.chr17.cis.feature.gz, 1136435 rows) with columns frag1, frag2, cov_frag1, cov_frag2, freq, dist
- pcHi-C PO-interaction feature table (e.g. GM12878.po.no0.feature.gz) with promoter-side and other-side coverage columns
- 40 kb-resolution coverage profiles computed with BEDTools coverage on filtered BAM files

## Outputs

- Filtered data.frame (raw_data_filter) with zero-valued fragment pairs removed and fragments below the 200 coverage cut-off at 40 kb resolution excluded
- Coverage-normalized downstream table produced by covNormRpkg::normCoverage (result_df with residuals from the negative binomial GLM)

## How to apply

Build the fragment-pair table at 40 kb resolution using BEDTools coverage, then run covNormRpkg::filterInputDF on the raw data.frame, which removes zero-valued DNA fragment pairs and applies a coverage cut-off threshold (default 200 at 40 kb resolution, user-definable). For pcHi-C PO-interactions, use asymmetric thresholds (promoter side 200, other side 50) and restrict dist to 15 kb–2 Mb. Upstream read-level filters that should already be applied when generating the table include MAPQ < 10 removal, deletion of putative self-ligation reads within 15 kb, PCR duplicate removal with Picard MarkDuplicates, and filtering of inter-chromosomal interactions since only cis-interactions are analyzed. The rationale is that zero-coverage pairs carry no normalization signal and low-coverage fragments destabilize the negative binomial GLM fitted by covNormRpkg::normCoverage (log(u) = beta0 + beta1*C_frag1 + beta2*C_frag2).

## Related tools

- **covNorm** (R package whose filterInputDF function removes zero-valued DNA fragment pairs and applies the default 200 coverage cut-off at 40 kb resolution) — https://github.com/kaistcbfg/covNormRpkg
- **R** (Runtime environment in which the covNormRpkg filtering and normalization functions are executed on the input data.frame) — https://github.com/kaistcbfg/covNormRpkg
- **BEDTools coverage** (Computes the 40 kb-resolution bin coverage profiles (cov_frag1, cov_frag2) used by the filtering step) — https://github.com/arq5x/bedtools2
- **Picard MarkDuplicates** (Upstream removal of PCR duplicates from the filtered BAM before coverage profiles and fragment pairs are computed)

## Examples

```
raw_data <- read.table(gzfile('GM19204.chr17.cis.feature.gz'), head=TRUE); raw_data_filter <- covNormRpkg::filterInputDF(raw_data)
```

## Evaluation signals

- Output data.frame contains no rows where cov_frag1 or cov_frag2 is zero
- All retained fragments have coverage values above the 200 cut-off at 40 kb resolution (or the user-specified threshold; 200/50 for promoter/other sides in pcHi-C)
- All remaining fragment pairs are cis-interactions (same chromosome in frag1 and frag2) and, for pcHi-C, have dist between 15 kb and 2 Mb
- filterInputDF completes without error and the row count decreases relative to the raw table (e.g. from 783588 rows for GM19204.chr17.cis.feature.gz)
- For downsampling reproducibility tasks, each subsample undergoes the same filtering before covNormRpkg::normCoverage so HiC-spector scores are comparable across 15%-50% sampling ratios

## Limitations

- The default coverage cut-off of 200 at 40 kb resolution is a heuristic; the article notes the user can define this threshold, and results are sensitive to its choice
- Only cis-interactions are retained; inter-chromosomal interactions are deliberately filtered, so trans analyses are out of scope
- In pcHi-C PO-interaction normalization, interactions are restricted to 15 kb–2 Mb, excluding longer-range promoter contacts

## Evidence
