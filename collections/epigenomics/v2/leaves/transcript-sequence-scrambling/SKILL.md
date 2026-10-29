---
name: transcript-sequence-scrambling
description: Use when benchmarking TPX prediction tools on peak regions and you need
  shuffled-transcript negative controls that preserve nucleotide composition while
  destroying genuine triplex-forming signal.
when_to_use_negative:
- When the negative-control strategy of interest is randomised genomic regions (bedtools
  shuffle excluding positives, genome gaps and ENCODE blacklist regions) rather than
  scrambled transcripts — that is a separate strategy in the paper's benchmark.
- When sequence order information must be preserved (e.g. k-mer frequency analyses
  sensitive to local context), since -kmer 1 shuffling destroys all di- and higher-order
  structure.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0659
tools:
- name: MEME suite
  role: Provides fasta-shuffle-letters used with -kmer 1 to scramble transcript sequences
    into composition-matched negatives
- name: 3plex
  role: TPX prediction tool rerun on positives with scrambled transcripts; scored
    per peak by Best stability
  repo: https://github.com/molinerisLab/3plex
- name: TriplexAligner
  role: TPX prediction tool rerun on scrambled transcripts; scored per peak by best
    -log10E (infinite scores replaced by 1000)
- name: fasim-LongTarget
  role: TPX prediction tool rerun on scrambled transcripts; scored per peak by best
    MeanStability
- name: pROC
  role: R package (v1.17.0.1) used to compute ROC curves and AUCs under the shuffled-transcript
    strategy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:c7d0f1df3b98492a8733b918a18795c939206417f2f81096d0a1ab57fe130953
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Transcript sequence scrambling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates negative-control sequences by shuffling transcript nucleotides with fasta-shuffle-letters (MEME suite, -kmer 1) so that TPX prediction tools can be benchmarked against matched-composition but non-functional transcript sequences.

## When to use

Use when benchmarking TPX prediction tools on peak regions and you need shuffled-transcript negative controls that preserve nucleotide composition while destroying genuine triplex-forming signal.

## When NOT to use

- When the negative-control strategy of interest is randomised genomic regions (bedtools shuffle excluding positives, genome gaps and ENCODE blacklist regions) rather than scrambled transcripts — that is a separate strategy in the paper's benchmark.
- When sequence order information must be preserved (e.g. k-mer frequency analyses sensitive to local context), since -kmer 1 shuffling destroys all di- and higher-order structure.

## Inputs

- GENCODE v32 lncRNA transcript sequences (FASTA) corresponding to the 7 TPX-validated lncRNAs
- Top1000 RNA:DNA interaction peak regions (BED/interval set) for the positive class
- fasta-shuffle-letters from the MEME suite

## Outputs

- Scrambled transcript FASTA files (single-nucleotide-shuffled, -kmer 1)
- Per-peak prediction scores from each tool on scrambled transcripts
- ROC curves and AUC values under the shuffled-transcript negative strategy (pROC v1.17.0.1)

## How to apply

Take the GENCODE v32 lncRNA transcript sequences underlying the positive (Top1000 RADICL-seq peak) regions and scramble them with fasta-shuffle-letters from the MEME suite using -kmer 1, which preserves mononucleotide composition while breaking any triplex-forming pattern. Rerun each TPX prediction tool (3plex, TriplexAligner, fasim-LongTarget, Triplexator) with default parameters on the positive regions paired with the scrambled transcripts instead of the native ones. Assign a single per-peak score for ROC analysis: Best stability for 3plex, best -log10E for TriplexAligner, best MeanStability for fasim-LongTarget; score 0 when no interaction is predicted, and 1000 in place of infinite TriplexAligner scores. Compute ROC curves and AUC with the pROC R package (v1.17.0.1) and compare against the randomised-genomic-region negative-control strategy to assess each tool's discrimination under composition-matched negatives.

## Related tools

- **MEME suite** (Provides fasta-shuffle-letters used with -kmer 1 to scramble transcript sequences into composition-matched negatives)
- **3plex** (TPX prediction tool rerun on positives with scrambled transcripts; scored per peak by Best stability) — https://github.com/molinerisLab/3plex
- **TriplexAligner** (TPX prediction tool rerun on scrambled transcripts; scored per peak by best -log10E (infinite scores replaced by 1000))
- **fasim-LongTarget** (TPX prediction tool rerun on scrambled transcripts; scored per peak by best MeanStability)
- **pROC** (R package (v1.17.0.1) used to compute ROC curves and AUCs under the shuffled-transcript strategy)

## Examples

```
fasta-shuffle-letters -kmer 1 < lncRNA_transcripts.fa > lncRNA_scrambled.fa
```

## Evaluation signals

- Scrambled sequences must retain mononucleotide composition of the originals (guaranteed by fasta-shuffle-letters -kmer 1).
- AUC values must be reported per tool and compared against the randomised-region strategy (reference AUCs: 3plex 0.660 ± 0.002, fasim-LongTarget 0.595 ± 0.004, TriplexAligner 0.519 ± 0.004 under random-region negatives).
- Peaks with no predicted interaction must receive score 0; infinite TriplexAligner scores must be replaced by 1000, so the score table contains no NaN/Inf values.
- AUC standard deviations must be estimable via 1000 negative resamplings, consistent with the paper's protocol.

## Limitations

- The AUC value of TriplexAligner under this strategy was considerably different from the value reported by its authors, so cross-tool comparison of published AUCs may not be reproducible.
- -kmer 1 shuffling preserves only mononucleotide composition; tools relying on dinucleotide or higher-order patterns may behave differently than on true biological negatives.
- Per-peak score assignment conventions (0 for no prediction, 1000 for infinite TriplexAligner scores) are arbitrary and can affect ROC computation.

## Evidence

- [methods] Strategy 2 (shuffled-transcript negatives): rerun the tools on the positive regions using transcript sequences scrambled with fasta-shuffle-letters from the MEME suite with -kmer 1: [evidence span withheld — anchor status: not_found]
- [methods] assign per-peak scores (Best stability for 3plex, best -log10E for TriplexAligner, best MeanStability for fasim-LongTarget, 0 when no interaction is predicted, and 1000 in place of infinite TriplexAligner scores): [evidence span withheld — anchor status: not_found]
- [methods] Compute ROC curves and AUC values for all tools under both strategies with the pROC R package (v1.17.0.1): [evidence span withheld — anchor status: not_found]
- [results] 3plex outperformed both tools, with resampling AUCs of TriplexAligner 0.519 ± 0.004, fasim-LongTarget 0.595 ± 0.004, and 3plex 0.660 ± 0.002: [evidence span withheld — anchor status: not_found]
