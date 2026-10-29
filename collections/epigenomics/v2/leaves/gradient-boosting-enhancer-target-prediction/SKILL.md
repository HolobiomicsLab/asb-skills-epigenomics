---
name: gradient-boosting-enhancer-target-prediction
description: Use when you need to classify candidate enhancer-target (cCRE-ELS to
  gene) pairs as interacting or not in a specific cell type from user-supplied histone
  ChIP-seq BAM files and RNA-seq TPM values, without needing massive interaction datasets.
when_to_use_negative:
- Candidate pairs are located more than 500 kb apart — CENTRE limits the ET pair search
  to a distance of 500 KB
- You need cross-cell-line extrapolation from models trained on interaction-rich cell
  lines — this is a known failure mode of related methods like TargetFinder, and CENTRE
  requires CT-specific H3K27ac/H3K4me1/H3K4me3 and RNA-seq for the target cell type
- Input is already a labeled feature table and you only want to train/evaluate a classifier
  rather than apply the pre-trained user-facing pipeline
edam_operation: http://edamontology.org/operation_0439
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: CENTRE
  role: Bioconductor-style R package providing createPairs, computeGenericFeatures,
    computeCellTypeFeatures and centrePrediction for the end-to-end prediction workflow
  repo: https://github.com/slrvv/CENTRE
- name: XGBoost
  role: Pre-trained extreme gradient boosting classifier that computes the interaction
    probability for each annotated ET pair
- name: crupR
  role: Runs normalization in computeCellTypeFeatures and provides CRUP-EP/CRUP-PP
    enhancer and promoter probabilities and regulatory distance features
- name: ExperimentHub / CENTREprecomputed
  role: Source of the precomputed Wilcoxon rank-sum test data (CENTREprecomputed package
    / PrecomputedData.db)
  repo: https://bioconductor.org/packages/release/data/experiment/src/contrib/CENTREprecomputed_1.2.0.tar.gz
- name: GenomicRanges
  role: Genomic interval arithmetic (e.g. findOverlaps) for collecting cCRE-ELS within
    500 kb of target TSSs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:ae468c9c8cb28bd4a8b4f5a44097d880667d05d2bb9c190abb8e23ce709d371f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slrvv/CENTRE
  tools:
  - CENTRE
  - ExperimentHub
  - ExperimentHub / CENTREprecomputed
  - GenomicRanges
  - XGBoost
  - centrePrediction
  - computeCellTypeFeatures
  - computeGenericFeatures
  - createPairs
  - crupR
  edam_operation: http://edamontology.org/operation_0439
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# gradient-boosting-enhancer-target-prediction

## Summary

CENTRE is a pre-trained XGBoost-based framework that predicts cell-type-specific enhancer-target interactions using only RNA-seq and ChIP-seq data for H3K27ac, H3K4me1, and H3K4me3 of the cell type of interest, combining CT-specific and cell-type-agnostic features.

## When to use

Use when you need to classify candidate enhancer-target (cCRE-ELS to gene) pairs as interacting or not in a specific cell type from user-supplied histone ChIP-seq BAM files and RNA-seq TPM values, without needing massive interaction datasets.

## When NOT to use

- Candidate pairs are located more than 500 kb apart — CENTRE limits the ET pair search to a distance of 500 KB
- You need cross-cell-line extrapolation from models trained on interaction-rich cell lines — this is a known failure mode of related methods like TargetFinder, and CENTRE requires CT-specific H3K27ac/H3K4me1/H3K4me3 and RNA-seq for the target cell type
- Input is already a labeled feature table and you only want to train/evaluate a classifier rather than apply the pre-trained user-facing pipeline

## Inputs

- Vector/data.frame of GENCODE/ENSEMBL gene IDs of target genes (or enhancer-target pairs of interest)
- Cell-type specific H3K27ac, H3K4me1 and H3K4me3 ChIP-seq BAM files with matching input control
- RNA-seq TPM data.frame with three columns: gene_id (ENSEMBL), transcript_ids, TPM
- Precomputed PrecomputedData.db (Wilcoxon rank-sum tests on CAGE-seq, DNase hypersensitivity, DNAse-seq gene expression datasets)

## Outputs

- predictions data.frame with one row per enhancer_id-gene_id pair, each with an interacting/active label and a probability value (interacting if probability > 0.5)
- Feature tables from computeGenericFeatures (distance, cor_CRUP, combined_tests P-value) and computeCellTypeFeatures (CRUP-EP/CRUP-PP probabilities, regulatory distances, TPM)

## How to apply

First call createPairs(ids) on ENSEMBL gene IDs so that all cCRE-ELS enhancers within 500 kb of each gene's transcription start site are collected and every possible enhancer-gene pair is formed. Then call computeGenericFeatures(pairs) to attach distance between enhancer midpoint and TSS, the precomputed cor_CRUP correlation, and the Fisher-combined Wilcoxon P-value statistic (negative log of the combined P-value). Build a metaData data.frame pointing to cell-type-specific H3K4me1, H3K4me3 and H3K27ac ChIP-seq BAM files (with matching input control), load the RNA-seq TPM data.frame (gene_id, transcript_ids, TPM), and call computeCellTypeFeatures() to run crupR normalization and derive CRUP-EP/CRUP-PP probabilities, regulatory distances and the TPM feature. Finally call centrePrediction(features_generic, features_celltype) with the default pre-trained model: pairs with probability higher than 0.5 are labeled as interacting. Precomputed Wilcoxon rank-sum test data must be fetched via CENTRE::downloadPredcomputedData() or from the PrecomputedData.db URL and placed in /inst/extdata.

## Related tools

- **CENTRE** (Bioconductor-style R package providing createPairs, computeGenericFeatures, computeCellTypeFeatures and centrePrediction for the end-to-end prediction workflow) — https://github.com/slrvv/CENTRE
- **XGBoost** (Pre-trained extreme gradient boosting classifier that computes the interaction probability for each annotated ET pair)
- **crupR** (Runs normalization in computeCellTypeFeatures and provides CRUP-EP/CRUP-PP enhancer and promoter probabilities and regulatory distance features)
- **ExperimentHub / CENTREprecomputed** (Source of the precomputed Wilcoxon rank-sum test data (CENTREprecomputed package / PrecomputedData.db)) — https://bioconductor.org/packages/release/data/experiment/src/contrib/CENTREprecomputed_1.2.0.tar.gz
- **GenomicRanges** (Genomic interval arithmetic (e.g. findOverlaps) for collecting cCRE-ELS within 500 kb of target TSSs)

## Examples

```
pairs <- createPairs(ids); fg <- computeGenericFeatures(pairs); fc <- computeCellTypeFeatures(metaData, replicate, input.free, cores, sequencing, tpmData, pairs); preds <- centrePrediction(fg, fc)
```

## Evaluation signals

- Returned predictions data.frame contains exactly one row per enhancer_id-gene_id pair, each with a label and a probability value
- All predicted pairs lie within 500 kb of the target gene's TSS (any pair beyond this threshold indicates a createPairs misconfiguration)
- Pairs with probability higher than 0.5 are labeled as interacting — check label/probability consistency
- Generic features are non-missing: distance, cor_CRUP and combined_tests Fisher-combined P-value present for every pair
- On a benchmark such as GM12878 RNAPII-ChIAPET, combining all feature sources should achieve the highest AUPRC relative to single-feature baselines (distance-only baseline, generic-only features)

## Limitations

- ET pair search is limited to a distance of 500 KB; extending this interval requires future methodological improvements
- Generic features (Fisher's combined probability, Pearson correlation) perform poorly on distinguishing true and false ET interactions on the GM12878 cell line, being inferior to the baseline distance method
- F1 scores remain low on many datasets; performance depends strongly on the choice of validation data
- A Control ChIP-seq experiment to match the HM ChIP-seq is strongly advised but CENTRE can also run without it
- Only three histone marks (H3K27ac, H3K4me1, H3K4me3) are used; inclusion of more HMs could improve cCRE and regulatory domain annotation but would require extra experiments

## Evidence

- [results] CENTRE extracts all the cCRE-ELS within 500 KB of target genes and computes CT-specific and generic features for all potential ET pairs: "CENTRE extracts all the cCRE-ELS within 500 KB of target genes and computes CT-specific and generic features for all potential ET pairs"
- [results] Then a pre-trained extreme gradient boosting classifier computes a probability for an annotated ET pair to interact: "Then a pre-trained extreme gradient boosting classifier (Chen and Guestrin 2016) computes a probability for an annotated ET pair to interact"
- [results] ET pairs with higher probability than 0.5 are labeled as interacting pairs: "ET pairs with higher probability than 0.5 are labeled as interacting pairs"
- [methods] With the function centrePrediction(features_generic, features_celltype, model) the gene enhancer targets are classified as active or inactive: "With the function `centrePrediction(features_generic, features_celltype, model)` the gene enhancer targets are classified as active or inactive"
- [readme] Cell-type specific histone ChIP-seq in BAM format for H3K27ac, H3K4me3 and H3K4me1: [evidence span withheld — anchor status: external_doc]
- [results] We however limit the search for ET pairs to a distance of 500 KB which is a commonly accepted threshold: "We however limit the search for ET pairs to a distance of 500 KB which is a commonly accepted threshold"
