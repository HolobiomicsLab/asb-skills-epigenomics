---
name: randomized-sequence-negative-control
description: Use when computing horizontal or vertical alignment scores over chromatin
  state sequences (e.g. EpiAlign on Roadmap 500 kb regions) and raw scores need a
  null baseline to separate real from randomized signal.
when_to_use_negative:
- Input data are raw multi-track epigenomic signal matrices rather than discrete chromatin
  state sequences — ChromHMM annotation (e.g. 15-state model on Roadmap data) must
  precede this control.
- The comparison targets cross-sample biology (e.g. cell-type grouping or gene-level
  vertical alignment between male/female samples with Wilcoxon tests) where a randomized
  null is not the stated normalization design in the article.
- Sequences are uncompressed — long quiescent stretches dominate alignment and, per
  the paper's benchmark, uncompressed alignment fails to distinguish real chromosomes
  from randomized controls.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0080
tools:
- name: EpiAlign
  role: Performs the dynamic-programming local alignment of compressed chromatin state
    sequences against real and randomized control sequences with epsilon_N = 1.5 and
    epsilon_D = 1.
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Upstream chromatin state annotation that encodes multi-track epigenomic signals
    (e.g. NIH Roadmap marks) into the single-track state sequences to be randomized
    and aligned.
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:d7eb4c4c37074a2752823d7bc098e8cac97b77d938cb401325e81df0c193d9f7
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
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# randomized-sequence-negative-control

## Summary

A normalization/negative-control procedure from EpiAlign in which each query chromatin state sequence is aligned to randomized chromatin state sequences, so that raw horizontal alignment scores can be interpreted relative to a null distribution and real epigenomes distinguished from random ones.

## When to use

Use when computing horizontal or vertical alignment scores over chromatin state sequences (e.g. EpiAlign on Roadmap 500 kb regions) and raw scores need a null baseline to separate real from randomized signal.

## When NOT to use

- Input data are raw multi-track epigenomic signal matrices rather than discrete chromatin state sequences — ChromHMM annotation (e.g. 15-state model on Roadmap data) must precede this control.
- The comparison targets cross-sample biology (e.g. cell-type grouping or gene-level vertical alignment between male/female samples with Wilcoxon tests) where a randomized null is not the stated normalization design in the article.
- Sequences are uncompressed — long quiescent stretches dominate alignment and, per the paper's benchmark, uncompressed alignment fails to distinguish real chromosomes from randomized controls.

## Inputs

- Compressed chromatin state sequences (e.g. ChromHMM state labels per 500 kb genomic region from a Roadmap epigenome such as E003 or E071)
- EpiAlign alignment parameters: mismatch penalty epsilon_N = 1.5, deletion penalty epsilon_D = 1, weighted matching/mismatching/deletion score functions
- Randomized chromatin state sequences generated as negative controls

## Outputs

- Per-query real-alignment and randomized-control alignment scores (e.g. alignment_scores.csv)
- Overlay/comparison of the two score distributions (e.g. score_distributions.png)
- Separation statistics, e.g. top 500 regions scoring significantly higher than randomized and hybrid epigenomes

## How to apply

After running EpiAlign's local (horizontal) alignment — dynamic programming with 0-floored matrix update, weighted matching/mismatching/deletion functions, mismatch penalty epsilon_N = 1.5 and deletion penalty epsilon_D = 1 — on compressed chromatin state sequences (consecutive identical states collapsed, e.g. abbcc -> abc), generate randomized chromatin state sequences as negative controls. Align every query chromatin state sequence (e.g. 500 kb regions from a Roadmap epigenome such as E003 ESC or E071 brain) to the randomized sequences using identical EpiAlign settings. Tabulate the real-alignment and randomized-control scores per query and compare the two distributions: regions in the real epigenome score above 0 on average while randomized controls score close to 0. This also serves as score normalization, since the paper aligns every query to randomized sequences 'which serve as a negative control'; the same design powers the benchmark in which compressed alignment distinguishes real chromosomes from randomized controls while uncompressed alignment fails.

## Related tools

- **EpiAlign** (Performs the dynamic-programming local alignment of compressed chromatin state sequences against real and randomized control sequences with epsilon_N = 1.5 and epsilon_D = 1.) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Upstream chromatin state annotation that encodes multi-track epigenomic signals (e.g. NIH Roadmap marks) into the single-track state sequences to be randomized and aligned.)

## Examples

```
python run_epialign.py --query regions_E003_compressed.txt --control randomized_seqs.txt --epsN 1.5 --epsD 1 --out alignment_scores.csv
```

## Evaluation signals

- Real-query score distribution is visibly shifted above the randomized negative-control distribution; real regions average above 0 while randomized controls score close to 0.
- Top 500 regions of the real epigenome score significantly higher than those of randomized and hybrid epigenomes.
- With the compression step enabled, compressed alignment separates real chromosomes from randomized controls (and the uncompressed baseline takes roughly 20x more computation time while failing to separate).
- Identical alignment settings (epsilon_N = 1.5, epsilon_D = 1, same weighting scheme) are used for both real and randomized alignments so the comparison is a fair null.

## Limitations

- The paper's alignment results are based on ChromHMM annotations; noisy ChromHMM states that need further biological validation were observed (see Supplementary section 12, not included here).
- The quiescent state 'o' lacks a good biological interpretation, and long stretches of it motivate the compression step — uncompressed sequences let quiescent runs dominate the score.
- How to construct specialized substitution matrices for chromatin states remains an open question in the article's discussion.

## Evidence

- [results] we align every query chromatin state sequence to randomized chromatin state sequences, which serve as a negative control: "we align every query chromatin state sequence to randomized chromatin state sequences, which serve as a negative control"
- [methods] Second, we compress consecutive occurrences of the same state into a state label.: "Second, we compress consecutive occurrences of the same state into a state label."
- [results] Regions in the real ESC epigenome have average horizontal alignment scores higher than 0, while regions in the randomized epigenome score close to 0: [evidence span withheld — anchor status: not_found]
- [methods] The motivation of adding a compression step lies in the fact that most uncompressed (chromatin state) sequences contain long stretches of a single chromatin state: "The motivation of adding a compression step lies in the fact that most uncompressed (chromatin state) sequences contain long stretches of a single chromatin state"
