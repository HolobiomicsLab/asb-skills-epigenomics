---
name: chromatin-contact-table-processing
description: Use when Hi-C or capture Hi-C fragment-pair tables have already been
  coverage-normalized (bias-removed residuals with freq and dist columns) and the
  goal is to remove the distance-dependent decay background to obtain normalized interaction
  frequencies or call significant interactions.
when_to_use_negative:
- Input is raw interaction data that has not yet passed the negative binomial coverage
  normalization (normCoverage) step — distance normalization assumes bias-removed
  residuals Rij
- Working with pcHi-C PO-interactions restricted to the 15 kb–2 Mb promoter-centered
  window without the modified normCoverage call (do_shuffle=FALSE, cov1_thresh=200,
  cov2_thresh=50) done first
- Only inter-chromosomal contacts are of interest — the covNorm workflow filters to
  cis-interactions and normalizes against a genomic-distance background that only
  applies intra-chromosomally
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: covNorm
  role: Provides the normDistance function that performs the distance-dependent background
    normalization on coverage-normalized fragment-pair tables
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: R
  role: Language in which the covNormRpkg package is implemented and executed (devtools::install_github
    / R CMD INSTALL)
- name: bedtools
  role: coverageBed computes the bin coverage values (cov_frag1, cov_frag2) in the
    input fragment-pair table
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:d389d3f804fc7bc125ff2fae539f5928766a8a1af6ed76aaf7ec4e0c70a2ca83
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
  - bedtools
  - covNorm
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# chromatin-contact-table-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements covNorm's genomic distance-dependent background normalization step: after coverage normalization, fit expected ligation frequencies as a function of genomic distance and remove that background from fragment-pair interaction frequencies using covNormRpkg's normDistance function.

## When to use

Use when Hi-C or capture Hi-C fragment-pair tables have already been coverage-normalized (bias-removed residuals with freq and dist columns) and the goal is to remove the distance-dependent decay background to obtain normalized interaction frequencies or call significant interactions.

## When NOT to use

- Input is raw interaction data that has not yet passed the negative binomial coverage normalization (normCoverage) step — distance normalization assumes bias-removed residuals Rij
- Working with pcHi-C PO-interactions restricted to the 15 kb–2 Mb promoter-centered window without the modified normCoverage call (do_shuffle=FALSE, cov1_thresh=200, cov2_thresh=50) done first
- Only inter-chromosomal contacts are of interest — the covNorm workflow filters to cis-interactions and normalizes against a genomic-distance background that only applies intra-chromosomally

## Inputs

- Coverage-normalized fragment-pair data.frame from covNormRpkg::normCoverage with columns frag1, frag2, cov_frag1, cov_frag2, freq, dist (e.g. GM19204.chr17.cis.feature.gz / GM19240.chr17.cis.feature.gz after step 3)
- Distance cutoff parameter (max_dist), e.g. 2000000 bp in the example Hi-C workflow

## Outputs

- Result dataframe (dist_result$result_df) with an added dist_res column of distance-normalized interaction frequencies for all fragment pairs within the 2 Mb cutoff
- QC plots from checkFreqDistPCC (QCplot_dist_PCC.pdf) and plotDistNormRes (QCplot_dist_hexmap.pdf)
- Optionally, significant interactions from contactPval written to a tab-delimited gzipped output table

## How to apply

Load the coverage-normalized fragment-pair dataframe (columns frag1, frag2, cov_frag1, cov_frag2, freq, dist) produced upstream by covNormRpkg::normCoverage. Fit the distance model log(uij) = β0 + β1Dij on the residual values to estimate Ed, the expected ligation frequency at genomic distance d. Run covNormRpkg::normDistance with max_dist = 2000000 so only fragment pairs within 2 Mb are normalized against the distance-dependent background, applying the pseudocount-adjusted formula (Rij + avg(Rij)) / (Ed + avg(Rij)) where avg(Rij) is the pseudocount parameter. Extract dist_result$result_df, whose dist_res column holds the distance-normalized interaction frequencies, then optionally pass it to contactPval for significance calling. Validate that dist_res is populated for all fragment pairs within the 2 Mb cutoff and matches the pseudocount-adjusted ratio computed from the fitted Ed.

## Related tools

- **covNorm** (Provides the normDistance function that performs the distance-dependent background normalization on coverage-normalized fragment-pair tables) — https://github.com/kaistcbfg/covNormRpkg
- **R** (Language in which the covNormRpkg package is implemented and executed (devtools::install_github / R CMD INSTALL))
- **bedtools** (coverageBed computes the bin coverage values (cov_frag1, cov_frag2) in the input fragment-pair table) — https://github.com/arq5x/bedtools2

## Examples

```
dist_result <- covNormRpkg::normDistance(cov_df, max_dist=2000000); dist_df <- dist_result$result_df; write.table(dist_df, file=gzfile("outFileName2"), row.names=FALSE, col.names=TRUE, sep="\t", quote=FALSE)
```

## Evaluation signals

- Output dataframe contains a non-empty dist_res column populated for every fragment pair within the max_dist=2000000 cutoff
- dist_res values match the pseudocount-adjusted ratio (Rij + avg(Rij)) / (Ed + avg(Rij)) recomputed from the fitted distance model
- QC diagnostics pass: checkFreqDistPCC and plotDistNormRes render without errors on dist_df
- Downstream contactPval on dist_df completes and produces a final table writable via write.table with row.names=FALSE and tab separation
- Pairs at or beyond the 2 Mb distance cutoff are excluded from the distance-normalized background fitting

## Limitations

- Only fragment pairs within the user-defined max_dist (2 Mb in the example) are normalized against the distance-dependent background
- Putative self-ligation reads closer than 15 kb are removed upstream, and inter-chromosomal interactions are filtered, so the skill applies to cis-interactions only
- No license terms are stated in the article; the software is copyright KAIST and registered at the Korea Copyright Commission, and commercial use requires contacting the authors
- No version number is stated for covNorm in the article's discussion section

## Evidence
