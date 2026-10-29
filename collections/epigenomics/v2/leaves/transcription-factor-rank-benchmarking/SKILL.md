---
name: transcription-factor-rank-benchmarking
description: Use when comparing TR-identification tools on query region sets where
  the true perturbed/active TR is known, and a rank-based metric (MRR) over ranked
  TR output is needed for fair method comparison.
when_to_use_negative:
- No ground-truth perturbed TR is known for the query region set, so a reciprocal
  rank cannot be computed.
- Input is raw sequencing data rather than processed peak/region sets — peak calling
  (e.g., MACS2 q-value 0.01) must precede this benchmarking step.
- Only one method is being run; MRR is a cross-method comparison metric and is uninformative
  without competing ranked outputs.
edam_topics:
- http://edamontology.org/topic_0749
tools:
- name: BIT
  role: Bayesian hierarchical model that ranks TRs from user-provided epigenomic region
    sets, producing ranking tables with BIT_score and Rank columns
  repo: https://github.com/ZeyuL01/BIT
- name: BART
  role: ChIP-seq binding-based comparator run in region mode on the scored genomic
    region (DAR) BED inputs
  repo: https://github.com/zanglab/bart2
- name: ChIP-Atlas
  role: ChIP-seq-based comparator that ranks TRs using p values from Fisher's exact
    tests
- name: I-cisTarget
  role: Motif-based comparator using ENCODE references and recovery analysis
- name: WhichTF
  role: Motif-based comparator leveraging JASPAR/TRANSFAC references with a hypergeometric
    test
- name: HOMER
  role: Motif-based comparator leveraging JASPAR/TRANSFAC references
- name: R
  role: Language of the BIT R package used to run BIT and compute MRR values
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:a45f2ebb099dedc6c494712bbef85c771dbdb691b2fffcacbedd920e00592114
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# transcription-factor-rank-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark TR-identification methods by computing mean reciprocal rank (MRR) of known perturbed TRs across methods' ranked TR output lists, as done to show BIT outperforms BART, ChIP-Atlas, i-cisTarget, WhichTF, and HOMER on five bulk perturbation DAR datasets (Fig. 3f).

## When to use

Use when comparing TR-identification tools on query region sets where the true perturbed/active TR is known, and a rank-based metric (MRR) over ranked TR output is needed for fair method comparison.

## When NOT to use

- No ground-truth perturbed TR is known for the query region set, so a reciprocal rank cannot be computed.
- Input is raw sequencing data rather than processed peak/region sets — peak calling (e.g., MACS2 q-value 0.01) must precede this benchmarking step.
- Only one method is being run; MRR is a cross-method comparison metric and is uninformative without competing ranked outputs.

## Inputs

- Differentially accessible region BED files (hg38 or mm10) per perturbed TR: CTCF, ZBTB7A, FOXA2, RUNX1, BCL11B
- Ranked TR output tables per method (e.g., BIT ranking tables with BIT_score and Rank columns)
- Same underlying ChIP-seq profiling datasets / reference libraries for all methods (GTRD, CistromeDB, ENCODE, JASPAR/TRANSFAC, GTRD-derived BIT library)

## Outputs

- Per-method, per-TR rank and reciprocal rank (1/rank) values
- Per-method MRR values (mean reciprocal rank over the five perturbed TRs) written to mrr_benchmark.csv
- Bar chart of MRR per method in the style of Fig. 3f

## How to apply

Start from differentially accessible region BED files for each perturbation (e.g., DiffBind report generation and export.bed in the RUNX1 cKO workflow) for CTCF, ZBTB7A, FOXA2, RUNX1, and BCL11B. Run each method on the identical DAR inputs with the same underlying epigenomic profiling data: ChIP-seq binding-based methods (BART in region mode, ChIP-Atlas), motif-based methods with ENCODE references (i-cisTarget), and motif-based methods with JASPAR/TRANSFAC references (WhichTF, HOMER), plus BIT on the matching genome build. From each method's ranked TR output, extract the rank of the perturbed TR, compute reciprocal rank (1/rank) per TR, and average over the five TRs to get per-method MRR. Write per-method MRR values to mrr_benchmark.csv and visualize as a bar chart in the style of Fig. 3f.

## Related tools

- **BIT** (Bayesian hierarchical model that ranks TRs from user-provided epigenomic region sets, producing ranking tables with BIT_score and Rank columns) — https://github.com/ZeyuL01/BIT
- **BART** (ChIP-seq binding-based comparator run in region mode on the scored genomic region (DAR) BED inputs) — https://github.com/zanglab/bart2
- **ChIP-Atlas** (ChIP-seq-based comparator that ranks TRs using p values from Fisher's exact tests)
- **I-cisTarget** (Motif-based comparator using ENCODE references and recovery analysis)
- **WhichTF** (Motif-based comparator leveraging JASPAR/TRANSFAC references with a hypergeometric test)
- **HOMER** (Motif-based comparator leveraging JASPAR/TRANSFAC references)
- **R** (Language of the BIT R package used to run BIT and compute MRR values)

## Examples

```
bart2 region -i ChIPpeak.bed -c 4 -s hg38 --outdir bart2_output
```

## Evaluation signals

- Computed MRR per method reproduces the reported Fig. 3f ordering, with BIT attaining the highest MRR among the six methods.
- For each of the five perturbed TRs (CTCF, ZBTB7A, FOXA2, RUNX1, BCL11B), the perturbed TR appears in each method's ranked TR output so a 1/rank value exists.
- Per-method MRR equals the mean of 1/rank over exactly the five bulk perturbation TRs (spot-checkable against the published figure).
- All methods were run on the same DAR BED inputs and the same underlying epigenomic profiling datasets and genome build, so ranks are comparable.
- Output CSV (mrr_benchmark.csv) contains one MRR row per method (BIT, BART, ChIP-Atlas, I-cisTarget, WhichTF, HOMER).

## Limitations

- Jaccard similarity among the compared approaches is generally low, so methods disagree substantially on ranked TR lists; supplementary figures (9, 10a-c) needed for full comparison are not included in the main text.
- BIT's performance can be reduced by biological complexities of TR regulation, including dynamic TR binding across cell types and time, chromatin structure and epigenetic modifications, and cooperative binding among TRs.
- Sequencing biases such as GC content bias and PCR amplification bias can reduce the reliability of input epigenomic region sets and binding profiles; BIT's Gibbs sampler can become time-consuming as the number of ChIP-seq experiments grows.

## Evidence

- [readme] What BIT consumes as input: [evidence span withheld — anchor status: not_found]
- [readme] BART region-mode invocation for scored region sets: [evidence span withheld — anchor status: not_found]
