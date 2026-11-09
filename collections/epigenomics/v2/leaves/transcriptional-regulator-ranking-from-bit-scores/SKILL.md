---
name: transcriptional-regulator-ranking-from-bit-scores
description: Use when you have a processed peak/DAR region set (BED with chromosome
  coordinates, e.g., from DiffBind or MACS2) and need to identify and rank candidate
  transcriptional regulators with uncertainty estimates.
when_to_use_negative:
- Input is a gene set rather than an epigenomic region set — BART's geneset mode or
  motif-based tools like WhichTF/HOMER are alternatives, but BIT specifically requires
  region coordinates
- You need frequentist enrichment p-values (e.g., Fisher exact test rankings as in
  ChIP-Atlas) rather than Bayesian posterior scores with credible intervals
- The region set comes from a genome build not supported by the pre-compiled reference
  libraries (hg38 or mm10)
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3179
- http://edamontology.org/topic_0749
tools:
- name: BIT
  role: 'Core tool: Bayesian hierarchical model that transforms region coordinates
    to binary overlap vectors against the TR ChIP-seq library and samples the joint
    posterior via Gibbs sampler MCMC to assign per-TR BIT scores'
  repo: https://github.com/ZeyuL01/BIT
- name: R
  role: Runtime environment; BIT is distributed as an R package with a detailed manual
    on Read the Docs
  repo: https://github.com/ZeyuL01/BIT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:6ff92e40d0c2dc2a1bc8974a0cea243886e60180cc5d60e48a4c323248853216
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
  - R
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3179
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# transcriptional-regulator-ranking-from-bit-scores

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs the BIT R package on a user-provided epigenomic region set (e.g., an mm10 RUNX1 KO differentially accessible region BED file) to produce a ranked table of transcriptional regulators based on posterior-derived BIT scores from a Bayesian hierarchical model sampled via Gibbs-sampler MCMC.

## When to use

Use when you have a processed peak/DAR region set (BED with chromosome coordinates, e.g., from DiffBind or MACS2) and need to identify and rank candidate transcriptional regulators with uncertainty estimates.

## When NOT to use

- Input is a gene set rather than an epigenomic region set — BART's geneset mode or motif-based tools like WhichTF/HOMER are alternatives, but BIT specifically requires region coordinates
- You need frequentist enrichment p-values (e.g., Fisher exact test rankings as in ChIP-Atlas) rather than Bayesian posterior scores with credible intervals
- The region set comes from a genome build not supported by the pre-compiled reference libraries (hg38 or mm10)

## Inputs

- BED file of differentially accessible regions with chromosome numbers and coordinates (e.g., mm10_RUNX1.bed from DiffBind analysis)
- Genome build specification (e.g., 'mm10' or 'hg38')
- BIT R package with pre-compiled TR ChIP-seq reference library

## Outputs

- rank_table.csv with columns TR, Theta_i, lower, upper, BIT_score, BIT_score_lower, BIT_score_upper, Rank
- Per-TR BIT scores with 95% credible intervals and ranks (e.g., top TRs Bcl11b, Cbfb, Runx1 for the RUNX1 KO DAR set)

## How to apply

Load the BIT R package and import the chained differentially accessible region BED file (e.g., mm10_RUNX1.bed produced by upstream DiffBind analysis). Execute BIT(RUNX1_BED, output_path, genome='mm10'), which transforms the region coordinates into fixed-length binary overlap vectors of 0s and 1s against the TR ChIP-seq reference library (5,681 mouse TR ChIP-seq datasets; 10,140 human datasets covering 988 TRs), then samples the joint posterior via a Gibbs sampler MCMC. Read back the generated output with read.csv('./test/mm10_RUNX1_rank_table.csv', row.names=1) to obtain per-TR Theta_i estimates, 95% credible interval bounds, BIT scores, and ranks. The rationale is that BIT assesses genome-wide consistency between the query region set and TR binding profiles while accounting for between- and within-TR heterogeneity and quantifying uncertainty, unlike frequentist enrichment methods.

## Related tools

- **BIT** (Core tool: Bayesian hierarchical model that transforms region coordinates to binary overlap vectors against the TR ChIP-seq library and samples the joint posterior via Gibbs sampler MCMC to assign per-TR BIT scores) — https://github.com/ZeyuL01/BIT
- **R** (Runtime environment; BIT is distributed as an R package with a detailed manual on Read the Docs) — https://github.com/ZeyuL01/BIT

## Examples

```
BIT(RUNX1_BED, output_path, genome="mm10"); rank_table <- read.csv("./test/mm10_RUNX1_rank_table.csv", row.names=1)
```

## Evaluation signals

- rank_table.csv exists at the specified output path and contains the expected columns: TR, Theta_i, lower, upper, BIT_score, BIT_score_lower, BIT_score_upper, Rank
- Top-ranked TRs match the reported head-of-table output (e.g., Bcl11b, Cbfb, Runx1 for the mm10 RUNX1 KO DAR set)
- Every TR row has non-decreasing lower <= Theta_i <= upper across the 95% credible interval bounds
- Ranks in the Rank column are a complete, consistent ordering of the BIT_score column (descending)
- Input BED coordinates are valid for the declared genome build (mm10) so binary overlap vectors against the reference library are formed correctly

## Limitations

- BIT's performance can be reduced by biological complexities of TR regulation, including dynamic TR binding across cell types and time, chromatin structure and epigenetic modifications affecting binding affinity/specificity, and cooperative binding among TRs
- Sequencing biases such as GC content bias (e.g., on the Illumina platform) and PCR amplification bias can reduce the reliability of the input epigenomic region set and binding profiles generated by NGS
- With the growing availability of ChIP-seq experiments, the number of parameters for BIT to infer is expected to increase significantly, and analyzing dozens or hundreds of epigenomic profiling datasets with the Gibbs sampler can become increasingly time-consuming

## Evidence
