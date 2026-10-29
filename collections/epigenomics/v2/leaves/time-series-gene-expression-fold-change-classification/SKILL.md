---
name: time-series-gene-expression-fold-change-classification
description: Use when a time-series expression matrix (e.g. LR RNA-seq time course)
  must be split into submatrices at candidate branching points of a tree-structure
  model, prior to Baum-Welch training in TSMiner.
when_to_use_negative:
- The expression data are not time-series (no ordered time points to compute fold
  changes against the preceding time point).
- The goal is static gene-set or pathway enrichment without needing temporal bifurcation
  structure or TF timing (e.g. single-condition RNA-seq).
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: TSMiner
  role: Implements the bifurcation-pattern classification, nc vector computation,
    likelihood-ratio partitioning, and Baum-Welch initialization on the time-series
    expression matrix
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime platform required to execute TSMiner.jar (Java 1.5 or later, 1024
    MB heap)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:aea8d6557450b9bd444b19bf44b1daa3e347f1d561746142c91b0909ddb02de5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/free1234hm/tsminer-tool
  repo_url: https://github.com/free1234hm/tsminer-tool.git
  tools:
  - Java
  - TSMiner
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# time-series-gene-expression-fold-change-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies each gene at every candidate bifurcation time point of a time-series expression matrix as up-regulated, down-regulated, or unchanged based on its expression fold change relative to the preceding time point, building the three-dimensional comparison vector nc that TSMiner uses to define bifurcation patterns for tree-structure model initialization.

## When to use

Use when a time-series expression matrix (e.g. LR RNA-seq time course) must be split into submatrices at candidate branching points of a tree-structure model, prior to Baum-Welch training in TSMiner.

## When NOT to use

- The expression data are not time-series (no ordered time points to compute fold changes against the preceding time point).
- The goal is static gene-set or pathway enrichment without needing temporal bifurcation structure or TF timing (e.g. single-condition RNA-seq).

## Inputs

- Time-series expression matrix (e.g. mouse liver regeneration LR RNA-seq dataset, 'Time-series data' directory)
- TF-gene interaction file ('TF-gene file' directory)
- Parameter settings file (Defaults_for_case_analysis.txt)

## Outputs

- Per-gene three-dimensional comparison vector nc (up/down/unchanged counts over subsequent time points)
- Partition of gene set G into bifurcation-pattern groups defining candidate branching points
- Initialized input-output hidden Markov tree-structure model parameters for Baum-Welch training

## How to apply

Load the time-series expression data (from the 'Time-series data' directory) and the TF-gene interaction file (from the 'TF-gene file' directory) with the parameters listed in Defaults_for_case_analysis.txt. At each candidate bifurcation time point, classify every gene as up-regulated, down-regulated, or unchanged by comparing its expression fold change to the preceding time point. For each gene, summarize these calls over subsequent time points into a three-dimensional comparison vector nc (up/down/unchanged counts). Apply a likelihood ratio test to partition the gene set G into groups sharing a bifurcation pattern; these groups define the candidate split (branching) points of the tree-structure model. The bifurcation patterns then initialize the Baum-Welch training of the input-output hidden Markov model, iterating until the likelihood score converges; the rationale is that TSMiner addresses DREM's limitation of dependence on initial parameters without effective initialization.

## Related tools

- **TSMiner** (Implements the bifurcation-pattern classification, nc vector computation, likelihood-ratio partitioning, and Baum-Welch initialization on the time-series expression matrix) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime platform required to execute TSMiner.jar (Java 1.5 or later, 1024 MB heap))

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- Every gene at each candidate bifurcation time point has exactly one of three calls: up-regulated, down-regulated, or unchanged.
- Each gene yields a well-formed 3D comparison vector nc whose components are non-negative counts summing over subsequent time points.
- The likelihood-ratio-test partition of G covers all genes and the resulting splits correspond to candidate branching points of the tree.
- Baum-Welch training starting from these bifurcation patterns converges (likelihood score stops increasing).
- The trained model loads in TSMiner and its Viterbi-derived bifurcation structure is consistent with the example saved models in the 'Saved model' directory.

## Limitations

- Fold-change classification at bifurcation points assumes ordered, comparable time points; other time-series omics types (e.g. proteomics with PPI information) are not yet demonstrated applicability.
- Results depend on the TF-gene interaction input; negatively correlated TF-TG interactions are underrepresented because known repressors of many genes are limited.
- Requires Java 1.5+ and at least 1024 MB Java heap memory; precise regulatory mechanisms require more rigorous evaluation.

## Evidence

- [other] TSMiner divides each path in the current model based on a complete set of bifurcation patterns, calculates initial parameters for each division, and trains the parameters using the Baum–Welch algorithm, repeating until the likelihood score converges.: [evidence span withheld — anchor status: not_found]
- [methods] Classify each gene at every candidate bifurcation time point as up-regulated, down-regulated, or unchanged based on its expression fold change relative to the preceding time point.: [evidence span withheld — anchor status: not_found]
