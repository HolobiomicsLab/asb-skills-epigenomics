---
name: dynamic-programming-alignment-scoring
description: Use when two or more genomic regions must be compared by their single-track
  chromatin state sequences (e.g. ChromHMM output) rather than raw multi-track signal
  matrices, and ordered state patterns—not just state frequencies—must drive the similarity
  score.
when_to_use_negative:
- The analysis only needs overall state composition—comparing state frequency vectors
  without sequential order is the naive baseline, and the alignment machinery is unnecessary
  overhead
- Input is raw multi-track epigenomic signal matrices of differing dimensions rather
  than discrete chromatin state label sequences; annotation (e.g. ChromHMM) must be
  applied first
- Motifs or regions dominated by the quiescent state 'o', which lacks good biological
  interpretation and was excluded in the paper's motif analyses
edam_operation: http://edamontology.org/operation_0496
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: EpiAlign
  role: Implements the compression step and the weighted local-alignment dynamic programming
    recurrence that produces the alignment scores
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Encodes multi-track epigenomic signals into the single-track chromatin state
    sequences that EpiAlign aligns
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_007
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:8779a944dff2d555ed56bfd41be5692737e1e9276cc918d0215c695f8195c24d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_0496
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# dynamic-programming-alignment-scoring

## Summary

EpiAlign-style local alignment of chromatin state sequences via a modified Smith–Waterman dynamic programming recurrence with state-weight-based match (MF = W(k,S1)+W(l,S2)), mismatch (NF = -1.5*(W(k,S1)+W(l,S2))) and deletion (DF = -1*W(k,S)) score functions, applied after run-length compression of ChromHMM state labels.

## When to use

Use when two or more genomic regions must be compared by their single-track chromatin state sequences (e.g. ChromHMM output) rather than raw multi-track signal matrices, and ordered state patterns—not just state frequencies—must drive the similarity score.

## When NOT to use

- The analysis only needs overall state composition—comparing state frequency vectors without sequential order is the naive baseline, and the alignment machinery is unnecessary overhead
- Input is raw multi-track epigenomic signal matrices of differing dimensions rather than discrete chromatin state label sequences; annotation (e.g. ChromHMM) must be applied first
- Motifs or regions dominated by the quiescent state 'o', which lacks good biological interpretation and was excluded in the paper's motif analyses

## Inputs

- ChromHMM-annotated single-track chromatin state sequences for genomic regions (e.g. NIH Roadmap Epigenomics samples such as E071)
- Coordinates of query and target regions (e.g. NRG3, GRIA1, GENCODE v10 protein-coding gene set)
- Weight mode choice: equal weights (all states = 1) or frequency-based weights

## Outputs

- Per-region-pair local alignment scores (matrix entries M[n1,n2]) written to alignment_scores.csv
- Compressed chromatin state sequences (compressed_sequences.txt)
- Ranked gene tables such as nrg3_alignment_scores.csv or sex_differential_alignment_results.csv with scores and per-gene Wilcoxon P-values

## How to apply

First annotate multi-track epigenomic signals with a chromatin state method such as ChromHMM, then compress each state sequence by collapsing consecutive identical labels (abbcc -> abc) so long quiescent stretches do not dominate the alignment. Define the weight function W(k,S) with either equal weights (all states weight 1) or frequency-based weights that assign larger weights to less common states. Run the local-alignment dynamic program: maintain matrix M, include 0 in the max so scores never go negative, initialize the first row and column to zero, and report M[n1,n2] as the final score, using mismatch epsilon_N = 1.5 and deletion epsilon_D = 1 as defaults. Validate on the paper's toy example S1=abca vs S2=aba, which must yield the alignment triplets F=mmd1m with U1=1234 and U2=1203 before applying to real Roadmap data (e.g. 19,935 GENCODE v10 protein-coding genes across E071 and other samples).

## Related tools

- **EpiAlign** (Implements the compression step and the weighted local-alignment dynamic programming recurrence that produces the alignment scores) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Encodes multi-track epigenomic signals into the single-track chromatin state sequences that EpiAlign aligns)

## Evaluation signals

- Toy example check: aligning S1=abca against S2=aba must reproduce the triplets F=mmd1m, U1=1234, U2=1203
- Local-alignment invariant: the DP max includes 0, so no alignment score is ever negative
- Biological check: with compression on Roadmap sample E071, the top-ranked partner of brain-specific gene NRG3 must be GRIA1
- Statistical check: one-sided Wilcoxon rank-sum test on male-vs-male vs male-vs-female alignment scores must show significantly higher same-sex scores (P < 0.05), and top 200 P-value genes should be predominantly X-linked (paper reports 188/200)
- Compression invariant: consecutive identical state labels collapse to one label (abbcc -> abc) in compressed_sequences.txt

## Limitations

- Results depend on the upstream annotation tool: 'our current alignment results are based on ChromHMM', and no version numbers were stated for the tools used
- How to construct specialized substitution matrices for chromatin states remains an open future research question
- Uncompressed sequences contain long stretches of a single chromatin state whose lengths are not useful for comparison (Supplementary Figure S1), so the compression step is effectively required
- Downloading the package as a zip file may cause errors in links; the README recommends 'git clone' instead

## Evidence

- [methods] Weighted DP score functions with default penalties: "We design the mismatch and deletion score functions based on the weight of each chromatin state in each sequence."
- [methods] Compression step rationale: "The motivation of adding a compression step lies in the fact that most uncompressed (chromatin state) sequences contain long stretches of a single chromatin state"
- [methods] Dynamic programming core: "This optimization problem can be approached by dynamic programming, an algorithm that iteratively maintains and updates a matrix M that stores dynamic alignment results."
- [abstract] Algorithm purpose: "EpiAlign is a dynamic programming algorithm that novelly incorporates varying lengths and frequencies of chromatin states."
- [results] Repository availability: "which is available at GitHub: https://github.com/zzz3639/EpiAlign"
- [readme] Installation caveat: "we recommend you to download our package by "git clone" ruther than downloading the zip file."
