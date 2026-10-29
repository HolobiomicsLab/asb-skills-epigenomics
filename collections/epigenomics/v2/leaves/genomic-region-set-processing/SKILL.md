---
name: genomic-region-set-processing
description: Use when you hold a set of chromosome coordinates from epigenomic profiling
  (e.g., ATAC-seq DAR peaks called by MACS2 or DiffBind) and need to identify transcriptional
  regulators for that region set via BIT.
when_to_use_negative:
- Your input is already a ranked gene set or scored list rather than a genomic region
  set — BART's geneset mode (bart2 geneset -i genelist.txt) is designed for gene-set
  queries instead
- You need motif-based TR identification from sequences rather than overlap against
  real ChIP-seq binding profiles — BIT's reference-library approach explicitly bypasses
  computationally predicted motifs
- Your regions are on a genome build not covered by BIT's reference libraries (hg38
  or mm10)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: BIT
  role: 'Core tool: transforms region coordinates into fixed-length binary vectors,
    runs the Bayesian hierarchical model with Gibbs-sampler MCMC, and emits the TR
    rank table'
  repo: https://github.com/ZeyuL01/BIT
- name: R
  role: Runtime environment; BIT is distributed as an R package with a detailed manual
    on Read the Docs
  repo: https://github.com/ZeyuL01/BIT
- name: MACS2
  role: Upstream peak caller producing the chromosome/coordinate region set that BIT
    accepts as input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:4ae0f455a60c981ab8f0f2eb4f55a8a306eceddafc69fe19341e03e48177faae
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZeyuL01/BIT
  repo_url: https://github.com/ZeyuL01/BIT
  tools:
  - BIT
  - MACS2
  - R
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# genomic-region-set-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts a processed genomic region set (e.g., a BED file of differentially accessible peaks) into the fixed-length binary overlap vectors that BIT's Bayesian hierarchical model requires, then samples the joint posterior via a Gibbs-sampler MCMC to produce a per-TR ranked table (rank_table.csv) with Theta_i estimates, 95% credible intervals, and BIT scores.

## When to use

Use when you hold a set of chromosome coordinates from epigenomic profiling (e.g., ATAC-seq DAR peaks called by MACS2 or DiffBind) and need to identify transcriptional regulators for that region set via BIT.

## When NOT to use

- Your input is already a ranked gene set or scored list rather than a genomic region set — BART's geneset mode (bart2 geneset -i genelist.txt) is designed for gene-set queries instead
- You need motif-based TR identification from sequences rather than overlap against real ChIP-seq binding profiles — BIT's reference-library approach explicitly bypasses computationally predicted motifs
- Your regions are on a genome build not covered by BIT's reference libraries (hg38 or mm10)

## Inputs

- BED-format file of processed epigenomic regions containing chromosome numbers and coordinates (e.g., mm10_RUNX1.bed DAR peak set, peaks callable by MACS2)
- Genome build identifier (e.g., "mm10")
- BIT pre-processed TR ChIP-seq reference library (5,681 mouse / 10,140 human TR ChIP-seq datasets)

## Outputs

- rank_table.csv with columns TR, Theta_i, lower, upper, BIT_score, BIT_score_lower, BIT_score_upper, Rank
- Per-TR BIT scores and ranks derived from posterior draws

## How to apply

Import the chained differentially accessible region BED file (e.g., mm10_RUNX1.bed produced by the upstream DiffBind analysis) and load the BIT R package (ZeyuL01/BIT). Execute BIT(RUNX1_BED, output_path, genome="mm10"): BIT first transforms the coordinates of the processed peak file into fixed-length binary vectors of 0s and 1s, then formulates a Bayesian hierarchical model integrating the user-provided regions with TR binding information from the pre-compiled reference library (over 5,000 mouse TR ChIP-seq datasets; over 10,000 human datasets for hg38), sampling the joint posterior with a Gibbs-sampler MCMC. Read back the generated rank table with read.csv("./test/mm10_RUNX1_rank_table.csv", row.names=1) to obtain per-TR Theta_i estimates, lower/upper credible interval bounds, BIT scores (BIT_score, BIT_score_lower, BIT_score_upper), and Rank. Rationale: BIT leverages real ChIP-seq binding profiles rather than predicted motifs, retaining context-specific binding patterns and enabling uncertainty quantification via 95% credible intervals from posterior sampling.

## Related tools

- **BIT** (Core tool: transforms region coordinates into fixed-length binary vectors, runs the Bayesian hierarchical model with Gibbs-sampler MCMC, and emits the TR rank table) — https://github.com/ZeyuL01/BIT
- **R** (Runtime environment; BIT is distributed as an R package with a detailed manual on Read the Docs) — https://github.com/ZeyuL01/BIT
- **MACS2** (Upstream peak caller producing the chromosome/coordinate region set that BIT accepts as input)

## Examples

```
BIT(RUNX1_BED, output_path, genome="mm10"); rank_tab <- read.csv("./test/mm10_RUNX1_rank_table.csv", row.names=1)
```

## Evaluation signals

- rank_table.csv exists in the specified output path after BIT() completes
- Output table contains the expected columns: TR, Theta_i, lower, upper, BIT_score, BIT_score_lower, BIT_score_upper, Rank
- Head-of-table top-ranked TRs match reported results (e.g., Bcl11b, Cbfb, Runx1 for the RUNX1 KO DAR set)
- Credible interval bounds are coherent: lower <= Theta_i <= upper, and BIT_score lies within BIT_score_lower/BIT_score_upper
- Input BED is a valid region set with chromosome numbers and coordinates on the declared genome build (mm10)

## Limitations

- BIT can suffer from biological complexities of TR regulation (dynamic binding across cell types and time, chromatin structure and epigenetic modifications, cooperative binding), which can reduce performance
- Sequencing biases such as GC content bias or PCR amplification bias can reduce the reliability of the input epigenomic region set and binding profiles
- As ChIP-seq experiment availability grows, the number of parameters BIT infers increases significantly; analyzing dozens or hundreds of epigenomic profiling datasets with the Gibbs sampler can become time-consuming
- Currently gathered context-specific ChIP-seq binding profiles cover only a tiny part of all possible experiments

## Evidence
