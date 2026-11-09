---
name: statistical-significance-testing-of-model-comparison
description: Use when comparing two or more scoring/prediction methods on the same
  positive/negative region sets and you must show AUC differences are statistically
  significant, not just numerically larger.
when_to_use_negative:
- Predictions come from different methods on non-paired or differently sized region
  sets, invalidating paired DeLong comparison of ROC curves
- You only need overall software benchmarking against published AUCs (e.g. TriplexAligner
  comparison) rather than paired testing of scoring variants on the same data
- The question is classification accuracy at a fixed threshold rather than ranking/discriminative
  power measured by AUC
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: pROC
  role: R package (v1.17.0.1) used to compute ROC curves/AUCs and run roc.test (two-sided
    DeLong's test) comparing scoring methods per lncRNA
- name: R
  role: Environment in which roc.test, one-sided Mann-Whitney wilcox.test and p.adjust
    (method 'BH') are executed
- name: 3plex
  role: Produces the stability-derived (Normalised stability) scores being compared
    against match-count scoring
  repo: https://github.com/molinerisLab/3plex
- name: Triplexator
  role: Reference match-count-based Triplexator potential scoring used as the baseline
    in the comparison
- name: bedtools
  role: Generates matched negative regions (bedtools shuffle) and merges overlapping
    TTSs for the Normalised stability score
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:235da1cc1b53a130c312b9091803663cafc982a82ab41c04f19fe2edf851d884
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
  - R
  - Triplexator
  - bedtools
  - pROC
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Statistical significance testing of model comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Formally tests whether two scoring strategies (e.g. stability-derived vs match-count-based TPX scoring) differ in discriminative power, using paired ROC comparisons (DeLong's test) per target and non-parametric score-separation tests with multiple-testing correction.

## When to use

Use when comparing two or more scoring/prediction methods on the same positive/negative region sets and you must show AUC differences are statistically significant, not just numerically larger.

## When NOT to use

- Predictions come from different methods on non-paired or differently sized region sets, invalidating paired DeLong comparison of ROC curves
- You only need overall software benchmarking against published AUCs (e.g. TriplexAligner comparison) rather than paired testing of scoring variants on the same data
- The question is classification accuracy at a fixed threshold rather than ranking/discriminative power measured by AUC

## Inputs

- Per-region scores from each scoring method on Top1000 positive peak sets and bedtools-shuffled matched negative regions for 7 TPX-validated lncRNAs
- Positive/negative labels per scored region (positive = Top1000 RNA:DNA interaction peaks, negative = shuffled matched regions excluding positives, genome gaps, ENCODE blacklist)

## Outputs

- ROC curves and AUC values per lncRNA and per scoring method (pROC)
- DeLong roc.test p-values comparing scoring methods per lncRNA
- Mann-Whitney wilcox.test p-values for positive-vs-negative score separation, BH-corrected
- AUC table written to scoring_method_auc_comparison.csv (columns: lncRNA, scoring method, AUC)

## How to apply

For each of the 7 TPX-validated lncRNAs, compute ROC curves and AUCs for each scoring method (Triplexator potential match-count scoring and 3plex Normalised stability scoring) with the pROC R package (version 1.17.0.1), then compare methods per lncRNA with roc.test using a two-sided DeLong's test on the paired ROC curves. Additionally assess whether positive-vs-negative score distributions separate with a one-sided Mann-Whitney wilcox.test, correcting p-values across lncRNAs with Benjamini-Hochberg (p.adjust, method 'BH'). Collect per-lncRNA, per-method AUCs into a table (columns: lncRNA, scoring method, AUC) written to scoring_method_auc_comparison.csv; validation expects the stability-derived Normalised stability scoring to attain an AUC at least as high as the match-count Triplexator potential scoring, reproducing the reported +0.01 average AUC increase for the Best stability method.

## Related tools

- **pROC** (R package (v1.17.0.1) used to compute ROC curves/AUCs and run roc.test (two-sided DeLong's test) comparing scoring methods per lncRNA)
- **R** (Environment in which roc.test, one-sided Mann-Whitney wilcox.test and p.adjust (method 'BH') are executed)
- **3plex** (Produces the stability-derived (Normalised stability) scores being compared against match-count scoring) — https://github.com/molinerisLab/3plex
- **Triplexator** (Reference match-count-based Triplexator potential scoring used as the baseline in the comparison)
- **bedtools** (Generates matched negative regions (bedtools shuffle) and merges overlapping TTSs for the Normalised stability score)

## Examples

```
roc_res <- roc.test(roc(scores$Triplexator_potential, labels), roc(scores$Normalised_stability, labels), method="delong", alternative="two.sided"); p_adj <- p.adjust(sapply(lncrnas, mw_p), method="BH")
```

## Evaluation signals

- scoring_method_auc_comparison.csv has exactly the columns lncRNA, scoring method, AUC and one row per lncRNA x method combination
- Stability-derived Normalised stability AUC >= Triplexator potential (match-count) AUC for the TPX-validated lncRNAs
- Best stability scoring shows an average AUC increase of ~0.01 over the Triplexator potential baseline
- DeLong roc.test p-values are reported per lncRNA with two-sided testing; Mann-Whitney p-values are BH-corrected via p.adjust(method='BH')
- pROC version is pinned to 1.17.0.1 so ROC/AUC and DeLong results are reproducible

## Limitations

- A modest average AUC gain (+0.01) may not reach per-lncRNA significance for every lncRNA; only 2 of 7 lncRNAs had IDR conservative peaks, so peak-set choice limits generalisation
- Comparison is restricted to the 7 TPX-validated lncRNAs with Top1000 peaks; results may not extend to lncRNAs lacking experimental TPX validation
- Stability values derive from the LongTarget in vitro denaturation experiment collection, so scores inherit the limitations of that dataset

## Evidence

- [results] The Best stability scoring method outperformed other scoring functions, producing an average AUC increase of 0.01 compared to the usual Triplexator potential (match-count-based) scoring.: [evidence span withheld — anchor status: not_found]
- [methods] Compare scoring methods per lncRNA with roc.test (two-sided DeLong's test) and assess positive-vs-negative score separation with one-sided Mann-Whitney wilcox.test and Benjamini-Hochberg correction (p.adjust, method 'BH').: [evidence span withheld — anchor status: not_found]
- [methods] Compute ROC curves and AUC values for each lncRNA and each scoring method with the pROC R package (version 1.17.0.1).: [evidence span withheld — anchor status: not_found]
