---
name: coding-and-noncoding-driver-evidence-classification
description: Use when you have DriverPower result tables ([outDir]/[name].result.tsv)
  of coding and non-coding driver candidates and need to estimate FDR by cross-referencing
  candidates against CGC and PCAWG-consensus driver catalogs and literature evidence.
when_to_use_negative:
- The aim is de novo driver discovery from raw mutations — use the DriverPower model/infer
  pipeline instead; this skill classifies and validates already-called candidates
- Input candidates are not from DriverPower (e.g. CGC or PCAWG-consensus lists themselves),
  since the exclusive-candidate set and FDR calculation are specific to DriverPower
  calls
- No external evidence catalogs (CGC, PCAWG-consensus) or literature annotations are
  available to adjudicate candidate support
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
tools:
- name: DriverPower
  role: Produces the coding (217) and non-coding (95) driver candidate result tables
    whose exclusive calls are classified and used for FDR estimation
  repo: https://github.com/smshuai/DriverPower
- name: Python >= 3.5
  role: Scripting the intersection of candidate lists with CGC/PCAWG-consensus catalogs,
    evidence annotation, and FDR computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:299645aae7311a0db6d6a68399f429cb713935a1cb9998f492d2cae18108e468
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python >=
  - Python >= 3.5
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# coding-and-noncoding-driver-evidence-classification

## Summary

Classifies DriverPower driver candidates called on the 2583-genome PCAWG data set as literature/orthogonal-evidence-supported or unsupported by intersecting them with the CGC and PCAWG-consensus catalogs, and converts the unsupported DriverPower-exclusive calls (11 coding, 17 non-coding) into estimated false discovery rates of 3.2% (7/217) coding and 16.8% (16/95) non-coding.

## When to use

Use when you have DriverPower result tables ([outDir]/[name].result.tsv) of coding and non-coding driver candidates and need to estimate FDR by cross-referencing candidates against CGC and PCAWG-consensus driver catalogs and literature evidence.

## When NOT to use

- The aim is de novo driver discovery from raw mutations — use the DriverPower model/infer pipeline instead; this skill classifies and validates already-called candidates
- Input candidates are not from DriverPower (e.g. CGC or PCAWG-consensus lists themselves), since the exclusive-candidate set and FDR calculation are specific to DriverPower calls
- No external evidence catalogs (CGC, PCAWG-consensus) or literature annotations are available to adjudicate candidate support

## Inputs

- DriverPower result tables of coding and non-coding driver candidates, files of the form [outDir]/[name].result.tsv (217 coding and 95 non-coding candidates from 2583 PCAWG genomes)
- CGC (Cancer Gene Census) driver catalog
- PCAWG-consensus driver catalog
- Literature/orthogonal evidence annotations for exclusive candidates

## Outputs

- driverpower_exclusive_candidates_fdr.tsv: exclusive-candidate table (11 coding, 17 non-coding) with evidence-status annotations
- Estimated FDR values: 3.2% (7/217) coding and 16.8% (16/95) non-coding

## How to apply

Load the DriverPower result tables containing the 217 coding and 95 non-coding driver candidates called across 2583 PCAWG cancer genomes, then intersect candidate gene/element lists with the CGC and PCAWG-consensus driver catalogs to extract candidates exclusive to DriverPower (11 coding, 17 non-coding). Annotate each exclusive candidate with its literature/orthogonal evidence status, marking 4/11 coding and 4/17 non-coding exclusive candidates as supported. Compute the per-category FDR as the fraction of total calls lacking evidence support — 7/217 (3.2%) for coding and 16/95 (16.8%) for non-coding — under the assumption that exclusive candidates without literature or orthogonal evidence are false positives. Write the annotated exclusive-candidate table with FDR estimates to driverpower_exclusive_candidates_fdr.tsv.

## Related tools

- **DriverPower** (Produces the coding (217) and non-coding (95) driver candidate result tables whose exclusive calls are classified and used for FDR estimation) — https://github.com/smshuai/DriverPower
- **Python >= 3.5** (Scripting the intersection of candidate lists with CGC/PCAWG-consensus catalogs, evidence annotation, and FDR computation)

## Examples

```
python classify_exclusive_candidates.py --results [outDir]/[name].result.tsv --cgc cgc_catalog.tsv --consensus pcawg_consensus.tsv --out driverpower_exclusive_candidates_fdr.tsv
```

## Evaluation signals

- Exclusive-candidate counts after intersecting with CGC and PCAWG-consensus catalogs equal exactly 11 coding and 17 non-coding
- Computed coding FDR reproduces 3.2% (7/217) and non-coding FDR reproduces 16.8% (16/95)
- Evidence annotation marks exactly 4/11 coding and 4/17 non-coding exclusive candidates as supported
- Output TSV (driverpower_exclusive_candidates_fdr.tsv) contains both the evidence-status column and the per-category FDR estimates
- Total candidate counts in the source result tables match 217 coding and 95 non-coding from the 2583 PCAWG genomes

## Limitations

- FDR estimates assume all exclusive candidates lacking literature or orthogonal evidence are false positives; some may be genuine novel drivers
- False-positive driver calls may arise from variant-calling artefacts and biological processes not captured by the BMR model (e.g. regional AID activity)
- Only ~3.7% of the genome was studied, and functional impact score schemes are biased toward coding mutations, affecting the non-coding candidate evidence assessment

## Evidence

- [abstract] DriverPower identifies 217 coding and 95 non-coding driver candidates across 2583 PCAWG cancer genomes: "Testing across a collection of 2583 cancer genomes from the PCAWG project, DriverPower identifies 217 coding and 95 non-coding driver candidates."
- [methods] DriverPower result tables loaded for candidate intersection: "The ``infer`` sub-command is used to call drivers from test data with pre-trained models."
- [discussion] False positives can arise from artefacts or unmodelled biology, motivating evidence-based FDR estimation: "False-positive driver calls may be caused by technical errors such as variant-calling artefacts that artificially increase the local mutation rate, or by biological processes that are not captured by"
- [readme] DriverPower is a tool for coding and non-coding driver discovery, per README: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
