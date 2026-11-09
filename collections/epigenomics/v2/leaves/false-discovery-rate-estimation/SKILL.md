---
name: false-discovery-rate-estimation
description: 'Use when a driver-discovery candidate list (e.g. DriverPower .result.tsv
  calls) needs an empirical FDR: intersect calls against known catalogs (CGC, PCAWG-consensus),
  isolate method-exclusive candidates, and compute the unsupported fraction per category.'
when_to_use_negative:
- Input is a raw mutation VCF or feature table rather than a called driver candidate
  list — call drivers first with driverpower model/infer
- A calibrated, model-based FDR (e.g. from null/permutation q-values) is required
  rather than an evidence-based empirical estimate on exclusive calls
- Comparing methods where every candidate is already in CGC or the PCAWG-consensus
  set — no exclusive calls exist to estimate the rate
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: DriverPower
  role: Produces the coding and non-coding candidate result tables whose exclusive
    calls are used to estimate the empirical FDR
  repo: https://github.com/smshuai/DriverPower
- name: Python >= 3.5
  role: Implements the catalog intersection, evidence annotation, and FDR computation
    writing driverpower_exclusive_candidates_fdr.tsv
- name: bedtools
  role: Interval intersection/anti-join of candidate elements against CGC and PCAWG-consensus
    catalogs to isolate DriverPower-exclusive candidates
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:74157f667a70f948dec83fe10a0f53f07d5b958c56d0ecb80ef1e99bb93e8552
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python >=
  - Python >= 3.5
  - bedtools
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# false-discovery-rate-estimation

## Summary

Estimates an empirical false discovery rate for DriverPower driver calls on the PCAWG data set by cataloguing candidates exclusive to DriverPower (absent from CGC and PCAWG-consensus) and treating unsupported exclusives as false positives: 3.2% (7/217) coding and 16.8% (16/95) non-coding.

## When to use

Use when a driver-discovery candidate list (e.g. DriverPower .result.tsv calls) needs an empirical FDR: intersect calls against known catalogs (CGC, PCAWG-consensus), isolate method-exclusive candidates, and compute the unsupported fraction per category.

## When NOT to use

- Input is a raw mutation VCF or feature table rather than a called driver candidate list — call drivers first with driverpower model/infer
- A calibrated, model-based FDR (e.g. from null/permutation q-values) is required rather than an evidence-based empirical estimate on exclusive calls
- Comparing methods where every candidate is already in CGC or the PCAWG-consensus set — no exclusive calls exist to estimate the rate

## Inputs

- DriverPower result tables for coding and non-coding candidates ([outDir]/[name].result.tsv)
- CGC cancer gene catalog
- PCAWG-consensus driver catalog
- Literature/orthogonal evidence annotations for exclusive candidates

## Outputs

- driverpower_exclusive_candidates_fdr.tsv — exclusive-candidate table annotated with evidence status plus per-category FDR estimates (3.2% coding, 16.8% non-coding)

## How to apply

Load the DriverPower result tables ([outDir]/[name].result.tsv) containing the 217 coding and 95 non-coding candidates called across 2583 PCAWG cancer genomes. Intersect the candidate gene/element lists with the CGC and PCAWG-consensus driver catalogs to extract the 11 coding and 17 non-coding candidates exclusive to DriverPower. Annotate each exclusive candidate with its literature/orthogonal evidence status, marking 4/11 coding and 4/17 non-coding as supported. Compute the FDR per category as the fraction of total calls lacking evidence support: 7/217 (3.2%) coding and 16/95 (16.8%) non-coding, under the assumption that unsupported exclusives are false positives — a rationale grounded in the paper's note that false-positive driver calls can arise from variant-calling artefacts or biological processes not captured by the BMR model.

## Related tools

- **DriverPower** (Produces the coding and non-coding candidate result tables whose exclusive calls are used to estimate the empirical FDR) — https://github.com/smshuai/DriverPower
- **Python >= 3.5** (Implements the catalog intersection, evidence annotation, and FDR computation writing driverpower_exclusive_candidates_fdr.tsv)
- **bedtools** (Interval intersection/anti-join of candidate elements against CGC and PCAWG-consensus catalogs to isolate DriverPower-exclusive candidates)

## Examples

```
python -c "print('coding FDR: %.1f%% (%d/217)' % (100*7/217,7)); print('non-coding FDR: %.1f%% (%d/95)' % (100*16/95,16))"  # then write driverpower_exclusive_candidates_fdr.tsv
```

## Evaluation signals

- Exclusive-candidate counts must equal 11 (coding) and 17 (non-coding) after intersection with CGC and PCAWG-consensus catalogs
- Reproduced FDR values must match the reported 3.2% (7/217) for coding and 16.8% (16/95) for non-coding
- Evidence-status annotation must mark 4/11 coding and 4/17 non-coding exclusive candidates as supported
- Output table driverpower_exclusive_candidates_fdr.tsv must contain every exclusive candidate with an evidence-status column and no overlap with CGC or PCAWG-consensus

## Limitations

- The FDR estimate assumes all unsupported exclusive candidates are false positives; true drivers lacking literature or orthogonal evidence inflate the rate
- False-positive driver calls may be caused by technical errors such as variant-calling artefacts or biological processes not captured by the BMR model (e.g. regional AID activity), with additional BMR features proposed but not yet incorporated
- Systematic undercalling of somatic variants in non-coding regions (~216 TERT promoter mutations estimated missed in PCAWG) can bias evidence-support annotations for non-coding exclusives

## Evidence

- [abstract] DriverPower identifies 217 coding and 95 non-coding driver candidates across 2583 PCAWG cancer genomes: "Testing across a collection of 2583 cancer genomes from the PCAWG project, DriverPower identifies 217 coding and 95 non-coding driver candidates."
- [discussion] False positives arise from artefacts or uncaptured biological processes: "False-positive driver calls may be caused by technical errors such as variant-calling artefacts that artificially increase the local mutation rate, or by biological processes that are not captured by"
- [discussion] Non-coding variant undercalling bias: "Potential artefacts include systematic undercalling of somatic variants in non-coding regions24, a problem that could be rectified by deeper coverage."
- [results] FDR figures from the source card: [evidence span withheld — anchor status: not_found]
- [readme] DriverPower scope from README: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
