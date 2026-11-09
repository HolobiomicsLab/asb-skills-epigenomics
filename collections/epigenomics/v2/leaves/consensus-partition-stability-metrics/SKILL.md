---
name: consensus-partition-stability-metrics
description: Use when you have per-k consensus partitioning results (individual partitions,
  consensus partition, sample-sample consensus matrix) and must choose the number
  of subgroups k using stability metrics.
when_to_use_negative:
- Input is a raw expression/methylation matrix that has not yet been through cola
  consensus partitioning — run preprocessing (adjust_matrix), feature selection (SD,
  MAD, CV, ATC) and consensus partitioning first
- You need subgroup signature analysis rather than best-k selection — note that cola
  separately removes samples with silhouette scores less than 0.5 from the signature
  analysis
- The number of samples is very large single-cell data where cola's runtime/memory
  is unacceptable
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: cola
  role: R/Bioconductor package whose consensus partitioning framework and stability
    metric thresholds (1-PAC > 0.9, Jaccard > 0.95, 5% unstable-sample removal) this
    skill reproduces
  repo: https://github.com/jokergoo/cola_manuscript
- name: R
  role: Implementation language for computing silhouette, PAC, concordance and Jaccard
    metrics from cola output
- name: Bioconductor
  role: Distribution channel for the cola package used to generate the per-k consensus
    partitions consumed by this skill
- name: clue
  role: Provides cl_consensus() used to infer consensus subgroup labels from individual
    partitions prior to metric computation
- name: bsub
  role: Used to submit the cola consensus partitioning jobs to an LSF computing cluster
  repo: https://github.com/jokergoo/bsub
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:7cefed7c86bd3c70e0f5539f80ad10767aa89469f416a194e60b25b926ca2306
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
  - bsub
  - clue
  - cola
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# consensus-partition-stability-metrics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs cola's best-k selection logic by computing 1-PAC, mean silhouette, concordance and Jaccard metrics per tested k from consensus partitioning output, then applying cola's three-rule decision procedure to pick the best number of subgroups.

## When to use

Use when you have per-k consensus partitioning results (individual partitions, consensus partition, sample-sample consensus matrix) and must choose the best number of subgroups k using stability metrics.

## When NOT to use

- Input is a raw expression/methylation matrix that has not yet been through cola consensus partitioning — run preprocessing (adjust_matrix), feature selection (SD, MAD, CV, ATC) and consensus partitioning first
- You need subgroup signature analysis rather than best-k selection — note that cola separately removes samples with silhouette scores less than 0.5 from the signature analysis
- The number of samples is very large single-cell data where cola's runtime/memory is unacceptable

## Inputs

- Per-k consensus partition results from a cola run (individual partitions Pa from repeated random sampling of rows/columns)
- Consensus subgroup labels inferred from individual partitions (e.g. via cl_consensus() from the R package clue)
- Sample-sample consensus matrix for each tested k
- Sample distance matrix used for silhouette computation

## Outputs

- cola_metrics.csv with per-k columns for 1-PAC (PAC excluding the 5% most unstable samples), mean silhouette score, mean concordance between individual and consensus partitions, Jaccard index vs k-1, a stability flag, and the selected best k
- A single selected best-k value following cola's three-rule decision procedure

## How to apply

Load the per-k consensus partition results produced by a cola run (individual partitions, consensus partition and the sample-sample consensus matrix). For each k, compute the mean silhouette score of the consensus partition on the sample distance matrix, and compute PAC (proportion of ambiguous pairs in the consensus matrix) after first removing the 5% most unstable samples (those with the lowest mean consensus across their pairs), reporting 1-PAC. Also compute the concordance between each individual partition and the consensus partition for that k (averaged over individual partitions) and the Jaccard index between the k-1 and k partitions based on pairwise sample co-membership. Apply cola's three-rule decision: (a) drop any k whose Jaccard index with k-1 is > 0.95 as redundant; (b) among remaining k, take the maximal k with 1-PAC > 0.9; (c) if none satisfies 1-PAC > 0.9, take the majority vote over the four metrics. Flag each k as stable when 1-PAC > 0.9 and write all metric values, flags and the chosen k to cola_metrics.csv.

## Related tools

- **cola** (R/Bioconductor package whose consensus partitioning framework and stability metric thresholds (1-PAC > 0.9, Jaccard > 0.95, 5% unstable-sample removal) this skill reproduces) — https://github.com/jokergoo/cola_manuscript
- **R** (Implementation language for computing silhouette, PAC, concordance and Jaccard metrics from cola output)
- **Bioconductor** (Distribution channel for the cola package used to generate the per-k consensus partitions consumed by this skill)
- **clue** (Provides cl_consensus() used to infer consensus subgroup labels from individual partitions prior to metric computation)
- **bsub** (Used to submit the cola consensus partitioning jobs to an LSF computing cluster) — https://github.com/jokergoo/bsub

## Examples

```
library(cola); mat2 = adjust_matrix(mat)  # impute missing values, adjust outliers, remove low-variance rows before consensus partitioning
```

## Evaluation signals

- cola_metrics.csv contains all four metric columns (1-PAC, silhouette, concordance, Jaccard) for every tested k plus the stability flag and the chosen k
- PAC values are recomputed after excluding the 5% most unstable samples (lowest mean consensus across their pairs), not on the full consensus matrix
- The chosen k reproduces the three-rule procedure: k with Jaccard vs k-1 > 0.95 dropped as redundant, maximal k with 1-PAC > 0.9 preferred, majority vote fallback
- Each k is flagged stable exactly when its 1-PAC > 0.9
- Concordance per k equals the average agreement between each individual partition and the consensus partition at that k

## Limitations

- cola runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells
- Downsampling strategies (randomly sampling rows/columns to generate individual partitions) may miss small cell clusters
- Samples with silhouette scores less than 0.5 are considered ambiguous and removed by default from downstream signature analysis

## Evidence

- [intro] cola provides a complete set of tools for comprehensive subgroup analysis, including partitioning, signature analysis, functional enrichment: "cola provides a complete set of tools for comprehensive subgroup analysis, including partitioning, signature analysis, functional enrichment"
- [abstract] We implemented a new R/Bioconductor package, cola, that provides a general framework for consensus partitioning.: "We implemented a new R/Bioconductor package, cola, that provides a general framework for consensus partitioning."
- [intro] By default, cola removes samples with silhouette scores less than 0.5 from the signature analysis because they are ambiguous: "By default, cola removes samples with silhouette scores less than 0.5 from the signature analysis because they are ambiguous"
- [intro] A subset of rows or columns are randomly sampled from M with a probability of p by applying a certain partitioning method, generating a partition Pa: "A subset of rows or columns are randomly sampled from M with a probability of p by applying a certain partitioning method, generating a partition Pa"
- [readme] This repository contains scripts for the manuscript '**cola: an R/Bioconductor Package for Consensus Partitioning through a General Framework**'.: "This repository contains scripts for the manuscript '**cola: an R/Bioconductor Package for Consensus Partitioning through a General Framework**'."
