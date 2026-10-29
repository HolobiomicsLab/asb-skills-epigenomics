---
name: pairwise-local-alignment-score-computation
description: Use when the goal is to score similarity between genomic regions represented
  as chromatin state sequences (e.g. 500 kb Roadmap query regions vs. other regions
  or randomized negative controls) with a local alignment algorithm.
when_to_use_negative:
- Input is raw multi-track epigenomic signal (e.g. ChIP-seq BED/HDF5 tracks) rather
  than discrete chromatin state sequences — a state annotation step such as ChromHMM
  must be applied first
- The comparison target is multi-epigenome sample grouping via vertical alignment
  of gene-region sequences rather than within-epigenome horizontal alignment of 500
  kb regions
- Uncompressed state sequences dominate runtime — note the compression step changes
  both runtime (~20x faster) and separability, so uncompressed runs are not interchangeable
  with compressed ones
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: EpiAlign
  role: Performs the modified Smith-Waterman dynamic-programming local alignment of
    compressed chromatin state sequences with weighted mismatch/deletion functions
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Encodes multi-track epigenomic signals (e.g. NIH Roadmap mark data) into the
    single-track chromatin state sequences that EpiAlign aligns
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:2e10e90969e3735c4359bb02c81028584f16248658417be8c08729e9945b33ce
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pairwise-local-alignment-score-computation

## Summary

Compute EpiAlign's Smith-Waterman-style local alignment scores between chromatin state sequences (e.g. ChromHMM annotations of Roadmap epigenomes) using a 0-floored dynamic-programming matrix with frequency-based weights and penalties epsilon_N = 1.5 (mismatch) and epsilon_D = 1 (deletion), after compressing consecutive identical states.

## When to use

Use when the goal is to score similarity between genomic regions represented as chromatin state sequences (e.g. 500 kb Roadmap query regions vs. other regions or randomized negative controls) with a local alignment algorithm.

## When NOT to use

- Input is raw multi-track epigenomic signal (e.g. ChIP-seq BED/HDF5 tracks) rather than discrete chromatin state sequences — a state annotation step such as ChromHMM must be applied first
- The comparison target is multi-epigenome sample grouping via vertical alignment of gene-region sequences rather than within-epigenome horizontal alignment of 500 kb regions
- Uncompressed state sequences dominate runtime — note the compression step changes both runtime (~20x faster) and separability, so uncompressed runs are not interchangeable with compressed ones

## Inputs

- ChromHMM chromatin state annotation files for an NIH Roadmap epigenome (e.g. E003 ESC, E071 brain), bundled with the EpiAlign GitHub data
- 500 kb query region definitions partitioning the epigenome
- Randomized chromatin state sequences for negative-control normalization
- Alignment parameters: epsilon_N = 1.5 (mismatch penalty), epsilon_D = 1 (deletion penalty), frequency-based weights

## Outputs

- Per-query local alignment scores against other real regions (best-match assignments)
- Per-query negative-control alignment scores against randomized sequences
- alignment_scores.csv of real vs. randomized scores per query
- score_distributions.png comparing real and randomized score distributions

## How to apply

First obtain single-track chromatin state sequences by running a state annotation method such as ChromHMM on multi-track epigenomic signals (e.g. NIH Roadmap samples like E003 ESC or E071 brain). Compress consecutive occurrences of the same state into a single state label (abbcc -> abc) so long quiescent stretches do not dominate alignment, and assign frequency-based weights so less common states count more. Run EpiAlign's modified Smith-Waterman dynamic programming, maintaining and updating matrix M with a 0-floored update, weighted matching/mismatching/deletion functions, mismatch penalty epsilon_N = 1.5 and deletion penalty epsilon_D = 1. Divide the epigenome into 500 kb query regions and align each query to other regions (horizontal alignment) to find its best match. Normalize scores by aligning every query to randomized chromatin state sequences as a negative control. Tabulate real vs. randomized scores (e.g. alignment_scores.csv) and verify the real distribution is shifted above zero while the randomized distribution is close to 0.

## Related tools

- **EpiAlign** (Performs the modified Smith-Waterman dynamic-programming local alignment of compressed chromatin state sequences with weighted mismatch/deletion functions) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Encodes multi-track epigenomic signals (e.g. NIH Roadmap mark data) into the single-track chromatin state sequences that EpiAlign aligns)

## Examples

```
git clone https://github.com/zzz3639/EpiAlign  # then run EpiAlign horizontal alignment on compressed ChromHMM sequences with eps_N=1.5, eps_D=1
```

## Evaluation signals

- Real-query alignment scores average above 0 while scores against randomized chromatin state sequences are close to 0, yielding visibly separated distributions (score_distributions.png)
- The dynamic-programming matrix update is 0-floored, so scores cannot go negative — check no negative intermediate alignment scores are produced
- With the compression step enabled, per-query runtime is roughly 20x lower than uncompressed alignment (compression_benchmark.csv runtime ratio)
- Compressed alignment separates real chromosomes from randomized controls while uncompressed alignment fails to do so
- Top 500 regions of real epigenomes score significantly higher than those of randomized and hybrid epigenomes

## Limitations

- Uncompressed chromatin state sequences contain long stretches of a single state (e.g. quiescent 'o'), which can dominate alignment; omitting the compression step inflates runtime ~20x and degrades real-vs-random separation
- The quiescent state 'o' lacks a good biological interpretation and is excluded when extracting recurrent motifs
- Alignment results depend on the ChromHMM annotation; noisy ChromHMM annotations may need further biological validation (paper Supplementary section 12, not provided)
- The README warns that downloading the package as a zip file directly may cause errors in links; use 'git clone' instead
- How to construct more specialized substitution matrices for chromatin states remains an open research question

## Evidence

- [abstract] EpiAlign is a dynamic programming algorithm that novelly incorporates varying lengths and frequencies of chromatin states.: "EpiAlign is a dynamic programming algorithm that novelly incorporates varying lengths and frequencies of chromatin states."
- [methods] This optimization problem can be approached by dynamic programming, an algorithm that iteratively maintains and updates a matrix M that stores dynamic alignment results.: "This optimization problem can be approached by dynamic programming, an algorithm that iteratively maintains and updates a matrix M that stores dynamic alignment results."
- [methods] The motivation of adding a compression step lies in the fact that most uncompressed (chromatin state) sequences contain long stretches of a single chromatin state: "The motivation of adding a compression step lies in the fact that most uncompressed (chromatin state) sequences contain long stretches of a single chromatin state"
- [results] we align every query chromatin state sequence to randomized chromatin state sequences, which serve as a negative control: "we align every query chromatin state sequence to randomized chromatin state sequences, which serve as a negative control"
- [readme] if an error occurs,  we recommend you to download our package by "git clone" ruther than downloading the zip file.: "if an error occurs,  we recommend you to download our package by "git clone" ruther than downloading the zip file."
