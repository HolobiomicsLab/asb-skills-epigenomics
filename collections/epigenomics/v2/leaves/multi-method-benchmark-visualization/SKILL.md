---
name: multi-method-benchmark-visualization
description: Use when comparing TR-identification methods on the same epigenomic query
  region sets and a known ground-truth perturbed TR exists, so ranks can be converted
  into mean reciprocal rank values for head-to-head benchmarking.
when_to_use_negative:
- No ground-truth perturbed TR is known for the input region set, so reciprocal ranks
  cannot be computed and the benchmark degenerates to qualitative comparison.
- Methods cannot be run on the same DAR BED inputs with the same underlying epigenomic
  profiling datasets — the comparison would confound input differences with method
  performance.
- The goal is single-method TR identification rather than head-to-head method benchmarking.
edam_operation: http://edamontology.org/operation_2928
tools:
- name: BIT
  role: Bayesian TR identification method under benchmark; produces ranked TR tables
    with BIT_score and Rank columns
  repo: https://github.com/ZeyuL01/BIT
- name: BART
  role: ChIP-seq binding-based comparison method run on the same DAR region sets
  repo: https://github.com/zanglab/bart2
- name: ChIP-Atlas
  role: ChIP-seq binding-based comparison method ranking TRs by Fisher's exact test
    p values
- name: I-cisTarget
  role: Motif-based comparison method using ENCODE references and recovery analysis
- name: WhichTF
  role: Motif-based comparison method using JASPAR/TRANSFAC references and hypergeometric
    test
- name: HOMER
  role: Motif-based comparison method using JASPAR/TRANSFAC references
- name: R
  role: Environment for running the BIT R package and computing MRR/plotting the benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:0da4d9f56e679d3cb3f9e655d8b86acffd016d3bb71b0591d78b0dfa1727cd09
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
  - ChIP-Atlas
  - HOMER
  - I-cisTarget
  - R
  - WhichTF
  edam_operation: http://edamontology.org/operation_2928
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# multi-method-benchmark-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark multiple TR-identification methods (BIT vs. BART, ChIP-Atlas, i-cisTarget, WhichTF, HOMER) by extracting each perturbed TR's rank from method-specific outputs, computing per-method mean reciprocal rank (MRR) over five bulk perturbation datasets, and reporting results as a Fig. 3f-style bar chart.

## When to use

Use when comparing TR-identification methods on the same epigenomic query region sets and a known ground-truth perturbed TR exists, so ranks can be converted into mean reciprocal rank values for head-to-head benchmarking.

## When NOT to use

- No ground-truth perturbed TR is known for the input region set, so reciprocal ranks cannot be computed and the benchmark degenerates to qualitative comparison.
- Methods cannot be run on the same DAR BED inputs with the same underlying epigenomic profiling datasets — the comparison would confound input differences with method performance.
- The goal is single-method TR identification rather than head-to-head method benchmarking.

## Inputs

- Differentially accessible region (DAR) BED files for CTCF, ZBTB7A, FOXA2, RUNX1, and BCL11B perturbations (e.g., via DiffBind report generation and export.bed)
- Ranked TR output tables from each method (BIT ranking tables with BIT_score and Rank columns; BART *_bart_results.txt; motif- and ChIP-based method outputs)
- Genome build specification matching the DARs (e.g., hg38 or mm10)

## Outputs

- Per-method MRR values written to mrr_benchmark.csv
- Bar chart of MRR per method in the style of Fig. 3f
- Per-TR reciprocal rank (1/rank) table per method

## How to apply

Run every method on identical DAR BED inputs derived from the five perturbed-TR bulk datasets (CTCF, ZBTB7A, FOXA2, RUNX1, BCL11B), using the same underlying epigenomic profiling data and matching genome build per method (BIT via its R package, BART via `bart2 region`, ChIP-Atlas via Fisher's exact test p values, i-cisTarget via ENCODE motif recovery, WhichTF/HOMER via JASPAR/TRANSFAC motifs). From each method's ranked TR output, extract the rank of the known perturbed TR for each dataset. Compute the reciprocal rank (1/rank) per TR and average over the five TRs to obtain the per-method MRR, then write the values to mrr_benchmark.csv and render a bar chart in the style of Fig. 3f. BIT's ranked output is identified by its BIT_score and Rank columns produced by the Gibbs-sampler-backed Bayesian hierarchical model.

## Related tools

- **BIT** (Bayesian TR identification method under benchmark; produces ranked TR tables with BIT_score and Rank columns) — https://github.com/ZeyuL01/BIT
- **BART** (ChIP-seq binding-based comparison method run on the same DAR region sets) — https://github.com/zanglab/bart2
- **ChIP-Atlas** (ChIP-seq binding-based comparison method ranking TRs by Fisher's exact test p values)
- **I-cisTarget** (Motif-based comparison method using ENCODE references and recovery analysis)
- **WhichTF** (Motif-based comparison method using JASPAR/TRANSFAC references and hypergeometric test)
- **HOMER** (Motif-based comparison method using JASPAR/TRANSFAC references)
- **R** (Environment for running the BIT R package and computing MRR/plotting the benchmark)

## Examples

```
bart2 region -i DARs.bed -c 4 -s hg38 --outdir bart2_output
```

## Evaluation signals

- Computed MRR values reproduce the reported Fig. 3f result in which BIT attains the highest MRR among the six methods.
- Per-method MRR values are checkable against the published figure bar heights.
- Each method's ranked output contains the perturbed TR so a finite rank (and thus reciprocal rank) is extractable for all five TRs.
- mrr_benchmark.csv contains exactly six methods (BIT, BART, ChIP-Atlas, i-cisTarget, WhichTF, HOMER) with MRRs each an average of five reciprocal ranks (each between 0 and 1).
- All methods were run on the same DAR BED inputs with the same underlying epigenomic profiling datasets and genome build.

## Limitations

- Benchmark results depend on NGS data quality; sequencing biases such as GC content bias and PCR amplification bias can reduce the reliability of the input epigenomic region sets and binding profiles.
- BIT's performance can be reduced by biological complexities of TR regulation, including dynamic TR binding across cell types and time, chromatin structure and epigenetic modifications, and cooperative binding among TRs.
- Comparison coverage is limited: currently gathered context-specific ChIP-seq binding profiles cover only a tiny part of all possible experiments, constraining methods like BIT and BART.

## Evidence

- [readme] BART region-mode invocation for DAR BED inputs: [evidence span withheld — anchor status: not_found]
