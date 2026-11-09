---
name: epigenomic-query-search-horizontal-alignment
description: Use when you have ChromHMM chromatin state sequences for a single Roadmap
  epigenome and want to find recurrent local chromatin state patterns or -matching
  genomic regions for a query region.
when_to_use_negative:
- Your goal is comparing chromatin state patterns across multiple epigenomes (vertical
  alignment between samples) rather than finding recurrent patterns within a single
  epigenome
- Your input is raw multi-track epigenomic signal matrices rather than discrete chromatin
  state sequences — apply ChromHMM state annotation and the compression step first
- You need to compare regions at single-nucleotide or sequence (DNA) level; EpiAlign
  operates only on coarse chromatin state label sequences
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: EpiAlign
  role: Performs Smith-Waterman-style local alignment of compressed chromatin state
    sequences between 500 kb regions to extract recurrent horizontal patterns
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Encodes multi-track epigenomic signals into single-track chromatin state sequences
    that are the input to the horizontal alignment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:cebf98a3be376fd8393cd75e7aa360917bcf2b5210a6532670f4103956d618d3
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
n_claims: 13
---

# epigenomic-query-search-horizontal-alignment

## Summary

Extract recurrent chromatin state patterns within a single epigenome by dividing it into 500 kb regions and running EpiAlign's Smith-Waterman-style local alignment of each query region against all others, revealing cell-type-specific chromatin patterns.

## When to use

Use when you have ChromHMM chromatin state sequences for a single Roadmap epigenome and want to find recurrent local chromatin state patterns or best-matching genomic regions for a query region.

## When NOT to use

- Your goal is comparing chromatin state patterns across multiple epigenomes (vertical alignment between samples) rather than finding recurrent patterns within a single epigenome
- Your input is raw multi-track epigenomic signal matrices rather than discrete chromatin state sequences — apply ChromHMM state annotation and the compression step first
- You need to compare regions at single-nucleotide or sequence (DNA) level; EpiAlign operates only on coarse chromatin state label sequences

## Inputs

- ChromHMM-derived chromatin state sequences (single-track state labels per genomic region) for a target Roadmap epigenome (e.g. E071 brain sample)
- 500 kb genomic query regions from the epigenome
- Tissue-specific gene sets (e.g. brain-specific genes such as NRG3 and GRIA1) for annotation
- Roadmap sample metadata (e.g. Roadmap ID E003 ESC, E071 brain)

## Outputs

- Best-match alignment for each query region against all other regions
- Normalized horizontal alignment score per query (from randomized-sequence negative control)
- Table of recurrent chromatin state patterns (motifs) annotated with tissue/cell-type association, saved as epialign_horizontal_patterns.tsv

## How to apply

First obtain ChromHMM-derived chromatin state sequences for the target epigenome (e.g. Roadmap brain sample E071) and compress consecutive occurrences of the same state into a single state label, since uncompressed sequences contain long stretches of one state. Divide the epigenome into 500 kb regions and run EpiAlign's local alignment of each query region's compressed state sequence against all other regions to find the best match, using mismatch penalty epsilon_N = 1.5, deletion penalty epsilon_D = 1, and frequency-based weights that assign larger weights to less common chromatin states. As a negative control, align every query to randomized chromatin state sequences to normalize the horizontal alignment scores. Collect the recurrent aligned patterns (motifs), excluding any motif containing the quiescent state 'o' because it lacks a good biological interpretation, then annotate patterns with tissue-specific gene sets and sample metadata. Validate by checking that known tissue-specific gene pairs are recovered, e.g. NRG3 best-aligning with GRIA1 in E071.

## Related tools

- **EpiAlign** (Performs Smith-Waterman-style local alignment of compressed chromatin state sequences between 500 kb regions to extract recurrent horizontal patterns) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Encodes multi-track epigenomic signals into single-track chromatin state sequences that are the input to the horizontal alignment)

## Evaluation signals

- Known tissue-specific gene pairs are recovered: in brain sample E071, NRG3 has the best alignment with another brain-specific gene GRIA1
- Hierarchical clustering of epigenomes based on horizontal alignment score vectors clusters samples from the same tissue together
- Normalized scores against randomized chromatin state sequences (negative control) show real matches score substantially above the randomized baseline
- Output table epialign_horizontal_patterns.tsv exists and contains motifs, with all reported motifs excluding the quiescent state 'o'
- Consecutive occurrences of the same state are compressed to single state labels before alignment (compression step applied)

## Limitations

- Motifs containing the quiescent state 'o' are excluded because the state lacks a good biological interpretation
- Uncompressed chromatin state sequences contain long stretches of a single state, making compression necessary before comparison
- Results depend on ChromHMM annotations; the discussion notes likely noisy ChromHMM annotations that need further biological validation
- How to construct more specialized substitution matrices for chromatin states remains an open research question

## Evidence

- [results] Horizontal alignment of 500 kb regions: "We first divide a given epigenome into regions of 500 kb length, and then we align the chromatin state sequence of each region (i.e. the ‘query’) to those of other regions"
- [results] Randomized negative control normalization: "we align every query chromatin state sequence to randomized chromatin state sequences, which serve as a negative control"
- [methods] Compression step rationale: "The motivation of adding a compression step lies in the fact that most uncompressed (chromatin state) sequences contain long stretches of a single chromatin state"
- [methods] NRG3–GRIA1 validation in E071: "in the brain sample E071, when we apply EpiAlign with the compression step, the brain-specific gene NRG3 has the best alignment with another brain-specific gene GRIA1"
- [results] Exclusion of quiescent state motifs: "the state ‘o’ denotes the quiescent state and lacks a good biological interpretation, so we only consider the motifs without ‘o’"
- [readme] Repository availability: "## README: zzz3639/EpiAlign
Downloading zip file directly may cause errors in links."
