---
name: silhouette-score-computation
description: Use when a consensus partitioning run (e.g. cola over multiple k values)
  needs per-sample or per-k stability assessment, or when ambiguous samples must be
  excluded before PAC calculation or signature analysis.
when_to_use_negative:
- No subgroup labels or consensus partition are available yet — silhouette requires
  class membership plus a sample distance matrix, not a raw feature matrix
- The goal is row/feature filtering before partitioning (e.g. low-variance rows, ATC
  top-value selection) rather than sample-level stability assessment
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: cola
  role: Generates the consensus partitions per k whose silhouette scores are computed
    and consumed by the best-k selection rules
  repo: https://github.com/jokergoo/cola_manuscript
- name: R
  role: Implementation environment for computing the silhouette statistic on the sample
    distance matrix
- name: Bioconductor
  role: Distribution framework for the cola package used in the partitioning pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:860d5a56d20096579e4454bb8d522a8d4072889ab5da7b8c10ee081876d9834c
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
  - Bioconductor
  - R
  - cola
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# silhouette-score-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes mean silhouette scores of consensus subgroup assignments on the sample distance matrix, as used in the cola framework to quantify sample stability across consensus partitions and to flag ambiguous samples.

## When to use

Use when a consensus partitioning run (e.g. cola over multiple k values) needs per-sample or per-k stability assessment, or when ambiguous samples must be excluded before PAC calculation or signature analysis.

## When NOT to use

- No subgroup labels or consensus partition are available yet — silhouette requires class membership plus a sample distance matrix, not a raw feature matrix
- The goal is row/feature filtering before partitioning (e.g. low-variance rows, ATC top-value selection) rather than sample-level stability assessment

## Inputs

- Consensus partition results per tested k (individual partitions, consensus subgroup labels, sample-sample consensus matrix) from a cola consensus partitioning run
- Sample-by-sample distance matrix on which the silhouette statistic is computed

## Outputs

- Per-sample silhouette scores for each tested k
- Mean silhouette per k recorded in cola_metrics.csv (with 1-PAC, concordance, Jaccard columns, stability flag 1-PAC > 0.9, and the selected best k)
- Set of ambiguous samples (silhouette < 0.5) excluded from signature analysis

## How to apply

Load the per-k consensus partition results (individual partitions, consensus partition, and the sample-sample consensus matrix) and the sample distance matrix. For each tested k, assign each sample its consensus subgroup label and compute its silhouette score on the sample distance matrix; the mean silhouette per k serves as one of the four metrics (alongside 1-PAC, concordance, and Jaccard) used in cola's three-rule best-k selection. Downstream of the metric computation, drop any k whose Jaccard index with k-1 is > 0.95, then take the maximal k with 1-PAC > 0.9, else a majority vote over the four metrics. Additionally, exclude samples with silhouette scores less than 0.5 from signature analysis because they are ambiguous, and exclude the 5% most unstable samples (lowest mean consensus) before recomputing the PAC score. Rationale: low silhouette indicates a sample sits ambiguously between subgroups, so it should not drive signature discovery or inflate PAC.

## Related tools

- **cola** (Generates the consensus partitions per k whose silhouette scores are computed and consumed by the best-k selection rules) — https://github.com/jokergoo/cola_manuscript
- **R** (Implementation environment for computing the silhouette statistic on the sample distance matrix)
- **Bioconductor** (Distribution framework for the cola package used in the partitioning pipeline)

## Examples

```
library(cluster); sil <- silhouette(consensus_labels, as.dist(sample_dist)); mean_sil <- mean(sil[, 3])
```

## Evaluation signals

- The output cola_metrics.csv contains a mean-silhouette column for every tested k in addition to 1-PAC, concordance, Jaccard, the stability flag, and the chosen k
- The stability flag is set exactly when 1-PAC > 0.9, and the Jaccard > 0.95 drop rule reproduces cola's thresholds
- PAC is recomputed after excluding the 5% most unstable samples (lowest mean consensus), matching cola's behaviour
- Samples flagged with silhouette scores < 0.5 are absent from the downstream signature analysis
- Mean silhouette values fall in [-1, 1] and higher k candidates that are Jaccard-redundant with k-1 are dropped before best-k selection

## Limitations

- cola's runtime and memory usage may be unacceptable for single-cell RNA-Seq datasets with several thousands of cells, limiting applicability of per-k silhouette computation at that scale
- Downsampling strategies used by consensus partitioning might miss small cell clusters, which distorts silhouette-based stability assessment
- No version number stated for the cola package or R in the manuscript discussion, limiting reproducibility of versioned silhouette results

## Evidence
