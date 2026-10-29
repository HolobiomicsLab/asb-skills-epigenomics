---
name: r-package-function-implementation
description: 'Use when you need to reproduce or extend cola''s -k selection logic
  in R code: you have per-k consensus matrices, individual partitions, and consensus
  partitions and must compute PAC/silhouette/concordance/Jaccard metrics to choose
  k.'
when_to_use_negative:
- You need to run the full consensus partitioning pipeline itself (matrix adjustment
  with adjust_matrix(), top-value feature selection, scaling) rather than score already-produced
  per-k consensus results
- The dataset is scRNA-Seq with several thousands of cells, where the article warns
  cola's runtime and memory usage 'might not be acceptable'
- There is no consensus matrix or multiple individual partitions per k, so PAC and
  concordance cannot be defined
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: cola
  role: Source package whose consensus partitioning results and best-k selection rules
    (1-PAC, silhouette, concordance, Jaccard) are reimplemented as R functions
  repo: https://github.com/jokergoo/cola_manuscript
- name: R
  role: Implementation language for the metric functions and the CSV export
  repo: https://github.com/jokergoo/cola_manuscript
- name: clue
  role: Provides cl_consensus(), the function cola uses to infer consensus subgroup
    labels from individual partitions, which the metrics operate on
- name: impute
  role: Provides impute.knn() for missing-value imputation in upstream matrix preparation
    feeding the consensus runs
- name: bsub
  role: Used to submit the cola analyses (whose results the metric functions consume)
    to LSF computing clusters from R
  repo: https://github.com/jokergoo/bsub
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:76da0715be087e267b702db355f06189eda5eb669ddc1c97fcf03e06918bd535
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
  - clue
  - cola
  - impute
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# r-package-function-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implement R functions that recompute cola's per-k consensus-partitioning evaluation metrics — 1-PAC (after excluding the 5% most unstable samples), mean silhouette, partition-concordance, and Jaccard index between consecutive k — and apply cola's three-rule best-k selection procedure.

## When to use

Use when you need to reproduce or extend cola's best-k selection logic in R code: you have per-k consensus matrices, individual partitions, and consensus partitions and must compute PAC/silhouette/concordance/Jaccard metrics to choose k.

## When NOT to use

- You need to run the full consensus partitioning pipeline itself (matrix adjustment with adjust_matrix(), top-value feature selection, scaling) rather than score already-produced per-k consensus results
- The dataset is scRNA-Seq with several thousands of cells, where the article warns cola's runtime and memory usage 'might not be acceptable'
- There is no consensus matrix or multiple individual partitions per k, so PAC and concordance cannot be defined

## Inputs

- Per-k individual partitions from cola consensus partitioning
- Per-k consensus partition (sample subgroup labels, e.g. inferred with cl_consensus() from the R package clue)
- Per-k sample-sample consensus matrix
- Sample distance matrix used for silhouette computation

## Outputs

- cola_metrics.csv with one row per tested k and columns for 1-PAC, mean silhouette, partition concordance, Jaccard index, stability flag (1-PAC > 0.9), and the selected best k

## How to apply

Load the per-k consensus partitioning results from a cola run (individual partitions, consensus partition labels, and the sample-sample consensus matrix for each tested k). For each k, compute the mean silhouette of the consensus partition on the sample distance matrix, compute the PAC as the proportion of ambiguous pairs in the consensus matrix (converting to 1-PAC) after removing the 5% most unstable samples (those with the lowest mean consensus across their pairs, mirroring cola's exclusion of unstable samples), average the concordance between each individual partition and the consensus partition, and compute the Jaccard index of pairwise co-membership between the k-1 and k partitions. Then apply the decision procedure: drop any k whose Jaccard with k-1 exceeds 0.95 as redundant; among the remainder, take the maximal k whose 1-PAC > 0.9; otherwise take the majority vote over the four metrics. Write all per-k metric values, stability flags (1-PAC > 0.9), and the chosen k to cola_metrics.csv.

## Related tools

- **cola** (Source package whose consensus partitioning results and best-k selection rules (1-PAC, silhouette, concordance, Jaccard) are reimplemented as R functions) — https://github.com/jokergoo/cola_manuscript
- **R** (Implementation language for the metric functions and the CSV export) — https://github.com/jokergoo/cola_manuscript
- **clue** (Provides cl_consensus(), the function cola uses to infer consensus subgroup labels from individual partitions, which the metrics operate on)
- **impute** (Provides impute.knn() for missing-value imputation in upstream matrix preparation feeding the consensus runs)
- **bsub** (Used to submit the cola analyses (whose results the metric functions consume) to LSF computing clusters from R) — https://github.com/jokergoo/bsub

## Evaluation signals

- cola_metrics.csv contains all four metric columns (1-PAC, silhouette, concordance, Jaccard) for every tested k plus the stability flag and chosen k
- The PAC computation reproduces cola's exclusion of the 5% most unstable samples (lowest mean consensus) before scoring
- Any k whose Jaccard index with k-1 exceeds 0.95 is dropped as redundant before selecting the best k
- The stability threshold (1-PAC > 0.9) flags exactly the k values cola would call stable, and the maximal such k is selected when at least one exists
- When no k has 1-PAC > 0.9, the chosen k equals the majority vote over the four metrics

## Limitations

- The best-k rules rely on cola-specific thresholds (Jaccard > 0.95, 1-PAC > 0.9, 5% sample exclusion) that are calibrated for cola's consensus framework, not arbitrary clustering outputs
- cola's runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells, limiting upstream result availability for this skill
- No version number for the cola package or R is stated in the manuscript context, which limits reproducibility of versioned metric results

## Evidence
