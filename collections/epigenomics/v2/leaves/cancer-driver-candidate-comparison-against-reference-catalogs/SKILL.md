---
name: cancer-driver-candidate-comparison-against-reference-catalogs
description: Use when you have a tool-called driver candidate list (e.g. DriverPower
  [outDir]/[name].result.tsv) and need to benchmark its novelty and error rate against
  CGC or PCAWG-consensus catalogs.
when_to_use_negative:
- Input is a raw mutation VCF or an unprocessed BMR model feature table rather than
  a filtered driver candidate list — run the DriverPower model/infer pipeline first
- The goal is method performance comparison (F1 score vs. six published PCAWG methods)
  rather than catalog-overlap FDR estimation
- Your candidate list was called on a different cohort than the 2583-genome PCAWG
  data set, making counts like 11/17 exclusive candidates non-comparable
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_2640
tools:
- name: DriverPower
  role: Generates the coding and non-coding driver candidate result tables being compared
    against the CGC and PCAWG-consensus catalogs
  repo: github.com/smshuai/DriverPower
- name: Python
  role: Intersects candidate lists with reference catalogs, annotates evidence status,
    and computes FDR fractions (requires Python >= 3.5)
- name: bedtools
  role: Set/interval intersection of driver candidate elements against reference catalog
    intervals
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:e3a975baef16d10cfa7e987a8c4e059117dd06fca9728e436d31c16bd7201ebe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DriverPower
  - Python
  - Python >=
  - bedtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Cancer driver candidate comparison against reference catalogs and FDR estimation

## Summary

Compares DriverPower-called driver candidates from the 2583-genome PCAWG cohort against the CGC and PCAWG-consensus driver catalogs to isolate tool-exclusive candidates and estimate the empirical false discovery rate (3.2% coding, 16.8% non-coding).

## When to use

Use when you have a tool-called driver candidate list (e.g. DriverPower [outDir]/[name].result.tsv) and need to benchmark its novelty and error rate against CGC or PCAWG-consensus catalogs.

## When NOT to use

- Input is a raw mutation VCF or an unprocessed BMR model feature table rather than a filtered driver candidate list — run the DriverPower model/infer pipeline first
- The goal is method performance comparison (F1 score vs. six published PCAWG methods) rather than catalog-overlap FDR estimation
- Your candidate list was called on a different cohort than the 2583-genome PCAWG data set, making counts like 11/17 exclusive candidates non-comparable

## Inputs

- DriverPower result TSV files of the form [outDir]/[name].result.tsv (217 coding and 95 non-coding candidates)
- CGC (Cancer Gene Census) driver catalog
- PCAWG-consensus driver catalog
- Literature/orthogonal evidence annotations for exclusive candidates

## Outputs

- driverpower_exclusive_candidates_fdr.tsv: exclusive-candidate table with evidence-status annotations
- Per-category FDR estimates: 3.2% (7/217) coding and 16.8% (16/95) non-coding

## How to apply

Load the DriverPower result tables ([outDir]/[name].result.tsv) containing the 217 coding and 95 non-coding candidates called across 2583 PCAWG cancer genomes. Intersect candidate gene/element identifiers with the CGC and PCAWG-consensus catalogs to extract the 11 coding and 17 non-coding candidates exclusive to DriverPower. Annotate each exclusive candidate by evidence status: assuming candidates lacking literature or orthogonal evidence are false positives, only 4/11 coding and 4/17 non-coding exclusive candidates had support. Compute FDR as the fraction of total calls that are unsupported exclusive candidates, yielding 7/217 (3.2%) for coding and 16/95 (16.8%) for non-coding regions. The rationale is that uncontested calls are the most likely artefacts (e.g. variant-calling errors or biological processes not captured by the BMR model), so their proportion bounds the method's error rate.

## Related tools

- **DriverPower** (Generates the coding and non-coding driver candidate result tables being compared against the CGC and PCAWG-consensus catalogs) — github.com/smshuai/DriverPower
- **Python** (Intersects candidate lists with reference catalogs, annotates evidence status, and computes FDR fractions (requires Python >= 3.5))
- **bedtools** (Set/interval intersection of driver candidate elements against reference catalog intervals)

## Examples

```
bedtools intersect -a driverpower.result.tsv -b cgc_catalog.bed pcawg_consensus.bed -v > driverpower_exclusive_candidates.bed
```

## Evaluation signals

- Exclusive-candidate counts must equal exactly 11 coding and 17 non-coding candidates after intersection with CGC and PCAWG-consensus catalogs
- Computed FDR values must reproduce the reported 3.2% (7/217) for coding and 16.8% (16/95) for non-coding candidates
- Total candidate counts in the loaded result tables must match 217 coding and 95 non-coding calls
- Evidence-status annotation must mark 4/11 coding and 4/17 non-coding exclusive candidates as supported
- The output TSV must contain the evidence-status column and reproduce both FDR fractions on recomputation

## Limitations

- The FDR estimate is a lower/conditional bound: it assumes all exclusive candidates lacking literature or orthogonal evidence are false positives
- False-positive driver calls may arise from variant-calling artefacts or biological processes not captured by the BMR model (e.g. regional AID activity); additional BMR features are proposed but not yet incorporated
- Many tumour types in the 2583-genome PCAWG cohort have fewer than 30 samples, limiting statistical power and potentially inflating exclusive-candidate counts

## Evidence

- [abstract] Testing across a collection of 2583 cancer genomes from the PCAWG project, DriverPower identifies 217 coding and 95 non-coding driver candidates.: "Testing across a collection of 2583 cancer genomes from the PCAWG project, DriverPower identifies 217 coding and 95 non-coding driver candidates."
- [discussion] False positives arise from artefacts or uncaptured biology: "False-positive driver calls may be caused by technical errors such as variant-calling artefacts that artificially increase the local mutation rate, or by biological processes that are not captured by"
- [readme] DriverPower discovers coding and non-coding driver elements from somatic mutations: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
- [discussion] DriverPower balance of precision and recall: "DriverPower provides the best balance of precision and recall, although is not always the top-ranked method when either metric is considered independently"
