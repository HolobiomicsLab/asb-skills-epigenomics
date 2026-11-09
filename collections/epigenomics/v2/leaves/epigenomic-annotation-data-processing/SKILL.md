---
name: epigenomic-annotation-data-processing
description: Use when multi-track epigenomic signals have been reduced to chromatin
  state annotations and you need compressed state sequences to compute horizontal
  alignment scores, or to distinguish real epigenomes from randomized negative controls.
when_to_use_negative:
- Input is raw multi-track epigenomic signal (e.g. ChIP-seq bigWig tracks) rather
  than discrete chromatin state annotations — a chromatin state annotation method
  such as ChromHMM must be applied first.
- The goal is vertical (cross-sample, gene-level) alignment rather than horizontal
  (within-epigenome, 500 kb region) alignment.
- Sequences are uncompressed and long quiescent stretches would dominate the alignment
  score — apply the compression step first.
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_3676
tools:
- name: EpiAlign
  role: Performs the local alignment (dynamic programming with weighted matching/mismatch/deletion
    functions) of compressed chromatin state sequences and computes horizontal alignment
    scores; data files bundled with its GitHub package supply the Roadmap annotations.
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Upstream chromatin state annotation method that encodes multi-track epigenomic
    signals into the single-track state sequences this skill processes.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:8815a75809903da9bd1b9c7980160730acc16e4a7b788e82360341658c965ff9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_3676
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# epigenomic-annotation-data-processing

## Summary

Preprocess chromatin state annotations (e.g. ChromHMM output from NIH Roadmap epigenomes) into compressed single-track state sequences suitable for EpiAlign local alignment scoring, including real-vs-randomized negative-control comparison of horizontal alignment scores.

## When to use

Use when multi-track epigenomic signals have been reduced to chromatin state annotations and you need compressed state sequences to compute horizontal alignment scores, or to distinguish real epigenomes from randomized negative controls.

## When NOT to use

- Input is raw multi-track epigenomic signal (e.g. ChIP-seq bigWig tracks) rather than discrete chromatin state annotations — a chromatin state annotation method such as ChromHMM must be applied first.
- The goal is vertical (cross-sample, gene-level) alignment rather than horizontal (within-epigenome, 500 kb region) alignment.
- Sequences are uncompressed and long quiescent stretches would dominate the alignment score — apply the compression step first.

## Inputs

- NIH Roadmap chromatin state annotation files (ChromHMM state labels per genomic region)
- Chromatin state sequences for a target epigenome (e.g. Roadmap E003 ESC)
- Randomized chromatin state sequences (negative control)

## Outputs

- Compressed chromatin state sequences per 500 kb query region
- alignment_scores.csv with real-alignment and randomized-control alignment scores per query
- score_distributions.png comparing real vs randomized horizontal alignment score distributions

## How to apply

Load NIH Roadmap ChromHMM chromatin state annotation files (e.g. bundled with the EpiAlign GitHub data) and extract the chromatin state sequence for the epigenome of interest (e.g. ESC sample Roadmap E003). Compress consecutive occurrences of the same state into a single state label (e.g. abbcc -> abc), because uncompressed sequences contain long stretches of a single state (notably quiescent 'o') that would dominate alignment. Divide the epigenome into 500 kb query regions and score each query with EpiAlign's dynamic-programming local alignment using weighted matching/mismatching/deletion functions, mismatch penalty epsilon_N = 1.5 and deletion penalty epsilon_D = 1, with a 0-floored matrix update. Generate randomized chromatin state sequences as a negative control and align each query to them with identical settings; tabulate per-query scores (e.g. alignment_scores.csv) and compare the two score distributions (e.g. score_distributions.png).

## Related tools

- **EpiAlign** (Performs the local alignment (dynamic programming with weighted matching/mismatch/deletion functions) of compressed chromatin state sequences and computes horizontal alignment scores; data files bundled with its GitHub package supply the Roadmap annotations.) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Upstream chromatin state annotation method that encodes multi-track epigenomic signals into the single-track state sequences this skill processes.)

## Examples

```
git clone https://github.com/zzz3639/EpiAlign
```

## Evaluation signals

- Regions in the real ESC (E003) epigenome have average horizontal alignment scores higher than 0.
- Regions in the randomized (negative-control) epigenome score close to 0.
- The real-query score distribution is visibly shifted above the randomized negative-control distribution in score_distributions.png.
- The separation is reproduced only when the compression step (consecutive same-state runs collapsed to one label) is enabled.
- Each uncompressed annotation row's consecutive repeated state labels collapse to a single label (e.g. abbcc -> abc) before alignment.

## Limitations

- Long stretches of a single chromatin state (e.g. the quiescent state 'o') make uncompressed sequences uninformative; the compression step is required to avoid quiescent stretches dominating alignment.
- Alignment results depend on the upstream ChromHMM annotations; noisy ChromHMM annotations may need further biological validation (referenced as Supplementary section 12 in the paper).
- Downloading the package as a zip file directly may cause errors in links; the README recommends 'git clone' instead.

## Evidence

- [methods] Second, we compress consecutive occurrences of the same state into a state label.: "Second, we compress consecutive occurrences of the same state into a state label."
- [results] We first divide a given epigenome into regions of 500 kb length, and then we align the chromatin state sequence of each region (i.e. the ‘query’) to those of other regions to find the best match: "We first divide a given epigenome into regions of 500 kb length, and then we align the chromatin state sequence of each region (i.e. the ‘query’) to those of other regions to find the best match"
- [results] we align every query chromatin state sequence to randomized chromatin state sequences, which serve as a negative control: "we align every query chromatin state sequence to randomized chromatin state sequences, which serve as a negative control"
- [readme] Downloading zip file directly may cause errors in links.: "Downloading zip file directly may cause errors in links."
