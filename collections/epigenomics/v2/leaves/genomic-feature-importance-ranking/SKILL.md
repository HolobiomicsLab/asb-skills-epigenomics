---
name: genomic-feature-importance-ranking
description: Use when you have labeled genomic interval sets (e.g. HOT vs COLD regions)
  with coordinates and reference genome sequences, and need to rank sequence or genomic
  features by predictive importance.
when_to_use_negative:
- Input is already a pre-computed feature table with ranked importances, so k-mer
  computation and classifier fitting would be redundant
- The goal is functional annotation of HOT regions (e.g. house-keeping gene promoter
  overlap or expression analysis) rather than sequence-feature discrimination
- Regions are too few or too unbalanced between HOT and COLD classes to support 10-fold
  cross-validated elastic net fitting
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0102
tools:
- name: glmnet
  role: Fits the elastic net classifiers with 10-fold cross-validation to tune the
    regularization parameter and yields coefficients used as the feature-importance
    ranking
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:917d525f4c9c8d0051a9f33b9fe407b683050f5cc778629ff14a5bb422149689
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - glmnet
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# genomic-feature-importance-ranking

## Summary

Trains species-specific elastic net classifiers to rank genomic sequence features (k-mer frequencies, GC content, CpG O/E ratio) by their ability to discriminate HOT from COLD regions, mirroring the feature-importance analysis of the 'HOT or not' study.

## When to use

Use when you have labeled genomic interval sets (e.g. HOT vs COLD regions) with coordinates and reference genome sequences, and need to rank sequence or genomic features by predictive importance.

## When NOT to use

- Input is already a pre-computed feature table with ranked importances, so k-mer computation and classifier fitting would be redundant
- The goal is functional annotation of HOT regions (e.g. house-keeping gene promoter overlap or expression analysis) rather than sequence-feature discrimination
- Regions are too few or too unbalanced between HOT and COLD classes to support 10-fold cross-validated elastic net fitting

## Inputs

- HOT and COLD region coordinate sets (BED-style intervals) per species
- Reference genome sequences for the analyzed species
- Class labels (HOT/COLD) for each region

## Outputs

- Feature-importance table of top-ranked model coefficients per species (k-mers, GC content, GC skew, CpG O/E ratio)
- Cross-validated AUC for each species-specific elastic net classifier

## How to apply

Extract the genomic sequences underlying each HOT and COLD interval, then compute k-mer frequency vectors for k = 2, 3, and 4 plus GC content, GC skew, and CpG observed/expected (O/E) ratio per region, and assemble these into a feature matrix with HOT/COLD class labels. Fit one elastic net classifier per species with glmnet, tuning the regularization parameter by 10-fold cross-validation (the paper trains four species-specific classifiers this way). Extract the top-ranked model coefficients into a feature-importance table and record each classifier's cross-validated AUC. The rationale is that CpG/GC-rich k-mers and CpG O/E ratio are expected to rank highest because GC/CpG-rich sequence is the discriminatory signal for HOT regions, which the study links to ChIP-seq artifacts such as R-loops and G-quadruplex DNA.

## Related tools

- **glmnet** (Fits the elastic net classifiers with 10-fold cross-validation to tune the regularization parameter and yields coefficients used as the feature-importance ranking)

## Examples

```
cvfit <- cv.glmnet(X, y, family='binomial', alpha=0.5, type.measure='auc', nfolds=10); coef(cvfit, s='lambda.min')
```

## Evaluation signals

- Each species-specific classifier's cross-validated AUC falls within the reported 0.82–0.94 range
- CpG/GC-rich k-mers and the CpG observed/expected ratio rank among the top features in the coefficient table for all species
- The feature matrix contains k-mer frequencies for k = 2, 3, and 4 plus GC content, GC skew, and CpG O/E ratio columns matching each region label
- The regularization parameter lambda is chosen by 10-fold cross-validation for every fitted model

## Limitations

- HOT regions lack canonical motifs for transcription factors thought to be bound there, so motif features alone will not discriminate them
- The discriminatory GC/CpG-rich features may reflect ChIP-seq artifacts (hyper-ChIPable regions, R-loops, G-quadruplex DNA) rather than true biological co-binding
- No cross-species comparison of HOT region sequence features existed prior to this analysis, so learned feature importances may not transfer across species despite shared CpG/GC signals

## Evidence

- [results] Species-specific machine-learning models discriminate HOT from COLD regions with high accuracy (cross-validation AUC 0.82–0.94): [evidence span withheld — anchor status: not_found]
- [abstract] HOT regions have discriminatory sequence characteristics and genomic features including GC/CpG-rich k-mers, R-loops, and G-quadruplex DNA: "We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures"
- [intro] To date, there has been no cross-species comparison of HOT regions in terms of sequence features.: "To date, there has been no cross-species comparison of HOT regions in terms of sequence features."
- [readme] The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R).: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
- [abstract] HOT regions are solely defined using ChIP-seq experiments and shown to lack canonical motifs: "HOT regions are solely defined using ChIP-seq experiments and shown to lack canonical motifs for transcription factors that are thought to be bound there."
