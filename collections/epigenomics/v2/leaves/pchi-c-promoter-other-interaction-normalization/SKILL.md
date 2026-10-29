---
name: pchi-c-promoter-other-interaction-normalization
description: Use when normalizing pcHi-C replicates into promoter-centered long-range
  P-O cis-interaction calls, or when benchmarking covNorm's significant interaction
  calls against a CHiCAGO baseline via ROC AUC and hypergeometric overlap.
when_to_use_negative:
- Input is regular in situ Hi-C (not promoter-capture) — use the standard covNorm
  cis-interaction workflow without the P-O promoter/other threshold split (cov1_thresh=200,
  cov2_thresh=50)
- Interactions of interest are inter-chromosomal — the workflow explicitly filters
  inter-chromosomal interactions and focuses on cis-interactions only
- The goal is contact-map reproducibility scoring (HiC-spector) or compartment A/B
  correlation rather than significant P-O interaction calling
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: covNorm
  role: R package performing negative binomial coverage normalization (normCoverage),
    distance-dependent background removal (normDistance), and significant interaction
    calling (contactPval) for P-O pcHi-C interactions
  repo: https://github.com/kaistcbfg/covNormRpkg
- name: CHiCAGO
  role: Baseline significant-interaction caller for pcHi-C used as the reference against
    which covNorm's P-O calls are scored (ROC AUC ~85%)
- name: BEDTools coverage
  role: Computes per-bin coverage values (cov_frag1, cov_frag2) supplied as covNorm
    input
  repo: https://github.com/arq5x/bedtools2
- name: R
  role: Runtime environment for covNorm and for computing ROC AUC and phyper overlap
    statistics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.041
    title: 'covNorm: An R package for coverage based normalization of Hi-C and capture
      Hi-C data'
  source_content_hash: sha256:d9eed14b92cdfe58e322a785f99c3189e97d96538cb4468705bb53b17559c419
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# pcHi-C-promoter-other-interaction-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies covNorm's PO-interaction normalization to promoter-capture Hi-C (pcHi-C) data — negative binomial coverage normalization with asymmetric promoter/other coverage thresholds, distance-dependent background removal, and Weibull-based significant interaction calling — to detect reproducible promoter-other (P-O) cis-interactions comparable to CHiCAGO.

## When to use

Use when normalizing pcHi-C replicates into promoter-centered long-range P-O cis-interaction calls, or when benchmarking covNorm's significant interaction calls against a CHiCAGO baseline via ROC AUC and hypergeometric overlap.

## When NOT to use

- Input is regular in situ Hi-C (not promoter-capture) — use the standard covNorm cis-interaction workflow without the P-O promoter/other threshold split (cov1_thresh=200, cov2_thresh=50)
- Interactions of interest are inter-chromosomal — the workflow explicitly filters inter-chromosomal interactions and focuses on cis-interactions only
- The goal is contact-map reproducibility scoring (HiC-spector) or compartment A/B correlation rather than significant P-O interaction calling

## Inputs

- Preprocessed pcHi-C promoter-other interaction feature table (.feature.gz) with columns frag1, frag2, cov_frag1, cov_frag2, freq, dist (e.g. GM12878.po.no0.feature.gz, GM19240.po.no0.feature.gz)
- pcHi-C raw data (FASTQ from SRA) plus hg38 alignment, PCR-duplicate removal, and bedtools coverage for fragment coverage computation (if preprocessing from scratch)
- CHiCAGO output with 'score' field for baseline significant P-O cis-interactions

## Outputs

- Coverage-normalized P-O interaction data.frame (normCoverage result_df)
- Distance-normalized P-O interaction data.frame (normDistance result_df)
- Significant P-O cis-interaction calls with p-values/FDR from the Weibull fit (contactPval final_df)
- ROC AUC of covNorm vs CHiCAGO agreement and between replicates (pochic_rocauc_results.csv) with hypergeometric overlap p-values (phyper)

## How to apply

Load preprocessed promoter-other interaction tables (e.g. GM12878.po.no0.feature.gz, GM19240.po.no0.feature.gz) as an R data.frame with columns frag1/frag2, cov_frag1, cov_frag2, freq, and dist, where frag1 is the promoter site. Run filterInputDF, then restrict to promoter-centered long-range interactions over 15 kb and within 2 Mb (dist<2000000) and apply normCoverage with do_shuffle=FALSE, cov1_thresh=200 (promoter side), and cov2_thresh=50 (other side), since the 'other' side requires a lower coverage threshold in PO-interaction normalization. Sequentially apply normDistance (max_dist=2000000) to remove genomic distance-dependent background, then contactPval to fit normalized interaction frequencies to a three-parameter Weibull distribution and compute p-values/FDR for significant interactions. For benchmarking, run CHiCAGO on the same pcHi-C input, use its 'score' field as the baseline significance threshold, compute ROC AUC of covNorm calls against the CHiCAGO baseline and across biological replicates within 2 Mb, and assess overlap significance with the R phyper function (paper reports hypergeometric p-values ~0 and ROC AUC ~85% for both methods, Fig. 4D).

## Related tools

- **covNorm** (R package performing negative binomial coverage normalization (normCoverage), distance-dependent background removal (normDistance), and significant interaction calling (contactPval) for P-O pcHi-C interactions) — https://github.com/kaistcbfg/covNormRpkg
- **CHiCAGO** (Baseline significant-interaction caller for pcHi-C used as the reference against which covNorm's P-O calls are scored (ROC AUC ~85%))
- **BEDTools coverage** (Computes per-bin coverage values (cov_frag1, cov_frag2) supplied as covNorm input) — https://github.com/arq5x/bedtools2
- **R** (Runtime environment for covNorm and for computing ROC AUC and phyper overlap statistics)

## Examples

```
cov_result <- covNormRpkg::normCoverage(raw_data_filter, do_shuffle=FALSE, cov1_thresh=200, cov2_thresh=50)
```

## Evaluation signals

- ROC AUC of covNorm-derived significant P-O interactions against the CHiCAGO baseline (and between biological replicates) is ~0.85, matching Fig. 4D of the paper
- Hypergeometric overlap p-values (phyper) between covNorm and CHiCAGO significant interactions within 2 Mb are ~0, mirroring the paper's reproducibility benchmark
- Input data.frame matches the required schema (frag1, frag2, cov_frag1, cov_frag2, freq, dist) with frag1 as the promoter site and self-ligation interactions (<15 kb) removed
- QC plots from checkFreqCovPCC/plotCovNormRes and checkFreqDistPCC/plotDistNormRes show successful coverage and distance normalization before p-value calling
- Zero-valued DNA fragment pairs are removed from the input prior to normalization, as done in the demonstrated study

## Limitations

- P-O normalization uses an asymmetric coverage threshold split (promoter side 200, other side 50); applying the default 200 threshold uniformly to both sides is inappropriate for pcHi-C
- do_shuffle=FALSE is used for PO-interaction normalization; the default shuffling behavior (50% of coverage values) would otherwise be applied to prevent order-related biases
- The coverage threshold of 200 is a default at 40 kb resolution and is user-adjustable; coverage values depend on the binning/fragment definition (midpoint between start and end used for pcHi-C distance)
- No version number for covNorm is stated beyond the README's covNormRpkg_1.1.0 tarball; no license terms beyond copyright registration are given

## Evidence
