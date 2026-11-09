---
name: likelihood-ratio-test-gene-set-partitioning
description: Use when a time-series expression matrix must be split into bifurcation
  groups to define candidate branching points of a tree-structure (input-output hidden
  Markov) model prior to Baum-Welch training, as in TSMiner's LR mouse liver regeneration
  RNA-seq analysis.
when_to_use_negative:
- The expression data are not time-series (no successive time points), so fold changes
  relative to a preceding time point and the nc comparison vector cannot be computed
- The goal is static clustering of profiles without a tree-structure model — the article
  notes clustering methods cannot evaluate the temporal dynamics of regulators or
  pathways, so this partitioning is instead intended to seed bifurcations of a time-specific
  regulatory model
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: TSMiner
  role: Tool whose bifurcation-pattern division module implements the fold-change
    classification, nc vector computation, and likelihood ratio test partitioning
    of G before Baum-Welch initialization
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Language in which TSMiner's core algorithms (including this partitioning and
    Baum-Welch training) are written; run via java -mx1024M -jar TSMiner.jar
- name: DREM
  role: Prior tree-structure framework whose reliance on Baum-Welch training without
    effective initialization of parameters motivated TSMiner's bifurcation-pattern-based
    partitioning
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:239e2d03cc264068ded3b03a7b9921026dca0a9e6285652750837512bc9eabca
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
  - DREM
  - Java
  - TSMiner
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# likelihood-ratio-test-gene-set-partitioning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A statistical partitioning step from TSMiner that classifies genes at candidate bifurcation time points of a time-series expression matrix (up-, down-, or unchanged relative to the preceding time point) and applies a likelihood ratio test to divide the gene set G into groups, defining the candidate branching (split) points of a tree-structure model.

## When to use

Use when a time-series expression matrix must be split into bifurcation groups to define candidate branching points of a tree-structure (input-output hidden Markov) model prior to Baum-Welch training, as in TSMiner's LR mouse liver regeneration RNA-seq analysis.

## When NOT to use

- The expression data are not time-series (no successive time points), so fold changes relative to a preceding time point and the nc comparison vector cannot be computed
- The goal is static clustering of profiles without a tree-structure model — the article notes clustering methods cannot evaluate the temporal dynamics of regulators or pathways, so this partitioning is instead intended to seed bifurcations of a time-specific regulatory model

## Inputs

- Time-series expression matrix (e.g. the mouse liver regeneration RNA-seq dataset from the 'Time-series data' directory)
- Candidate bifurcation time points of the current tree-structure model path
- TF-gene interaction file (from the 'TF-gene file' directory), used downstream for network construction

## Outputs

- Partitioned gene set G, with genes grouped by bifurcation pattern via the likelihood ratio test
- Candidate split (branching) points of the tree-structure model
- Initial parameters for each bifurcation division, used to start Baum-Welch training

## How to apply

From the time-series expression matrix, classify each gene at every candidate bifurcation time point as up-regulated, down-regulated, or unchanged based on its expression fold change relative to the preceding time point. For each gene, summarize these calls into a three-dimensional comparison vector nc holding the counts of up/down/unchanged states over the subsequent time points. Apply a likelihood ratio test to the nc vectors to partition the gene set G into groups with statistically similar bifurcation behavior; these groups define the candidate split points of the tree-structure model. The resulting bifurcation patterns are then used to calculate initial parameters for each division and to initialize Baum-Welch training of TSMiner's input-output hidden Markov model, iterating until the likelihood score converges, after which Viterbi decoding assigns each TF to the time point at which it acts on its targets.

## Related tools

- **TSMiner** (Tool whose bifurcation-pattern division module implements the fold-change classification, nc vector computation, and likelihood ratio test partitioning of G before Baum-Welch initialization) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Language in which TSMiner's core algorithms (including this partitioning and Baum-Welch training) are written; run via java -mx1024M -jar TSMiner.jar)
- **DREM** (Prior tree-structure framework whose reliance on Baum-Welch training without effective initialization of parameters motivated TSMiner's bifurcation-pattern-based partitioning)

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- Every gene in G is assigned to exactly one partition group, with each group's membership determined by the likelihood ratio test on the three-dimensional nc comparison vector (up/down/unchanged counts)
- The partitioned bifurcation patterns yield initial parameters whose Baum-Welch training reaches a converged likelihood score
- The trained model loads in TSMiner and its Viterbi-derived bifurcation structure is consistent with the example saved models in the 'Saved model' directory
- TF assignment times from Viterbi decoding agree with the example saved models provided with the tool
- Reproducing the mouse LR case analysis with the parameters in Defaults_for_case_analysis.txt yields the default partitioning behavior

## Limitations

- DREM and its extended methods depend on initial parameters without effective initialization — this partitioning step exists specifically to supply that initialization, so skipping it risks poor tree-structure model fitting
- Fewer results arise from negative TF-TG interactions than positive ones because known repressors of many genes are limited, which constrains downstream interpretation of repressor bifurcations
- Requires Java 1.5 or later and at least 1024 MB Java heap memory; the run environment must support java -mx1024M

## Evidence
