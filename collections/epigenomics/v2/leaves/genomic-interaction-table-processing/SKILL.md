---
name: genomic-interaction-table-processing
description: Use when input is a tab-delimited Hi-C or capture Hi-C fragment-pair
  table (frag1, frag2, cov_frag1, cov_frag2, freq, dist) and the goal is to remove
  coverage-driven ligation bias from raw interaction frequencies before distance normalization
  or contact calling.
when_to_use_negative:
- When the goal is genomic distance-dependent background removal or significance calling
  — use covNormRpkg::normDistance and contactPval on the coverage-normalized table
  instead.
- For pcHi-C PO-interaction data with promoter-side/other-side coverages, the default
  single-threshold filtering is inappropriate; use cov1_thresh=200 and cov2_thresh=50
  with do_shuffle=FALSE.
- When input rows have zero-valued fragment pairs or unfiltered low-coverage fragments
  without first running filterInputDF — feeding raw unfiltered tables to normCoverage
  violates the demonstrated pipeline.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: covNorm
  role: R package providing normCoverage, filterInputDF, and the negative binomial
    GLM-based implicit coverage normalization
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: R
  role: Runtime environment in which covNormRpkg (with MASS::glm.nb for model fitting)
    is executed
  repo: https://github.com/kaistcbfg/covNormRpkg
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:c3eea516cbb9bf6da854dfcbb34aa79b6b4fa9a438f1a60f70512c78ee422ad5
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
  - R
  - covNorm
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# genomic-interaction-table-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Coverage normalization of Hi-C/capture Hi-C fragment-pair interaction tables using the covNormRpkg normCoverage function, which fits a negative binomial GLM (via MASS::glm.nb) on fragment coverages to produce bias-removed interaction frequencies.

## When to use

Use when input is a tab-delimited Hi-C or capture Hi-C fragment-pair table (frag1, frag2, cov_frag1, cov_frag2, freq, dist) and the goal is to remove coverage-driven ligation bias from raw interaction frequencies before distance normalization or contact calling.

## When NOT to use

- When the goal is genomic distance-dependent background removal or significance calling — use covNormRpkg::normDistance and contactPval on the coverage-normalized table instead.
- For pcHi-C PO-interaction data with promoter-side/other-side coverages, the default single-threshold filtering is inappropriate; use cov1_thresh=200 and cov2_thresh=50 with do_shuffle=FALSE.
- When input rows have zero-valued fragment pairs or unfiltered low-coverage fragments without first running filterInputDF — feeding raw unfiltered tables to normCoverage violates the demonstrated pipeline.

## Inputs

- Tab-delimited gzipped fragment-pair table with columns frag1, frag2, cov_frag1, cov_frag2, freq, dist (covNorm input format, e.g. GM19204.chr17.cis.feature.gz)
- Bin coverage values computed with bedtools coverageBed at 40 kb resolution

## Outputs

- Coverage-normalized interaction frequency table (result_df written via write.table with gzfile), including the capture_res column
- Fitted GLM bias coefficients (cov_result$coeff_cov1, cov_result$coeff_cov2)

## How to apply

Load the sample input (e.g. GM19204.chr17.cis.feature.gz from http://junglab.kaist.ac.kr/Dataset/) as a data.frame, then run covNormRpkg::filterInputDF to remove zero-valued fragment pairs and filter low-coverage fragments using the default coverage cut-off of 200 at 40 kb resolution. Next call covNormRpkg::normCoverage on the filtered data frame; it internally shuffles 50% of fragment 1 and 2 coverage values to prevent biases from the sorted input order, then fits the negative binomial GLM log(uij) = beta0 + beta1*Ci + beta2*Cj with MASS::glm.nb. The function returns an object whose coeff_cov1/coeff_cov2 hold the fitted bias coefficients and whose result_df contains the bias-removed residuals Rij = Yij / exp(beta0_hat + beta1_hat*Ci + beta2_hat*Cj); write result_df out as the coverage normalization result. For pcHi-C PO-interaction normalization, restrict dist to <2 Mb and call normCoverage with do_shuffle=FALSE, cov1_thresh=200, cov2_thresh=50. Validation: each output table should contain the capture_res column with coverage-normalized interaction frequencies equal to raw freq divided by the fitted GLM expectation within numerical tolerance.

## Related tools

- **covNorm** (R package providing normCoverage, filterInputDF, and the negative binomial GLM-based implicit coverage normalization) — https://github.com/kaistcbfg/covNormRpkg
- **R** (Runtime environment in which covNormRpkg (with MASS::glm.nb for model fitting) is executed) — https://github.com/kaistcbfg/covNormRpkg

## Examples

```
cov_result <- covNormRpkg::normCoverage(raw_data_filter); cov_df <- cov_result$result_df; write.table(cov_df, file=gzfile("outFileName1"), row.names=FALSE, sep="\t", quote=FALSE)
```

## Evaluation signals

- A normalized output table exists for each input sample and contains the capture_res field populated with coverage-normalized interaction frequencies
- Normalized frequencies equal raw freq divided by the fitted GLM expectation exp(beta0_hat + beta1_hat*Ci + beta2_hat*Cj) within numerical tolerance
- filterInputDF removed zero-valued fragment pairs and low-coverage fragments below the default coverage cut-off of 200 at 40 kb resolution
- QC plots (checkFreqCovPCC output coverage-vs-capture_res PCC) show reduced coverage-frequency correlation after normalization

## Limitations

- 50% of coverage values are shuffled before fitting to prevent biases caused by the sorted order of the input data; skipping shuffling (or disabling it, as in pcHi-C PO normalization with do_shuffle=FALSE) changes the fit
- Default coverage cut-off of 200 at 40 kb resolution may need user-defined adjustment; for pcHi-C the 'other' side uses a lower threshold of 50
- No version number or changelog is stated for covNormRpkg in the article or README

## Evidence

- [readme] README pipeline step writing coverage-normalized result: [evidence span withheld — anchor status: not_found]
