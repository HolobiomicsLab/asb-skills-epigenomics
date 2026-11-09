---
name: auc-metric-computation
description: Use when a ranked, score-filtered list of candidate target regions (e.g.
  from adaptive_liftover plus gr_candidate_filter) must be evaluated against matched
  orthologous ground-truth regions, or when tuning a window-size parameter via AUROC/AUPR
  in leave-one-out cross-validation.
when_to_use_negative:
- There is no matched orthologous ground truth (e.g. no corresponding human epigenome
  peaks for the held-out dataset), so positives/negatives cannot be defined and AUROC/AUPR
  are meaningless.
- Candidate regions are unscored peak intervals that have not yet gone through adaptive_liftover,
  similarity computation, and gr_candidate_filter — ranking-based AUC evaluation would
  be premature.
- 'The task is parameter learning rather than evaluation: use training_module() to
  estimate logistic regression parameters and suggest an optimal score threshold instead.'
tools:
- name: AdaLiftOver
  role: Produces the scored and filtered candidate target regions whose ranking quality
    is measured by AUROC/AUPR
  repo: https://github.com/keleslab/AdaLiftOver
- name: PRROC
  role: R package dependency of AdaLiftOver used for ROC and PR curve computation
  repo: https://github.com/keleslab/AdaLiftOver
- name: gr_candidate_filter
  role: Retains at most best_k candidates per query region above a score threshold,
    defining the ranked lists evaluated
  repo: https://github.com/keleslab/AdaLiftOver
- name: GenomicRanges
  role: Operates on genomic intervals when classifying candidates as correct by overlap
    with matched orthologous regions
  repo: https://github.com/keleslab/AdaLiftOver
- name: data.table
  role: Fast data manipulation/aggregation of per-fold AUROC/AUPR results across the
    window-size grid
  repo: https://github.com/keleslab/AdaLiftOver
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:c8b18727568f07264ad6c2a1c909f801d015e733e5f879a61007b1fecc2d7f94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/keleslab/AdaLiftOver
  tools:
  - AdaLiftOver
  - GenomicRanges
  - PRROC
  - R
  - data.table
  - gr_candidate_filter
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# auc-metric-computation

## Summary

Computes area under the ROC (AUROC) and area under the PR (AUPR) curves to evaluate how well AdaLiftOver's scored candidate target regions rank true orthologous regions, sweeping the query-extension window size over a LOOCV grid to select the optimal local window.

## When to use

Use when a ranked, score-filtered list of candidate target regions (e.g. from adaptive_liftover plus gr_candidate_filter) must be evaluated against matched orthologous ground-truth regions, or when tuning a window-size parameter via AUROC/AUPR in leave-one-out cross-validation.

## When NOT to use

- There is no matched orthologous ground truth (e.g. no corresponding human epigenome peaks for the held-out dataset), so positives/negatives cannot be defined and AUROC/AUPR are meaningless.
- Candidate regions are unscored peak intervals that have not yet gone through adaptive_liftover, similarity computation, and gr_candidate_filter — ranking-based AUC evaluation would be premature.
- The task is parameter learning rather than evaluation: use training_module() to estimate logistic regression parameters and suggest an optimal score threshold instead.

## Inputs

- Scored candidate target regions per query region (GRangesList with logistic probability scores from adaptive_liftover, compute_similarity_epigenome, compute_similarity_grammar)
- Binary ground-truth labels defined by overlap with matched orthologous regions (e.g. ENCODE human epigenome peaks for each held-out mouse dataset)
- Window-size grid: 0 to 5 kb in 400 bp increments (13 window sizes)
- 67 matched human-mouse ENCODE ChIP-seq/DNase-seq datasets and the mm10.hg38 reciprocal-best chain file

## Outputs

- Per-window-size AUROC and AUPR with mean and standard deviation across all 67 LOOCV folds
- loocv_results.csv summary table of AUROC/AUPR by window size
- Selected optimal window size (~2 kb) for AdaLiftOver's local query extension

## How to apply

For each leave-one-out fold (67 matched human-mouse ENCODE TF/DNA methylation ChIP-seq and DNase-seq datasets), map the held-out dataset's query regions with adaptive_liftover while excluding that dataset from the epigenomic signal similarity calculations, aggregate epigenome signal and sequence grammar similarities as logistic probability scores, and apply gr_candidate_filter with best_k and a score threshold. Label the top-scored candidate target region as correct (positive) if it overlaps the matched orthologous region of the held-out dataset. Sweep the query-region extension window size from 0 to 5 kb in 400 bp increments (13 window sizes) and, for each size, compute AUROC and AUPR across all 67 folds (e.g. with the PRROC package, a dependency of AdaLiftOver). Record mean and standard deviation per window size and write the summary to loocv_results.csv; the grid yielded ~2 kb as optimal (AUROC 0.820 ± 0.0113, AUPR 0.604 ± 0.0371), which AdaLiftOver uses as its default local window.

## Related tools

- **AdaLiftOver** (Produces the scored and filtered candidate target regions whose ranking quality is measured by AUROC/AUPR) — https://github.com/keleslab/AdaLiftOver
- **PRROC** (R package dependency of AdaLiftOver used for ROC and PR curve computation) — https://github.com/keleslab/AdaLiftOver
- **gr_candidate_filter** (Retains at most best_k candidates per query region above a score threshold, defining the ranked lists evaluated) — https://github.com/keleslab/AdaLiftOver
- **GenomicRanges** (Operates on genomic intervals when classifying candidates as correct by overlap with matched orthologous regions) — https://github.com/keleslab/AdaLiftOver
- **data.table** (Fast data manipulation/aggregation of per-fold AUROC/AUPR results across the window-size grid) — https://github.com/keleslab/AdaLiftOver

## Examples

```
gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- loocv_results.csv contains one AUROC/AUPR row per window size for all 13 sizes spanning 0–5 kb in 400 bp increments.
- AUROC at the ~2 kb window falls in the expected range (0.820 ± 0.0113) and AUPR at 0.604 ± 0.0371 across the 67 folds.
- Positives are defined only by overlap of top-scored candidates with the matched orthologous region of the held-out dataset, and the held-out dataset is excluded from epigenomic signal similarity calculations in each fold.
- Candidate scores are logistic probabilities filtered with gr_candidate_filter (best_k and score threshold), so evaluated lists should contain at most best_k entries per query region.
- Mean and standard deviation are reported per window size, not only per-fold point estimates.

## Limitations

- Existing sequence-conservation methods for mapping non-coding regions across species suffer from multi-mapping, low precision, and low mapping rate issues, motivating ranking-based AUPR evaluation rather than accuracy alone.
- For organisms other than mouse (e.g. rat), a user-defined orthologous epigenome repertoire must be collected and logistic regression parameters learned with training_module() before scored candidates can be generated and evaluated.
- The ~2 kb optimal window was calibrated on the 67 matched human-mouse ENCODE datasets via LOOCV and is applied as AdaLiftOver's default; other species/repertoires may warrant re-tuning.

## Evidence

- [readme] For each query region, AdaLiftOver outputs a scored and filtered list of candidate target regions that are most similar to the query region in terms of regulatory information.: "For each query region, AdaLiftOver outputs a scored and filtered list of candidate target regions that are most similar to the query region in terms of regulatory information."
- [other] Optimizing the area under the ROC and PR curves across the LOOCV grid yielded ~2 kb as the optimal window size (AUROC: 0.820 ± 0.0113, AUPR: 0.604 ± 0.0371): [evidence span withheld — anchor status: not_found]
- [intro] To conduct LOOCV, we first apply AdaLiftOver to map each rat epigenome dataset to human while excluding the current dataset for epigenomic signal similarity calculations.: "To conduct LOOCV, we first apply AdaLiftOver to map each rat epigenome dataset to human while excluding the current dataset for epigenomic signal similarity calculations."
- [readme] ## PRROC
install.packages("PRROC"): "## PRROC
install.packages("PRROC")"
