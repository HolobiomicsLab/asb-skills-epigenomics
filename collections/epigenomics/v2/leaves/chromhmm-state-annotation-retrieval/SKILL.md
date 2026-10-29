---
name: chromhmm-state-annotation-retrieval
description: Use when a query genomic region's ChromHMM-annotated chromatin state
  sequence (e.g. from NIH Roadmap sample E071) must be compared against all gene regions
  to rank local-alignment similarity, especially when long quiescent stretches would
  otherwise dominate uncompressed alignments.
when_to_use_negative:
- Comparing raw multi-track epigenomic signal matrices directly rather than derived
  chromatin state sequences — reduce the problem to single-track state sequences first
- Alignments where long uncompressed quiescent-state stretches are meaningful signal
  rather than noise to be removed by the compression step
- Cross-sample epigenome grouping/discrimination tasks better served by horizontal
  alignment of 500 kb query regions or vertical gene-by-sample score comparisons with
  Wilcoxon/GO enrichment
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: EpiAlign
  role: Performs the compression of consecutive same-state runs and the modified Smith–Waterman
    local alignment with state-weight-based match/mismatch/deletion score functions
    to score the query against all gene regions
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Encodes multi-track epigenomic signals into the single-track chromatin state
    sequences that serve as EpiAlign's input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:1f9167629e6321cfac4d7df78b62ac0f01da40d08d6090ae236f1ac24bb9bcca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# chromhmm-state-annotation-retrieval

## Summary

Apply EpiAlign's compression step plus modified Smith–Waterman local alignment to ChromHMM chromatin state sequences in order to retrieve genomic regions (e.g. protein-coding genes) whose chromatin state patterns best match a query region, as in the NRG3–GRIA1 alignment in Roadmap brain sample E071.

## When to use

Use when a query genomic region's ChromHMM-annotated chromatin state sequence (e.g. from NIH Roadmap sample E071) must be compared against all gene regions to rank local-alignment similarity, especially when long quiescent stretches would otherwise dominate uncompressed alignments.

## When NOT to use

- Comparing raw multi-track epigenomic signal matrices directly rather than derived chromatin state sequences — reduce the problem to single-track state sequences first
- Alignments where long uncompressed quiescent-state stretches are meaningful signal rather than noise to be removed by the compression step
- Cross-sample epigenome grouping/discrimination tasks better served by horizontal alignment of 500 kb query regions or vertical gene-by-sample score comparisons with Wilcoxon/GO enrichment

## Inputs

- ChromHMM-annotated single-track chromatin state sequences for the sample of interest (e.g. NIH Roadmap Epigenomics release 9, brain sample E071)
- Genomic coordinates of the query gene region (e.g. NRG3)
- Coordinates of all candidate protein-coding gene regions (GENCODE v10 compatible with the Roadmap database)
- EpiAlign penalty parameters (epsilon_N = 1.5, epsilon_D = 1) and choice of state weights (equal or frequency-based)

## Outputs

- Ranked table of local alignment scores M(n1, n2) for every candidate gene vs. the query (e.g. nrg3_alignment_scores.csv)
- Top-scoring best-alignment gene for the query (e.g. GRIA1 for NRG3 in E071)
- Compressed chromatin state sequences for the query and each candidate gene

## How to apply

First load ChromHMM-annotated single-track chromatin state sequences for the target sample (e.g. Roadmap E071, 25 bp-window state labels) and the coordinates of the query gene and all protein-coding genes (GENCODE v10, Roadmap-compatible). Apply EpiAlign's compression step to collapse consecutive occurrences of the same chromatin state into a single state label for the query and every candidate sequence — this removes long quiescent/low-state stretches that would otherwise dominate the alignment. For each candidate, run EpiAlign's modified Smith–Waterman local alignment (dynamic programming matrix M with match, mismatch, and deletion score functions based on state weights, using default penalties epsilon_N = 1.5 for mismatch and epsilon_D = 1 for deletion). Rank all candidates by the final local alignment score M(n1, n2) and write the ranked table (e.g. nrg3_alignment_scores.csv); the top-scoring gene is the retrieved best match. Use equal weights or frequency-based weights for states, the latter assigning larger weights to less common chromatin states.

## Related tools

- **EpiAlign** (Performs the compression of consecutive same-state runs and the modified Smith–Waterman local alignment with state-weight-based match/mismatch/deletion score functions to score the query against all gene regions) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Encodes multi-track epigenomic signals into the single-track chromatin state sequences that serve as EpiAlign's input)

## Examples

```
git clone https://github.com/zzz3639/EpiAlign
```

## Evaluation signals

- The top-ranked gene in the output score table (nrg3_alignment_scores.csv) is GRIA1, matching the paper's reported NRG3 best alignment in E071
- Every candidate sequence is compressed before alignment (no uncompressed state sequences enter the dynamic programming matrix)
- The alignment uses the stated default penalties epsilon_N = 1.5 (mismatch) and epsilon_D = 1 (deletion)
- Scores are local alignment scores M(n1, n2) from the dynamic programming matrix, not global alignment or raw shared-state counts
- Query-localized recurrent patterns carry cell-type-specific characteristics — retrieved matches should be biologically consistent (both brain-specific for a brain query sample)

## Limitations

- The compression step is motivated by the fact that most uncompressed chromatin state sequences contain long stretches of a single chromatin state whose lengths are not useful for comparison (see Supplementary Figure S1, referenced but not included)
- Alignment results depend on ChromHMM annotations; the paper notes likely noisy ChromHMM annotations that need further biological validation (Supplementary section 12)
- How to construct more specialized substitution matrices for chromatin states remains an open research question in the paper's discussion

## Evidence

- [methods] in the brain sample E071, when we apply EpiAlign with the compression step, the brain-specific gene NRG3 has the best alignment with another brain-specific gene GRIA1: "in the brain sample E071, when we apply EpiAlign with the compression step, the brain-specific gene NRG3 has the best alignment with another brain-specific gene GRIA1"
- [methods] Second, we compress consecutive occurrences of the same state into a state label.: "Second, we compress consecutive occurrences of the same state into a state label."
- [results] we use all protein-coding genes in GENCODE v10 (28) that are compatible with the Roadmap database: "we use all protein-coding genes in GENCODE v10 (28) that are compatible with the Roadmap database"
- [results] which is available at GitHub: https://github.com/zzz3639/EpiAlign: "which is available at GitHub: https://github.com/zzz3639/EpiAlign"
- [readme] we recommend you to download our package by "git clone" ruther than downloading the zip file.: "we recommend you to download our package by "git clone" ruther than downloading the zip file."
