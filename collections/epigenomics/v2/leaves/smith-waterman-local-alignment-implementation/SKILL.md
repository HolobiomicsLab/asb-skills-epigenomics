---
name: smith-waterman-local-alignment-implementation
description: Use when two genomic regions are represented as single-track chromatin
  state label sequences (e.g. from ChromHMM) and you need a local alignment score
  capturing locally similar chromatin patterns.
when_to_use_negative:
- Input is still multi-track epigenomic signal (e.g. raw ChIP-seq/Histone-mark matrices)
  rather than discrete chromatin state label sequences — annotate states first (e.g.
  with ChromHMM)
- A global end-to-end alignment of the full sequences is required; EpiAlign's recurrence
  includes the 0 option in the DP max specifically to report locally aligned chromatin
  regions
- Sequential order of chromatin states is irrelevant and per-window state overlaps
  suffice — this skill exists to incorporate sequential information that prior comparisons
  ignored
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: EpiAlign
  role: Implements the compression step, state-weighted scoring functions (MF/NF/DF),
    and the local-alignment dynamic programming recurrence; available at GitHub
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Upstream chromatin state annotation that encodes multi-track epigenomic signals
    into the single-track state sequences that the aligner consumes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:c94cf6b832991a6ee66b62508c697bd694b43fb0971d50f1edbbf2b1d8b6973f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Smith-Waterman local alignment implementation

## Summary

Implements the EpiAlign local-alignment dynamic programming recurrence (Smith-Waterman style, with a 0 option in the max) to score the similarity of two chromatin state sequences, using compression of consecutive states and state-weighted match/mismatch/deletion score functions.

## When to use

Use when two genomic regions are represented as single-track chromatin state label sequences (e.g. from ChromHMM) and you need a local alignment score capturing locally similar chromatin patterns.

## When NOT to use

- Input is still multi-track epigenomic signal (e.g. raw ChIP-seq/Histone-mark matrices) rather than discrete chromatin state label sequences — annotate states first (e.g. with ChromHMM)
- A global end-to-end alignment of the full sequences is required; EpiAlign's recurrence includes the 0 option in the DP max specifically to report locally aligned chromatin regions
- Sequential order of chromatin states is irrelevant and per-window state overlaps suffice — this skill exists to incorporate sequential information that prior comparisons ignored

## Inputs

- Two single-track chromatin state sequences for genomic regions (produced by a chromatin state annotation method such as ChromHMM)
- Weight specification: equal weights (W=1) or frequency-based weights per chromatin state
- Gap parameters epsilon_N (default 1.5) and epsilon_D (default 1)

## Outputs

- Local alignment score matrix M with final score M[n1,n2]
- alignment_scores.csv (one local alignment score per region pair)
- compressed_sequences.txt (compressed chromatin state sequences, e.g. abbcc -> abc)
- Alignment triplets (match/mismatch/deletion labels F and aligned position indices U1, U2, e.g. F=mmd1m, U1=1234, U2=1203)

## How to apply

First compress each chromatin state sequence by collapsing consecutive occurrences of the same state label (abbcc -> abc), since uncompressed sequences are dominated by long stretches of a single state such as the quiescent state. Assign each state a weight W(k,S), either equal weights (W=1 for all states) or frequency-based weights that assign larger weights to less common states. Define the match/mismatch score MF(k,l,S1,S2) = W(k,S1)+W(l,S2), the gap penalty NF = -epsilon_N * (W(k,S1)+W(l,S2)), and the deletion penalty DF(k,S) = -epsilon_D * W(k,S), with defaults epsilon_N=1.5 and epsilon_D=1. Run the Smith-Waterman dynamic programming recurrence (Equation 4), which crucially includes 0 in the max so scores never go negative, initializes the first row and column to zero, and reports the final local alignment score as M[n1,n2]. Emit compressed_sequences.txt and alignment_scores.csv and validate on the paper's example S1=abca vs S2=aba, which must reproduce alignment triplets F=mmd1m, U1=1234, U2=1203.

## Related tools

- **EpiAlign** (Implements the compression step, state-weighted scoring functions (MF/NF/DF), and the local-alignment dynamic programming recurrence; available at GitHub) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Upstream chromatin state annotation that encodes multi-track epigenomic signals into the single-track state sequences that the aligner consumes)

## Examples

```
M[i][j] = max(0, M[i-1][j-1] + MF(k,l,S1,S2), M[i][j-1] + NF, M[i-1][j] + DF)  # EpiAlign local-alignment recurrence, score = M[n1,n2]
```

## Evaluation signals

- Validation example: aligning S1=abca against S2=aba must reproduce alignment triplets F=mmd1m, U1=1234, U2=1203
- Local-alignment invariant: the DP max includes the 0 option, so M[i,j] >= 0 and the final score M[n1,n2] is never negative
- Compression invariant: consecutive identical state labels collapse to a single label (abbcc -> abc) in compressed_sequences.txt
- Gap-penalty invariant: NF = -1.5*(W(k,S1)+W(l,S2)) and DF = -1*W(k,S) with default epsilons; first DP row/column initialized to zero
- Sanity check on real data: alignment of a query to randomized chromatin state sequences serves as a negative control for score normalization

## Limitations

- Sequences dominated by long stretches of the quiescent state 'o' carry little comparative signal; the compression step and motif filtering (excluding 'o') address this
- How to construct specialized substitution matrices for chromatin states is an open future research question (discussion section)
- Alignment results depend on the upstream ChromHMM model; noisy ChromHMM annotations may need further biological validation (Supplementary section 12, not included here)
- Downloading the repository as a zip file may cause errors in links; the README recommends git clone

## Evidence

- [abstract] EpiAlign is a dynamic programming algorithm that novelly incorporates varying lengths and frequencies of chromatin states.: "EpiAlign is a dynamic programming algorithm that novelly incorporates varying lengths and frequencies of chromatin states."
- [methods] Second, we compress consecutive occurrences of the same state into a state label.: "Second, we compress consecutive occurrences of the same state into a state label."
- [methods] We design the mismatch and deletion score functions based on the weight of each chromatin state in each sequence.: "We design the mismatch and deletion score functions based on the weight of each chromatin state in each sequence."
- [methods] This optimization problem can be approached by dynamic programming, an algorithm that iteratively maintains and updates a matrix M that stores dynamic alignment results.: "This optimization problem can be approached by dynamic programming, an algorithm that iteratively maintains and updates a matrix M that stores dynamic alignment results."
- [readme] we recommend you to download our package by "git clone" ruther than downloading the zip file.: "we recommend you to download our package by "git clone" ruther than downloading the zip file."
