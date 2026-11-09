---
name: sensitivity-specificity-ratio-analysis
description: Use when you have a set of positive-control SVs with known disease-causative
  genes plus a larger patient/VUS SV cohort and must compare per-SV pathogenicity
  scores across multiple classifiers.
when_to_use_negative:
- The evaluation goal is recessive phenotypes — POSTRE is oriented towards dominant
  disorders and its capacity for recessive SVs has not been specifically evaluated
- The SVs act through long-range mechanisms and the tool being assessed is a coding-biased
  classifier such as TADA, which is reported as inappropriate for non-coding pathomechanisms
- Only a single SV needs interpretation (use POSTRE's Single SV submission instead
  of a multi-cohort benchmark)
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: POSTRE
  role: SV pathogenicity predictor under evaluation; run via POSTRE_multiSV() on both
    cohorts to obtain pathogenicityPrediction_per_SV and geneStats_recurrencyAndPathomech
  repo: https://github.com/vicsanga/Postre
- name: TADA
  role: Benchmark classifier, applied only to coding-disrupting effects
- name: STRVCTVRE
  role: Benchmark SV pathogenicity classifier on both cohorts
- name: SVScore
  role: Benchmark SV pathogenicity classifier on both cohorts
- name: CADD-SV
  role: Benchmark SV pathogenicity classifier on both cohorts
- name: TADeus2
  role: Benchmark classifier used as the reference for the 21-SV gene-ranking comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:858dec9014cc017774b3967fe1c00f55c5b9e841032e6e8392192bac19cff960
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/vicsanga/Postre
  repo_url: https://github.com/vicsanga/Postre
  tools:
  - CADD-SV
  - POSTRE
  - R
  - STRVCTVRE
  - SVScore
  - TADA
  - TADeus2
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# sensitivity-specificity-ratio-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark structural-variant pathogenicity predictors by comparing their classifications on a positive-control SV cohort against a patient cohort, computing sensitivity/(1-specificity) ratios per tool and comparing gene-ranking agreement on a 21-SV subset (e.g. POSTRE 18/21 = 86% versus TADeus2 14/21 = 67%).

## When to use

Use when you have a set of positive-control SVs with known disease-causative genes plus a larger patient/VUS SV cohort and must compare per-SV pathogenicity scores across multiple classifiers.

## When NOT to use

- The evaluation goal is recessive phenotypes — POSTRE is oriented towards dominant disorders and its capacity for recessive SVs has not been specifically evaluated
- The SVs act through long-range mechanisms and the tool being assessed is a coding-biased classifier such as TADA, which is reported as inappropriate for non-coding pathomechanisms
- Only a single SV needs interpretation (use POSTRE's Single SV submission instead of a multi-cohort benchmark)

## Inputs

- Positive-control SV cohort as a 7-column tab-separated table (POSTRE format: chr1, coord1, chr2, coord2, SV type, phenotypes, SV id) with known disease-causative genes, e.g. Table1_LongRange_SVs.tsv
- 270-patient SV cohort in the same 7-column tab-separated POSTRE format
- Per-SV pathogenicity scores from TADA, STRVCTVRE, SVScore, CADD-SV and TADeus2, each restricted to its stated applicability domain

## Outputs

- Per-SV score table with pathogenicity scores from all benchmarked tools on a normalized common scale (figure6_benchmark_comparison.tsv)
- Performance-ratio table of sensitivity and (1-specificity) per tool
- Gene-ranking comparison on the 21-SV subset (percentage agreement vs TADeus2: 86% vs 67%)

## How to apply

Load both cohorts as tab-separated tables in POSTRE's 7-column format (breakpoint chromosomes, hg19 coordinates, SV type, phenotype categories, unique SV id) using read.delim in R. Run each benchmark classifier — TADA, STRVCTVRE, SVScore, CADD-SV and TADeus2 — restricting each to its stated applicability (e.g. TADA only for coding-disrupting effects, since it is reported as inappropriate for non-coding long-range pathomechanisms). Run POSTRE on the same cohorts by sourcing Postre_app/POSTRE_multiSV.R and calling POSTRE_multiSV(SVs = svs_patients, pathTo_Postre_app_Folder = ...) to obtain pathogenicityPrediction_per_SV and geneStats_recurrencyAndPathomech. Normalize all tools' per-SV pathogenic scores onto a common scale for Figure 6-style comparison, then compute sensitivity and (1-specificity) ratios for each tool from positive-control versus patient-cohort classifications. Finally, on the 21-SV subset, compare each tool's gene rankings against TADeus2 and tabulate the percentage agreement (86% for POSTRE vs 67% for TADeus2).

## Related tools

- **POSTRE** (SV pathogenicity predictor under evaluation; run via POSTRE_multiSV() on both cohorts to obtain pathogenicityPrediction_per_SV and geneStats_recurrencyAndPathomech) — https://github.com/vicsanga/Postre
- **TADA** (Benchmark classifier, applied only to coding-disrupting effects)
- **STRVCTVRE** (Benchmark SV pathogenicity classifier on both cohorts)
- **SVScore** (Benchmark SV pathogenicity classifier on both cohorts)
- **CADD-SV** (Benchmark SV pathogenicity classifier on both cohorts)
- **TADeus2** (Benchmark classifier used as the reference for the 21-SV gene-ranking comparison)

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); res <- POSTRE_multiSV(SVs = svs_patients, pathTo_Postre_app_Folder = "/PathTo/Postre/")
```

## Evaluation signals

- Sensitivity/(1-specificity) ratios computed from positive-control vs 270-patient cohort classifications match the values reported in Figure 6 of the paper
- On the 21-SV subset, POSTRE predicts the correct disease-causative gene in 18/21 cases (86%) versus TADeus2's 14/21 (67%)
- Both input cohorts parse as 7-column tab-separated POSTRE-format tables with hg19 coordinates and valid SV types (Inversion, Translocation, Deletion, Duplication) and phenotype categories
- Per-SV scores from all tools are successfully normalized onto a common comparable scale before ratio computation
- TADA results are only counted where the SV has a coding-disrupting effect, consistent with its stated applicability

## Limitations

- Databases used to train SV classifiers are biased towards coding deleterious effects due to the scarcity of reported SVs with long-range pathogenic effects
- Machine learning classifiers risk bias through overfitting when training data does not faithfully recapitulate the general population
- Incomplete characterization of enhancers across human cell types limits prediction of long-range pathomechanisms (e.g. POSTRE fails on the SHH duplication case because bulk whole limb bud data cannot identify the ZRS enhancer)

## Evidence

- [results] POSTRE predicted the correct disease-causative gene in 18/21 positive-control SV cases (86%), outperforming TADEUS2, which predicted it in 14/21 cases (67%).: [evidence span withheld — anchor status: not_found]
