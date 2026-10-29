---
name: bayesian-hierarchical-model-mcmc-inference
description: Use when the input is a processed peak/region set (BED from MACS2, DiffBind,
  or ArchR DARs) and the goal is an in-silico ranked list of TRs with credible intervals
  rather than motif-based or frequentist enrichment.
when_to_use_negative:
- When the question is motif enrichment rather than context-specific binding — BIT
  deliberately bypasses computationally predicted motifs (as used by WhichTF/HOMER),
  so motif-only tools may be preferable for de novo motif discovery.
- When the input is raw sequencing data (FASTQ/BAM) — BIT only requires processed
  peak coordinates; upstream peak calling (MACS2 q 0.01) and DAR calling (DiffBind,
  FDR <= 0.05 & abs(Log2FC) >= 2) must be done first.
- When a very large collection of epigenomic profiling datasets must be analyzed and
  runtime is critical — the Gibbs sampler can become increasingly time-consuming as
  the number of ChIP-seq experiments grows.
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
tools:
- name: BIT
  role: Core R package implementing the Bayesian hierarchical model and Gibbs-sampler
    MCMC that converts a query region BED into a ranked TR table with credible intervals
  repo: https://github.com/ZeyuL01/BIT
- name: R
  role: Execution environment for the BIT package; read.csv() is used to load the
    returned rank table
- name: MACS2
  role: Upstream peak caller producing the processed region files that serve as BIT's
    input
- name: BART
  role: ChIP-seq-based frequentist alternative for TR identification against which
    BIT is benchmarked
  repo: https://github.com/zanglab/bart2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:49faef41cb099019bc0126e7276187046ef2ad165b1ba4b0fa7b2247a4b779e4
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
  - BART
  - BIT
  - MACS2
  - R
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bayesian-hierarchical-model-mcmc-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run BIT, an R package implementing a Bayesian hierarchical model with Gibbs-sampler MCMC, to convert a user-provided epigenomic region set (e.g., ATAC-seq DAR peaks such as mm10_RUNX1.bed) into a ranked table of transcriptional regulators with posterior uncertainty quantification.

## When to use

Use when the input is a processed peak/region set (BED from MACS2, DiffBind, or ArchR DARs) and the goal is an in-silico ranked list of TRs with credible intervals rather than motif-based or frequentist enrichment.

## When NOT to use

- When the question is motif enrichment rather than context-specific binding — BIT deliberately bypasses computationally predicted motifs (as used by WhichTF/HOMER), so motif-only tools may be preferable for de novo motif discovery.
- When the input is raw sequencing data (FASTQ/BAM) — BIT only requires processed peak coordinates; upstream peak calling (MACS2 q 0.01) and DAR calling (DiffBind, FDR <= 0.05 & abs(Log2FC) >= 2) must be done first.
- When a very large collection of epigenomic profiling datasets must be analyzed and runtime is critical — the Gibbs sampler can become increasingly time-consuming as the number of ChIP-seq experiments grows.

## Inputs

- BED file of query epigenomic regions with chromosome and coordinates (e.g., mm10_RUNX1.bed from DiffBind DAR analysis; peaks called by MACS2 also acceptable)
- Genome assembly identifier (e.g., "mm10" or human equivalent)
- BIT R package with its pre-processed TR ChIP-seq reference library (10,140 human / 5,681 mouse datasets)

## Outputs

- rank_table.csv (e.g., mm10_RUNX1_rank_table.csv) with per-TR columns: TR, Theta_i, lower, upper, BIT_score, BIT_score_lower, BIT_score_upper, Rank
- Ranked list of transcriptional regulators with 95% credible intervals from posterior sampling

## How to apply

Load the BIT R package (github.com/ZeyuL01/BIT) and call BIT(RUNX1_BED, output_path, genome="mm10"), passing a BED of differentially accessible regions (e.g., from DiffBind after dba.blacklist and dba.count). BIT transforms the region coordinates into fixed-length binary overlap vectors against its pre-compiled reference library of over 5,000 mouse (and 10,000+ human) TR ChIP-seq datasets, then formulates a Bayesian hierarchical model integrating the user data with TR binding information and samples the joint posterior via a Gibbs sampler MCMC. Posterior draws yield per-TR Theta_i estimates, 95% credible interval bounds, BIT scores, and ranks. Read the output with read.csv('./test/mm10_RUNX1_rank_table.csv', row.names=1) and inspect the top-ranked TRs. Rationale: this Bayesian approach assesses genome-wide consistency between the query regions and the binding library while bypassing motif prediction and retaining context-specific binding patterns.

## Related tools

- **BIT** (Core R package implementing the Bayesian hierarchical model and Gibbs-sampler MCMC that converts a query region BED into a ranked TR table with credible intervals) — https://github.com/ZeyuL01/BIT
- **R** (Execution environment for the BIT package; read.csv() is used to load the returned rank table)
- **MACS2** (Upstream peak caller producing the processed region files that serve as BIT's input)
- **BART** (ChIP-seq-based frequentist alternative for TR identification against which BIT is benchmarked) — https://github.com/zanglab/bart2

## Examples

```
BIT(RUNX1_BED, output_path, genome="mm10"); rank_table <- read.csv("./test/mm10_RUNX1_rank_table.csv", row.names=1)
```

## Evaluation signals

- The output rank table exists and contains the expected columns: TR, Theta_i, lower, upper, BIT_score, BIT_score_lower, BIT_score_upper, Rank.
- The head of the rank table matches reported top-ranked TRs (e.g., Bcl11b, Cbfb, Runx1 for the Runx1 KO DAR set).
- The 95% credible interval columns (lower/upper, BIT_score_lower/BIT_score_upper) are populated and consistent with posterior sampling, confirming the Gibbs sampler converged to usable draws.
- Each TR in the output is a member of the reference library (988 human TRs / mouse equivalent), confirming the binary overlap vectors were built against the correct genome build (e.g., mm10).
- Downstream validation: top TRs can be cross-checked via GO enrichment (one-sided Fisher's exact tests, Benjamini–Hochberg adjusted p values) or perturbation/DepMap data as done in the paper.

## Limitations

- BIT can suffer from the biological complexities of TR regulation — dynamic TR binding across cell types and time, chromatin structure and epigenetic modifications affecting binding affinity/specificity, and cooperative binding among TRs.
- Performance depends on NGS data quality: GC content bias (e.g., Illumina platform) and PCR amplification bias can reduce the reliability of the input epigenomic region set and the TR binding profiles.
- The number of parameters to infer grows with the expanding ChIP-seq reference library, and analyzing dozens or hundreds of epigenomic profiling datasets with the Gibbs sampler can become increasingly time-consuming.
- Context-specific ChIP-seq binding profiles cover only a tiny fraction of all possible experiments, limiting coverage of TR–context combinations.

## Evidence

- [results] Bayesian hierarchical model with MCMC sampling: "BIT formulates a Bayesian hierarchical model to integrate user-provided information with TR binding information from the reference library and then uses Markov Chain Monte Carlo (MCMC) to sample from"
- [results] Gibbs sampler draws posterior samples: "A Gibbs sampler is used to draw posterior samples and estimate key model parameters."
