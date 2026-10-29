---
name: r-package-data-workflow
description: Use when raw Hi-C or pcHi-C interaction data is provided as fragment-pair
  feature tables (frag1/frag2 coordinates, coverages, raw frequencies, distances)
  and reproducible, coverage-normalized contact maps are needed.
when_to_use_negative:
- Data are already normalized contact matrices (e.g. .hic format) rather than fragment-pair
  feature tables with coverage values — covNorm's input format is the fragment-pair
  data.frame, not matrices
- Inter-chromosomal (trans) interactions need analysis — the method focuses on cis-interactions
  only and inter-chromosomal interactions are filtered
- Very short-range contacts (under ~15 kb, putative self-ligations) are the target
  — these are removed by filtering
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: covNormRpkg
  role: R package implementing filterInputDF, normCoverage (negative binomial GLM
    coverage normalization), normDistance, contactPval and QC plotting
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: R
  role: Runtime environment; covNorm is distributed and run as an R package using
    MASS::glm.nb for model fitting
- name: BEDTools coverage
  role: Computes 40 kb resolution bin coverage values (cov_frag1, cov_frag2) required
    in the covNorm input format
  repo: https://github.com/arq5x/bedtools2
- name: HiC-spector
  role: Used downstream to score reproducibility (run_reproducibility_v2.py) of covNorm-normalized
    contact maps between replicates
  repo: https://github.com/gersteinlab/HiC-spector
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:5ebcdbe623047fb958b485a69ffedc6f3f1a31c0ef38859e9d53621670b5f29a
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
  - HiC-spector
  - R
  - covNorm
  - covNormRpkg
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# r-package-data-workflow

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

End-to-end Hi-C/capture Hi-C data processing with the covNorm R package, which performs negative binomial model-based coverage normalization, genomic distance-dependent background normalization, and significant interaction calling from fragment-pair feature tables.

## When to use

Use when raw Hi-C or pcHi-C interaction data is provided as fragment-pair feature tables (frag1/frag2 coordinates, coverages, raw frequencies, distances) and reproducible, coverage-normalized contact maps are needed.

## When NOT to use

- Data are already normalized contact matrices (e.g. .hic format) rather than fragment-pair feature tables with coverage values — covNorm's input format is the fragment-pair data.frame, not matrices
- Inter-chromosomal (trans) interactions need analysis — the method focuses on cis-interactions only and inter-chromosomal interactions are filtered
- Very short-range contacts (under ~15 kb, putative self-ligations) are the target — these are removed by filtering

## Inputs

- Tab-delimited gzipped feature table (data.frame in R) with columns frag1, frag2 (dot-spliced chr.start.end), cov_frag1, cov_frag2, freq, dist (e.g. GM19204.chr17.cis.feature.gz, 783588 rows)
- Coverage profiles computed with bedtools coverageBed at 40 kb resolution
- pcHi-C PO-interaction tables with promoter-side (frag1) and other-side (frag2) fragments

## Outputs

- Coverage normalization result table (result_df written as gzipped tab-delimited file) with a capture_res column of coverage-normalized interaction frequencies
- Distance-normalized table (normDistance result_df)
- Final table of significant interactions from contactPval
- QC PDFs: coverage PCC plot, coverage normalization heatmap, distance PCC plot, distance hexmap, fit.pdf

## How to apply

Load the sample feature files (e.g. GM19204.chr17.cis.feature.gz from http://junglab.kaist.ac.kr/Dataset/) as data.frames with columns frag1, frag2, cov_frag1, cov_frag2, freq, dist. Filter with covNormRpkg::filterInputDF, which removes zero-valued fragment pairs and applies coverage cut-off thresholds (default 200 at 40 kb resolution; pcHi-C PO-normalization uses cov1_thresh=200 for the promoter side and cov2_thresh=50 for the 'other' side). Run covNormRpkg::normCoverage, which fits a negative binomial GLM (MASS::glm.nb, log(uij) = β0 + β1*Ci + β2*Cj) after shuffling 50% of coverage values to avoid input-order bias, and returns residuals Rij = Yij / exp(β̂0 + β̂1*Ci + β̂2*Cj) in result_df, written out with a capture_res column holding coverage-normalized interaction frequencies. Optionally continue with normDistance (max_dist=2000000) and contactPval to call significant interactions, using checkFreqCovPCC/plotCovNormRes for QC plots.

## Related tools

- **covNormRpkg** (R package implementing filterInputDF, normCoverage (negative binomial GLM coverage normalization), normDistance, contactPval and QC plotting) — https://github.com/kaistcbfg/covNormRpkg
- **R** (Runtime environment; covNorm is distributed and run as an R package using MASS::glm.nb for model fitting)
- **BEDTools coverage** (Computes 40 kb resolution bin coverage values (cov_frag1, cov_frag2) required in the covNorm input format) — https://github.com/arq5x/bedtools2
- **HiC-spector** (Used downstream to score reproducibility (run_reproducibility_v2.py) of covNorm-normalized contact maps between replicates) — https://github.com/gersteinlab/HiC-spector

## Examples

```
cov_result <- covNormRpkg::normCoverage(raw_data_filter); cov_df <- cov_result$result_df; write.table(cov_df, file=gzfile("outFileName1"), sep="\t", quote=FALSE)
```

## Evaluation signals

- A normalized output table exists for each input sample and contains the capture_res column with coverage-normalized interaction frequencies
- Normalized frequencies equal raw freq divided by the fitted GLM expectation exp(β̂0 + β̂1*Ci + β̂2*Cj) within numerical tolerance
- filterInputDF removed zero-valued fragment pairs and rows with coverage below the threshold (default 200 at 40 kb; pcHi-C 'other' side 50)
- QC plot checkFreqCovPCC shows coverage PCC reduced after normalization (coverage bias removed)
- Downstream reproducibility (HiC-spector score) between GM19204 and GM19240 replicates matches or exceeds raw-data scores

## Limitations

- No version number or license terms stated for covNorm beyond copyright registration at the Korea Copyright Commission (C-2021-022800); commercial use requires contacting the authors
- The default coverage cut-off threshold of 200 was selected for 40 kb resolution and may need user re-definition at other resolutions
- Coverage-value shuffling (50%) is applied by default to prevent sorted-input bias; pcHi-C PO-normalization must disable it with do_shuffle=FALSE
- No changelog is available for the software repository

## Evidence
