---
name: roc-auc-tool-comparison
description: Use when multiple TPX-prediction tools or scoring functions must be compared
  quantitatively on the same positive (Top1000 RNA:DNA peaks) and negative (bedtools-shuffled
  genomic) region sets.
when_to_use_negative:
- When the goal is de novo TPX discovery on a single transcript rather than a head-to-head
  accuracy benchmark against experimental positive/negative peaks.
- When no experimentally validated positive regions (e.g. RADICL-seq Top1000 peaks)
  exist for the lncRNAs, since ROC/AUC labels cannot be defined.
- When comparing TriplexFPP predictions, since the article reports no good agreement
  between TriplexFPP and rule-based tool outputs.
tools:
- name: 3plex
  role: TPX predictor under evaluation; supplies Best stability per-peak scores
  repo: https://github.com/molinerisLab/3plex
- name: Triplexator
  role: Match-count baseline predictor scored with Triplexator potential
- name: TriplexAligner
  role: Probabilistic-model predictor scored with best -log10E per peak
- name: fasim-LongTarget
  role: Thermal-stability predictor scored with best MeanStability per peak
- name: bedtools
  role: Generates length-matched shuffled negative regions and merges overlapping
    TTSs for stability scoring
- name: pROC
  role: Computes ROC curves, AUCs and DeLong roc.test comparisons (v1.17.0.1)
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:7fe404de18667b67e04590f743afe8ed5db04b84a57c0bef3bd5bec2a305349d
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
  - R
  - RNAplfold
  - TriplexAligner
  - TriplexFPP
  - Triplexator
  - ViennaRNA
  - bedtools
  - fasim-LongTarget
  - pROC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# roc-auc-tool-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark triplex (TPX) prediction tools by scoring positive vs negative regions, computing ROC curves and AUC with the pROC R package (v1.17.0.1), and comparing AUCs across tools and scoring strategies.

## When to use

Use when multiple TPX-prediction tools or scoring functions must be compared quantitatively on the same positive (Top1000 RNA:DNA peaks) and negative (bedtools-shuffled genomic) region sets.

## When NOT to use

- When the goal is de novo TPX discovery on a single transcript rather than a head-to-head accuracy benchmark against experimental positive/negative peaks.
- When no experimentally validated positive regions (e.g. RADICL-seq Top1000 peaks) exist for the lncRNAs, since ROC/AUC labels cannot be defined.
- When comparing TriplexFPP predictions, since the article reports no good agreement between TriplexFPP and rule-based tool outputs.

## Inputs

- Top1000 RNA:DNA interaction peak sets (BED) for the 7 TPX-validated lncRNAs as positives
- GENCODE v32 lncRNA transcript sequences (FASTA)
- Negative regions generated with bedtools shuffle (length-matched, excluding positives, genome gaps and ENCODE blacklist regions)
- Per-peak tool scores from 3plex, Triplexator, TriplexAligner or fasim-LongTarget with assigned class labels (positive/negative)

## Outputs

- ROC curves and AUC values per tool and lncRNA (pROC v1.17.0.1)
- AUC standard deviations from 1000 negative-region resamplings
- roc.test DeLong p-values and BH-corrected Mann-Whitney wilcox.test p-values
- AUC comparison tables, e.g. scoring_method_auc_comparison.csv and ss_masking_auc_comparison.csv (rows: lncRNA x method/masking level, AUC in [0,1])

## How to apply

Load the Top1000 RNA:DNA interaction peaks for the 7 TPX-validated lncRNAs as positives and generate matched negatives with bedtools shuffle, preserving the peak length distribution while excluding positives, genome gaps and ENCODE blacklist regions. Run each tool (3plex, Triplexator, TriplexAligner, fasim-LongTarget) with default parameters and assign one score per peak (Best stability for 3plex, best -log10E for TriplexAligner, best MeanStability for fasim-LongTarget; 0 when no interaction is predicted, 1000 in place of infinite TriplexAligner scores). Repeat the negative-region sampling 1000 times to estimate AUC standard deviations, and compute ROC curves and AUC values with pROC (v1.17.0.1). Compare scoring methods or tools per lncRNA using roc.test with two-sided DeLong's test, and positive-vs-negative score separation with a one-sided Mann-Whitney wilcox.test plus Benjamini-Hochberg correction (p.adjust, method 'BH'). Validation requires 3plex to reach AUCs comparable or better than the other tools (e.g. randomised negative-control AUCs: TriplexAligner 0.519 ± 0.004, fasim-LongTarget 0.595 ± 0.004, 3plex 0.660 ± 0.002) and stability-derived Best stability scoring to beat the match-count Triplexator potential by ~0.01 average AUC.

## Related tools

- **3plex** (TPX predictor under evaluation; supplies Best stability per-peak scores) — https://github.com/molinerisLab/3plex
- **Triplexator** (Match-count baseline predictor scored with Triplexator potential)
- **TriplexAligner** (Probabilistic-model predictor scored with best -log10E per peak)
- **fasim-LongTarget** (Thermal-stability predictor scored with best MeanStability per peak)
- **bedtools** (Generates length-matched shuffled negative regions and merges overlapping TTSs for stability scoring)
- **pROC** (Computes ROC curves, AUCs and DeLong roc.test comparisons (v1.17.0.1))

## Examples

```
library(pROC); roc1 <- roc(response=labels, predictor=scores_3plex); roc2 <- roc(response=labels, predictor=scores_triplexator); auc(roc1); roc.test(roc1, roc2, method="delong")
```

## Evaluation signals

- 3plex AUC under randomised genomic-region negatives must be highest among tools (0.660 ± 0.002 vs fasim-LongTarget 0.595 ± 0.004 and TriplexAligner 0.519 ± 0.004).
- AUC standard deviations must be derived from 1000 negative-region resamplings and be small (±0.002–0.004).
- scoring_method_auc_comparison.csv must show Best stability scoring exceeding the Triplexator potential match-count scoring by about 0.01 average AUC.
- Every reported AUC must lie within [0, 1], and each lncRNA x method combination must be present (e.g. 7 x 4 for the masking table).
- Per-lncRNA roc.test (two-sided DeLong) and BH-corrected one-sided Mann-Whitney tests must support positive-vs-negative score separation for TPX-validated lncRNAs.

## Limitations

- TriplexAligner's AUC under the article's negative-control strategy differed considerably from the value reported by its own authors, so cross-paper AUC comparisons are not directly transferable.
- TriplexFPP did not agree with rule-based predictions and failed to identify experimentally known TPX-forming lncRNAs (CDKN2B-AS1, lncSmad7, MEG3, NEAT1, TERC).
- Only 2 of 7 lncRNAs had IDR conservative peak sets; the other 5 showed insufficient replicate reproducibility for that filtering method.

## Evidence
