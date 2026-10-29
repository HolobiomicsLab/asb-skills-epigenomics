---
name: permutation-based-alignment-score-normalization
description: Use when computing EpiAlign horizontal alignment scores within an epigenome
  (e.g. 500 kb regions of a Roadmap sample) and raw Smith-Waterman-style scores must
  be calibrated against a random background before ranking or clustering.
when_to_use_negative:
- Input regions are short or dominated entirely by the quiescent state 'o', whose
  long uniform stretches make compressed sequences uninformative for alignment (see
  Supplementary Figure S1 motivation)
- The analysis compares chromatin state sequences across two fixed epigenomes vertically
  (e.g. per-gene comparisons of 19,935 protein-coding genes between sample pairs)
  rather than horizontal within-epigenome scoring that needs a null background
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: EpiAlign
  role: Performs the Smith-Waterman-style local alignment of compressed chromatin
    state sequences, run both on real query-target pairs and on query-randomized pairs
    to build the null distribution used for score normalization
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Produces the single-track chromatin state sequences from multi-track Roadmap
    epigenomic signals that are compressed, aligned, and randomized for the normalization
    step
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:814979cc42dc8f8e0db7f92a6c71f0292765345ec9a5be34023b39fca0a671a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# permutation-based-alignment-score-normalization

## Summary

Normalizes EpiAlign local alignment scores by re-aligning each query chromatin state sequence to randomized chromatin state sequences as a negative control, so horizontal-alignment scores reflect enrichment over chance and are comparable across regions and epigenomes.

## When to use

Use when computing EpiAlign horizontal alignment scores within an epigenome (e.g. 500 kb regions of a Roadmap sample) and raw Smith-Waterman-style scores must be calibrated against a random background before ranking or clustering.

## When NOT to use

- Input regions are short or dominated entirely by the quiescent state 'o', whose long uniform stretches make compressed sequences uninformative for alignment (see Supplementary Figure S1 motivation)
- The analysis compares chromatin state sequences across two fixed epigenomes vertically (e.g. per-gene comparisons of 19,935 protein-coding genes between sample pairs) rather than horizontal within-epigenome scoring that needs a null background

## Inputs

- ChromHMM-derived compressed chromatin state sequences for each 500 kb region of a Roadmap epigenome (e.g. E071 brain sample, E003 ESC)
- Per-query best-match alignment scores from EpiAlign horizontal alignment
- Randomized chromatin state sequences generated as negative-control queries/targets

## Outputs

- Null-distribution alignment scores per query from query-vs-randomized alignments
- Normalized horizontal alignment score vectors per epigenome
- Ranked query-target region pairs suitable for recurrent-pattern extraction (patterns excluding the quiescent state 'o')

## How to apply

After running EpiAlign's local alignment of each 500 kb query region against all other regions (with compression of consecutive identical states, frequency-based weights, mismatch penalty epsilon_N = 1.5 and deletion penalty epsilon_D = 1), perform the negative-control step: align every query chromatin state sequence to randomized chromatin state sequences, which serve as a negative control. The randomized alignments provide a null distribution of scores accounting for chromatin state lengths and frequencies (e.g. the long stretches of the quiescent state 'o'), and each observed query-vs-target score can then be interpreted relative to this null. Because frequency-based weights assign larger weights to less common chromatin states, randomization is essential to prevent common states from dominating raw scores. The normalized scores then feed downstream analyses such as extracting recurrent chromatin state patterns (excluding motifs containing state 'o') or hierarchical clustering of Roadmap epigenomes by horizontal alignment score vectors.

## Related tools

- **EpiAlign** (Performs the Smith-Waterman-style local alignment of compressed chromatin state sequences, run both on real query-target pairs and on query-randomized pairs to build the null distribution used for score normalization) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Produces the single-track chromatin state sequences from multi-track Roadmap epigenomic signals that are compressed, aligned, and randomized for the normalization step)

## Examples

```
python run_epialign.py --epigenome E071 --region-size 500kb --penalties --epsN 1.5 --epsD 1 --weights frequency --normalize randomized-control --exclude-states o
```

## Evaluation signals

- Observed query-vs-target scores should be separable from the query-vs-randomized negative-control distribution; queries whose best real match does not exceed the random null indicate no recurrent pattern
- Downstream hierarchical clustering of epigenomes on normalized horizontal alignment score vectors should cluster samples from the same tissue together (e.g. Roadmap brain samples such as E071, as in Figure 5)
- Recovered best-matching region pairs should overlap known tissue-specific gene pairs, e.g. in E071 the brain-specific gene NRG3 has the best alignment with the brain-specific gene GRIA1
- Extracted recurrent motifs should exclude the quiescent state 'o', which the authors note lacks a good biological interpretation
- Normalization should be run with the same penalty parameters (epsilon_N = 1.5, epsilon_D = 1) and weight scheme (frequency-based) as the real alignments so scores are comparable

## Limitations

- Alignment results depend on ChromHMM annotation quality; the authors report finding likely noisy ChromHMM annotations that need further biological validation (Supplementary section 12)
- The quiescent state 'o' produces long stretches with no useful comparison signal, motivating the compression step and the exclusion of 'o'-containing motifs
- How to construct more specialized substitution matrices for chromatin states is an open future research question, so current scores rely on the designed mismatch/deletion functions and frequency-based weights

## Evidence

- [results] negative-control randomization: "we align every query chromatin state sequence to randomized chromatin state sequences, which serve as a negative control"
- [results] horizontal alignment setup: "We first divide a given epigenome into regions of 500 kb length, and then we align the chromatin state sequence of each region (i.e. the ‘query’) to those of other regions to find the best match"
- [methods] compression rationale: "The motivation of adding a compression step lies in the fact that most uncompressed (chromatin state) sequences contain long stretches of a single chromatin state"
- [methods] frequency-based weights: "(ii) frequency-based weights assign larger weights to less common chromatin states (see Supplementary section 1 for details)"
- [results] motif filter: "the state ‘o’ denotes the quiescent state and lacks a good biological interpretation, so we only consider the motifs without ‘o’"
