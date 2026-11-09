---
name: optimal-k-decision-rule-implementation
description: Use when you have per-k consensus partition outputs (individual partitions,
  consensus partition, sample-sample consensus matrices) from a cola run and need
  to reconstruct or audit how the k is chosen.
when_to_use_negative:
- Input is a single fixed-k clustering with no consensus matrix or per-k partitions
  — there is nothing to compare across k values
- You are doing signature analysis sample filtering, where cola instead removes samples
  with silhouette scores < 0.5 — a different rule than the 5%-most-unstable PAC exclusion
- The dataset is scRNA-Seq with several thousands of cells, where the article notes
  cola runtime and memory usage may be unacceptable
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: cola
  role: Source of the consensus partitioning framework whose best-k selection statistics
    (PAC, silhouette, concordance) are being reconstructed; R/Bioconductor package
  repo: https://github.com/jokergoo/cola_manuscript
- name: R
  role: Implementation language for recomputing the per-k metrics and decision rules
- name: Bioconductor
  role: Ecosystem in which the cola package providing the consensus partition results
    is distributed
- name: clue
  role: R package whose cl_consensus() function infers the consensus subgroup labels
    from individual partitions that feed the metric computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:26f1ef1d4d90e6e244980bd9fb53a05074c4d788f19e3466ec70f88180acd8d6
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
  - clue
  - cola
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# optimal-k-decision-rule-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reimplements cola's three-rule procedure for selecting the best number of subgroups k from consensus partitioning results, using 1-PAC, mean silhouette, partition concordance, and Jaccard index between consecutive k values, writing all per-k metrics to a CSV for auditability.

## When to use

Use when you have per-k consensus partition outputs (individual partitions, consensus partition, sample-sample consensus matrices) from a cola run and need to reconstruct or audit how the best k is chosen.

## When NOT to use

- Input is a single fixed-k clustering with no consensus matrix or per-k partitions — there is nothing to compare across k values
- You are doing signature analysis sample filtering, where cola instead removes samples with silhouette scores < 0.5 — a different rule than the 5%-most-unstable PAC exclusion
- The dataset is scRNA-Seq with several thousands of cells, where the article notes cola runtime and memory usage may be unacceptable

## Inputs

- Per-k consensus partition results from a cola run (individual partitions, consensus partition, sample-sample consensus matrix)
- Sample distance matrix used for silhouette computation
- Vector of tested k values

## Outputs

- cola_metrics.csv with four metric columns (silhouette, 1-PAC, concordance, Jaccard) for every tested k plus stability flag and selected k
- Selected best k with the decision rule that produced it

## How to apply

Load the per-k consensus partition results from an upstream cola run; for each tested k compute the mean silhouette score of the consensus partition on the sample distance matrix. Compute PAC (proportion of ambiguous pairs in the consensus matrix) and convert to 1-PAC, recomputing PAC after excluding the 5% most unstable samples (those with the lowest mean consensus across their pairs) — cola's own stability filtering also removes ambiguous samples by silhouette score (default silhouette < 0.5 for signature analysis). Compute the concordance of each individual partition against the consensus partition for that k (averaged over individual partitions), and the Jaccard index between k-1 and k partitions based on pairwise sample co-membership. Apply the three-rule decision: (a) drop any k with Jaccard > 0.95 with k-1 as redundant; (b) among remaining k, take the maximal k with 1-PAC > 0.9; (c) if no k satisfies 1-PAC > 0.9, take the majority vote over the four metrics. Flag each k as stable when 1-PAC > 0.9 and write all metric values, flags, and the chosen k to cola_metrics.csv.

## Related tools

- **cola** (Source of the consensus partitioning framework whose best-k selection statistics (PAC, silhouette, concordance) are being reconstructed; R/Bioconductor package) — https://github.com/jokergoo/cola_manuscript
- **R** (Implementation language for recomputing the per-k metrics and decision rules)
- **Bioconductor** (Ecosystem in which the cola package providing the consensus partition results is distributed)
- **clue** (R package whose cl_consensus() function infers the consensus subgroup labels from individual partitions that feed the metric computation)

## Examples

```
consensus = cl_consensus(np_partitions); per_k_stats <- cbind(k = ks, silhouette = s, one_minus_pac = 1 - pac, concordance = conc, jaccard = jac); write.csv(per_k_stats, "cola_metrics.csv", row.names = FALSE)
```

## Evaluation signals

- cola_metrics.csv contains all four metric columns (silhouette, 1-PAC, concordance, Jaccard) for every tested k plus the stability flag and chosen k
- PAC values are recomputed after excluding the 5% most unstable samples (lowest mean consensus), not on the full consensus matrix
- Decision rules reproduce cola's thresholds: Jaccard > 0.95 drop rule, 1-PAC > 0.9 stability threshold, and majority-vote fallback when no k passes 1-PAC > 0.9
- Per-k stability flag equals (1-PAC > 0.9) and any k with Jaccard > 0.95 against k-1 is excluded from selection
- Concordance per k equals the mean agreement between individual partitions and the consensus partition for that k

## Limitations

- The 5%-most-unstable sample exclusion and the silhouette < 0.5 signature-analysis filter are distinct rules; applying the wrong one changes PAC and signature results
- The Jaccard > 0.95 redundancy rule can discard a larger k whose 1-PAC is better, which is intentional but may surprise users expecting a pure stability optimum
- cola runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells

## Evidence
