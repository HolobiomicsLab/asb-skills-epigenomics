---
name: cross-tissue-epigenome-comparison
description: Use when the goal is to detect tissue-specific genes or tissue-specific
  chromatin patterns by comparing compressed ChromHMM chromatin state sequences across
  two Roadmap epigenome samples at gene regions.
when_to_use_negative:
- When the question is about aligning raw multi-track epigenomic signal matrices rather
  than compressed chromatin state sequences — EpiAlign compares state sequences, not
  signal matrices
- When genes on chromosome Y are in scope — the paper's differential analysis excludes
  chromosome Y genes
- When motifs containing the quiescent state 'o' matter — the article restricts motif
  consideration to non-'o' states because 'o' lacks biological interpretation
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: EpiAlign
  role: Performs the modified Smith-Waterman local alignment (dynamic programming,
    0 reset, epsilon_N = 1.5, epsilon_D = 1) on compressed chromatin state sequences
    of gene regions
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Encodes multi-track epigenomic signals from Roadmap samples into single-track
    chromatin state sequences prior to compression and alignment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:d8251cd865be82cbad30d17391db2d22ec7160596bac605d23e0c3160d2a3427
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cross-tissue-epigenome-comparison

## Summary

Differentially aligns chromatin state sequences between tissue epigenomes (e.g. brain E071 vs heart Roadmap samples) using EpiAlign's modified Smith-Waterman local alignment to identify cell-type/tissue-specific genes such as STMN4 without expression data.

## When to use

Use when the goal is to detect tissue-specific genes or tissue-specific chromatin patterns by comparing compressed ChromHMM chromatin state sequences across two Roadmap epigenome samples at gene regions.

## When NOT to use

- When the question is about aligning raw multi-track epigenomic signal matrices rather than compressed chromatin state sequences — EpiAlign compares state sequences, not signal matrices
- When genes on chromosome Y are in scope — the paper's differential analysis excludes chromosome Y genes
- When motifs containing the quiescent state 'o' matter — the article restricts motif consideration to non-'o' states because 'o' lacks biological interpretation

## Inputs

- Compressed ChromHMM chromatin state sequences for the Roadmap brain sample E071 and the heart sample
- Protein-coding gene coordinates from GENCODE v10 compatible with the Roadmap database (19,935 genes, excluding chromosome Y)
- Brain-specific and heart-specific gene labels for validation/enrichment

## Outputs

- Per-gene differential alignment P-values (two-sample one-sided Wilcoxon rank-sum test)
- Ranked gene list with STMN4 expected as the top hit
- Hypergeometric enrichment counts of tissue-specific genes in the top 100 (P < 10^-30)
- GOrilla GO terms for top-ranked aligned genes at P < 0.0001

## How to apply

Encode multi-track Roadmap epigenomic signals into single-track chromatin state sequences with ChromHMM, then compress consecutive occurrences of the same state into a single state label (compression removes long stretches that are uninformative). Run EpiAlign's local alignment — dynamic programming with a 0 reset option, mismatch penalty epsilon_N = 1.5, deletion penalty epsilon_D = 1, match score = sum of state weights — on the compressed state sequences of each protein-coding gene region between the two tissue samples. For each gene, compute a differential alignment P-value with a two-sample one-sided Wilcoxon rank-sum test comparing within-tissue alignment scores against cross-tissue scores, rank all genes by P-value, and test enrichment of known tissue-specific genes in the top 100 via a hypergeometric test (expected ~9 brain-specific and ~4 heart-specific genes, P < 10^-30). Optionally run GOrilla GO enrichment on the top-ranked aligned genes and extract terms with P < 0.0001; frequency-based weights (larger weights for less common chromatin states) can replace equal weights.

## Related tools

- **EpiAlign** (Performs the modified Smith-Waterman local alignment (dynamic programming, 0 reset, epsilon_N = 1.5, epsilon_D = 1) on compressed chromatin state sequences of gene regions) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Encodes multi-track epigenomic signals from Roadmap samples into single-track chromatin state sequences prior to compression and alignment)

## Examples

```
git clone https://github.com/zzz3639/EpiAlign.git  # then run EpiAlign local alignment on compressed E071 (brain) vs heart ChromHMM gene-region state sequences
```

## Evaluation signals

- STMN4 receives the top (smallest) differential alignment P-value in the brain-vs-heart ranking
- Hypergeometric enrichment of known tissue-specific genes in the top 100: ~9 brain-specific and ~4 heart-specific genes with P < 10^-30
- GOrilla GO enrichment of top-ranked aligned genes yields terms at P < 0.0001
- Compression step applied: consecutive occurrences of the same chromatin state are collapsed to a single state label before alignment
- Alignment uses the specified score functions (mismatch penalty epsilon_N = 1.5, deletion penalty epsilon_D = 1, match = sum of state weights) with the 0 reset option

## Limitations

- Long stretches of a single chromatin state in uncompressed sequences carry little comparative information, motivating the compression step (Supplementary Figure S1 covers quiescent/low state lengths but was not included in the provided content)
- Alignment results depend on ChromHMM annotation quality; likely noisy ChromHMM annotations need further biological validation (Supplementary section 12 referenced but not provided)
- How to construct specialized substitution matrices for chromatin states is an open question in the paper
- Downloading the EpiAlign zip file directly may cause errors in links; git clone is recommended

## Evidence

- [intro] EpiAlign compares two chromatin state sequences by calculating a local alignment score.: "EpiAlign compares two chromatin state sequences by calculating a local alignment score."
- [methods] Compression and gene-region alignment workflow: "Second, we compress consecutive occurrences of the same state into a state label."
- [results] Wilcoxon differential test: "we perform the two-sample one-sided Wilcoxon test between male-vs-male scores and male-vs-female scores for each protein-coding gene"
- [methods] Mismatch and deletion scores based on state weights: "We design the mismatch and deletion score functions based on the weight of each chromatin state in each sequence."
- [readme] Recommend git clone over zip download: "Downloading zip file directly may cause errors in links."
