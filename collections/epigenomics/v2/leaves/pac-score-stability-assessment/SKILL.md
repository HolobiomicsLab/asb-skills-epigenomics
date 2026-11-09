---
name: pac-score-stability-assessment
description: Use when you have per-k consensus partitioning results (individual partitions,
  consensus partition, sample-sample consensus matrices) and must select the k using
  cola's PAC-based stability metrics and decision rules.
when_to_use_negative:
- Single-cell datasets with several thousands of cells — the cola paper notes runtime
  and memory usage may be unacceptable at that scale, and small clusters may be missed
  by downsampling
- 'The upstream consensus partitions have not been generated yet (e.g. the matrix
  has not gone through cola''s adjust_matrix preprocessing: impute.knn for missing
  values, winsorizing at the 5th/95th percentiles, low-variance row removal)'
- You need to compare clustering methods rather than evaluate stability/select k for
  one consensus partitioning run
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: cola
  role: Source of the PAC/silhouette/concordance/Jaccard stability metrics and the
    best-k decision rules being reproduced; provides consensus partitioning and adjust_matrix()
    preprocessing
  repo: https://github.com/jokergoo/cola_manuscript
- name: R
  role: Implementation language for recomputing per-k metrics (silhouette, PAC, concordance,
    Jaccard) and writing cola_metrics.csv
- name: Bioconductor
  role: Package ecosystem in which cola is distributed and from which companion packages
    (e.g. impute) are used
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:a8823154f32743128dc0e39406d6b8035706ca0c9cb5b238a48bb8949d233262
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# pac-score-stability-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Replicates cola's best-k selection statistics (1-PAC with 5% most unstable samples excluded, mean silhouette, concordance, and Jaccard between consecutive k) and its three-rule decision procedure for picking the best number of subgroups.

## When to use

Use when you have per-k consensus partitioning results (individual partitions, consensus partition, sample-sample consensus matrices) and must select the best k using cola's PAC-based stability metrics and decision rules.

## When NOT to use

- Single-cell datasets with several thousands of cells — the cola paper notes runtime and memory usage may be unacceptable at that scale, and small clusters may be missed by downsampling
- The upstream consensus partitions have not been generated yet (e.g. the matrix has not gone through cola's adjust_matrix preprocessing: impute.knn for missing values, winsorizing at the 5th/95th percentiles, low-variance row removal)
- You need to compare clustering methods rather than evaluate stability/select k for one consensus partitioning run

## Inputs

- Per-k consensus partition results from a cola run: individual partitions, consensus partition, and sample-sample consensus matrix for each tested k
- Sample distance matrix (for silhouette computation)

## Outputs

- cola_metrics.csv with one row per tested k containing 1-PAC, mean silhouette, concordance, Jaccard index (vs k-1), stability flag (1-PAC > 0.9), and the selected best k

## How to apply

For each tested k, first compute the mean silhouette score of the consensus partition on the sample distance matrix; then compute the PAC (proportion of ambiguous pairs in the sample-sample consensus matrix), converting it to 1-PAC, after excluding the 5% most unstable samples — the samples with the lowest mean consensus across their pairs (cola similarly drops samples with silhouette scores < 0.5 from downstream signature analysis because they are ambiguous). Also compute the concordance between each individual partition and the consensus partition (averaged over individual partitions) and the Jaccard index between the k-1 and k partitions based on pairwise sample co-membership. Apply cola's three-rule decision: (a) drop any k whose Jaccard index with k-1 is > 0.95 as redundant; (b) among remaining k, take the maximal k whose 1-PAC > 0.9; (c) if no k satisfies 1-PAC > 0.9, take the majority vote over the four metrics. Flag each k as stable when 1-PAC > 0.9 and write all per-k metric values, flags, and the chosen k to cola_metrics.csv.

## Related tools

- **cola** (Source of the PAC/silhouette/concordance/Jaccard stability metrics and the best-k decision rules being reproduced; provides consensus partitioning and adjust_matrix() preprocessing) — https://github.com/jokergoo/cola_manuscript
- **R** (Implementation language for recomputing per-k metrics (silhouette, PAC, concordance, Jaccard) and writing cola_metrics.csv)
- **Bioconductor** (Package ecosystem in which cola is distributed and from which companion packages (e.g. impute) are used)

## Evaluation signals

- cola_metrics.csv contains all four metric columns (1-PAC, mean silhouette, concordance, Jaccard) plus the stability flag and chosen k for every tested k
- PAC is recomputed after excluding the 5% most unstable samples (lowest mean consensus across pairs), matching cola's procedure
- k values with Jaccard index to k-1 > 0.95 are dropped as redundant before selection
- No k with 1-PAC <= 0.9 is flagged stable; selection follows the maximal-k-with-1-PAC > 0.9 rule or the four-metric majority vote fallback
- Silhouette scores are computed on the sample distance matrix against the consensus partition, and samples with silhouette < 0.5 are treated as ambiguous (consistent with cola's default)

## Limitations

- cola runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells
- Downsampling strategies used by cola may miss small cell clusters
- No version number stated for the cola package in the source, limiting reproducibility of versioned metric implementations

## Evidence
