---
name: sv-score-normalization-across-classifiers
description: Use when comparing SV pathogenicity predictions from tools that output
  scores on different scales, especially when ranking the disease-causative gene for
  positive-control SVs against a patient-cohort background.
when_to_use_negative:
- Evaluating a tool on non-coding long-range mechanisms using TADA — the article reports
  TADA 'might not be appropriate for the prediction of non-coding (e.g. long-range)
  pathomechanisms', so it must be restricted to coding-disrupting effects in the benchmark
- Single-SV clinical interpretation rather than cohort-scale benchmarking — the online
  POSTRE version supports only single SV submissions; multiple-SV benchmarking requires
  the local R function
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0622
tools:
- name: POSTRE
  role: Reference classifier; run via source(POSTRE_multiSV.R) then POSTRE_multiSV(SVs,
    pathTo_Postre_app_Folder) to get pathogenicityPrediction_per_SV and geneStats_recurrencyAndPathomech
  repo: https://github.com/vicsanga/Postre
- name: TADA
  role: Benchmark classifier, applied only to coding-disrupting SV effects
- name: STRVCTVRE
  role: Benchmark classifier scored on the same SV cohorts
- name: SVScore
  role: Benchmark classifier scored on the same SV cohorts
- name: CADD-SV
  role: Benchmark classifier scored on the same SV cohorts
- name: TADeus2
  role: Reference tool for the gene-ranking agreement comparison on the 21-SV subset
    (14/21 = 67% correct gene)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:8b46eec2ff1b372a28abef84eec3ed9b343c175214cbaf7578a6db2859eec093
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 21
---

# sv-score-normalization-across-classifiers

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark protocol for placing per-SV pathogenicity scores from heterogeneous classifiers (POSTRE, TADA, STRVCTVRE, SVScore, CADD-SV, TADeus2) onto a common scale so that sensitivity/(1-specificity) ratios and disease-gene ranking agreement can be computed, as done for Figure 6 of the POSTRE paper.

## When to use

Use when comparing SV pathogenicity predictions from tools that output scores on different scales, especially when ranking the disease-causative gene for positive-control SVs against a patient-cohort background.

## When NOT to use

- Evaluating a tool on non-coding long-range mechanisms using TADA — the article reports TADA 'might not be appropriate for the prediction of non-coding (e.g. long-range) pathomechanisms', so it must be restricted to coding-disrupting effects in the benchmark
- Single-SV clinical interpretation rather than cohort-scale benchmarking — the online POSTRE version supports only single SV submissions; multiple-SV benchmarking requires the local R function

## Inputs

- Positive-control SV cohort (21 SVs with known disease-causative genes) as a 7-column tab-separated hg19 table (Table1_LongRange_SVs.tsv style)
- Patient SV cohort (270 patients) in the same 7-column POSTRE format (breakpoint chr1, coords1, chr2, coords2, SV type, comma-separated phenotypes, SV id)
- Raw per-SV pathogenicity scores from TADA, STRVCTVRE, SVScore, CADD-SV, TADeus2 and POSTRE

## Outputs

- Per-SV score table with all classifiers' predictions normalized onto a common scale
- Performance-ratio table (sensitivity and (1-specificity) per tool)
- Gene-ranking comparison table for the 21-SV subset (e.g. figure6_benchmark_comparison.tsv)

## How to apply

Load the positive-control SVs (21 cases) and the 270-patient SV cohort as tab-separated tables in the POSTRE 7-column format (breakpoint chromosomes, hg19 coordinates, SV type, phenotype categories such as head_neck/limbs/neurodevelopmental/cardiovascular, unique SV identifier) using read.delim in R. Run each benchmark classifier restricted to its stated applicability — e.g. TADA only for coding-disrupting effects since it is reported as inappropriate for non-coding long-range pathomechanisms — then run POSTRE by sourcing Postre_app/POSTRE_multiSV.R and calling POSTRE_multiSV() to obtain pathogenicityPrediction_per_SV and geneStats_recurrencyAndPathomech. Normalize the per-SV scores from all tools onto a common scale so they can be tabulated together as in Figure 6. Compute sensitivity and (1-specificity) ratios from positive-control versus patient-cohort classifications, and compare each tool's gene rankings on the 21-SV subset against TADeus2 (POSTRE: 18/21 = 86% vs TADEUS2: 14/21 = 67%). Write the per-SV score table, performance-ratio table and gene-ranking comparison to a TSV output such as figure6_benchmark_comparison.tsv.

## Related tools

- **POSTRE** (Reference classifier; run via source(POSTRE_multiSV.R) then POSTRE_multiSV(SVs, pathTo_Postre_app_Folder) to get pathogenicityPrediction_per_SV and geneStats_recurrencyAndPathomech) — https://github.com/vicsanga/Postre
- **TADA** (Benchmark classifier, applied only to coding-disrupting SV effects)
- **STRVCTVRE** (Benchmark classifier scored on the same SV cohorts)
- **SVScore** (Benchmark classifier scored on the same SV cohorts)
- **CADD-SV** (Benchmark classifier scored on the same SV cohorts)
- **TADeus2** (Reference tool for the gene-ranking agreement comparison on the 21-SV subset (14/21 = 67% correct gene))

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); res <- POSTRE_multiSV(SVs = svs_patients, pathTo_Postre_app_Folder = "/PathTo/Postre/")
```

## Evaluation signals

- POSTRE predicts the correct disease-causative gene in 18/21 positive-control SV cases (86%) versus TADEUS2 14/21 (67%) in the gene-ranking comparison
- Computed sensitivity and (1-specificity) ratios for each tool match the values reported in Figure 6 of the paper
- All per-SV scores fall on a comparable normalized scale before ranking (no raw mixed-scale mixing)
- Input tables conform to the 7-column POSTRE format: chromosome 1, coordinates 1 (hg19), chromosome 2, coordinates 2, SV type (Inversion/Translocation/Deletion/Duplication), comma-separated phenotypes, unique SV id
- TADA predictions are absent or restricted for non-coding/long-range SVs, consistent with its stated applicability

## Limitations

- Training databases for SV classifiers are biased towards coding deleterious effects due to the scarcity of reported long-range pathogenic SVs, which disadvantages long-range prediction in the benchmark
- POSTRE failed the SHH duplication case because bulk whole limb bud data cannot identify the ZRS enhancer, so SHH appears as an inactive gene
- Lack of genomic data for the appropriate developmental stages or differentiation time points may affect identification of pathologically relevant genes

## Evidence
