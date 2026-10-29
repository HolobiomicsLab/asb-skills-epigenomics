---
name: gene-ranking-comparison-across-tools
description: Use when a set of SVs with known disease-causative genes (positive controls)
  must be scored by multiple pathogenicity classifiers and ranked-gene agreement between
  tools (e.g. POSTRE vs TADEUS2) quantified.
when_to_use_negative:
- The SVs act through non-coding long-range mechanisms and TADA is being compared
  without restriction — TADA is reported as inappropriate for non-coding (e.g. long-range)
  pathomechanisms
- The task is recessive-disorder SV interpretation — POSTRE is oriented towards dominant
  disorders and its capacity for recessive phenotypes has not been specifically evaluated
- Only a single tool's predictions are available with no comparator cohort — sensitivity/(1-specificity)
  ratios and ranking agreement require both positive controls and a second classifier
  such as TADeus2
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: POSTRE
  role: Candidate classifier under benchmarking; run via POSTRE_multiSV() to obtain
    pathogenicityPrediction_per_SV and geneStats_recurrencyAndPathomech for both cohorts
  repo: https://github.com/vicsanga/Postre
- name: TADeus2
  role: Reference comparator whose gene rankings define the agreement baseline for
    the 21-SV positive-control subset
- name: TADA
  role: Benchmark classifier, restricted to coding-disrupting SV effects only
- name: STRVCTVRE
  role: Benchmark classifier scored on both cohorts and normalized to the common scale
- name: SVScore
  role: Benchmark classifier scored on both cohorts and normalized to the common scale
- name: CADD-SV
  role: Benchmark classifier scored on both cohorts and normalized to the common scale
- name: R
  role: Runtime environment (>=3.5.0) in which POSTRE_multiSV is sourced and the benchmark
    tables are computed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:54c9a2753bcc2131dca068a20e17590044b5c3cf2082aa3d1ccca9226f558fbf
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
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# Gene-ranking comparison across tools

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a candidate SV pathogenicity predictor (POSTRE) against competing classifiers (TADA, STRVCTVRE, SVScore, CADD-SV, TADeus2) by comparing which tool ranks the correct disease-causative gene on a set of positive-control structural variants. Provides the reproduction of Figure 6's head-to-head gene-ranking agreement metrics.

## When to use

Use when a set of SVs with known disease-causative genes (positive controls) must be scored by multiple pathogenicity classifiers and ranked-gene agreement between tools (e.g. POSTRE vs TADEUS2) quantified.

## When NOT to use

- The SVs act through non-coding long-range mechanisms and TADA is being compared without restriction — TADA is reported as inappropriate for non-coding (e.g. long-range) pathomechanisms
- The task is recessive-disorder SV interpretation — POSTRE is oriented towards dominant disorders and its capacity for recessive phenotypes has not been specifically evaluated
- Only a single tool's predictions are available with no comparator cohort — sensitivity/(1-specificity) ratios and ranking agreement require both positive controls and a second classifier such as TADeus2

## Inputs

- Positive-control SV cohort as a 7-column tab-separated file (breakpoint chr1, hg19 coordinates 1, chr2, coordinates 2, SV type, phenotype categories, unique SV id), e.g. Table1_LongRange_SVs.tsv
- 270-patient SV cohort in the same 7-column POSTRE format
- Per-SV pathogenicity scores from TADA, STRVCTVRE, SVScore, CADD-SV and TADeus2 run on both cohorts

## Outputs

- Normalized per-SV pathogenic score table across all benchmarked tools
- Sensitivity and (1-specificity) ratio table per tool (positive controls vs patient cohort)
- 21-SV gene-ranking comparison table (percentage agreement per tool vs TADeus2), written to figure6_benchmark_comparison.tsv

## How to apply

Load the positive-control and 270-patient SV cohorts as tab-separated tables in the POSTRE 7-column format (breakpoint chromosomes, hg19 coordinates, SV type, phenotype categories, unique SV identifier) using read.delim in R. Run each benchmark classifier on both cohorts, restricting each tool to its stated applicability — TADA only for coding-disrupting effects, since it is reported as inappropriate for non-coding long-range pathomechanisms. Run POSTRE on the same cohorts by sourcing Postre_app/POSTRE_multiSV.R and calling POSTRE_multiSV(SVs = svs_patients, pathTo_Postre_app_Folder = ...) to obtain pathogenicityPrediction_per_SV and geneStats_recurrencyAndPathomech. Normalize the per-SV pathogenic scores from all tools onto a common scale, then compute sensitivity and (1-specificity) ratios from the positive-control versus patient-cohort classifications. On the 21-SV positive-control subset, compare gene rankings produced by each tool against TADeus2 and tabulate the percentage agreement, writing results to a figure6_benchmark_comparison.tsv table.

## Related tools

- **POSTRE** (Candidate classifier under benchmarking; run via POSTRE_multiSV() to obtain pathogenicityPrediction_per_SV and geneStats_recurrencyAndPathomech for both cohorts) — https://github.com/vicsanga/Postre
- **TADeus2** (Reference comparator whose gene rankings define the agreement baseline for the 21-SV positive-control subset)
- **TADA** (Benchmark classifier, restricted to coding-disrupting SV effects only)
- **STRVCTVRE** (Benchmark classifier scored on both cohorts and normalized to the common scale)
- **SVScore** (Benchmark classifier scored on both cohorts and normalized to the common scale)
- **CADD-SV** (Benchmark classifier scored on both cohorts and normalized to the common scale)
- **R** (Runtime environment (>=3.5.0) in which POSTRE_multiSV is sourced and the benchmark tables are computed)

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); results <- POSTRE_multiSV(SVs = svs_patients, pathTo_Postre_app_Folder = "/PathTo/Postre/Postre_app")
```

## Evaluation signals

- POSTRE predicts the correct disease-causative gene in 18/21 positive-control SV cases (86%), matching Figure 6
- TADEUS2 predicts the correct gene in 14/21 cases (67%), matching Figure 6
- Computed sensitivity/(1-specificity) ratios for each tool reproduce the values reported in Figure 6
- Per-SV scores from all tools lie on a common normalized scale before ranking comparison
- Output table figure6_benchmark_comparison.tsv contains the per-SV scores, performance ratios and gene-ranking agreement columns

## Limitations

- TADA is reported as inappropriate for the prediction of non-coding (e.g. long-range) pathomechanisms, so it must be restricted to coding-disrupting effects in the comparison
- Databases used to train SV classifiers are biased towards coding deleterious effects due to the scarcity of reported SVs with long-range pathogenic effects, which can deflate competitor scores on long-range controls
- POSTRE's bulk whole limb bud data cannot identify the ZRS enhancer, so it fails the SHH duplication case — some positive controls will be missed for data-coverage reasons
- Lack of genomic data for the appropriate developmental stages or differentiation time points may affect identification of pathologically relevant genes

## Evidence

- [results] POSTRE predicted the correct disease-causative gene in 18/21 positive-control SV cases (86%), outperforming TADEUS2, which predicted it in 14/21 cases (67%).: [evidence span withheld — anchor status: not_found]
- [methods] To use it, it requires 2 mandatory arguments. SVs: Data frame with 7 columns containing the SVs information (coordinates have to be in hg19).: [evidence span withheld — anchor status: not_found]
