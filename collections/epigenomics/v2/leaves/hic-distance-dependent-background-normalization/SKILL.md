---
name: hic-distance-dependent-background-normalization
description: Use when you have coverage-normalized Hi-C fragment-pair tables (bias-removed
  residuals with freq and dist columns) and need to remove the genomic distance-dependent
  background to obtain distance-normalized interaction frequencies.
when_to_use_negative:
- Input is still raw interaction frequency data that has not passed covNormRpkg::normCoverage;
  the distance normalization assumes bias-removed residuals Rij
- Interaction pairs lie beyond the max_dist cutoff (e.g. > 2 Mb) and are not meant
  to be distance-normalized
- Working with inter-chromosomal interactions; the article's pipeline focuses on cis-interactions
  only, as inter-chromosomal interactions were filtered upstream
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: covNorm
  role: R package whose normDistance function performs the genomic distance-dependent
    background normalization step
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: R
  role: Implementation language/environment in which covNormRpkg::normDistance is
    executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:99390fb32c6f44d3f4045f56748b6962a8ff2c06a2d5ef3bc5b42c16fe8ebe9f
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
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# hic-distance-dependent-background-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Performs the genomic distance-dependent background normalization step of the covNorm R package (covNormRpkg::normDistance), which removes the expected distance-decay of Hi-C ligation frequencies after coverage normalization, producing the dist_res column of distance-normalized interaction frequencies.

## When to use

Use when you have coverage-normalized Hi-C fragment-pair tables (bias-removed residuals with freq and dist columns) and need to remove the genomic distance-dependent background to obtain distance-normalized interaction frequencies.

## When NOT to use

- Input is still raw interaction frequency data that has not passed covNormRpkg::normCoverage; the distance normalization assumes bias-removed residuals Rij
- Interaction pairs lie beyond the max_dist cutoff (e.g. > 2 Mb) and are not meant to be distance-normalized
- Working with inter-chromosomal interactions; the article's pipeline focuses on cis-interactions only, as inter-chromosomal interactions were filtered upstream

## Inputs

- Coverage-normalized chr17 fragment-pair data.frame from covNormRpkg::normCoverage (columns: frag1, frag2, cov_frag1, cov_frag2, freq, dist; bias-removed residuals Rij), e.g. GM19204.chr17.cis.feature.gz or GM19240.chr17.cis.feature.gz after normCoverage
- max_dist parameter (example value 2000000, i.e. 2 Mb cutoff)

## Outputs

- Result dataframe with the added dist_res column of distance-normalized interaction frequencies for every fragment pair within the 2 Mb distance cutoff
- Optional QC plots via checkFreqDistPCC and plotDistNormRes (e.g. QCplot_dist_PCC.pdf, QCplot_dist_hexmap.pdf)

## How to apply

After running covNormRpkg::normCov on the filtered fragment-pair table (frag1, frag2, cov_frag1, cov_frag2, freq, dist), take the coverage-normalized residuals Rij as input. Run covNormRpkg::normDistance with max_dist=2000000 so only fragment pairs within 2 Mb are normalized against the distance-dependent background; the function fits the distance model log(uij) = beta0 + beta1*Dij to estimate Ed, the expected ligation frequency at distance d. Compute the background-removed signal with the pseudocount-adjusted formula (Rij + avg(Rij)) / (Ed + avg(Rij)), where avg(Rij) serves as the pseudocount parameter. The returned result_df contains the added dist_res column of distance-normalized frequencies, which can be passed downstream to contactPval for significant interaction calling. Rationale: the distance-decay of ligation frequency reflects genomic proximity rather than true spatial contact strength, so removing Ed enables comparison of long-range contacts at equal distance.

## Related tools

- **covNorm** (R package whose normDistance function performs the genomic distance-dependent background normalization step) — https://github.com/kaistcbfg/covNormRpkg
- **R** (Implementation language/environment in which covNormRpkg::normDistance is executed)

## Examples

```
dist_result <- covNormRpkg::normDistance(cov_df, max_dist=2000000); dist_df <- dist_result$result_df
```

## Evaluation signals

- Output file exists and the dist_res column is non-empty and populated for all fragment pairs within the 2 Mb distance cutoff
- dist_res values match the pseudocount-adjusted ratio (Rij + avg(Rij)) / (Ed + avg(Rij)) computed from the fitted distance model Ed
- Coverage normalization step (normCoverage) completed first, verified by checkFreqCovPCC/plotCovNormRes QC plots before running normDistance
- QC plots checkFreqDistPCC (QCplot_dist_PCC.pdf) and plotDistNormRes (QCplot_dist_hexmap.pdf) generated from dist_df show the expected distance-normalized behavior
- Downstream contactPval succeeds on dist_df and the final written table contains distance-normalized values plus significance scores

## Limitations

- Only fragment pairs within the max_dist cutoff (2 Mb in the example workflow) are normalized against the distance-dependent background; longer-range pairs fall outside the distance normalization
- No version number is stated for covNorm in the paper's discussion, and no changelog is available for the repository
- Zero-valued DNA fragment pairs should be removed from the input before normalization

## Evidence
