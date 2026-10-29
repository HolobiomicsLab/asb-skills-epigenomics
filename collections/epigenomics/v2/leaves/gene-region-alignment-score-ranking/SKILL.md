---
name: gene-region-alignment-score-ranking
description: Use when a query gene's chromatin state sequence must be compared against
  every protein-coding gene region in one Roadmap/ChromHMM-annotated sample to produce
  a ranked local-alignment score table identifying the -matching gene region.
when_to_use_negative:
- The goal is cross-sample comparison of all genes between pairs of epigenomes with
  downstream Wilcoxon testing and GO enrichment, not ranking regions against a single
  query gene.
- The sequences are uncompressed chromatin state sequences and long quiescent stretches
  should dominate; skipping the compression step changes which gene ranks best.
- The input is a 500 kb region-level 'horizontal' alignment query across an epigenome
  rather than a gene-region ranking task.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: EpiAlign
  role: Performs the compression step and the modified Smith–Waterman local alignment
    with state-weight-based mismatch/deletion score functions used to score and rank
    gene regions against the query.
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Produces the single-track chromatin state sequences for the Roadmap sample
    that serve as the alignment input.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:4991dc585ff318c0cf436d71614843bf5a343c0ab1fe6ece3e16c6593096d902
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# gene-region-alignment-score-ranking

## Summary

Ranks all protein-coding gene regions in a Roadmap sample by EpiAlign local alignment score against a query gene's compressed ChromHMM chromatin state sequence, so the best epigenomically similar gene region can be identified (e.g. NRG3 aligns best with GRIA1 in brain sample E071).

## When to use

Use when a query gene's chromatin state sequence must be compared against every protein-coding gene region in one Roadmap/ChromHMM-annotated sample to produce a ranked local-alignment score table identifying the best-matching gene region.

## When NOT to use

- The goal is cross-sample comparison of all genes between pairs of epigenomes with downstream Wilcoxon testing and GO enrichment, not ranking regions against a single query gene.
- The sequences are uncompressed chromatin state sequences and long quiescent stretches should dominate; skipping the compression step changes which gene ranks best.
- The input is a 500 kb region-level 'horizontal' alignment query across an epigenome rather than a gene-region ranking task.

## Inputs

- ChromHMM-annotated single-track chromatin state sequences for the sample (e.g. NIH Roadmap brain sample E071)
- GENCODE v10 coordinates of all protein-coding genes compatible with the Roadmap database (excluding chromosome Y genes per the paper's filter)
- Coordinates of the query gene region (e.g. NRG3)
- EpiAlign penalty parameters (epsilon_N = 1.5 mismatch, epsilon_D = 1 deletion) and chosen state-weight scheme (equal or frequency-based)

## Outputs

- Ranked table of local alignment scores M(n1, n2) for every protein-coding gene against the query (e.g. nrg3_alignment_scores.csv)
- Top-scoring gene region (best chromatin-state-sequence match to the query gene)

## How to apply

Load the ChromHMM-annotated single-track chromatin state sequences for the sample (e.g. brain sample E071) plus coordinates of the query gene (e.g. NRG3) and all protein-coding genes from GENCODE v10 compatible with the Roadmap database. Apply EpiAlign's compression step to collapse consecutive occurrences of the same chromatin state into a single state label, because long quiescent/low-state stretches would otherwise dominate the alignment. For each protein-coding gene, run EpiAlign's modified Smith–Waterman local alignment between the compressed query sequence and the compressed gene sequence using default penalty parameters epsilon_N = 1.5 (mismatch) and epsilon_D = 1 (deletion), with mismatch and deletion score functions derived from state weights (equal weights, or frequency-based weights that assign larger weights to less common states). Record each gene's final local alignment score M(n1, n2) and rank all genes; write the ranked table (e.g. nrg3_alignment_scores.csv) and validate that the top gene matches the reported finding (GRIA1 for NRG3 in E071 with compression).

## Related tools

- **EpiAlign** (Performs the compression step and the modified Smith–Waterman local alignment with state-weight-based mismatch/deletion score functions used to score and rank gene regions against the query.) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Produces the single-track chromatin state sequences for the Roadmap sample that serve as the alignment input.)

## Evaluation signals

- With compression applied in brain sample E071, the top-ranked gene for the NRG3 query in nrg3_alignment_scores.csv must be GRIA1, matching the paper's finding.
- Compressed sequences contain no consecutive occurrences of the same chromatin state label.
- Every protein-coding gene in GENCODE v10 (minus chromosome Y genes) receives a finite local alignment score and the table is fully ranked.
- Rerunning with identical parameters (epsilon_N = 1.5, epsilon_D = 1, same weight scheme) reproduces the same ranking order.
- Scores are local alignment scores M(n1, n2) from the dynamic programming matrix, not global alignment or raw similarity metrics.

## Limitations

- Uncompressed chromatin state sequences contain long stretches of a single (quiescent) state that dominate alignments, so the compression step is essential; the quiescent state 'o' also lacks a good biological interpretation and was excluded from motif analyses in the paper.
- Alignment results depend on ChromHMM annotations, which the authors note can be noisy and may need further biological validation; no tool version numbers are stated.
- The EpiAlign README warns that downloading the zip file directly may cause errors in links; the package should be installed via git clone instead, and no changelog exists in the repository.

## Evidence

- [methods] in the brain sample E071, when we apply EpiAlign with the compression step, the brain-specific gene NRG3 has the best alignment with another brain-specific gene GRIA1: "in the brain sample E071, when we apply EpiAlign with the compression step, the brain-specific gene NRG3 has the best alignment with another brain-specific gene GRIA1"
- [methods] Second, we compress consecutive occurrences of the same state into a state label.: "Second, we compress consecutive occurrences of the same state into a state label."
- [methods] We design the mismatch and deletion score functions based on the weight of each chromatin state in each sequence.: "We design the mismatch and deletion score functions based on the weight of each chromatin state in each sequence."
- [results] we use all protein-coding genes in GENCODE v10 (28) that are compatible with the Roadmap database: "we use all protein-coding genes in GENCODE v10 (28) that are compatible with the Roadmap database"
- [readme] if an error occurs,  we recommend you to download our package by "git clone" ruther than downloading the zip file.: "if an error occurs,  we recommend you to download our package by "git clone" ruther than downloading the zip file."
