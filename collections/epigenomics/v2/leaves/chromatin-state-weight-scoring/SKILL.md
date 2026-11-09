---
name: chromatin-state-weight-scoring
description: Use when two genomic regions are represented as ChromHMM-derived chromatin
  state label sequences and you need a similarity score that accounts for varying
  state lengths, state frequencies, mismatches, and deletions (e.g. -match queries
  in 500 kb regions).
when_to_use_negative:
- Input is still raw multi-track epigenomic signal matrices rather than single-track
  chromatin state sequences produced by ChromHMM or a similar annotator
- The comparison requires aligning genomic regions whose sequences are dominated by
  long single-state stretches and cannot be meaningfully compressed first (e.g. quiescent
  stretches lacking biological interpretation, per Supplementary Figure S1)
- The task is clustering whole epigenomes by global genome-wide profiles rather than
  scoring local pairwise similarity of chromatin state sequences
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_3302
tools:
- name: EpiAlign
  role: Implements the weight function W(k,S), the MF/NF/DF score functions, and the
    local-alignment dynamic programming recurrence for chromatin state sequences
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Upstream chromatin state annotation that encodes multi-track epigenomic signals
    into the single-track state sequences the scoring operates on
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:a354e5739002af0465d40f2d5a12de6ea184d3f94ff6806400aa05652988ba56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_3302
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# chromatin-state-weight-scoring

## Summary

Weight-based scoring scheme from EpiAlign (NAR 2019) that quantifies similarity between two compressed chromatin state sequences using a weight function W(k,S) — equal or frequency-based — feeding match/mismatch (MF), mismatch (NF = -epsilon_N * (W+W)), and deletion (DF = -epsilon_D * W) score functions inside a local-alignment dynamic programming recurrence.

## When to use

Use when two genomic regions are represented as ChromHMM-derived chromatin state label sequences and you need a similarity score that accounts for varying state lengths, state frequencies, mismatches, and deletions (e.g. best-match queries in 500 kb regions).

## When NOT to use

- Input is still raw multi-track epigenomic signal matrices rather than single-track chromatin state sequences produced by ChromHMM or a similar annotator
- The comparison requires aligning genomic regions whose sequences are dominated by long single-state stretches and cannot be meaningfully compressed first (e.g. quiescent stretches lacking biological interpretation, per Supplementary Figure S1)
- The task is clustering whole epigenomes by global genome-wide profiles rather than scoring local pairwise similarity of chromatin state sequences

## Inputs

- ChromHMM-derived chromatin state label sequences for genomic regions (single-track state strings per region, e.g. from 500 kb Roadmap epigenome windows)
- Choice of weight scheme: equal weights or frequency-based weights
- Penalty parameters epsilon_N (default 1.5) and epsilon_D (default 1)

## Outputs

- compressed_sequences.txt (state sequences after collapsing consecutive identical states)
- alignment_scores.csv (local alignment score per sequence pair, M[n1,n2])
- Alignment trace triplets (F, U1, U2) such as F=mmd1m, U1=1234, U2=1203 for S1=abca vs S2=aba

## How to apply

First compress each chromatin state sequence by collapsing consecutive occurrences of the same state into a single state label (abbcc -> abc), since uncompressed sequences contain long stretches of one state that are not useful for comparison. Choose a weighting scheme: equal weights (W=1 for all states) or frequency-based weights, which assign larger weights to less common chromatin states. Implement the score functions MF(k,l,S1,S2) = W(k,S1)+W(l,S2), NF = -epsilon_N * (W(k,S1)+W(l,S2)), and DF(k,S) = -epsilon_D * W(k,S) with defaults epsilon_N=1.5 and epsilon_D=1. Run the Smith-Waterman-style local-alignment dynamic programming recurrence (Equation 4) that includes 0 in the max, initializes the first row/column to zero, and reports the score as M[n1,n2]. For horizontal alignment analyses, normalize scores by aligning every query to randomized chromatin state sequences as a negative control.

## Related tools

- **EpiAlign** (Implements the weight function W(k,S), the MF/NF/DF score functions, and the local-alignment dynamic programming recurrence for chromatin state sequences) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Upstream chromatin state annotation that encodes multi-track epigenomic signals into the single-track state sequences the scoring operates on)

## Examples

```
git clone https://github.com/zzz3639/EpiAlign
```

## Evaluation signals

- The aligner reproduces the paper's example alignment triplets F=mmd1m, U1=1234, U2=1203 for S1=abca against S2=aba
- The compression module maps abbcc to abc
- The local-alignment DP includes the 0 option in the max so scores never go negative
- Alignment scores are normalized against randomized chromatin state sequences used as a negative control
- In Roadmap E071 (brain), scoring recovers brain-specific gene NRG3 best-aligning with brain-specific gene GRIA1

## Limitations

- Mismatch/deletion penalties are fixed defaults (epsilon_N=1.5, epsilon_D=1); how to construct more specialized substitution matrices for chromatin states is stated as an important future research question
- Long stretches of a single chromatin state (e.g. quiescent) are not useful for comparison and require the compression step
- Results depend on ChromHMM annotations, which may contain noisy states needing further biological validation (Supplementary section 12); motifs containing the quiescent state 'o' were excluded in the paper's analyses
- No version numbers stated for the tools; the repository provides no changelog and recommends git clone because direct zip download may break links

## Evidence

- [abstract] EpiAlign is a dynamic programming algorithm that novelly incorporates varying lengths and frequencies of chromatin states.: "EpiAlign is a dynamic programming algorithm that novelly incorporates varying lengths and frequencies of chromatin states."
- [methods] Second, we compress consecutive occurrences of the same state into a state label.: "Second, we compress consecutive occurrences of the same state into a state label."
- [methods] We design the mismatch and deletion score functions based on the weight of each chromatin state in each sequence.: "We design the mismatch and deletion score functions based on the weight of each chromatin state in each sequence."
- [methods] (ii) frequency-based weights assign larger weights to less common chromatin states (see Supplementary section 1 for details): "(ii) frequency-based weights assign larger weights to less common chromatin states (see Supplementary section 1 for details)"
- [results] we align every query chromatin state sequence to randomized chromatin state sequences, which serve as a negative control: "we align every query chromatin state sequence to randomized chromatin state sequences, which serve as a negative control"
- [readme] if an error occurs,  we recommend you to download our package by "git clone" ruther than downloading the zip file.: "if an error occurs,  we recommend you to download our package by "git clone" ruther than downloading the zip file."
