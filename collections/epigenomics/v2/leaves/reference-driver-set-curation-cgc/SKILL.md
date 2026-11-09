---
name: reference-driver-set-curation-cgc
description: Use when comparing driver discovery methods (e.g., DriverPower vs ActiveDriverWGS,
  ExInAtor, LARVA, MutSig, ncdDetect, oncodriveFML) on the same PCAWG cohorts and
  you need CGC-anchored precision/recall/F1 metrics.
when_to_use_negative:
- Input is a single method's raw driver call table with no comparator methods or CGC
  reference set available — no precision/recall benchmark can be computed.
- The goal is de novo driver discovery on a new cohort rather than evaluating/benchmarking
  existing driver calls against a gold standard.
- Tumour cohorts outside the non-melanoma/lymphoma PCAWG set used in the benchmark,
  where the reported F1 baselines (0.81 coding, 0.91 splice site, 0.79 non-coding)
  do not apply as validation anchors.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
tools:
- name: DriverPower
  role: Source of the evaluated coding and non-coding driver candidate calls ([outDir]/[name].result.tsv
    from driverpower infer)
  repo: https://github.com/smshuai/DriverPower
- name: bedtools
  role: Genomic interval intersection of each method's called elements with CGC/PCAWG
    reference sets to define true positives
- name: Python
  role: Computation of precision, recall, and F1 scores per method per element class
    and writing the comparison table (requires Python >= 3.5)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:9734fbae5956aeb83aa5c779a8180258c0f890c5158d271fa416725d70a7915b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python
  - bedtools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Reference driver set curation against CGC gold standard

## Summary

Benchmark cancer driver discovery calls against the Cancer Gene Census (CGC) gold-standard reference set using bedtools interval intersection to compute precision, recall, and F1 per method and element class (coding, splice site, non-coding).

## When to use

Use when comparing driver discovery methods (e.g., DriverPower vs ActiveDriverWGS, ExInAtor, LARVA, MutSig, ncdDetect, oncodriveFML) on the same PCAWG cohorts and you need CGC-anchored precision/recall/F1 metrics.

## When NOT to use

- Input is a single method's raw driver call table with no comparator methods or CGC reference set available — no precision/recall benchmark can be computed.
- The goal is de novo driver discovery on a new cohort rather than evaluating/benchmarking existing driver calls against a gold standard.
- Tumour cohorts outside the non-melanoma/lymphoma PCAWG set used in the benchmark, where the reported F1 baselines (0.81 coding, 0.91 splice site, 0.79 non-coding) do not apply as validation anchors.

## Inputs

- DriverPower per-cohort driver calls: [outDir]/[name].result.tsv from the driverpower infer sub-command (26 non-melanoma/lymphoma PCAWG cohorts)
- Driver calls from six comparator methods: ActiveDriverWGS, ExInAtor, LARVA, MutSig, ncdDetect, oncodriveFML (PCAWG Drivers and Functional Interpretation Working Group)
- CGC gold-standard reference element set (plus optional PCAWG-consensus/raw reference sets)
- Genomic interval files (BED) of called elements per method

## Outputs

- True-positive element sets per method (bedtools intersection with CGC)
- Per-method precision, recall, and F1 score per element class (coding, splice site, non-coding)
- F1 comparison table: f1_scores_comparison.tsv

## How to apply

Load each method's driver calls — e.g., DriverPower per-cohort result tables ('[outDir]/[name].result.tsv' produced by the driverpower infer sub-command) for the 26 non-melanoma/lymphoma PCAWG cohorts, plus calls from ActiveDriverWGS, ExInAtor, LARVA, MutSig, ncdDetect, and oncodriveFML as used by the PCAWG Drivers and Functional Interpretation Working Group. Intersect each method's called elements with CGC (and optionally PCAWG-consensus/raw) reference sets using bedtools genomic interval intersection to define true positives. Compute precision, recall, and F1 per method per element class in Python, and tabulate all F1 scores (e.g., f1_scores_comparison.tsv). Validate that computed DriverPower F1 values reproduce the reported benchmarks — 0.81 coding (precision 0.84, recall 0.79), 0.91 splice site (vs ncdDetect 0.65 and oncodriveFML 0.32), and 0.79 non-coding — within tolerance, and that DriverPower ranks highest overall.

## Related tools

- **DriverPower** (Source of the evaluated coding and non-coding driver candidate calls ([outDir]/[name].result.tsv from driverpower infer)) — https://github.com/smshuai/DriverPower
- **bedtools** (Genomic interval intersection of each method's called elements with CGC/PCAWG reference sets to define true positives)
- **Python** (Computation of precision, recall, and F1 scores per method per element class and writing the comparison table (requires Python >= 3.5))

## Examples

```
bedtools intersect -u -a driverpower_calls.bed -b cgc_elements.bed > driverpower_true_positives.bed
```

## Evaluation signals

- Computed DriverPower coding F1 matches the reported 0.81 (precision 0.84, recall 0.79) within an acceptable tolerance.
- Computed splice-site F1 matches 0.91, with comparator values near ncdDetect 0.65 and oncodriveFML 0.32.
- Computed non-coding F1 matches 0.79 within tolerance.
- DriverPower ranks highest in F1 across all six comparator methods for both coding and non-coding driver discovery.
- f1_scores_comparison.tsv is produced with one row per method per element class and no missing comparator entries.

## Limitations

- DriverPower provides the best balance of precision and recall but is not always the top-ranked method when either metric is considered independently.
- CGC-based validation is biased toward well-known coding drivers; functional impact score schemes are currently biased toward coding mutations, limiting non-coding benchmark sensitivity.
- False-positive driver calls may arise from variant-calling artefacts or biological processes not captured by the background mutation model, affecting precision estimates.

## Evidence

- [abstract] DriverPower achieved the highest F1 score for coding driver discovery: "Comparing to six published methods used by the PCAWG Drivers and Functional Interpretation Working Group, DriverPower has the highest F1 score for both coding and non-coding driver discovery."
- [discussion] DriverPower best precision/recall balance: "DriverPower provides the best balance of precision and recall, although is not always the top-ranked method when either metric is considered independently"
- [methods] DriverPower result output used for benchmarking: "The ``infer`` sub-command is used to call drivers from test data with pre-trained models."
- [readme] DriverPower tool scope: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
