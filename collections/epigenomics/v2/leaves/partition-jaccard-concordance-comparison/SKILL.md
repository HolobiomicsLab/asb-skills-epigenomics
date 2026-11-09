---
name: partition-jaccard-concordance-comparison
description: Decision support for choosing the number of subgroups k from cola consensus
  partitioning results, when per-k consensus matrices and partitions are available
  and a reproducible, threshold-based k-selection rule is needed.
when_to_use_negative:
- Samples already have trusted subgroup labels, so no k-selection is needed
- Only a single k was tested — the Jaccard redundancy rule requires consecutive pairs
  of k values (k-1 vs k)
- The input is an scRNA-Seq dataset with several thousands of cells, where cola's
  runtime and memory usage may be unacceptable
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: cola
  role: R/Bioconductor consensus partitioning package whose best-k selection rules
    (1-PAC with 5% unstable-sample exclusion, Jaccard > 0.95 drop rule, 1-PAC > 0.9
    stability threshold) this skill reproduces
  repo: https://github.com/jokergoo/cola_manuscript
- name: R
  role: Language for recomputing silhouette, 1-PAC, concordance and Jaccard metrics
    from cola output objects
- name: Bioconductor
  role: Distribution ecosystem for the cola package used to generate the per-k partitions
    and consensus matrices
- name: clue
  role: Provides cl_consensus() for inferring consensus subgroup labels from individual
    partitions, the object on which concordance is computed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:04fe368d8491b4fb6521e7405b785a6bcf43a9471e0e7047f0cf2a19b7cca027
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
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# partition-jaccard-concordance-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs cola's best-k selection decision procedure by computing four per-k metrics — 1-PAC (proportion of ambiguous pairs, with the 5% most unstable samples excluded), mean silhouette, partition-concordance, and consecutive-k Jaccard indices — and applying cola's three-rule decision procedure to choose the optimal number of subgroups.

## When to use

Decision support for choosing the best number of subgroups k from cola consensus partitioning results, when per-k consensus matrices and partitions are available and a reproducible, threshold-based k-selection rule is needed.

## When NOT to use

- Samples already have trusted subgroup labels, so no k-selection is needed
- Only a single k was tested — the Jaccard redundancy rule requires consecutive pairs of k values (k-1 vs k)
- The input is an scRNA-Seq dataset with several thousands of cells, where cola's runtime and memory usage may be unacceptable

## Inputs

- Per-k individual partitions from a cola consensus partitioning run
- Per-k consensus partition (subgroup labels per sample)
- Per-k sample-sample consensus matrix
- Sample distance matrix (for silhouette computation)

## Outputs

- cola_metrics.csv with four metric columns (1-PAC, silhouette, concordance, Jaccard) for every tested k
- Per-k stability flag (1-PAC > 0.9)
- Selected best k value

## How to apply

Load the per-k outputs of a cola consensus partitioning run: individual partitions, the consensus partition, and the sample-sample consensus matrix. For each tested k, compute the mean silhouette score of the consensus partition on the sample distance matrix, and compute PAC (proportion of ambiguous pairs in the consensus matrix) converted to 1-PAC, recalculating PAC after excluding the 5% most unstable samples (those with the lowest mean consensus across their pairs). Also compute the concordance between each individual partition and the consensus partition (averaged over individual partitions), and the Jaccard index between the k-1 and k partitions based on pairwise sample co-membership. Apply cola's three-rule decision procedure: (a) drop any k whose Jaccard index with k-1 is > 0.95 as redundant; (b) among remaining k, take the maximal k with 1-PAC > 0.9; (c) if no k satisfies 1-PAC > 0.9, take the majority vote over the four metrics. Write all per-k metric values, a stability flag (1-PAC > 0.9), and the selected k to a metrics CSV for downstream checks.

## Related tools

- **cola** (R/Bioconductor consensus partitioning package whose best-k selection rules (1-PAC with 5% unstable-sample exclusion, Jaccard > 0.95 drop rule, 1-PAC > 0.9 stability threshold) this skill reproduces) — https://github.com/jokergoo/cola_manuscript
- **R** (Language for recomputing silhouette, 1-PAC, concordance and Jaccard metrics from cola output objects)
- **Bioconductor** (Distribution ecosystem for the cola package used to generate the per-k partitions and consensus matrices)
- **clue** (Provides cl_consensus() for inferring consensus subgroup labels from individual partitions, the object on which concordance is computed)

## Examples

```
consensus <- clue::cl_consensus(individual_partitions)  # consensus partition on which silhouette, 1-PAC and concordance per k are computed
```

## Evaluation signals

- cola_metrics.csv contains all four metric columns (1-PAC, silhouette, concordance, Jaccard) for every tested k, plus the stability flag and the chosen k
- PAC values are recomputed after excluding the 5% most unstable samples (lowest mean consensus across their pairs), matching cola's internal rule
- Decision rules reproduce cola thresholds: Jaccard index > 0.95 drop rule, 1-PAC > 0.9 stability threshold, maximal-k rule, and majority-vote fallback
- Each k flagged stable satisfies 1-PAC > 0.9
- Jaccard indices are computed from pairwise sample co-membership between consecutive partitions (k-1 vs k), not from label agreement

## Limitations

- For scRNA-Seq datasets with several thousands of cells, cola's runtime and memory usage might not be acceptable
- Downsampling strategies in cola might not detect cell clusters with small sizes
- Excluding the 5% most unstable samples means the reported 1-PAC is not a raw PAC over all samples and can differ from PAC computed elsewhere

## Evidence

- [other] 1-PAC as best-k metric with 5% unstable-sample exclusion: [evidence span withheld — anchor status: not_found]
