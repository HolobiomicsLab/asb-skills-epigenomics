---
name: chromatin-state-sequence-compression
description: Use when chromatin state sequences (e.g. from ChromHMM on Roadmap epigenomes)
  are about to be aligned with EpiAlign, or when uncompressed sequences contain long
  runs of a single state such as the quiescent state 'o'.
when_to_use_negative:
- Input is still raw multi-track epigenomic signal (ChIP-seq/ATAC-seq tracks) that
  has not yet been converted to discrete chromatin state labels — run chromatin state
  annotation (e.g. ChromHMM) first
- Analysis intentionally depends on state duration/run lengths, e.g. reproducing the
  uncompressed baseline that the paper shows fails to distinguish real from randomized
  epigenomes and takes ~20x longer
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3173
tools:
- name: EpiAlign
  role: Consumes the compressed sequences for modified Smith-Waterman local alignment
    of chromatin state sequences; compression is an explicit preprocessing module
    of the EpiAlign workflow
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Produces the single-track chromatin state sequences (hidden Markov model labels
    over multi-track epigenomic marks) that the compression step acts upon
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_005
  - task_006
  - task_008
  - task_004
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:631cc802f00faa066de78f978c400ce4fde7e29b62871404a937547c4afd1571
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# chromatin-state-sequence-compression

## Summary

Run-length compression of ChromHMM-style chromatin state sequences, collapsing consecutive occurrences of the same state into a single label (abbcc -> abc), so that long quiescent stretches do not dominate EpiAlign local alignments.

## When to use

Use when chromatin state sequences (e.g. from ChromHMM on Roadmap epigenomes) are about to be aligned with EpiAlign, or when uncompressed sequences contain long runs of a single state such as the quiescent state 'o'.

## When NOT to use

- Input is still raw multi-track epigenomic signal (ChIP-seq/ATAC-seq tracks) that has not yet been converted to discrete chromatin state labels — run chromatin state annotation (e.g. ChromHMM) first
- Analysis intentionally depends on state duration/run lengths, e.g. reproducing the uncompressed baseline that the paper shows fails to distinguish real from randomized epigenomes and takes ~20x longer

## Inputs

- ChromHMM-derived single-track chromatin state label sequences (e.g. NIH Roadmap epigenome annotation files such as brain sample E071)
- Per-gene chromatin state sequences for protein-coding gene regions (GENCODE v10 compatible with the Roadmap database)
- 500 kb epigenomic query region state sequences for horizontal alignment

## Outputs

- Compressed chromatin state sequences (e.g. compressed_sequences.txt) with consecutive identical state labels collapsed to one label
- Compressed vs uncompressed variant pairs for runtime and real-vs-randomized separation benchmarks (compression_benchmark.csv)

## How to apply

Take each single-track chromatin state sequence produced by a state annotation method such as ChromHMM and replace every maximal run of identical consecutive state labels with one copy of that label (abbcc -> abc). The rationale is that most uncompressed sequences contain long stretches of a single chromatin state (often quiescent), whose lengths are not useful for comparing chromatin state sequences (Supplementary Figure S1). Apply compression identically to query and target sequences — e.g. to all gene-region sequences for the 19,935 GENCODE v10 protein-coding genes, or to 500 kb query regions before horizontal alignment — keeping the uncompressed sequences as an explicit baseline variant when benchmarking. Downstream alignment uses EpiAlign's modified Smith-Waterman local alignment with mismatch penalty epsilon_N = 1.5 and deletion penalty epsilon_D = 1 on the compressed sequences. When comparing against an uncompressed run, record wall-clock time of both variants: the paper reports uncompressed alignment takes roughly 20 times more computation time than compressed alignment.

## Related tools

- **EpiAlign** (Consumes the compressed sequences for modified Smith-Waterman local alignment of chromatin state sequences; compression is an explicit preprocessing module of the EpiAlign workflow) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Produces the single-track chromatin state sequences (hidden Markov model labels over multi-track epigenomic marks) that the compression step acts upon)

## Examples

```
compressed = ''.join(s[i] for i in range(len(s)) if i == 0 or s[i] != s[i-1])  # abbcc -> abc
```

## Evaluation signals

- Unit check: abbcc compresses to abc, and no two adjacent identical state labels remain in any output sequence
- Every compressed sequence is shorter than or equal to its uncompressed counterpart, with length reduced most for sequences containing long quiescent runs
- Downstream invariant: EpiAlign real-vs-randomized separation (real 500 kb queries score above 0 on average, randomized close to 0) holds with compression but fails on uncompressed sequences
- Benchmark check: uncompressed alignment wall-clock time is roughly 20x the compressed alignment time on the same queries
- Biological check: with compression, NRG3's best alignment partner in brain sample E071 is GRIA1, matching the paper's reported result

## Limitations

- Compression discards state duration information — the paper motivates this by showing lengths of long single-state (quiescent/low state) stretches are not useful for comparison (Supplementary Figure S1)
- Quality of compressed sequences depends on the underlying ChromHMM annotations; the paper notes likely noisy ChromHMM annotations that need further biological validation (Supplementary section 12)
- Downloading the repository as a zip file may cause errors in links; git clone is recommended per the README

## Evidence

- [methods] The motivation of adding a compression step lies in the fact that most uncompressed (chromatin state) sequences contain long stretches of a single chromatin state: "The motivation of adding a compression step lies in the fact that most uncompressed (chromatin state) sequences contain long stretches of a single chromatin state"
- [methods] Second, we compress consecutive occurrences of the same state into a state label.: "Second, we compress consecutive occurrences of the same state into a state label."
- [discussion] whose lengths are not useful for comparing chromatin state sequences (see Supplementary Figure S1): "whose lengths are not useful for comparing chromatin state sequences (see Supplementary Figure S1)"
- [readme] if an error occurs,  we recommend you to download our package by "git clone" ruther than downloading the zip file.: "if an error occurs,  we recommend you to download our package by "git clone" ruther than downloading the zip file."
