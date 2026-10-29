---
name: genomic-region-shuffling-negative-control
description: Use when benchmarking TPX-prediction software (3plex, Triplexator, TriplexAligner,
  fasim-LongTarget) and a matched randomised genomic negative set is needed to compute
  ROC AUCs against experimentally derived positive peaks.
when_to_use_negative:
- The analysis goal is to test sequence-level rather than region-level discrimination
  — use the shuffled-transcript strategy (fasta-shuffle-letters, -kmer 1) on the positive
  regions instead.
- No experimentally supported positive peak set exists; AUC benchmarking against randomised
  negatives is not meaningful without true positives.
- Only a small positive set is available (e.g. fewer than the Top1000 peaks), making
  1000 resampled AUC estimates unstable.
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0092
tools:
- name: bedtools
  role: Shuffles positive peaks into matched randomised genomic negative regions preserving
    the length distribution while excluding positives, genome gaps and ENCODE blacklist
    regions
- name: 3plex
  role: TPX prediction tool under evaluation; run with default parameters on positives
    and shuffled negatives, scored by Best stability
  repo: https://github.com/molinerisLab/3plex
- name: Triplexator
  role: Baseline TPX prediction tool run on the same positive/negative sets under
    the Hoogsteen pairing-rules algorithm
- name: TriplexAligner
  role: Comparison tool run with default parameters; per-peak score is the best -log10E
    (1000 substituted for infinite scores)
- name: fasim-LongTarget
  role: Comparison tool run with default parameters; per-peak score is the best MeanStability
- name: pROC
  role: R package (v1.17.0.1) used to compute ROC curves and AUC values for every
    tool across the 1000 negative resamplings
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:a093555e49e832cb67dfd0fbb52ae07770d0430c3305024c27f5b288546dc2ef
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/molinerisLab/3plex
  repo_url: https://github.com/molinerisLab/3plex
  tools:
  - 3plex
  - MEME suite
  - TriplexAligner
  - TriplexFPP
  - Triplexator
  - bedtools
  - fasim-LongTarget
  - pROC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# genomic-region-shuffling-negative-control

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates matched negative-control genomic regions with bedtools shuffle to benchmark RNA:DNA triplex (TPX) prediction tools, resampling the negatives 1000 times to estimate AUC standard deviation when comparing 3plex against Triplexator, TriplexAligner and fasim-LongTarget.

## When to use

Use when benchmarking TPX-prediction software (3plex, Triplexator, TriplexAligner, fasim-LongTarget) and a matched randomised genomic negative set is needed to compute ROC AUCs against experimentally derived positive peaks.

## When NOT to use

- The analysis goal is to test sequence-level rather than region-level discrimination — use the shuffled-transcript strategy (fasta-shuffle-letters, -kmer 1) on the positive regions instead.
- No experimentally supported positive peak set exists; AUC benchmarking against randomised negatives is not meaningful without true positives.
- Only a small positive set is available (e.g. fewer than the Top1000 peaks), making 1000 resampled AUC estimates unstable.

## Inputs

- Positive peak BED set (e.g. Top1000 RNA:DNA interaction peaks for 7 TPX-validated lncRNAs)
- Genome chromosome-sizes file for bedtools shuffle
- ENCODE blacklist regions BED (to exclude)
- Genome gaps track (to exclude)
- ssRNA transcript sequences in FASTA (GENCODE v32 lncRNA transcripts)
- dsDNA target sequences in multi-FASTA

## Outputs

- Shuffled negative-region BED sets (1000 resamplings)
- Per-peak tool scores for positives and negatives (e.g. Best stability, -log10E, MeanStability)
- ROC curves and AUC values with standard deviations from pROC (v1.17.0.1)
- Tool-accuracy comparison table (e.g. TriplexAligner 0.519 ± 0.004, fasim-LongTarget 0.595 ± 0.004, 3plex 0.660 ± 0.002)

## How to apply

Take experimentally supported positive regions — e.g. the Top1000 RNA:DNA interaction peaks for the 7 TPX-validated lncRNAs — and shuffle them across the genome with bedtools shuffle, preserving the peak length distribution while excluding the positive regions themselves, genome gaps and ENCODE blacklist regions. Run each prediction tool with default parameters on both positives and shuffled negatives, repeating the negative sampling 1000 times so that an AUC standard deviation can be estimated per tool. Assign a per-peak score per tool (e.g. Best stability for 3plex, best -log10E for TriplexAligner, best MeanStability for fasim-LongTarget, 0 when no interaction is predicted) and compute ROC curves and AUCs with the pROC R package (v1.17.0.1). The rationale is that shuffled regions control for peak length and mappability without discarding the enrichment signal captured by real positive loci; a complementary strategy 2 (shuffling transcript sequence with fasta-shuffle-letters -kmer 1) can be run in parallel to disentangle sequence-level from region-level discrimination.

## Related tools

- **bedtools** (Shuffles positive peaks into matched randomised genomic negative regions preserving the length distribution while excluding positives, genome gaps and ENCODE blacklist regions)
- **3plex** (TPX prediction tool under evaluation; run with default parameters on positives and shuffled negatives, scored by Best stability) — https://github.com/molinerisLab/3plex
- **Triplexator** (Baseline TPX prediction tool run on the same positive/negative sets under the Hoogsteen pairing-rules algorithm)
- **TriplexAligner** (Comparison tool run with default parameters; per-peak score is the best -log10E (1000 substituted for infinite scores))
- **fasim-LongTarget** (Comparison tool run with default parameters; per-peak score is the best MeanStability)
- **pROC** (R package (v1.17.0.1) used to compute ROC curves and AUC values for every tool across the 1000 negative resamplings)

## Examples

```
bedtools shuffle -i Top1000_peaks.bed -g hg38.chrom.sizes -excl encode_blacklist.merged.bed > negatives_shuffled.bed
```

## Evaluation signals

- Shuffled negatives must match the positive peak length distribution and contain no overlap with positives, genome gaps or ENCODE blacklist regions.
- AUC standard deviations must be derived from 1000 negative resamplings per tool (e.g. 3plex 0.660 ± 0.002 vs TriplexAligner 0.519 ± 0.004).
- All tools must be run with default parameters and identical positive sets so AUCs are directly comparable.
- Per-peak scoring rules must be applied consistently (score 0 when no interaction is predicted; 1000 in place of infinite TriplexAligner scores).
- Reproduced AUC table should show 3plex achieving comparable or better accuracy than the other tools under this negative-control strategy.

## Limitations

- TriplexAligner's AUC under this strategy differed considerably from the value reported by its own authors, so cross-tool AUC comparisons may not match vendor-reported numbers.
- Only 2 of 7 lncRNAs had IDR conservative peaks; region-level benchmarking therefore relies on weaker peak sets (e.g. Top1000) for most transcripts.
- Randomised genomic negatives control for region properties but not for sequence composition; pair with the shuffled-transcript strategy to separate the two effects.

## Evidence
