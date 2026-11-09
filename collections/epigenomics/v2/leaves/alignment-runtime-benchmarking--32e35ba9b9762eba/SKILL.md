---
name: alignment-runtime-benchmarking
description: Use when benchmarking the computational cost and discriminative power
  of a chromatin state sequence alignment method, specifically when validating that
  compressing consecutive identical ChromHMM states preserves EpiAlign's ability to
  separate real from randomized epigenomes while reducing runtime.
when_to_use_negative:
- When the input is raw multi-track epigenomic signal (e.g. ChIP-seq/DNase bigWig)
  rather than already-annotated ChromHMM chromatin state sequences — chromatin state
  annotation must be done first with ChromHMM before benchmarking the compression
  step
- When evaluating vertical (gene-by-gene) alignment between sample pairs rather than
  horizontal (500 kb region) alignment within an epigenome
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: EpiAlign
  role: Modified Smith–Waterman dynamic-programming local alignment of chromatin state
    sequences, run on both compressed and uncompressed variants to measure runtime
    and score separation
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Produces the chromatin state annotations whose sequences are compressed and
    aligned for benchmarking
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:4d183763cca6ab7d3cc188e7e4a245d8152c0e161a35befe5251b1b91011de21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# alignment-runtime-benchmarking

## Summary

Benchmarks EpiAlign's compression step by running its modified Smith–Waterman local alignment on both uncompressed and state-compressed ChromHMM chromatin state sequences, recording wall-clock runtime and real-vs-randomized alignment score separation. Validates the paper's claim that compression yields ~20x speedup while preserving (and enabling) the ability to distinguish real Roadmap epigenomes from randomized negative controls.

## When to use

Use when benchmarking the computational cost and discriminative power of a chromatin state sequence alignment method, specifically when validating that compressing consecutive identical ChromHMM states preserves EpiAlign's ability to separate real from randomized epigenomes while reducing runtime.

## When NOT to use

- When the input is raw multi-track epigenomic signal (e.g. ChIP-seq/DNase bigWig) rather than already-annotated ChromHMM chromatin state sequences — chromatin state annotation must be done first with ChromHMM before benchmarking the compression step
- When evaluating vertical (gene-by-gene) alignment between sample pairs rather than horizontal (500 kb region) alignment within an epigenome

## Inputs

- ChromHMM chromatin state sequences for a Roadmap epigenome sample (e.g. E071 brain), uncompressed
- Compressed variant of the same sequences (consecutive identical states condensed to single state labels)
- Randomized chromatin state sequences (negative control)
- EpiAlign alignment parameters: mismatch penalty εN=1.5, deletion penalty εD=1

## Outputs

- compression_benchmark.csv with runtime ratio (uncompressed/compressed) and real-vs-randomized alignment score separation for both variants
- Per-region horizontal alignment scores for compressed and uncompressed runs
- Wall-clock computation time per run for each sequence variant

## How to apply

Load ChromHMM chromatin state sequences for a Roadmap sample (e.g. E071 brain) in both uncompressed form and compressed form, where consecutive occurrences of the same state are condensed into a single state label. Run EpiAlign's modified Smith–Waterman dynamic-programming local alignment separately on each variant using the default mismatch penalty εN=1.5 and deletion penalty εD=1, recording wall-clock time for each run. For each variant, align every query chromatin state sequence to randomized chromatin state sequences as a negative control to obtain normalized alignment scores. Tabulate the runtime ratio and the real-versus-randomized score separation (mean score above 0 for real regions vs. scores near 0 for randomized) for both variants. The validation target is that uncompressed alignment takes roughly 20 times more computation time on average than compressed alignment, and that compressed alignment distinguishes real chromosomes from randomized controls while uncompressed alignment fails to do so.

## Related tools

- **EpiAlign** (Modified Smith–Waterman dynamic-programming local alignment of chromatin state sequences, run on both compressed and uncompressed variants to measure runtime and score separation) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Produces the chromatin state annotations whose sequences are compressed and aligned for benchmarking)

## Examples

```
git clone https://github.com/zzz3639/EpiAlign.git
```

## Evaluation signals

- Uncompressed alignment wall-clock time is approximately 20x that of compressed alignment on average
- Real epigenome regions score above 0 on average under compressed alignment; randomized control scores are close to 0
- Top 500 real regions score significantly higher than those of randomized and hybrid epigenomes
- Uncompressed alignment fails to separate real chromosomes from randomized controls
- compression_benchmark.csv contains runtime ratio and score-separation columns for both compressed and uncompressed variants

## Limitations

- The ~20x speedup and score-separation claims are grounded in EpiAlign's specific implementation (ChromHMM-derived state sequences, εN=1.5, εD=1) and may not generalize to other annotation tools or parameter settings
- No version numbers stated for ChromHMM or EpiAlign in the article, and no changelog found for the EpiAlign repository
- The motivation for compression is that most uncompressed sequences contain long stretches of a single chromatin state; results depend on this property holding for the input data
- Downloading the zip file directly may cause errors in links; the README recommends using 'git clone' to obtain the EpiAlign package

## Evidence

- [methods] Second, we compress consecutive occurrences of the same state into a state label.: "Second, we compress consecutive occurrences of the same state into a state label."
- [methods] The motivation of adding a compression step lies in the fact that most uncompressed (chromatin state) sequences contain long stretches of a single chromatin state: "The motivation of adding a compression step lies in the fact that most uncompressed (chromatin state) sequences contain long stretches of a single chromatin state"
- [results] we align every query chromatin state sequence to randomized chromatin state sequences, which serve as a negative control: "we align every query chromatin state sequence to randomized chromatin state sequences, which serve as a negative control"
- [readme] if an error occurs,  we recommend you to download our package by "git clone" ruther than downloading the zip file.: "if an error occurs,  we recommend you to download our package by "git clone" ruther than downloading the zip file."
