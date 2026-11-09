---
name: roc-auc-evaluation-of-genomic-predictions
description: Use when predictions of genomic binding regions must be scored against
  curated positive peaks and matched negative regions, and you need AUC comparison
  across parameter settings plus significance testing of parameter effects.
when_to_use_negative:
- When you need tool-vs-tool prediction agreement rather than accuracy against experimental
  peaks — the article notes no good agreement was found between 3plex and TriplexFPP
  predictions
- When your positives are not replicate-filtered peaks and negatives are not length-matched
  shuffled regions — the AUC comparison assumes the described evaluation-set construction
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: 3plex
  role: Generates the TPX predictions (TFO:TTS matches) that are scored by ROC/AUC
    across parameter settings
  repo: https://github.com/molinerisLab/3plex
- name: Triplexator
  role: Provides the potential score and the canonical Hoogsteen pairing rules baseline
    used when excluding 3plex-specific stability and secondary-structure parameters
- name: bedtools
  role: Generates length-matched negative regions (bedtools shuffle) excluding genome
    gaps and ENCODE blacklist regions
- name: pROC
  role: R package (v1.17.0.1) used to compute ROC curves and AUC values for each parameter
    combination
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:b14fcdae0198fd77543f6c4dae79a67bb395f9c36dae0caad4b16e43d987b158
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
  - Triplexator
  - bedtools
  - pROC
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# roc-auc-evaluation-of-genomic-predictions

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates genomic interaction predictions (e.g. ssRNA:dsDNA triplex predictions from 3plex/Triplexator) by computing ROC curves and AUC values per parameter combination with the pROC R package, then testing which parameters (peak filtering method, minimal TPX length, ssRNA masking) affect accuracy via linear-model ANOVA and a 10,000-permutation randomisation test.

## When to use

Use when predictions of genomic binding regions must be scored against curated positive peaks and matched negative regions, and you need AUC comparison across parameter settings plus significance testing of parameter effects.

## When NOT to use

- When you need tool-vs-tool prediction agreement rather than accuracy against experimental peaks — the article notes no good agreement was found between 3plex and TriplexFPP predictions
- When your positives are not replicate-filtered peaks and negatives are not length-matched shuffled regions — the AUC comparison assumes the described evaluation-set construction

## Inputs

- Positive binding-site regions from peak filtering methods (IDR conservative, IDR optimal, Overlap conservative filtered at SPP q<0.05, Top1000, author-provided peaks)
- Negative regions generated with bedtools shuffle, length-matched to positives, excluding genome gaps and ENCODE blacklist regions
- Predictions for 7 TPX-validated lncRNAs (e.g. CDKN2B-AS1, lncSmad7, MEG3, NEAT1, TERC) across parameter combinations (minimal TPX length 16/12/10/8)
- AUC values per parameter combination, in an R data frame with columns for method, ssRNA, n peaks, and minimal length

## Outputs

- ROC curves and AUC values for each parameter combination (computed with pROC v1.17.0.1)
- Panoval from R ANOVA comparison of full vs nested lm models
- Permutation-based relevance probability of the tested parameter from 10,000 permutations
- Parameter-performance ranking (e.g. minimal lengths 8 and 10 best, default 16 worst with 0.1 average AUC reduction)

## How to apply

Build evaluation sets by using peak filtering method outputs (IDR conservative, IDR optimal, Overlap conservative filtered at SPP q<0.05, Top1000, author-provided peaks) as positive regions and drawing negatives with bedtools shuffle, matching the positive length distribution and excluding genome gaps and ENCODE blacklist regions. Run the TPX search (e.g. 3plex or Triplexator) on the 7 TPX-validated lncRNAs for each parameter combination, keeping other parameters at suggested values (minimal length=10, error rate=20%, guanine rate=40%) and scoring with the Triplexator potential score, excluding 3plex-specific stability and secondary-structure parameters. Compute ROC curves and AUCs for each combination with the pROC R package (v1.17.0.1). Fit a full linear model with R lm (AUC ~ peak filtering method + ssRNA + n peaks + minimal length) and a nested model without the minimal length term, then compare with R ANOVA to obtain Panoval; estimate relevance probability via 10,000 permutations of the AUC-versus-parameter association, refitting full and nested models on each permuted dataset while preserving the correlation structure.

## Related tools

- **3plex** (Generates the TPX predictions (TFO:TTS matches) that are scored by ROC/AUC across parameter settings) — https://github.com/molinerisLab/3plex
- **Triplexator** (Provides the potential score and the canonical Hoogsteen pairing rules baseline used when excluding 3plex-specific stability and secondary-structure parameters)
- **bedtools** (Generates length-matched negative regions (bedtools shuffle) excluding genome gaps and ENCODE blacklist regions)
- **pROC** (R package (v1.17.0.1) used to compute ROC curves and AUC values for each parameter combination)

## Examples

```
Rscript -e 'full <- lm(AUC ~ method + ssRNA + n_peaks + min_length, d); nested <- update(full, . ~ . - min_length); anova(full, nested)'
```

## Evaluation signals

- Reproduced ANOVA comparison of full vs nested models yields Panoval < 2.2e-16 for the minimal length parameter
- Permutation-based probability of the minimal length parameter is estimated below 1e-4 (0 of 10,000 permutations yield Panova_random <= Panoval)
- Shorter minimal lengths (8 and 10) show the best AUC performance, minimal length 16 the worst (average AUC reduced by ~0.1), and 12 intermediate
- Negative regions are length-matched to positives and exclude genome gaps and ENCODE blacklist regions (verifiable in the shuffled BED output)
- AUC computed with pROC v1.17.0.1 for every combination of peak filtering method and minimal length (16, 12, 10, 8)

## Limitations

- Only 2 of 7 lncRNAs had IDR conservative peaks; the other 5 showed insufficient reproducibility among replicates to apply this method
- Minimal TPX length of 6 gave no further improvement in a subsample of the parameter space
- Shorter minimal lengths increase predictions but TPXs tend to be shorter and more degenerated than previously expected, so AUC gains may reflect relaxed constraints
- Reported statistics (p-value < 1e-04) come from a permutation test on the collected experimental binding-site dataset and may not transfer to other lncRNAs or assays

## Evidence

- [other] pROC evaluation of parameter combinations: [evidence span withheld — anchor status: not_found]
- [other] Linear model and ANOVA specification: [evidence span withheld — anchor status: not_found]
- [other] Permutation test on minimal length parameter: [evidence span withheld — anchor status: not_found]
