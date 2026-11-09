---
name: roc-auc-baseline-comparison
description: Use when a new significant-interaction caller for pcHi-C (or another
  'C' technology) needs to be validated by ROC AUC against an established baseline
  like CHiCAGO, plus replicate-overlap significance testing.
when_to_use_negative:
- Input is raw FASTQ/BAM data rather than preprocessed feature tables — preprocessing
  (BWA-mem alignment to hg38, Picard MarkDuplicates, BEDTools coverage) must be done
  first
- Evaluating contact-map reproducibility (HiC-spector score) or compartment A/B correlation
  rather than significant P-O interaction calling
- Interactions outside the 15 kb–2 Mb promoter-centered cis range, or inter-chromosomal
  interactions, which the paper's workflow filters out
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: covNorm
  role: Normalizes P-O pcHi-C interactions (normCoverage, normDistance) and calls
    significant interactions (contactPval) for ROC AUC evaluation
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: CHiCAGO
  role: Baseline significant-interaction caller for pcHi-C whose 'score' field defines
    the reference for the ROC comparison
- name: R
  role: Implementation environment for covNormRpkg, ROC AUC computation, and phyper
    overlap tests
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:842251e0ac06fc9e46f6438f1ee2ce7403902fe6865055967a6b7ae98209ff3e
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
  - CHiCAGO
  - R
  - covNorm
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# roc-auc-baseline-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark covNorm's significant P-O cis-interaction calls against the CHiCAGO baseline on pcHi-C replicates by computing ROC AUC and hypergeometric overlap statistics, mirroring the paper's reproducibility-based evaluation.

## When to use

Use when a new significant-interaction caller for pcHi-C (or another 'C' technology) needs to be validated by ROC AUC against an established baseline like CHiCAGO, plus replicate-overlap significance testing.

## When NOT to use

- Input is raw FASTQ/BAM data rather than preprocessed feature tables — preprocessing (BWA-mem alignment to hg38, Picard MarkDuplicates, BEDTools coverage) must be done first
- Evaluating contact-map reproducibility (HiC-spector score) or compartment A/B correlation rather than significant P-O interaction calling
- Interactions outside the 15 kb–2 Mb promoter-centered cis range, or inter-chromosomal interactions, which the paper's workflow filters out

## Inputs

- Preprocessed pcHi-C P-O interaction feature tables (.feature.gz, columns: frag1, frag2, cov_frag1, cov_frag2, freq, dist), e.g. GM12878.po.no0.feature.gz and GM19240.po.no0.feature.gz
- CHiCAGO output for the same pcHi-C input (P-O cis-interactions within 2 Mb with 'score' field)

## Outputs

- ROC AUC of covNorm significant P-O interaction calls versus the CHiCAGO baseline (expected ~0.85 per Fig. 4D)
- Hypergeometric overlap p-values (phyper) for reproducible significant interactions between replicates
- Results table, e.g. pochic_rocauc_results.csv

## How to apply

Load preprocessed promoter-other interaction tables (e.g. GM12878.po.no0.feature.gz and GM19240.po.no0.feature.gz with frag1/frag2, coverage, freq, dist columns) into R as covNorm input. Filter to promoter-centered long-range cis-interactions over 15 kb and within 2 Mb, then run covNormRpkg::normCoverage with do_shuffle=FALSE, cov1_thresh=200 (promoter side) and cov2_thresh=50 (other side), followed by normDistance (max_dist=2000000) to remove the genomic distance-dependent background and contactPval to call significant interactions via three-parameter Weibull p-values and 'fdr' correction. Run CHiCAGO on the same input as the baseline, collecting P-O cis-interactions within 2 Mb and using its 'score' field as the significance threshold. Compute the ROC AUC quantifying agreement of covNorm-derived significant P-O interactions with the CHiCAGO baseline (and between the two biological replicates) within 2 Mb, and assess overlap significance with R's phyper. Write the ROC AUC and overlap statistics to a results CSV (e.g. pochic_rocauc_results.csv).

## Related tools

- **covNorm** (Normalizes P-O pcHi-C interactions (normCoverage, normDistance) and calls significant interactions (contactPval) for ROC AUC evaluation) — https://github.com/kaistcbfg/covNormRpkg
- **CHiCAGO** (Baseline significant-interaction caller for pcHi-C whose 'score' field defines the reference for the ROC comparison)
- **R** (Implementation environment for covNormRpkg, ROC AUC computation, and phyper overlap tests)

## Examples

```
cov_result <- covNormRpkg::normCoverage(raw_data_filter, do_shuffle=FALSE, cov1_thresh=200, cov2_thresh=50); dist_result <- covNormRpkg::normDistance(cov_result$result_df, max_dist=2000000); final_df <- covNormRpkg::contactPval(dist_result$result_df, 'fit.pdf')
```

## Evaluation signals

- ROC AUC of covNorm vs CHiCAGO baseline should be approximately 0.85, matching the paper's Fig. 4D result
- Hypergeometric overlap p-values between covNorm and CHiCAGO calls should be ~0 (phyper)
- Coverage normalization coefficients (coeff_cov1, coeff_cov2) from normCoverage should be finite and QC plots (checkFreqCovPCC) should show reasonable PCC
- Significant-interaction p-values from contactPval should follow the fitted three-parameter Weibull distribution with FDR ('fdr' method) correction applied
- All retained interactions must be cis, promoter-centered, and within 15 kb–2 Mb

## Limitations

- The comparison is a reproducibility/agreement benchmark against CHiCAGO, not an absolute truth standard; covNorm is judged acceptable if reproducibility is enhanced or similar to baseline
- Zero-valued DNA fragment pairs were removed from the demonstrated input data, so results depend on that preprocessing step
- The 'other'-side coverage threshold of 50 is specific to pcHi-C P-O normalization; the default 200 coverage cut-off applies at 40 kb resolution for Hi-C
- No explicit version number or changelog is available for covNorm (v1.1.0 referenced only via R CMD INSTALL example)

## Evidence

- [results] For pcHi-C significant interaction calling, the ROC AUC of both covNorm and CHiCAGO was about 85% (Fig. 4D).: [evidence span withheld — anchor status: not_found]
