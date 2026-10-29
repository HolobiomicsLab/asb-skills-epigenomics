---
name: lncrna-binding-site-analysis
description: Use when comparing TPX prediction scoring functions (e.g. Triplexator
  potential match-count vs thermal stability) for lncRNA binding sites, or when validating
  that a stability-based TPX score discriminates experimentally supported peaks from
  matched genomic negatives.
when_to_use_negative:
- The goal is to compare whole prediction tools (3plex vs TriplexFPP, TriplexAligner)
  rather than scoring strategies within one tool — the paper found no good agreement
  between 3plex and TriplexFPP predictions and only 2 of 7 lncRNAs had IDR conservative
  peaks usable for tool comparison
- Predicting de novo TPX interactions without experimentally validated positive regions
  and matched negatives — AUC benchmarking requires a defined positive/negative peak
  framework
- Integrating RNA secondary-structure masking as the primary variable — that is a
  separate 3plex evaluation axis (ss10/ss20/ss50 masking extents), distinct from the
  stability-vs-match-count scoring comparison
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0203
tools:
- name: 3plex
  role: TPX prediction with Normalised stability scoring (maximal thermal stability
    among merged overlapping TTSs / dsDNA length, from LongTarget denaturation data)
  repo: https://github.com/molinerisLab/3plex
- name: Triplexator
  role: Baseline match-count-based Triplexator potential scoring of the same positive/negative
    regions
- name: bedtools
  role: bedtools shuffle to generate length-matched negatives and bedtools merge of
    overlapping TTSs for stability aggregation
- name: pROC
  role: ROC/AUC computation (v1.17.0.1) and two-sided DeLong roc.test per lncRNA
- name: LongTarget
  role: Source of the triplex denaturation experiment collection underlying 3plex's
    thermal stability scores
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:8442ea5a8898c9da6a6f7386cce873c8adf8e467b640d2e61bcf3d224e106b15
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
  - LongTarget
  - R
  - Triplexator
  - bedtools
  - pROC
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# lncrna-binding-site-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark competing lncRNA:dsDNA triplex (TPX) scoring strategies — match-count-based Triplexator potential versus 3plex's thermal-stability-derived scoring — on validated RNA:DNA binding peaks using ROC/AUC analysis. Grounded in the 3plex paper's demonstration that stability-derived measures outperform simple match counts.

## When to use

Use when comparing TPX prediction scoring functions (e.g. Triplexator potential match-count vs thermal stability) for lncRNA binding sites, or when validating that a stability-based TPX score discriminates experimentally supported peaks from matched genomic negatives.

## When NOT to use

- The goal is to compare whole prediction tools (3plex vs TriplexFPP, TriplexAligner) rather than scoring strategies within one tool — the paper found no good agreement between 3plex and TriplexFPP predictions and only 2 of 7 lncRNAs had IDR conservative peaks usable for tool comparison
- Predicting de novo TPX interactions without experimentally validated positive regions and matched negatives — AUC benchmarking requires a defined positive/negative peak framework
- Integrating RNA secondary-structure masking as the primary variable — that is a separate 3plex evaluation axis (ss10/ss20/ss50 masking extents), distinct from the stability-vs-match-count scoring comparison

## Inputs

- Top1000 RNA:DNA interaction peak sets (e.g. RADICL-seq NPM peaks) for the 7 TPX-validated lncRNAs as BED/positive regions
- Reference genome for bedtools shuffle negative generation and exclusion of gaps and ENCODE blacklist regions
- ssRNA transcript sequences in FASTA (one sequence per file, as required by 3plex)
- dsDNA target regions in multi-FASTA format
- LongTarget collection of triplex denaturation experiments (stability lookup used by 3plex)

## Outputs

- TFO:TTS match tables from 3plex (tab-delimited *_tpx.stability.gz with Score, Stability, Motif, Strand, Orientation columns)
- scoring_method_auc_comparison.csv (columns: lncRNA, scoring method, AUC)
- ROC curves and AUC values per lncRNA per scoring method (pROC objects)
- DeLong roc.test p-values and BH-corrected one-sided Mann-Whitney p-values per lncRNA

## How to apply

Load the Top1000 RADICL-seq RNA:DNA interaction peaks for the 7 TPX-validated lncRNAs as positive regions, then generate length-matched negatives with bedtools shuffle, excluding positives, genome gaps and ENCODE blacklist regions. Run Triplexator/3plex with the match-count-based Triplexator potential scoring and re-score the same regions with 3plex's Normalised stability score — the maximal thermal stability (from the LongTarget triplex denaturation collection, He et al. 2015) among bedtools-merged overlapping TTSs divided by dsDNA length. Compute per-lncRNA, per-method ROC curves and AUCs with the pROC R package (version 1.17.0.1), compare scoring methods per lncRNA using two-sided DeLong's test (roc.test), and assess positive-vs-negative score separation with a one-sided Mann-Whitney wilcox.test plus Benjamini-Hochberg correction (p.adjust, method 'BH'). Write the AUC table (columns: lncRNA, scoring method, AUC) to scoring_method_auc_comparison.csv and verify the stability-derived score attains AUC at least as high as the match-count score (paper reports an average AUC increase of 0.01 for the Best stability method).

## Related tools

- **3plex** (TPX prediction with Normalised stability scoring (maximal thermal stability among merged overlapping TTSs / dsDNA length, from LongTarget denaturation data)) — https://github.com/molinerisLab/3plex
- **Triplexator** (Baseline match-count-based Triplexator potential scoring of the same positive/negative regions)
- **bedtools** (bedtools shuffle to generate length-matched negatives and bedtools merge of overlapping TTSs for stability aggregation)
- **pROC** (ROC/AUC computation (v1.17.0.1) and two-sided DeLong roc.test per lncRNA)
- **LongTarget** (Source of the triplex denaturation experiment collection underlying 3plex's thermal stability scores)

## Examples

```
docker run -u `id -u`:`id -g` -it --rm -v $PWD:$PWD imolineris/3plex:v1.0 $PWD/ssRNA.fa $PWD/dsDNA.fa $PWD/results_3plex
```

## Evaluation signals

- scoring_method_auc_comparison.csv exists with exactly the columns lncRNA, scoring method, AUC and covers all 7 TPX-validated lncRNAs under both scoring methods
- The Normalised stability scoring attains an AUC at least as high as the Triplexator potential match-count scoring, reproducing the reported average AUC increase of 0.01
- Negative regions match the positive length distribution and contain no overlap with positives, genome gaps, or ENCODE blacklist regions
- One-sided Mann-Whitney tests with Benjamini-Hochberg correction (p.adjust, method 'BH') indicate significant positive-vs-negative score separation for the expected lncRNA/method combinations
- DeLong roc.test (two-sided) comparisons are computed per lncRNA between paired ROC curves from the same positive/negative sets

## Limitations

- Only 2 of 7 lncRNAs had defined IDR conservative peak sets; the other 5 showed insufficient replicate reproducibility for that peak filtering method
- The stability-scoring advantage is modest (average AUC increase of 0.01 over Triplexator potential), so per-lncRNA differences may not be individually significant
- 3plex requires ssRNA FASTA files containing only one sequence per file; exit code 139 can occur on Linux kernels >= 4.8 unless vsyscall is enabled in Docker

## Evidence
