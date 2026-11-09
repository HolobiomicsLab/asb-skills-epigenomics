---
name: pairwise-epigenome-local-alignment
description: Use when two genomic regions are represented as single-track chromatin
  state sequences (ChromHMM labels) and you need a local alignment score to find which
  regions/gene bodies are epigenomically similar to a query region.
when_to_use_negative:
- Input is still multi-track epigenomic signal matrices (raw 25 bp window signals)
  — ChromHMM state annotation must be applied first to obtain chromatin state sequences
- You need comparison of continuous signal intensities or substitution matrices over
  chromatin states — the paper notes constructing specialized substitution matrices
  for chromatin states is an open future research question
- Query motifs include the quiescent state 'o', which the paper excludes because it
  lacks a good biological interpretation
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: EpiAlign
  role: Performs the modified Smith–Waterman local alignment (dynamic programming
    with match, mismatch, and deletion score functions) on compressed chromatin state
    sequences
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Encodes multi-track epigenomic signals into the single-track chromatin state
    sequences that EpiAlign consumes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:4e15f82f7ceebe4481d5e43b76661fd6d036c98dd64907a406f960a3024953ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# pairwise-epigenome-local-alignment

## Summary

EpiAlign-based pairwise local alignment of ChromHMM-derived chromatin state sequences to score epigenomic similarity between two genomic regions (e.g., gene bodies) by compressing consecutive identical states and running a modified Smith–Waterman dynamic programming alignment.

## When to use

Use when two genomic regions are represented as single-track chromatin state sequences (ChromHMM labels) and you need a local alignment score to find which regions/gene bodies are epigenomically similar to a query region.

## When NOT to use

- Input is still multi-track epigenomic signal matrices (raw 25 bp window signals) — ChromHMM state annotation must be applied first to obtain chromatin state sequences
- You need comparison of continuous signal intensities or substitution matrices over chromatin states — the paper notes constructing specialized substitution matrices for chromatin states is an open future research question
- Query motifs include the quiescent state 'o', which the paper excludes because it lacks a good biological interpretation

## Inputs

- ChromHMM-annotated single-track chromatin state sequences for genomic regions (e.g., NIH Roadmap Epigenomics sample E071)
- Gene region coordinates for the query gene (e.g., NRG3) and candidate protein-coding genes (GENCODE v10, compatible with the Roadmap database)
- EpiAlign penalty parameters epsilon_N = 1.5 (mismatch) and epsilon_D = 1 (deletion); choice of equal or frequency-based state weights

## Outputs

- Local alignment score M(n1, n2) for each query–candidate region pair
- Ranked score table of candidate genes (e.g., nrg3_alignment_scores.csv) with the best-matching gene identified (expected: GRIA1 for NRG3 in E071)
- Locally aligned chromatin regions with their compressed state-sequence alignments

## How to apply

First annotate multi-track epigenomic signals (e.g., NIH Roadmap Epigenomics samples like brain sample E071) with ChromHMM to obtain single-track chromatin state sequences per region. Apply the compression step to collapse consecutive occurrences of the same state into a single state label — this removes long quiescent/low-state stretches that would otherwise dominate the alignment. Run EpiAlign's modified Smith–Waterman local alignment (dynamic programming maintaining matrix M) between the compressed query sequence and each candidate sequence, designing mismatch and deletion score functions from the weight of each chromatin state (equal weights of 1, or frequency-based weights that up-weight rarer states). Use the paper's default penalty parameters epsilon_N = 1.5 (mismatch) and epsilon_D = 1 (deletion). Record the final local alignment score M(n1, n2) for each pair, rank candidates, and validate against expected results (e.g., in brain sample E071, NRG3's best protein-coding-gene match is GRIA1 when using the compression step).

## Related tools

- **EpiAlign** (Performs the modified Smith–Waterman local alignment (dynamic programming with match, mismatch, and deletion score functions) on compressed chromatin state sequences) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Encodes multi-track epigenomic signals into the single-track chromatin state sequences that EpiAlign consumes)

## Examples

```
git clone https://github.com/zzz3639/EpiAlign
```

## Evaluation signals

- Compressed sequences contain no consecutive occurrences of the same state label (compression invariants hold)
- Reproduces the paper's benchmark: in brain sample E071 with the compression step, the top-ranked protein-coding gene aligned to NRG3 is GRIA1, matching the published finding
- Alignment score table (e.g., nrg3_alignment_scores.csv) contains a score M(n1, n2) for every candidate protein-coding gene from GENCODE v10 compatible with the Roadmap database
- Randomized chromatin state sequences used as negative controls yield lower alignment scores than real queries (score normalization check)
- Chosen weight scheme (equal weights of 1 vs frequency-based weights) and penalties epsilon_N = 1.5, epsilon_D = 1 are explicitly recorded and consistently applied

## Limitations

- Uncompressed chromatin state sequences contain long stretches of a single chromatin state, which dominate the alignment — the compression step is required; supplementary Figure S1 (quiescent/low state lengths) documents this
- The quiescent state 'o' is excluded from motif analysis because it lacks a good biological interpretation
- Alignment results depend on ChromHMM, which can produce likely noisy annotations needing further biological validation (Supplementary section 12)
- How to construct more specialized substitution matrices for chromatin states is an explicit open question

## Evidence

- [methods] Second, we compress consecutive occurrences of the same state into a state label.: "Second, we compress consecutive occurrences of the same state into a state label."
- [methods] EpiAlign then performs a local alignment between two genomic regions based on their compressed state sequences.: "EpiAlign then performs a local alignment between two genomic regions based on their compressed state sequences."
- [methods] We design the mismatch and deletion score functions based on the weight of each chromatin state in each sequence.: "We design the mismatch and deletion score functions based on the weight of each chromatin state in each sequence."
- [methods] in the brain sample E071, when we apply EpiAlign with the compression step, the brain-specific gene NRG3 has the best alignment with another brain-specific gene GRIA1: "in the brain sample E071, when we apply EpiAlign with the compression step, the brain-specific gene NRG3 has the best alignment with another brain-specific gene GRIA1"
- [results] we use all protein-coding genes in GENCODE v10 (28) that are compatible with the Roadmap database: "we use all protein-coding genes in GENCODE v10 (28) that are compatible with the Roadmap database"
- [results] we align every query chromatin state sequence to randomized chromatin state sequences, which serve as a negative control: "we align every query chromatin state sequence to randomized chromatin state sequences, which serve as a negative control"
- [readme] we recommend you to download our package by "git clone" ruther than downloading the zip file.: "we recommend you to download our package by "git clone" ruther than downloading the zip file."
