---
name: capture-hic-significant-interaction-calling
description: Use when you have promoter-other (P-O) pcHi-C fragment-pair tables with
  coverage values, interaction frequencies and distances, and need reproducible significant
  cis-interaction calls within 15 kb–2 Mb.
when_to_use_negative:
- Input is already a normalized contact map or feature table with significance calls
  — the skill operates on raw fragment-pair frequency tables with per-fragment coverage
  values
- Analyzing inter-chromosomal (trans) interactions — the article explicitly filters
  inter-chromosomal interactions and focuses on cis only
- Non-promoter-capture Hi-C data without promoter annotation, since the P-O split
  and asymmetric coverage thresholds (promoter 200 vs 'other' 50) depend on labeling
  frag1 as the promoter site
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: covNorm
  role: Performs negative binomial coverage normalization (normCoverage), distance-dependent
    background removal (normDistance), and Weibull-based significant interaction calling
    (contactPval)
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: CHiCAGO
  role: Baseline significant interaction caller for pcHi-C; its 'score' field defines
    the significance threshold used to compute ROC AUC for covNorm calls
- name: R
  role: Implementation language for covNorm package and for phyper-based overlap significance
    testing
- name: BEDTools coverage
  role: Computes per-bin DNA fragment coverage values (cov_frag1/cov_frag2) used as
    covNorm input
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:e5e8f28ba4cc37e50abf18eaa3bf53947f625bdbb3bfce12fc1796f999d816f5
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
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# capture-hic-significant-interaction-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calls significant promoter-centered long-range interactions from pcHi-C data using covNorm's two-stage normalization (negative binomial coverage normalization followed by distance-dependent background removal) with Weibull-based significance scoring, benchmarkable against CHiCAGO as a baseline.

## When to use

Use when you have promoter-other (P-O) pcHi-C fragment-pair tables with coverage values, interaction frequencies and distances, and need reproducible significant cis-interaction calls within 15 kb–2 Mb.

## When NOT to use

- Input is already a normalized contact map or feature table with significance calls — the skill operates on raw fragment-pair frequency tables with per-fragment coverage values
- Analyzing inter-chromosomal (trans) interactions — the article explicitly filters inter-chromosomal interactions and focuses on cis only
- Non-promoter-capture Hi-C data without promoter annotation, since the P-O split and asymmetric coverage thresholds (promoter 200 vs 'other' 50) depend on labeling frag1 as the promoter site

## Inputs

- Preprocessed pcHi-C P-O interaction table (tab-delimited .gz data.frame: frag1, frag2, cov_frag1, cov_frag2, freq, dist)
- Coverage values per genomic bin/fragment (computed with bedtools coverage)
- Same pcHi-C input formatted for CHiCAGO to serve as baseline

## Outputs

- Significant P-O cis-interaction table with added normalization and significance columns (p-value/FDR from Weibull fit)
- ROC AUC of covNorm significant calls versus CHiCAGO baseline and between replicates (e.g. pochic_rocauc_results.csv)
- Hypergeometric overlap p-values (phyper) for reproducibility between biological replicates

## How to apply

Load the preprocessed promoter-other interaction tables (e.g. GM12878.po.no0.feature.gz, GM19240.po.no0.feature.gz) with frag1/frag2, cov_frag1, cov_frag2, freq and dist columns, having removed zero-valued DNA fragment pairs, self-ligation reads (<15 kb), inter-chromosomal pairs, and PCR duplicates (Picard MarkDuplicates). Restrict to promoter-centered cis-interactions over 15 kb and within 2 Mb, then run covNormRpkg::normCoverage with do_shuffle=FALSE, cov1_thresh=200 (promoter side) and cov2_thresh=50 ('other' side). Sequentially run normDistance (max_dist=2000000) to remove the genomic distance-dependent background, then contactPval to fit normalized interaction frequencies to the three-parameter Weibull distribution and compute p-values with FDR correction. For benchmarking, run CHiCAGO on the same pcHi-C input, take its 'score' field as the significance threshold for P-O cis-interactions within 2 Mb, and quantify agreement of covNorm calls with the CHiCAGO baseline and between biological replicates via ROC AUC plus hypergeometric overlap tests (phyper).

## Related tools

- **covNorm** (Performs negative binomial coverage normalization (normCoverage), distance-dependent background removal (normDistance), and Weibull-based significant interaction calling (contactPval)) — https://github.com/kaistcbfg/covNormRpkg
- **CHiCAGO** (Baseline significant interaction caller for pcHi-C; its 'score' field defines the significance threshold used to compute ROC AUC for covNorm calls)
- **R** (Implementation language for covNorm package and for phyper-based overlap significance testing)
- **BEDTools coverage** (Computes per-bin DNA fragment coverage values (cov_frag1/cov_frag2) used as covNorm input) — https://github.com/arq5x/bedtools2

## Examples

```
cov_result <- covNormRpkg::normCoverage(raw_data_filter, do_shuffle=FALSE, cov1_thresh=200, cov2_thresh=50); final_df <- covNormRpkg::contactPval(dist_df, 'fit.pdf')
```

## Evaluation signals

- ROC AUC of covNorm significant P-O cis-interactions against the CHiCAGO baseline is about 85% (Fig. 4D in the paper)
- Hypergeometric overlap p-values between replicate significant-interaction sets (GM12878 vs GM19240) are ~0, mirroring the paper's reproducibility benchmark
- Output retains the input schema plus added normalization/significance columns; all retained interactions are cis, promoter-centered, and within 15 kb–2 Mb (dist < 2000000)
- Fragment pairs with zero values were removed before normalization; coverage filtering applied asymmetrically (cov1_thresh=200 promoter, cov2_thresh=50 other)
- QC plots (checkFreqCovPCC, checkFreqDistPCC, fit.pdf) are generated and support coverage and distance normalization quality

## Limitations

- Coverage thresholds are resolution-dependent: the default 200 cut-off was selected at 40 kb resolution, and users must adjust for other resolutions
- Zero-valued DNA fragment pairs must be removed beforehand, which limits application to sparse data with many zero-count pairs
- Coverage values must be precomputed externally with bedtools coverage; covNorm does not align reads itself

## Evidence

- [readme] raw_data_filter <- raw_data_filter[which(raw_data_filter$dist<2000000),] # Consider 15kb-2Mb interactions only in pcHi-C: "raw_data_filter <- raw_data_filter[which(raw_data_filter$dist<2000000),] # Consider 15kb-2Mb interactions only in pcHi-C"
- [readme] cov_result <- covNormRpkg::normCoverage(raw_data_filter, do_shuffle=FALSE, cov1_thresh=200, cov2_thresh=50): "cov_result <- covNormRpkg::normCoverage(raw_data_filter, do_shuffle=FALSE, cov1_thresh=200, cov2_thresh=50)"
- [methods] The provided example used 200 as a promoter side threshold but a value of 50 was used to filter the “other” side coverage.: "The provided example used 200 as a promoter side threshold but a value of 50 was used to filter the “other” side coverage."
- [methods] Putative self-ligation reads were removed by deleting ligated reads within a distance shorter than 15 kb.: "Putative self-ligation reads were removed by deleting ligated reads within a distance shorter than 15 kb."
- [results] Venn diagrams showing the fraction of reproducible significant interactions between GM12878 and GM19240 pcHi-C data (Hypergeometric p-values ~0): "Venn diagrams showing the fraction of reproducible significant interactions between GM12878 and GM19240 pcHi-C data (Hypergeometric p-values ~0)"
- [abstract] removal of genomic distance-dependent background and calling of the significant interactions can be applied sequentially: "removal of genomic distance-dependent background and calling of the significant interactions can be applied sequentially"
