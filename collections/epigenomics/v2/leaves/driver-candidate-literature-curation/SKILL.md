---
name: driver-candidate-literature-curation
description: Use when a driver-discovery result table contains candidates absent from
  CGC and PCAWG-consensus catalogs and you need to triage them via literature/orthogonal
  evidence and compute an empirical FDR.
when_to_use_negative:
- Input candidates have already been intersected with CGC and PCAWG-consensus catalogs,
  leaving no exclusive calls to curate
- The goal is a statistical FDR via q-values or permutations rather than an evidence-based
  estimate assuming unsupported exclusive calls are false positives
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
tools:
- name: DriverPower
  role: Source of the coding and non-coding driver candidate result tables being curated
    against literature and driver catalogs
  repo: https://github.com/smshuai/DriverPower
- name: Python >= 3.5
  role: Scripting the intersection with CGC/PCAWG-consensus catalogs, evidence annotation,
    and FDR computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:a14b99aee12ab3e04c9a482eb6bfd11157d890cd2e3530014b4a1ab088d77326
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python >=
  - Python >= 3.5
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# driver-candidate-literature-curation

## Summary

Curates driver candidate calls (e.g. DriverPower's 217 coding and 95 non-coding candidates from 2583 PCAWG genomes) against literature and catalogs such as CGC and PCAWG-consensus to classify exclusive candidates and derive a false discovery rate estimate (3.2% coding, 16.8% non-coding).

## When to use

Use when a driver-discovery result table contains candidates absent from CGC and PCAWG-consensus catalogs and you need to triage them via literature/orthogonal evidence and compute an empirical FDR.

## When NOT to use

- Input candidates have already been intersected with CGC and PCAWG-consensus catalogs, leaving no exclusive calls to curate
- The goal is a statistical FDR via q-values or permutations rather than an evidence-based estimate assuming unsupported exclusive calls are false positives

## Inputs

- DriverPower result tables of driver candidates, files of the form [outDir]/[name].result.tsv (217 coding and 95 non-coding candidates)
- CGC (Cancer Gene Census) driver catalog
- PCAWG-consensus driver catalog
- Literature / orthogonal-evidence annotations for candidate genes and elements

## Outputs

- driverpower_exclusive_candidates_fdr.tsv: exclusive-candidate table (11 coding, 17 non-coding) with evidence-status annotations and per-category FDR estimates (3.2% coding, 16.8% non-coding)

## How to apply

Load the DriverPower result tables ([outDir]/[name].result.tsv) containing the 217 coding and 95 non-coding driver candidates called across 2583 PCAWG cancer genomes, then intersect candidate genes/elements with the CGC and PCAWG-consensus driver catalogs to extract candidates exclusive to the method (11 coding, 17 non-coding in this analysis). Annotate each exclusive candidate with literature or orthogonal-evidence support; assume exclusive candidates lacking any support are false positives. Compute per-category FDR as the fraction of all calls lacking evidence support: 7/217 (3.2%) coding and 16/95 (16.8%) non-coding. Write the annotated exclusive-candidate table with FDR estimates to driverpower_exclusive_candidates_fdr.tsv.

## Related tools

- **DriverPower** (Source of the coding and non-coding driver candidate result tables being curated against literature and driver catalogs) — https://github.com/smshuai/DriverPower
- **Python >= 3.5** (Scripting the intersection with CGC/PCAWG-consensus catalogs, evidence annotation, and FDR computation)

## Examples

```
python curate_exclusive_candidates.py --results '[outDir]/[name].result.tsv' --catalogs cgc.tsv pcawg_consensus.tsv --out driverpower_exclusive_candidates_fdr.tsv
```

## Evaluation signals

- Exclusive-candidate counts match 11 coding and 17 non-coding candidates after CGC/PCAWG-consensus intersection
- Computed coding FDR reproduces 7/217 = 3.2%
- Computed non-coding FDR reproduces 16/95 = 16.8%
- Output table driverpower_exclusive_candidates_fdr.tsv contains an evidence-status column for every exclusive candidate
- Evidence-supported counts are 4/11 coding and 4/17 non-coding unique exclusive candidates

## Limitations

- False-positive driver calls may arise from variant-calling artefacts or biological processes not captured by the BMR model (e.g. regional AID activity), so 'unsupported' does not strictly mean false
- The FDR estimate assumes all exclusive candidates lacking literature or orthogonal evidence are false positives, which may overestimate FDR for candidates in poorly studied regulatory elements
- Systematic undercalling of somatic variants in non-coding regions (~216 TERT promoter mutations estimated missed in PCAWG) affects the candidate pool being curated

## Evidence

- [abstract] DriverPower identifies 217 coding and 95 non-coding driver candidates across 2583 PCAWG cancer genomes: "Testing across a collection of 2583 cancer genomes from the PCAWG project, DriverPower identifies 217 coding and 95 non-coding driver candidates."
- [discussion] False positives from artefacts limit evidence-based curation: "False-positive driver calls may be caused by technical errors such as variant-calling artefacts that artificially increase the local mutation rate"
- [methods] DriverPower result file format: "DriverPower runs on processed training and test data as described in :doc:`data format </data_format>`."
- [methods] DriverPower requires Python >= 3.5: "DriverPower requires Python >= 3.5 and some other computing packages."
- [readme] DriverPower is a tool for driver element discovery in whole genomes: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
