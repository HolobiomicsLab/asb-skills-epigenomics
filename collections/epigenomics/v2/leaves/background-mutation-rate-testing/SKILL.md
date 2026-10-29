---
name: background-mutation-rate-testing
description: Use when you have hg19 somatic mutations from WGS or WES tumour cohorts
  and need to model expected per-element mutation counts (nMut, nSample, length) so
  that excess burden can be tested for driver candidates.
when_to_use_negative:
- You only have a pre-trained BMR model and test data — the 'driverpower infer' sub-command,
  not model training, is the appropriate entry point.
- Input mutations are not hg19 whole-genome or whole-exome somatic calls, or the regions
  of interest are outside the well-annotated train/test element sets provided with
  the example data.
- The tumour type has very few samples (many PCAWG tumour types have fewer than 30),
  where BMR-based burden testing has limited statistical power.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DriverPower
  role: Implements the 'model' sub-command to train GLM (randomised lasso feature
    selection) or GBM BMR models and the 'infer' sub-command to call driver candidates
    with pre-trained models
  repo: https://github.com/smshuai/DriverPower
- name: XGBoost
  role: Gradient boosting backend (poisson deviance objective) used by DriverPower's
    GBM BMR models with k-fold cross-validation
  repo: https://github.com/dmlc/xgboost
- name: bedtools
  role: Builds response tables by intersecting somatic mutations with elements and
    callable regions (e.g. 'bedtools bed12tobed6 -i ./test_elements.bed12.gz | cut
    -f1-4 > test_elements.tsv') and computes per-element mean CADD scores via bedtools
    intersect/groupby
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:212fe836bc29b0cccf700035db630cf28481b6861d3b3b8634aa2a91dac25b99
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python >=
  - XGBoost
  - bedtools
  - conda
  - dmlc/xgboost
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# background-mutation-rate-testing

## Summary

Train and apply DriverPower background mutation rate (BMR) models that predict regional mutation burden from 1373 genomic features using either randomised-lasso-selected GLMs or k-fold cross-validated XGBoost GBMs, as the burden-test backbone for coding and non-coding cancer driver discovery on PCAWG WGS and TCGA WES data.

## When to use

Use when you have hg19 somatic mutations from WGS or WES tumour cohorts and need to model expected per-element mutation counts (nMut, nSample, length) so that excess burden can be tested for driver candidates.

## When NOT to use

- You only have a pre-trained BMR model and test data — the 'driverpower infer' sub-command, not model training, is the appropriate entry point.
- Input mutations are not hg19 whole-genome or whole-exome somatic calls, or the regions of interest are outside the well-annotated train/test element sets provided with the example data.
- The tumour type has very few samples (many PCAWG tumour types have fewer than 30), where BMR-based burden testing has limited statistical power.

## Inputs

- Somatic mutation file (hg19, e.g. PCAWG WGS or GDC MuTect2 WES callsets)
- Element BED files (train_elements.tsv.gz, test_elements.bed12.gz)
- Callable regions BED
- Training/test feature tables (train_feature.hdf5, test_feature.hdf5)
- Response tables (nMut, nSample, length per element, e.g. train_y.tsv, test_y.tsv)
- Per-element CADD PHRED score table (CADD_per_ele_score.tsv) for functional adjustment

## Outputs

- Trained BMR model files ([name].GBM.model.pkl or [name].GLM.model.pkl, model_info.pkl, XGBoost fold models)
- feature_importance.tsv from randomised lasso / GBM
- Predicted expected mutation burden and driver candidates in [outDir]/[name].result.tsv / DriverPower.result.tsv with BH-corrected q-values

## How to apply

Build per-element response tables (nMut, nSample, length) with the prepare.py helper, bedtools, and a callable-regions BED (use 'bedtools bed12tobed6' for the bed12 test elements), then merge the three split training features with 'cat train_feature.hdf5.part1 train_feature.hdf5.part2 train_feature.hdf5.part3 > train_feature.hdf5' and verify md5 cb4af6bc7979efa087955f94909dd065. Train the BMR model with 'driverpower model': for GBM use --gbmFold 3 (XGBoost, poisson deviance objective, default parameters from xgb_param_default.py) or --gbmFold 5 for the GLM-vs-GBM comparison; for GLM, randomised lasso retains features with importance score >= 0.5 (--featImpCut). At inference ('driverpower infer'), keep --method auto so a dispersion test of the training data selects binomial vs negative binomial p-values, keep --geoMean True (geometric mean of nMut and nSample), and optionally adjust with functional scores via --funcScore CADD_per_ele_score.tsv and --funcScoreCut 'CADD:0.01'. Finally apply Benjamini-Hochberg correction and retain driver candidates with q < 0.1; the pan-cancer BMR model explains up to 93% of mutation-rate variance on training elements and 83% on the held-out 2521 1-Mbp autosomal test elements.

## Related tools

- **DriverPower** (Implements the 'model' sub-command to train GLM (randomised lasso feature selection) or GBM BMR models and the 'infer' sub-command to call driver candidates with pre-trained models) — https://github.com/smshuai/DriverPower
- **XGBoost** (Gradient boosting backend (poisson deviance objective) used by DriverPower's GBM BMR models with k-fold cross-validation) — https://github.com/dmlc/xgboost
- **bedtools** (Builds response tables by intersecting somatic mutations with elements and callable regions (e.g. 'bedtools bed12tobed6 -i ./test_elements.bed12.gz | cut -f1-4 > test_elements.tsv') and computes per-element mean CADD scores via bedtools intersect/groupby)

## Examples

```
driverpower model --feature train_feature.hdf5 --response train_y.tsv --method GBM --gbmFold 3 --name tutorial --modelDir ./output
```

## Evaluation signals

- The merged training feature file matches md5 cb4af6bc7979efa087955f94909dd065.
- The tutorial GBM run logs 'Model metrics for training set: r2=0.63, Variance explained=0.63' and all named files (feature_importance.tsv, model_info.pkl, [name].GBM.model.pkl, result.tsv) exist in the output directory.
- For the pan-cancer cohort, model variance explained is consistent with the paper (~93% on the 867,266 training elements, 83% on the 2521 held-out 1-Mbp test elements).
- Applying the full pipeline to 2583 PCAWG genomes reproduces 217 coding and 95 non-coding driver candidates at q < 0.1.
- TCGA WES transfer runs reproduce 14 coding drivers in 364 TCGA-LIHC and 6 coding drivers in 180 TCGA-PAAD samples, with called genes matching known driver gene sets.

## Limitations

- The BMR is trained on predominantly neutral regions, so negatively selected sites are not modelled, which limits recall.
- False-positive driver calls can arise from variant-calling artefacts or biological processes (e.g. regional AID activity) not captured by the BMR features.
- Only ~3.7% of the genome was studied, and functional impact score schemes (e.g. CADD) are biased toward coding mutations, limiting non-coding discovery.

## Evidence

- [abstract] Using a total of 1373 genomic features derived from public sources, DriverPower's background mutation model explains up to 93% of the regional variance in the mutation rate across multiple tumour: "Using a total of 1373 genomic features derived from public sources, DriverPower's background mutation model explains up to 93% of the regional variance"
- [methods] ``--gbmFold``: [*optional*] Number of model fold to train for GBM. Fold must be an integer >= 2. Default value is 3.: "``--gbmFold``: [*optional*] Number of model fold to train for GBM. Fold must be an integer >= 2. Default value is 3."
- [methods] ``--featImpCut``: [*optional*] cutoff for feature importance score. Features with score >= cutoff will be used in the model. Only used for GLM.: "cutoff for feature importance score. Features with score >= cutoff will be used in the model. Only used for GLM."
- [readme] DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations.: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
- [discussion] only ~ 3.7% of the genome has been studied here. There may be more non-coding drivers in other types of regulatory elements: "only ~ 3.7% of the genome has been studied here. There may be more non-coding drivers in other types of regulatory elements"
