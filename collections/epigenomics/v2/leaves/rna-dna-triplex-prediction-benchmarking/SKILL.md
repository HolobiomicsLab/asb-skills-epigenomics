---
name: rna-dna-triplex-prediction-benchmarking
description: Use when comparing the prediction accuracy or runtime of triplex-forming
  lncRNA tools on experimentally validated RNA:DNA peaks and needing a fair, negative-control-based
  AUC benchmark with resampling-based confidence estimates.
when_to_use_negative:
- When the goal is predicting TPXs in novel lncRNAs rather than comparing tool accuracy
  against known experimental peaks — use a single tool's prediction mode instead of
  the full benchmark.
- When only non-human or non-GENCODE v32-annotated transcripts are available, since
  the protocol's positive sets and shuffle exclusions (ENCODE blacklist, genome gaps)
  are human-genome specific.
- When evaluating a tool under its own authors' optimised parameter settings rather
  than a default-parameter comparison, as the article notes the TriplexAligner AUC
  differed considerably from the value reported by its authors.
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3511
tools:
- name: 3plex
  role: Tool under benchmark and reference integrator of Hoogsteen pairing rules,
    RNAplfold secondary structure masking and LongTarget-derived thermal stability
    scoring
  repo: https://github.com/molinerisLab/3plex
- name: Triplexator
  role: Baseline TPX prediction tool (Hoogsteen pairing rules) benchmarked for accuracy
    and wall-clock time against 3plex
- name: TriplexAligner
  role: Probabilistic-model-based comparator tool; per-peak score is best -log10E
    with 1000 substituting infinite values
- name: fasim-LongTarget
  role: Thermal-stability-based comparator tool; per-peak score is best MeanStability
- name: TriplexFPP
  role: CNN-based comparator run via the authors' notebook ('triplex lncRNA prediction'
    section) for prediction-agreement checks
  repo: https://github.com/yuuuuzhang/TriplexFPP
- name: bedtools
  role: Generates random-region negative controls via bedtools shuffle preserving
    the peak length distribution
- name: MEME suite
  role: fasta-shuffle-letters with -kmer 1 produces shuffled-transcript negatives
- name: pROC
  role: R package (v1.17.0.1) computing ROC curves and AUC values for all tools under
    both strategies
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:6da4f16adf472c11926ceee42203c99363a54d3a6cfb77d2af60251ffc8d1df8
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
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# rna-dna-triplex-prediction-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking protocol for comparing RNA:DNA triplex (TPX) prediction tools — 3plex, Triplexator, TriplexAligner, fasim-LongTarget and TriplexFPP — under default parameters using Top1000 RADICL-seq peaks as positives and two negative-control strategies, with AUC comparison and wall-clock timing on AMD EPYC hardware.

## When to use

Use when comparing the prediction accuracy or runtime of triplex-forming lncRNA tools on experimentally validated RNA:DNA peaks and needing a fair, negative-control-based AUC benchmark with resampling-based confidence estimates.

## When NOT to use

- When the goal is predicting TPXs in novel lncRNAs rather than comparing tool accuracy against known experimental peaks — use a single tool's prediction mode instead of the full benchmark.
- When only non-human or non-GENCODE v32-annotated transcripts are available, since the protocol's positive sets and shuffle exclusions (ENCODE blacklist, genome gaps) are human-genome specific.
- When evaluating a tool under its own authors' optimised parameter settings rather than a default-parameter comparison, as the article notes the TriplexAligner AUC differed considerably from the value reported by its authors.

## Inputs

- Top1000 RNA:DNA interaction peak sets (BED) for 7 TPX-validated lncRNAs (RADICL-seq NPM data)
- GENCODE v32 lncRNA transcript sequences (FASTA)
- GENCODE v32 / M25 gene annotation
- GENOME gaps and ENCODE blacklist region tracks (BED) for exclusion during shuffle
- ChIP-seq IDR conservative thresholded peaks from ENCODE as true positive binding sites
- Tool outputs from 3plex, Triplexator, TriplexAligner, fasim-LongTarget run with default parameters

## Outputs

- AUC comparison table per tool under both negative-control strategies, with AUC standard deviations from 1000 negative resamplings (pROC v1.17.0.1)
- ROC curves for each tool and strategy
- Per-peak score vectors (Best stability, best -log10E, best MeanStability; 0 for no predicted interaction; 1000 for infinite TriplexAligner scores)
- TriplexFPP lncRNA prediction outputs compared against 3plex calls
- Average wall-clock times with standard deviations for 3plex and Triplexator on TERC (541 nt) and NEAT1 (22743 nt)

## How to apply

Load the Top1000 RNA:DNA interaction peak sets for the 7 TPX-validated lncRNAs plus their GENCODE v32 transcript sequences as positives. Build negatives two ways: (Strategy 1) bedtools shuffle of peak regions preserving the peak length distribution, excluding positives, genome gaps and ENCODE blacklist regions, resampled 1000 times to estimate AUC standard deviation; (Strategy 2) positive regions queried against transcript sequences scrambled with fasta-shuffle-letters (MEME suite, -kmer 1). Run 3plex, Triplexator, TriplexAligner and fasim-LongTarget with default parameters on both positive and negative sets, assigning per-peak scores — Best stability (3plex), best -log10E (TriplexAligner), best MeanStability (fasim-LongTarget), 0 when no interaction is predicted, and 1000 in place of infinite TriplexAligner scores. Also run TriplexFPP via the authors' notebook ('triplex lncRNA prediction' section) and compare predictions. Compute ROC curves and AUCs with pROC (v1.17.0.1), and benchmark computation time on a single core of an AMD EPYC 7313 node (250 GB RAM) by running 3plex and Triplexator 10 times each on TERC (541 nt) and NEAT1 (22743 nt) transcripts, recording average wall-clock times.

## Related tools

- **3plex** (Tool under benchmark and reference integrator of Hoogsteen pairing rules, RNAplfold secondary structure masking and LongTarget-derived thermal stability scoring) — https://github.com/molinerisLab/3plex
- **Triplexator** (Baseline TPX prediction tool (Hoogsteen pairing rules) benchmarked for accuracy and wall-clock time against 3plex)
- **TriplexAligner** (Probabilistic-model-based comparator tool; per-peak score is best -log10E with 1000 substituting infinite values)
- **fasim-LongTarget** (Thermal-stability-based comparator tool; per-peak score is best MeanStability)
- **TriplexFPP** (CNN-based comparator run via the authors' notebook ('triplex lncRNA prediction' section) for prediction-agreement checks) — https://github.com/yuuuuzhang/TriplexFPP
- **bedtools** (Generates random-region negative controls via bedtools shuffle preserving the peak length distribution)
- **MEME suite** (fasta-shuffle-letters with -kmer 1 produces shuffled-transcript negatives)
- **pROC** (R package (v1.17.0.1) computing ROC curves and AUC values for all tools under both strategies)

## Examples

```
docker run -u `id -u`:`id -g` -it --rm -v $PWD:$PWD imolineris/3plex:v1.0 $PWD/ssRNA.fa $PWD/dsDNA.fa $PWD/results_3plex
```

## Evaluation signals

- Under the randomised genomic region negative-control strategy, expected resampling AUCs are TriplexAligner 0.519 ± 0.004, fasim-LongTarget 0.595 ± 0.004, and 3plex 0.660 ± 0.002, with 3plex outperforming both.
- 3plex must achieve comparable or better accuracy than the other tools, with reported AUC standard deviations derived from the 1000 negative resamplings.
- The wall-clock time plot (Fig. 6 analogue) must reproduce average times with standard deviations below 5% over the 10 repeated runs on TERC and NEAT1.
- The reproduced AUC value for TriplexAligner will differ considerably from the value reported by its authors when run under this strategy.
- TriplexFPP predictions should show poor agreement with 3plex and should fail to correctly identify experimentally known TPX-forming lncRNAs such as CDKN2B-AS1, lncSmad7, MEG3, NEAT1 and TERC.

## Limitations

- Only 2 out of 7 lncRNAs had a defined set of IDR conservative peaks; the other 5 showed insufficient reproducibility among replicates to apply this method, so Top1000 peak sets were used instead.
- The TriplexAligner AUC under this benchmarking strategy was considerably different from the value reported by its authors, limiting cross-paper comparability.
- No good agreement was found between 3plex and TriplexFPP predictions, and TriplexFPP failed to identify several experimentally known TPX-forming lncRNAs (CDKN2B-AS1, lncSmad7, MEG3, NEAT1, TERC).
- Benchmarking uses default parameters for all tools; results may not reflect each tool's optimised settings.
- Timing benchmarks are hardware-specific (single core of an AMD EPYC 7313 node with 250 GB RAM).

## Evidence

- [results] Under the randomised genomic region negative-control strategy, 3plex outperformed both tools, with resampling AUCs of TriplexAligner 0.519 ± 0.004, fasim-LongTarget 0.595 ± 0.004, and 3plex 0.660 ± 0.002.: [evidence span withheld — anchor status: not_found]
- [other] assign per-peak scores (Best stability for 3plex, best -log10E for TriplexAligner, best MeanStability for fasim-LongTarget, 0 when no interaction is predicted, and 1000 in place of infinite TriplexAligner scores): [evidence span withheld — anchor status: not_found]
