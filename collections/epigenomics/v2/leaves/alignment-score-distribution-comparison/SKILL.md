---
name: alignment-score-distribution-comparison
description: 'Use when validating an alignment-based epigenomic method: you need to
  show horizontal alignment scores on real ChromHMM chromatin state sequences separate
  from a randomized negative control, or to normalize scores by a randomized baseline.'
when_to_use_negative:
- You need cross-sample (vertical) alignment of gene-region chromatin state sequences
  between epigenomes rather than within-epigenome horizontal score distributions.
- Input sequences are raw multi-track epigenomic signal matrices instead of compressed
  single-track chromatin state sequences from ChromHMM.
- You intend to compare alignment scores across different weight schemes or penalty
  settings — scores are only comparable under identical epsilon_N/epsilon_D and weight
  settings.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3656
tools:
- name: EpiAlign
  role: Performs the local (horizontal) alignment of compressed chromatin state sequences
    and computes the alignment scores for real queries and randomized negative controls
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Produces the chromatin state annotations that encode multi-track epigenomic
    signals into the single-track state sequences being aligned
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:e3dbb0618abdfef3379bee6b6da6a98bd0cc67865253e6a5165b42003c648866
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3656
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# alignment-score-distribution-comparison

## Summary

Reproduces EpiAlign's negative-control validation: horizontal alignment scores of real chromatin state sequence regions (e.g. Roadmap ESC sample E003, 500 kb queries) are compared against scores from alignments to randomized chromatin state sequences, demonstrating that real regions score clearly above 0 while randomized controls score near 0.

## When to use

Use when validating an alignment-based epigenomic method: you need to show horizontal alignment scores on real ChromHMM chromatin state sequences separate from a randomized negative control, or to normalize scores by a randomized baseline.

## When NOT to use

- You need cross-sample (vertical) alignment of gene-region chromatin state sequences between epigenomes rather than within-epigenome horizontal score distributions.
- Input sequences are raw multi-track epigenomic signal matrices instead of compressed single-track chromatin state sequences from ChromHMM.
- You intend to compare alignment scores across different weight schemes or penalty settings — scores are only comparable under identical epsilon_N/epsilon_D and weight settings.

## Inputs

- NIH Roadmap chromatin state annotation files (ChromHMM state labels per epigenome, e.g. ESC sample E003)
- 500 kb query regions with compressed chromatin state sequences
- Randomized chromatin state sequences generated as negative controls
- EpiAlign parameters: equal or frequency-based state weights, epsilon_N = 1.5, epsilon_D = 1

## Outputs

- alignment_scores.csv with real-alignment and randomized-control alignment scores per query region
- score_distributions.png overlaying the real vs randomized score distributions

## How to apply

Load NIH Roadmap chromatin state annotation files (e.g. bundled with the EpiAlign GitHub data) and extract chromatin state sequences for the epigenome of interest. Compress consecutive occurrences of the same chromatin state into a single state label (e.g. abbcc -> abc) so that long quiescent stretches do not dominate alignment. Divide the epigenome into 500 kb query regions and run EpiAlign's local alignment — dynamic programming with a 0-floored matrix update M, weighted matching/mismatching/deletion functions, mismatch penalty epsilon_N = 1.5 and deletion penalty epsilon_D = 1 — scoring each query against other real regions. Generate randomized chromatin state sequences as a negative control and align each query to them with identical EpiAlign settings. Tabulate both score sets per query (e.g. alignment_scores.csv), overlay the two score distributions, and check that the real-query distribution is visibly shifted above the randomized-control distribution.

## Related tools

- **EpiAlign** (Performs the local (horizontal) alignment of compressed chromatin state sequences and computes the alignment scores for real queries and randomized negative controls) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Produces the chromatin state annotations that encode multi-track epigenomic signals into the single-track state sequences being aligned)

## Examples

```
git clone https://github.com/zzz3639/EpiAlign  # then run EpiAlign local alignment (epsilon_N=1.5, epsilon_D=1) on compressed 500 kb query states vs real and randomized controls, writing alignment_scores.csv
```

## Evaluation signals

- Real-region average horizontal alignment scores are higher than 0, while randomized epigenome regions score close to 0.
- The real-query score distribution is visibly shifted above the randomized negative-control distribution in the overlaid plot.
- Every query region is 500 kb and its state sequence was compressed so no consecutive identical state labels remain.
- Real and randomized alignments were run with identical settings (same weights, epsilon_N = 1.5, epsilon_D = 1).
- alignment_scores.csv contains paired real/control scores for every query with no missing rows.

## Limitations

- The separation depends on enabling the compression step; uncompressed sequences with long stretches of a single chromatin state (e.g. quiescent state 'o') obscure the distinction.
- Alignment scores are not directly comparable across different weight schemes (equal vs frequency-based) or penalty settings.
- Randomized sequences are a simple negative control and do not capture higher-order chromatin structure of real epigenomes.

## Evidence

- [methods] compression step motivation: "The motivation of adding a compression step lies in the fact that most uncompressed (chromatin state) sequences contain long stretches of a single chromatin state"
- [results] horizontal alignment workflow: [evidence span withheld — anchor status: not_found]
- [results] randomized negative control: "we align every query chromatin state sequence to randomized chromatin state sequences, which serve as a negative control"
- [results] ESC E003 horizontal scores: "We use an ESC sample (Roadmap ID E003) as an example and calculate horizontal alignment scores"
- [abstract] dynamic programming core: "EpiAlign is a dynamic programming algorithm that novelly incorporates varying lengths and frequencies of chromatin states."
