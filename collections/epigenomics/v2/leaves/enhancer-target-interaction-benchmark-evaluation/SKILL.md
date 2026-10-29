---
name: enhancer-target-interaction-benchmark-evaluation
description: Use when a new or existing enhancer-target prediction method must be
  compared against TargetFinder and Random Forest baselines on BENGI ground-truth
  datasets with cross-validated AUPRC/F1 metrics.
when_to_use_negative:
- The goal is to train a new enhancer-target predictor from scratch rather than benchmark
  existing methods against published baselines
- The interaction ground truth or the BENGI chromosome-based CV grouping is unavailable,
  making a fair cross-validated comparison impossible
- Candidate ET pairs extend beyond 500 KB, outside the distance regime for which CENTRE
  and this benchmark were designed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3169
tools:
- name: CENTRE
  role: Pre-trained cell-type-specific XGBoost classifier evaluated on BENGI datasets
    as the method under benchmark
  repo: https://github.com/slrvv/CENTRE
- name: TargetFinder
  role: Baseline method; full GradientBoostingClassifier model (n_estimators=4000,
    learning_rate=0.1, max_depth=5, max_features='log2') reimplemented for five cell
    lines plus a reduced 13-feature model for tissue datasets
  repo: https://github.com/shwhalen/targetfinder
- name: XGBoost
  role: Provides the pre-trained extreme gradient boosting classifier used for CENTRE
    predictions in the benchmark
  repo: https://github.com/slrvv/CENTRE
- name: R
  role: Environment in which the ready-to-use CENTRE framework and benchmark pipeline
    are executed
  repo: https://github.com/slrvv/CENTRE_paper_version
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:d27eaf6fa699b69500d5ebd614f4c5b83f65dcc157fbbd04edc5563c4fd2b221
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slrvv/CENTRE
  tools:
  - CENTRE
  - R
  - TargetFinder
  - XGBoost
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# enhancer-target-interaction-benchmark-evaluation

## Summary

Benchmark enhancer-target (ET) prediction methods by running CENTRE, reimplemented TargetFinder, and a Random Forest baseline over the All-Pairs.Natural-Ratio BENGI datasets under a chromosome-based 12-fold cross-validation scheme, then comparing AUPRC and F1-score per dataset.

## When to use

Use when a new or existing enhancer-target prediction method must be compared against TargetFinder and Random Forest baselines on BENGI ground-truth datasets with cross-validated AUPRC/F1 metrics.

## When NOT to use

- The goal is to train a new enhancer-target predictor from scratch rather than benchmark existing methods against published baselines
- The interaction ground truth or the BENGI chromosome-based CV grouping is unavailable, making a fair cross-validated comparison impossible
- Candidate ET pairs extend beyond 500 KB, outside the distance regime for which CENTRE and this benchmark were designed

## Inputs

- All-Pairs.Natural-Ratio BENGI datasets (hg38 cCREs-ELS vs GENCODE v40 TSS pairs, filtered to 500 KB, with 12 chromosome-based CV groups)
- Pre-trained CENTRE XGBoost model (logistic regression objective, scale_pos_weight=5)
- TargetFinder pre-generated training data for LCLs, HeLa, K562, IMR-90 and NHEK (hg19 BENGI annotation, EPW regions)
- Reduced TargetFinder feature sets (DNase, H3K4me3, H3K27ac, CTCF, distance) for five eQTL tissue datasets

## Outputs

- Per-dataset method comparison table of AUPRC and F1-score for CENTRE, full TargetFinder, reduced TargetFinder, and Random Forest
- Held-out outer cross-validation predictions per method-dataset combination

## How to apply

Load the processed All-Pairs.Natural-Ratio BENGI datasets (hg38 cCREs-ELS vs GENCODE v40 TSS pairs, distance-filtered to 500 KB) together with the 12 cross-validation groups split by chromosome so testing always occurs in different genomic regions than training. Run the pre-trained CENTRE XGBoost classifier (logistic regression objective, scale_pos_weight=5) on each dataset without further hyperparameter optimization. Reimplement the full TargetFinder model (GradientBoostingClassifier with n_estimators=4000, learning_rate=0.1, max_depth=5, max_features='log2', 303 features on enhancer-promoter-window regions) for LCLs, HeLa, K562, IMR-90 and NHEK using the hg19 BENGI annotation, with CENTRE hg38 feature values averaged over pairs mapping to the same hg19 pair; run the reduced 13-feature TargetFinder model (DNase, H3K4me3, H3K27ac, CTCF plus distance) on tissue datasets lacking richer genomic data. Train the Random Forest baseline (n_estimators=300, max_depth=5, max_samples=0.9, class_weight='balanced_subsample') with analogous parameters. Compute AUPRC and F1-score from the held-out outer CV folds for every method-dataset combination and assemble the per-dataset comparison table.

## Related tools

- **CENTRE** (Pre-trained cell-type-specific XGBoost classifier evaluated on BENGI datasets as the method under benchmark) — https://github.com/slrvv/CENTRE
- **TargetFinder** (Baseline method; full GradientBoostingClassifier model (n_estimators=4000, learning_rate=0.1, max_depth=5, max_features='log2') reimplemented for five cell lines plus a reduced 13-feature model for tissue datasets) — https://github.com/shwhalen/targetfinder
- **XGBoost** (Provides the pre-trained extreme gradient boosting classifier used for CENTRE predictions in the benchmark) — https://github.com/slrvv/CENTRE
- **R** (Environment in which the ready-to-use CENTRE framework and benchmark pipeline are executed) — https://github.com/slrvv/CENTRE_paper_version

## Examples

```
estimator = GradientBoostingClassifier(n_estimators=4000, learning_rate=0.1, max_depth=5, max_features='log2', random_state=0); scores = cross_val_score(estimator, predictors_df, labels, scoring='f1', cv=cv, n_jobs=-1)
```

## Evaluation signals

- CENTRE outperforms TargetFinder in 11 out of 13 BENGI datasets (reproduces Fig. 4A ordering)
- CENTRE significantly outperforms the reduced TargetFinder model on the five eQTL tissue datasets (Fig. 4B)
- Testing chromosomes are disjoint from training chromosomes in every fold (12 chromosome-based CV groups)
- AUPRC and F1 are computed only from held-out outer CV folds, never from training folds
- TargetFinder feature counts match the specification: 303 features for the full model, 13 features for the reduced model

## Limitations

- F1 scores remain low on many datasets and performance depends strongly on the choice of validation data
- CENTRE is restricted to interactions within 500 kb; extending this interval requires future methodological improvements
- The full TargetFinder model can only be run for cell lines with rich annotation (LCLs, HeLa, K562, IMR-90, NHEK); tissues require the reduced 13-feature model

## Evidence

- [results] In the 12-fold cross-validated BENGI benchmark comparison, CENTRE outperforms TargetFinder in 11 out of 13 datasets: [evidence span withheld — anchor status: not_found]
- [results] the authors also provide 12 cross-validation (CV) groups split by chromosome. This ensures that testing is always performed in different genomic regions than training: "the authors also provide 12 cross-validation (CV) groups split by chromosome. This ensures that testing is always performed in different genomic regions than training"
- [results] TargetFinder was shown to be the best-performing method across all datasets: "TargetFinder (Whalen et al. 2016) which was shown to be the best-performing method across all datasets"
- [readme] The README documents the GradientBoostingClassifier parameters used for the TargetFinder baseline in the benchmark: [evidence span withheld — anchor status: external_doc]
- [other] The reduced TargetFinder model uses a small feature subset for tissues lacking full genomic data: [evidence span withheld — anchor status: not_found]
