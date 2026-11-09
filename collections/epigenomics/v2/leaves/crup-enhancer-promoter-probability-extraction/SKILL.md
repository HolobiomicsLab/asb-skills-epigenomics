---
name: crup-enhancer-promoter-probability-extraction
description: Use when you need cell-type-specific enhancer/promoter activity probabilities
  and regulatory distances for candidate enhancer-target pairs from H3K27ac/H3K4me1/H3K4me3
  ChIP-seq BAM files plus input controls.
when_to_use_negative:
- When only cell-type-agnostic features are needed — use computeGenericFeatures()
  instead of re-running crupR
- When the cell type lacks H3K27ac, H3K4me1 and H3K4me3 ChIP-seq data, since CENTRE's
  CT-specific features cannot be computed
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3173
tools:
- name: crupR
  role: Normalization of ChIP-seq BAM files and production of CRUP-EP/CRUP-PP probabilities
    per 100-bp bin, invoked inside computeCellTypeFeatures()
- name: CENTRE
  role: 'Bioconductor package wrapping the pipeline: computeCellTypeFeatures() and
    centrePrediction() consume the CRUP-derived features'
  repo: https://github.com/slrvv/CENTRE
- name: XGBoost
  role: Pre-trained gradient boosting classifier that consumes the CRUP-derived feature
    vector to assign interaction probabilities
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:b204149638fe7bbcacd7c4027612f15d45d4769e8e16c7a25f7461a8a6d7abcc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slrvv/CENTRE
  tools:
  - CENTRE
  - CRUP-EP
  - CRUP-PP
  - ExperimentHub
  - R
  - XGBoost
  - centrePrediction
  - computeCellTypeFeatures
  - computeGenericFeatures
  - createPairs
  - crupR
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# crup-enhancer-promoter-probability-extraction

## Summary

Extracts cell-type-specific enhancer (CRUP-EP) and promoter (CRUP-PP) probabilities per 100-bp genomic bin from H3K27ac, H3K4me1 and H3K4me3 ChIP-seq, and derives regulatory-distance features and correlations used by CENTRE's enhancer-target classifier.

## When to use

Use when you need cell-type-specific enhancer/promoter activity probabilities and regulatory distances for candidate enhancer-target pairs from H3K27ac/H3K4me1/H3K4me3 ChIP-seq BAM files plus input controls.

## When NOT to use

- When only cell-type-agnostic features are needed — use computeGenericFeatures() instead of re-running crupR
- When the cell type lacks H3K27ac, H3K4me1 and H3K4me3 ChIP-seq data, since CENTRE's CT-specific features cannot be computed

## Inputs

- Cell-type-specific histone ChIP-seq BAM files for H3K27ac, H3K4me1, H3K4me3 (with matching input control experiment)
- Cell-type-specific RNA-seq TPM data.frame with columns gene_id, transcript_ids, TPM
- metaData data.frame pointing to the BAM files (histone mark, replicate, input.free, cores, sequencing)
- Candidate enhancer-target pairs from createPairs() (cCRE-ELS within 500 KB of GENCODE TSS)

## Outputs

- CRUP-EP enhancer probability per 100-bp genomic bin
- CRUP-PP promoter probability per 100-bp genomic bin
- CRUP-EP- and CRUP-PP-derived regulatory distance features per enhancer-target pair
- cor_CRUP Pearson correlation and TPM feature per pair, ready for centrePrediction()

## How to apply

Run crupR normalization on the user-provided BAM files for H3K27ac, H3K4me1 and H3K4me3 (with matching input control) via computeCellTypeFeatures(), which calls CRUP-EP to output enhancer probability for every 100-bp genomic bin and CRUP-PP for promoter probability. Apply CRUP-EP and CRUP-PP to the window between each enhancer-target pair to extract the fraction of regions classified as enhancers or promoters, and derive regulatory distance features from these probabilities. Also compute the Pearson correlation (cor_CRUP) between enhancer activity and target expression across cell types, and pull the TPM of the target gene from the RNA-seq data (three columns: gene_id, transcript_ids, TPM). These CRUP-derived and TPM features are concatenated with the generic features (distance, cor_CRUP, combined_tests Fisher-combined Wilcoxon P-value) before classification with the pre-trained XGBoost model via centrePrediction().

## Related tools

- **crupR** (Normalization of ChIP-seq BAM files and production of CRUP-EP/CRUP-PP probabilities per 100-bp bin, invoked inside computeCellTypeFeatures())
- **CENTRE** (Bioconductor package wrapping the pipeline: computeCellTypeFeatures() and centrePrediction() consume the CRUP-derived features) — https://github.com/slrvv/CENTRE
- **XGBoost** (Pre-trained gradient boosting classifier that consumes the CRUP-derived feature vector to assign interaction probabilities)

## Examples

```
features_celltype <- computeCellTypeFeatures(metaData, replicate, input.free=FALSE, cores=4, sequencing="paired", tpmData, pairs)
```

## Evaluation signals

- CRUP-EP output must contain an enhancer probability for every 100-bp genomic bin in the analyzed window
- computeCellTypeFeatures() must return one feature row per enhancer_id-gene_id pair including CRUP-EP/CRUP-PP regulatory distances and the TPM value
- Pairs with probability higher than 0.5 from the final classifier are labeled as interacting — probabilities should lie in [0,1]
- The combined feature set including CRUP-derived features should achieve higher AUPRC than generic features alone (which are inferior even to the baseline distance method)
- metaData BAM paths must have matching input control experiments; running without them is possible but strongly advised against by the README

## Limitations

- Restricted to interactions within 500 KB of the target gene TSS; extending this interval requires future methodological improvements
- Input control ChIP-seq is strongly advised but CENTRE can run without it, potentially at reduced normalization quality
- Only three histone modifications are used; inclusion of more HMs could improve annotation but costs extra experiments
- F1 scores remain low on many BENGI datasets; performance depends strongly on the choice of validation data

## Evidence

- [methods] we applied the CRUP-EP algorithm. CRUP-EP outputs the enhancer probability for every 100-bp genomic bin: "we applied the CRUP-EP algorithm. CRUP-EP outputs the enhancer probability for every 100-bp genomic bin"
- [results] We apply CRUP-EP and CRUP-PP to the window between the two regulatory elements and extract the fraction of regions classified as enhancers or promoters: "We apply CRUP-EP and CRUP-PP to the window between the two regulatory elements and extract the fraction of regions classified as enhancers or promoters"
- [readme] Cell-type specific histone ChIP-seq in BAM format for H3K27ac, H3K4me3 and H3K4me1. Additionally, a Control ChIP-seq experiment to match the HM ChIP-seq is strongly advised: [evidence span withheld — anchor status: external_doc]
- [results] our algorithm only requires RNA-seq data as well as ChIP-seq data of the histone modifications H3K27ac, H3K4me1, and H3K4me3 determined for this particular CT: "our algorithm only requires RNA-seq data as well as ChIP-seq data of the histone modifications H3K27ac, H3K4me1, and H3K4me3 determined for this particular CT"
- [results] ET pairs with higher probability than 0.5 are labeled as interacting pairs: "ET pairs with higher probability than 0.5 are labeled as interacting pairs"
