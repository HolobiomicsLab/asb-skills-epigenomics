---
name: motif-enrichment-based-tr-ranking
description: Use when a differentially accessible region (DAR) BED file must be scored
  for candidate TRs by motif enrichment, and per-TR ranks are needed to compute mean
  reciprocal rank (MRR) in a multi-method benchmark such as Fig. 3f of the BIT paper.
when_to_use_negative:
- The input is a raw ATAC/ChIP peak file that first requires differential analysis
  (e.g., DiffBind dba.count with minOverlap = 2 and FDR <= 0.05 & abs(Log2FC) >= 2
  filtering) rather than an already-derived DAR region set
- The goal is to run the ChIP-seq-binding-based baselines (BART, ChIP-Atlas) or BIT
  itself, which bypass computationally predicted motifs entirely — BIT leverages over
  10,000 TR ChIP-seq datasets from GTRD instead of motif databases
- Query regions are a gene set rather than a genomic region set; gene-set-based TR
  association (e.g., BART geneset mode) is a different skill
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0749
tools:
- name: WhichTF
  role: Python motif-based TR identification tool; scores query regions against JASPAR/TRANSFAC
    motifs via a hypergeometric test to produce ranked TRs
- name: HOMER
  role: Motif-based comparison method that leverages TR binding motifs as reference
    to rank candidate TRs from the DAR input
- name: I-cisTarget
  role: Web-portal motif-based method using ENCODE references and recovery analysis
    to rank TRs for the query region set
- name: BIT
  role: ChIP-seq-based reference method whose MRR is the benchmark target; motif-based
    TR ranks are compared against BIT_score and Rank columns from the BIT R package
  repo: https://github.com/ZeyuL01/BIT
- name: BART
  role: ChIP-seq-binding-based comparison method run on the same DAR BED region input
    in region mode for the MRR benchmark
  repo: https://github.com/zanglab/bart2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:3fc12a619747da3a67c178c12b3f9f6b4e299403e4142e5064a4684bbccdd69a
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
  - HOMER
  - I-cisTarget
  - R
  - WhichTF
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# motif-enrichment-based-tr-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Rank transcriptional regulators (TRs) for a query region set (e.g., a DAR BED file) using motif-enrichment methods — WhichTF (JASPAR/TRANSFAC motifs, hypergeometric test), HOMER, and I-cisTarget (ENCODE-based recovery analysis) — and extract TR ranks for benchmarking against ChIP-seq-based methods such as BIT.

## When to use

Use when a differentially accessible region (DAR) BED file must be scored for candidate TRs by motif enrichment, and per-TR ranks are needed to compute mean reciprocal rank (MRR) in a multi-method benchmark such as Fig. 3f of the BIT paper.

## When NOT to use

- The input is a raw ATAC/ChIP peak file that first requires differential analysis (e.g., DiffBind dba.count with minOverlap = 2 and FDR <= 0.05 & abs(Log2FC) >= 2 filtering) rather than an already-derived DAR region set
- The goal is to run the ChIP-seq-binding-based baselines (BART, ChIP-Atlas) or BIT itself, which bypass computationally predicted motifs entirely — BIT leverages over 10,000 TR ChIP-seq datasets from GTRD instead of motif databases
- Query regions are a gene set rather than a genomic region set; gene-set-based TR association (e.g., BART geneset mode) is a different skill

## Inputs

- DAR BED files of differentially accessible regions from bulk TR perturbation datasets (e.g., DiffBind report/export.bed output for CTCF, ZBTB7A, FOXA2, RUNX1, BCL11B)
- Genome build specification matching the DAR files (e.g., hg38 or mm10)
- Motif reference databases: JASPAR/TRANSFAC (WhichTF, HOMER) and ENCODE references (I-cisTarget)

## Outputs

- Per-method ranked TR output tables from WhichTF, HOMER, and I-cisTarget
- Per-TR reciprocal ranks (1/rank) for the five perturbed TRs
- mrr_benchmark.csv with per-method mean reciprocal rank values visualizable as a Fig. 3f-style bar chart

## How to apply

Take the processed DAR BED files (e.g., DiffBind-exported regions from bulk perturbation datasets for CTCF, ZBTB7A, FOXA2, RUNX1, and BCL11B) and run each motif-based method on the same region input with the matching genome build. WhichTF (Python) scores TRs against JASPAR/TRANSFAC motif references using a frequentist hypergeometric test; HOMER likewise leverages TR binding motifs as reference; I-cisTarget, accessed through its web portal, uses ENCODE references and a recovery-analysis-based ranking. From each method's ranked TR output table, extract the numeric rank of the known perturbed TR for each of the five datasets, compute the reciprocal rank (1/rank) per TR, and average the five values to obtain the per-method MRR. These MRR values are written to mrr_benchmark.csv and compared directly against ChIP-seq-binding-based methods (BART, ChIP-Atlas) and BIT, which transforms peak coordinates into fixed-length binary vectors and assigns each TR a BIT score from posterior draws.

## Related tools

- **WhichTF** (Python motif-based TR identification tool; scores query regions against JASPAR/TRANSFAC motifs via a hypergeometric test to produce ranked TRs)
- **HOMER** (Motif-based comparison method that leverages TR binding motifs as reference to rank candidate TRs from the DAR input)
- **I-cisTarget** (Web-portal motif-based method using ENCODE references and recovery analysis to rank TRs for the query region set)
- **BIT** (ChIP-seq-based reference method whose MRR is the benchmark target; motif-based TR ranks are compared against BIT_score and Rank columns from the BIT R package) — https://github.com/ZeyuL01/BIT
- **BART** (ChIP-seq-binding-based comparison method run on the same DAR BED region input in region mode for the MRR benchmark) — https://github.com/zanglab/bart2

## Examples

```
bart2 region -i DAR.bed -c 4 -s mm10 --outdir bart2_output
```

## Evaluation signals

- Each method's ranked TR output must contain an extractable numeric rank for the known perturbed TR (CTCF, ZBTB7A, FOXA2, RUNX1, or BCL11B) for all five bulk datasets
- Reciprocal ranks must be averaged over exactly the five perturbed TRs to yield the per-method MRR; verify the count of contributing TRs equals five
- The computed MRR values, when plotted as a bar chart, should reproduce the reported Fig. 3f ordering in which BIT attains the highest MRR among the six methods
- The same DAR BED inputs and matching genome build must be used across all six methods for a fair comparison
- Per-method MRR values written to mrr_benchmark.csv should be checkable against the published figure values

## Limitations

- Motif-based approaches suffer from highly similar motifs shared among many TRs, which makes accurate TR identification challenging
- Motifs alone cannot characterize any context-specific binding pattern of a TR
- BIT's authors note existing frequentist motif-based methods lack coherent probabilistic interpretation or uncertainty estimation, unlike BIT's 95% credible intervals from posterior sampling
- Method performance depends on NGS data quality; sequencing biases such as GC content bias and PCR amplification bias can reduce reliability of the input epigenomic region set

## Evidence

- [readme] Region-set TR prediction input format for baseline methods: [evidence span withheld — anchor status: not_found]
