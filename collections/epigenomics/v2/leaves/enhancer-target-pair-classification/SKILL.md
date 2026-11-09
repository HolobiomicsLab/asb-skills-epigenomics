---
name: enhancer-target-pair-classification
description: Use when you need to predict which candidate cCRE-ELS enhancers within
  500 KB of a target gene's TSS actively interact with that gene in a specific cell
  type, given ChIP-seq (H3K27ac, H3K4me1, H3K4me3) and RNA-seq data for that cell
  type.
when_to_use_negative:
- Input is already a scored feature table with only generic features (distance, cor_CRUP,
  combined_tests) — the article shows generic features alone perform worse than the
  baseline distance method
- The enhancer-target candidates extend beyond 500 KB — CENTRE limits the ET pair
  search to a distance of 500 KB and extending this interval requires future methodological
  improvements
- No cell-type-specific RNA-seq or H3K27ac/H3K4me1/H3K4me3 ChIP-seq data are available
  for the CT of interest — the method requires exactly these inputs
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: CENTRE
  role: R/Bioconductor framework providing createPairs, computeGenericFeatures, computeCellTypeFeatures,
    and centrePrediction for the ET pair classification workflow
  repo: https://github.com/slrvv/CENTRE
- name: XGBoost
  role: Pre-trained extreme gradient boosting classifier (logistic objective, scale_pos_weight
    = 5) that assigns an interaction probability to each ET feature vector
- name: crupR
  role: Runs normalization and CRUP-EP/CRUP-PP enhancer/promoter probability prediction
    on user ChIP-seq BAMs to derive CT-specific features and regulatory distances
- name: R
  role: Language in which the pre-trained CENTRE framework is delivered as ready-to-use
    R software (xgboost, GenomicRanges, metapod, RSQLite dependencies)
  repo: https://github.com/slrvv/CENTRE
provenance:
  source_task_ids:
  - task_006
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:774e9e62aedf1cdf5475e4bb10212d6642d97e4ae87a25c5c293a33399d5cb2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slrvv/CENTRE
  tools:
  - CENTRE
  - ExperimentHub
  - R
  - XGBoost
  - centrePrediction
  - computeCellTypeFeatures
  - computeGenericFeatures
  - createPairs
  - crupR
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 20
---

# enhancer-target-pair-classification

## Summary

Train and apply a pre-trained XGBoost gradient-boosting classifier (CENTRE) to label enhancer-target (ET) pairs as interacting or not interacting in a cell-type-specific manner, using CT-specific features (CRUP-EP/CRUP-PP probabilities, regulatory distances, RNA-seq TPM) plus generic features (genomic distance, cor_CRUP, Fisher's combined Wilcoxon P-value).

## When to use

Use when you need to predict which candidate cCRE-ELS enhancers within 500 KB of a target gene's TSS actively interact with that gene in a specific cell type, given ChIP-seq (H3K27ac, H3K4me1, H3K4me3) and RNA-seq data for that cell type.

## When NOT to use

- Input is already a scored feature table with only generic features (distance, cor_CRUP, combined_tests) — the article shows generic features alone perform worse than the baseline distance method
- The enhancer-target candidates extend beyond 500 KB — CENTRE limits the ET pair search to a distance of 500 KB and extending this interval requires future methodological improvements
- No cell-type-specific RNA-seq or H3K27ac/H3K4me1/H3K4me3 ChIP-seq data are available for the CT of interest — the method requires exactly these inputs

## Inputs

- Cell-type-specific H3K27ac, H3K4me1, H3K4me3 ChIP-seq BAM files with matching input control (metaData data.frame)
- Cell-type-specific RNA-seq TPM data.frame with columns gene_id, transcript_ids, TPM
- Data.frame of GENCODE ENSEMBL gene IDs or enhancer(cCRE-ELS)-target gene pairs of interest
- Precomputed PrecomputedData.db database of Wilcoxon rank-sum tests (Andersson 2014 CAGE-seq, Thurman 2012 DNase hypersensitivity)
- All-Pairs.Natural-Ratio BENGI feature tables (for training/evaluation mode)

## Outputs

- predictions data.frame with one row per enhancer_id-gene_id pair, each with a label (active/inactive) and an interaction probability (centre_predictions.csv)
- Per-dataset AUPRC and F1-score when run in cross-validated evaluation mode

## How to apply

Form all enhancer-gene pairs by extracting every cCRE-ELS within 500 KB of each target gene's GENCODE TSS, keeping only pairs located within 500 KB. Compute generic, cell-type-agnostic features (genomic distance, precomputed cor_CRUP Pearson correlation, negative log of the Fisher's combined Wilcoxon rank-sum P-value) via computeGenericFeatures, and CT-specific features via computeCellTypeFeatures, which runs crupR normalization on the user's H3K27ac/H3K4me1/H3K4me3 BAM files to derive CRUP-EP and CRUP-PP probabilities per 100-bp bin, the four regulatory-distance features (reg_dist_enh, norm_reg_dist_enh, reg_dist_prom, norm_reg_dist_prom), and the TPM feature. Feed the combined feature vector to centrePrediction using the default pre-trained XGBoost classifier (logistic objective, scale_pos_weight = 5, colsample_bytree = 0.7, gamma = 1.0, learning_rate = 0.1, max_depth = 5, n_estimators = 300, reg_lambda = 0, subsample = 0.9); pairs with probability > 0.5 are labeled interacting. When retraining or evaluating rather than predicting, use the All-Pairs.Natural-Ratio BENGI datasets (hg38 cCREs-ELS and GENCODE v40 TSS, positives with TPM > 0, negatives matched to positives by TSS or cCRE-ELS) and evaluate with the 12-fold chromosome-split outer cross-validation so testing occurs in different genomic regions than training.

## Related tools

- **CENTRE** (R/Bioconductor framework providing createPairs, computeGenericFeatures, computeCellTypeFeatures, and centrePrediction for the ET pair classification workflow) — https://github.com/slrvv/CENTRE
- **XGBoost** (Pre-trained extreme gradient boosting classifier (logistic objective, scale_pos_weight = 5) that assigns an interaction probability to each ET feature vector)
- **crupR** (Runs normalization and CRUP-EP/CRUP-PP enhancer/promoter probability prediction on user ChIP-seq BAMs to derive CT-specific features and regulatory distances)
- **R** (Language in which the pre-trained CENTRE framework is delivered as ready-to-use R software (xgboost, GenomicRanges, metapod, RSQLite dependencies)) — https://github.com/slrvv/CENTRE

## Examples

```
pairs <- createPairs(gene_ids); feats <- centrePrediction(computeGenericFeatures(pairs), computeCellTypeFeatures(metaData, replicate, input.free, cores, sequencing, tpmData, pairs))
```

## Evaluation signals

- Returned predictions data.frame contains exactly one row per enhancer_id-gene_id pair, each with a label and a probability value
- On GM12878 RNAPII-ChIAPET, the combined feature set achieves the highest AUPRC versus CT-specific-only and generic-only conditions, reproducing the paper's ordering
- In RD ablation on GM12878 RNAPII-ChIAPET, the model including the four regulatory-distance features achieves higher outer-CV average precision than the RD-ablated model
- Pairs with predicted probability > 0.5 are labeled interacting, matching the paper's threshold convention
- For 334 LCL-interacting/HeLa-non-interacting ET pairs, the fraction of CRUP-predicted enhancers/promoters in the intervening window is smaller in LCLs than in HeLa

## Limitations

- ET pair search is restricted to 500 KB; extending this interval requires future methodological improvements
- F1 scores remain low on many BENGI datasets, and the choice of validation data strongly influences performance measurement
- Generic features (Fisher's combined probability, Pearson correlation) perform poorly on GM12878, inferior even to the baseline distance method
- Inclusion of more histone marks could improve cCRE and RD annotation but at the cost of extra experiments and harder practical application

## Evidence

- [results] CENTRE extracts all cCRE-ELS within 500 KB of target genes and computes CT-specific and generic features for all potential ET pairs: "CENTRE extracts all the cCRE-ELS within 500 KB of target genes and computes CT-specific and generic features for all potential ET pairs"
- [results] Pre-trained XGBoost assigns interaction probability; threshold 0.5: "ET feature vectors are then fed to a pre-trained XGBOOST classifier, and a probability of an interaction is assigned to ET pairs"
- [results] 12-fold chromosome-split cross-validation scheme: "the authors also provide 12 cross-validation (CV) groups split by chromosome. This ensures that testing is always performed in different genomic regions than training"
- [readme] User-provided data requirements from README: [evidence span withheld — anchor status: external_doc]
- [methods] Positive pairs require TPM > 0 filter: "The positive interactions were further processed so that the interacting gene had a TPM value > 0"
- [methods] All-Pairs.Natural-Ratio BENGI used for training/evaluation: "We used the All-Pairs.Natural-Ratio BENGI collection of datasets for the training and evaluation of CENTRE"
