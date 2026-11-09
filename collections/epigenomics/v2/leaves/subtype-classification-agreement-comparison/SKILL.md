---
name: subtype-classification-agreement-comparison
description: Use when a methylation (or expression) matrix with an established external
  subtype/label set is available and you must decide which top-value method (SD, ATC,
  etc.) yields partitions that agree with known subgroups.
when_to_use_negative:
- The input lacks an external, biologically validated subtype/label classification
  to compare partitions against — there is no agreement benchmark.
- The dataset is scRNA-Seq with several thousands of cells, where cola runtime and
  memory usage may be unacceptable per the article's discussion.
- The goal is differential expression or signature testing rather than evaluating
  which feature-selection top-value method best reproduces known subgroups.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: cola
  role: Runs adjust_matrix() preprocessing, run_all_consensus_partitions() with SD/ATC
    top values and the five partitioning methods, and generates the automated HTML
    report
  repo: https://github.com/jokergoo/cola_manuscript
- name: R
  role: Language/Bioconductor environment in which the cola comparison analysis and
    report generation are executed
- name: impute
  role: Provides impute.knn() used inside adjust_matrix() to impute missing methylation
    values before filtering
- name: bsub
  role: Submits the cola analysis jobs to an LSF computing cluster as done in the
    manuscript scripts
  repo: https://github.com/jokergoo/bsub
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:9f00e442f82b49df56b0d016a44a6efa7a6ea37f3caa51549475bda488a81d9a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/jokergoo/cola_manuscript
  repo_url: https://github.com/jokergoo/cola_manuscript
  tools:
  - R
  - bsub
  - cola
  - impute
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# subtype-classification-agreement-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares cola consensus partitions derived from different top-value feature-selection methods (SD vs ATC) against an established external subtype classification (e.g., DKFZ GBM subtypes) to judge which feature-selection strategy recovers known biology on the GBM 450K methylation dataset (GSE36278).

## When to use

Use when a methylation (or expression) matrix with an established external subtype/label set is available and you must decide which top-value method (SD, ATC, etc.) yields partitions that agree with known subgroups.

## When NOT to use

- The input lacks an external, biologically validated subtype/label classification to compare partitions against — there is no agreement benchmark.
- The dataset is scRNA-Seq with several thousands of cells, where cola runtime and memory usage may be unacceptable per the article's discussion.
- The goal is differential expression or signature testing rather than evaluating which feature-selection top-value method best reproduces known subgroups.

## Inputs

- GBM 450K DNA methylation beta-value matrix from GEO accession GSE36278
- DKFZ GBM subtype classification labels for the same samples
- Illumina 450K probe annotation with CpG-island association categories

## Outputs

- gbm_450k_sd_atc_comparison.csv containing SD-vs-ATC overlap, CpG-island fractions, and subtype-agreement statistics
- gbm_450k_cola_report.html (automated cola HTML report)
- Top-5000 probe sets by SD and ATC scores with their intersection
- Consensus subgroup assignments per top-value/partitioning method combination

## How to apply

Download the GBM 450K methylation matrix (GEO GSE36278) with GEOquery and preprocess it with cola's adjust_matrix(): remove probes on sex chromosomes and probes overlapping known SNPs, impute missing values with impute.knn() from the R package impute, winsorize outliers at the 5th/95th percentiles, and drop rows with variance below the 5th percentile. Run run_all_consensus_partitions() with top-value methods SD and ATC, partitioning methods hclust, kmeans, skmeans, pam and mclust (NMF excluded), and row scaling disabled. Take the top 5000 probes by each top-value score and quantify their overlap (expected near 1.76%), then annotate both probe sets with Illumina 450K CpG-island categories and compare CpG-island fractions (60.6% for SD vs 6.6% for ATC). Finally, compare the SD- and ATC-based consensus partitions to the DKFZ GBM subtype labels via partition agreement/label coincidence, writing statistics to gbm_450k_sd_atc_comparison.csv and generating the automated HTML report gbm_450k_cola_report.html. The rationale is that SD-selected probes recover the known subtype structure while ATC-selected probes mainly track global methylation level differences.

## Related tools

- **cola** (Runs adjust_matrix() preprocessing, run_all_consensus_partitions() with SD/ATC top values and the five partitioning methods, and generates the automated HTML report) — https://github.com/jokergoo/cola_manuscript
- **R** (Language/Bioconductor environment in which the cola comparison analysis and report generation are executed)
- **impute** (Provides impute.knn() used inside adjust_matrix() to impute missing methylation values before filtering)
- **bsub** (Submits the cola analysis jobs to an LSF computing cluster as done in the manuscript scripts) — https://github.com/jokergoo/bsub

## Examples

```
mat = adjust_matrix(mat); rl = run_all_consensus_partitions(mat, top_value_method = c("SD","ATC"), partition_method = c("hclust","kmeans","skmeans","pam","mclust"))
```

## Evaluation signals

- SD/ATC top-5000 probe intersection is near the reported 1.76% overlap
- CpG-island probe fraction among top-5000 probes is ~60.6% for SD vs ~6.6% for ATC
- Consensus partitions from SD-selected features agree with (recover) the DKFZ GBM subtype classification, while ATC partitions track global methylation level differences
- Output files gbm_450k_sd_atc_comparison.csv and gbm_450k_cola_report.html are produced with the expected statistics
- Preprocessed matrix has no sex-chromosome probes, no SNP-overlapping probes, no missing values, and no rows with variance below the 5th percentile

## Limitations

- cola runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells
- Downsampling strategies may miss small cell clusters
- Weighting consensus partitions across different methods is complex and merged classification can produce ambiguous labels

## Evidence
