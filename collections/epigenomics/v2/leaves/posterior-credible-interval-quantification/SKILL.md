---
name: posterior-credible-interval-quantification
description: Use when the input is an epigenomic region set (e.g., MACS2/DiffBind-derived
  ATAC-seq DARs as a BED file) and you need TR ranks with probabilistic uncertainty
  rather than a frequentist-only enrichment score.
when_to_use_negative:
- The query is a gene set rather than an epigenomic region set — BART's geneset mode
  (bart2 geneset) is the appropriate ChIP-seq-library-based alternative, not BIT's
  region-based Bayesian model
- A frequentist enrichment ranking without uncertainty is sufficient and MCMC runtime
  is prohibitive — ChIP-Atlas (Fisher's exact test) or I-cisTarget style recovery
  analysis are faster frequentist alternatives
- The region set comes from a genome without a prepared BIT reference library of TR
  ChIP-seq datasets
edam_operation: http://edamontology.org/operation_0306
tools:
- name: BIT
  role: Bayesian hierarchical model that samples the joint posterior via Gibbs-sampler
    MCMC and emits Theta_i, 95% credible intervals, and BIT scores for TR ranking
  repo: https://github.com/ZeyuL01/BIT
- name: R
  role: Runtime environment; BIT's R package is distributed on GitHub
  repo: https://github.com/ZeyuL01/BIT
- name: dplyr
  role: Splits the DiffBind report by Fold > 0 / Fold < 0 to produce increased- and
    decreased-accessibility BED inputs for separate posterior runs
- name: rtracklayer
  role: Imports the DAR BED files and exports headerless increased/decreased region
    BEDs
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:f1f39f806efae2126ddd2db2d27ca00feb229dc62dd84cd553510319c344283b
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
  - dplyr
  - rtracklayer
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# posterior-credible-interval-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify uncertainty in transcriptional regulator (TR) identification by running BIT's Bayesian hierarchical model, sampling the joint posterior via a Gibbs-sampler MCMC, and reporting per-TR Theta_i point estimates with 95% credible intervals alongside BIT_score_lower/BIT_score_upper bounds in a ranked table.

## When to use

Use when the input is an epigenomic region set (e.g., MACS2/DiffBind-derived ATAC-seq DARs as a BED file) and you need TR ranks with probabilistic uncertainty rather than a frequentist-only enrichment score.

## When NOT to use

- The query is a gene set rather than an epigenomic region set — BART's geneset mode (bart2 geneset) is the appropriate ChIP-seq-library-based alternative, not BIT's region-based Bayesian model
- A frequentist enrichment ranking without uncertainty is sufficient and MCMC runtime is prohibitive — ChIP-Atlas (Fisher's exact test) or I-cisTarget style recovery analysis are faster frequentist alternatives
- The region set comes from a genome without a prepared BIT reference library of TR ChIP-seq datasets

## Inputs

- Processed epigenomic region set as a BED file (e.g., mm10_RUNX1.bed with 15,652 DiffBind DAR ranges for mm10)
- Differential accessibility report (e.g., RUNX1_cKO_report.csv) with a Fold column for splitting increased/decreased regions
- BIT's pre-processed TR ChIP-seq reference library (human and mouse, bundled with the package)

## Outputs

- Rank table CSV (e.g., mm10_RUNX1_rank_table.csv) with per-TR Theta_i, 95% credible interval bounds (lower, upper), BIT_score, BIT_score_lower, BIT_score_upper, and Rank
- Per-TR posterior uncertainty quantification (95% credible intervals derived from posterior sampling)

## How to apply

Run BIT on the user-provided region BED (e.g., mm10_RUNX1.bed from a DiffBind analysis) with the correct genome argument, e.g. genome="mm10"; BIT transforms peak coordinates into fixed-length binary overlap vectors against a reference library of TR ChIP-seq datasets (10,140 human and 5,681 mouse datasets). It formulates a Bayesian hierarchical model integrating user-provided information with TR binding information across TRs and across multiple ChIP-seq datasets per TR, then uses a Gibbs sampler MCMC to draw from the joint posterior. Posterior draws yield per-TR Theta_i estimates, 95% credible interval bounds (lower/upper), and BIT scores with BIT_score_lower/BIT_score_upper, written to a rank_table.csv with columns TR, Theta_i, lower, upper, BIT_score, BIT_score_lower, BIT_score_upper, Rank. If uncertain-regulator identification is the goal, repeat the run on biologically split region sets (e.g., dplyr::filter(Fold > 0) for increased accessibility and Fold < 0 for decreased accessibility DARs) and compare which TRs reach Rank 1 in each.

## Related tools

- **BIT** (Bayesian hierarchical model that samples the joint posterior via Gibbs-sampler MCMC and emits Theta_i, 95% credible intervals, and BIT scores for TR ranking) — https://github.com/ZeyuL01/BIT
- **R** (Runtime environment; BIT's R package is distributed on GitHub) — https://github.com/ZeyuL01/BIT
- **dplyr** (Splits the DiffBind report by Fold > 0 / Fold < 0 to produce increased- and decreased-accessibility BED inputs for separate posterior runs)
- **rtracklayer** (Imports the DAR BED files and exports headerless increased/decreased region BEDs)

## Examples

```
BIT(RUNX1_BED, output_path, genome="mm10"); read.csv("./test/mm10_RUNX1_rank_table.csv", row.names=1)
```

## Evaluation signals

- Rank table exists and contains the expected columns: TR, Theta_i, lower, upper, BIT_score, BIT_score_lower, BIT_score_upper, Rank
- Each 95% credible interval brackets its point estimate (e.g., Bcl11b's reported interval 0.09860792-0.10371893 brackets Theta_i/BIT_score 0.10132149)
- Rank 1 entries reproduce reported values: Bcl11b 0.10132149 (full DAR set), Cdk6 0.09366096 (increased set), Runx1 0.06684889 (decreased set)
- Top-ranked TRs include the perturbed TR and experimentally validated partners (e.g., Bcl11b, Cbfb, Runx1 in the head-of-table output)
- Rows have no missing NA/NaN entries in the interval columns, confirming the Gibbs sampler converged to complete posterior draws

## Limitations

- Sequencing biases such as GC content bias (e.g., Illumina platform) and PCR amplification bias can reduce the reliability of the input epigenomic region set and binding profiles
- With growing ChIP-seq availability the number of parameters BIT must infer increases significantly; analyzing dozens or hundreds of epigenomic profiling datasets with the Gibbs sampler can become increasingly time-consuming
- Biological complexities of TR regulation — dynamic binding across cell types and time, chromatin structure and epigenetic modifications, and cooperative binding — can reduce BIT's performance

## Evidence

- [readme] Pre-processed reference library scale backing the posterior model: [evidence span withheld — anchor status: external_doc]
