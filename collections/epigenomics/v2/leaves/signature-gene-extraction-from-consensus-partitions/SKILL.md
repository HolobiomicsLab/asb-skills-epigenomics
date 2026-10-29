---
name: signature-gene-extraction-from-consensus-partitions
description: Use when a consensus partition result (e.g. from cola run_all_consensus_partitions
  over top-value methods SD/CV/MAD/ATC and partitioning methods such as skmeans) already
  exists and per-subgroup signature genes are needed for interpretation.
when_to_use_negative:
- Single-cell RNA-Seq datasets with several thousands of cells, where cola runtime
  and memory usage may be unacceptable
- Input is a raw, unprocessed expression matrix with missing values or outliers —
  run cola's preprocessing (adjust_matrix, impute.knn) and consensus partitioning
  first; signatures cannot be extracted before a stable partition exists
- No stable partition is available (e.g. 1-PAC scores are low for all method combinations)
  — signature extraction from an unstable partition is unreliable
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: cola
  role: Provides the consensus partitioning framework and signature extraction functionality;
    recommends the ATC top-value method with spherical k-means (skmeans) partitioning
    for subgroup classification
  repo: https://github.com/jokergoo/cola_manuscript
- name: impute
  role: impute.knn() fills missing expression values before partitioning so signatures
    are computed on a complete matrix
- name: NMF
  role: Partitioning method available within cola; ATC:NMF yielded a stable two-group
    partition (1-PAC = 0.97) on the Golub dataset
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:68c93254b888a1bcb2ca716944e98d3ca3762b4bdf478cbf558eb1c3ad73dc09
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
  - NMF
  - R
  - cola
  - impute
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# signature-gene-extraction-from-consensus-partitions

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract signature genes from a cola consensus partitioning result (e.g. the ATC:skmeans partition of the Golub leukemia dataset) by identifying rows whose expression uniquely defines each subgroup, while automatically excluding ambiguous samples with silhouette scores < 0.5.

## When to use

Use when a consensus partition result (e.g. from cola run_all_consensus_partitions over top-value methods SD/CV/MAD/ATC and partitioning methods such as skmeans) already exists and per-subgroup signature genes are needed for interpretation.

## When NOT to use

- Single-cell RNA-Seq datasets with several thousands of cells, where cola runtime and memory usage may be unacceptable
- Input is a raw, unprocessed expression matrix with missing values or outliers — run cola's preprocessing (adjust_matrix, impute.knn) and consensus partitioning first; signatures cannot be extracted before a stable partition exists
- No stable partition is available (e.g. 1-PAC scores are low for all method combinations) — signature extraction from an unstable partition is unreliable

## Inputs

- cola ConsensusPartition or ConsensusPartitionList result object (e.g. from run_all_consensus_partitions on the Golub leukemia matrix, k = 2 to 6)
- Preprocessed and row-scaled expression matrix (quantile-normalized, NA-imputed with impute.knn(), adjust_matrix()-cleaned)

## Outputs

- Signature gene list per subgroup (e.g. 1810 signature genes for ATC:skmeans k=2 on Golub)
- Saved result object golub_cola_results.rds
- Verification statistics table of extracted signature counts and stability metrics

## How to apply

First run cola consensus partitioning on the preprocessed expression matrix (quantile-normalized Golub data, values < 1 set to NA, imputed with impute.knn(), passed through adjust_matrix()) with row scaling applied before partitioning. Select the stable partition (e.g. ATC:skmeans with k=2, 1-PAC = 1) and call the cola signature extraction on that result for the chosen k. By default, cola removes samples with silhouette scores less than 0.5 from the signature analysis because they are ambiguous, so the signature set is computed only on confidently assigned samples. Verify the extracted signature set size against expected results — for the Golub ATC:skmeans k=2 partition, exactly 1810 signature genes — and save the full result object as an .rds file (golub_cola_results.rds) with verification statistics written to a table.

## Related tools

- **cola** (Provides the consensus partitioning framework and signature extraction functionality; recommends the ATC top-value method with spherical k-means (skmeans) partitioning for subgroup classification) — https://github.com/jokergoo/cola_manuscript
- **impute** (impute.knn() fills missing expression values before partitioning so signatures are computed on a complete matrix)
- **NMF** (Partitioning method available within cola; ATC:NMF yielded a stable two-group partition (1-PAC = 0.97) on the Golub dataset)

## Examples

```
res = golub_cola_results["ATC:skmeans"]; sig = get_signatures(res, k = 2)
```

## Evaluation signals

- The extracted ATC:skmeans k=2 signature set from the Golub dataset contains exactly 1810 genes
- Samples with silhouette scores < 0.5 are excluded from the signature analysis by default (they are ambiguous)
- Reported 1-PAC stability values for the selected partition lie in [0, 1] (ATC:kmeans = 1, ATC:NMF = 0.97, ATC:skmeans = 1 at k=2; ATC:skmeans = 0.96 at k=3)
- The full cola result object is saved as golub_cola_results.rds and verification statistics reproduce the reported values within a small numeric tolerance

## Limitations

- Samples with silhouette scores < 0.5 are dropped from signature analysis, which shrinks the effective sample set when many samples are ambiguous
- cola runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells
- No version number stated for the cola package or R in the manuscript, limiting reproducibility of versioned results

## Evidence
