---
name: cell-type-agnostic-feature-computation
description: Use when candidate enhancer-gene pairs (e.g. cCRE-ELS within 500 KB of
  target gene TSSs) have been formed with createPairs and the CENTRE workflow requires
  the non-cell-type-specific feature columns before calling centrePrediction.
when_to_use_negative:
- Pairs have not yet been generated — run createPairs() first to collect cCRE-ELS
  within the 500 KB window around target TSSs
- You need the cell-type-specific features (CRUP-EP/CRUP-PP probabilities, regulatory
  distance, TPM) — use computeCellTypeFeatures(metaData, replicate, input.free, cores,
  sequencing, tpmData, pairs) instead
- You intend to train a TargetFinder-style model requiring hundreds of cell-specific
  genomic datasets — this skill only computes CT-agnostic statistics
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CENTRE
  role: Bioconductor R package providing computeGenericFeatures() to attach CT-agnostic
    features (distance, cor_CRUP, combined_tests) to ET pairs
  repo: https://github.com/slrvv/CENTRE
- name: CENTREprecomputed
  role: ExperimentHub/precomputed package supplying PrecomputedData.db with Wilcoxon
    rank-sum test statistics underlying the combined_tests Fisher feature
  repo: https://bioconductor.org/packages/release/data/experiment/src/contrib/CENTREprecomputed_1.2.0.tar.gz
- name: XGBoost
  role: Pre-trained extreme gradient boosting classifier that consumes the generic
    features together with CT-specific features to score ET pairs
  repo: https://github.com/slrvv/CENTRE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:0e386d2bf48f25defc0a3fb503e44093a937d4217d25355bde5829dc5792a715
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slrvv/CENTRE
  tools:
  - CENTRE
  - CENTREprecomputed
  - ExperimentHub
  - XGBoost
  - centrePrediction
  - computeCellTypeFeatures
  - computeGenericFeatures
  - createPairs
  - crupR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# cell-type-agnostic-feature-computation

## Summary

Computes cell-type-agnostic features for candidate enhancer-target (ET) pairs in the CENTRE pipeline via computeGenericFeatures(pairs): genomic distance between enhancer midpoint and TSS, the precomputed cor_CRUP correlation, and the Fisher-combined Wilcoxon P-value statistic, all derived from precomputed public datasets rather than the cell type of interest.

## When to use

Use when candidate enhancer-gene pairs (e.g. cCRE-ELS within 500 KB of target gene TSSs) have been formed with createPairs and the CENTRE workflow requires the non-cell-type-specific feature columns before calling centrePrediction.

## When NOT to use

- Pairs have not yet been generated — run createPairs() first to collect cCRE-ELS within the 500 KB window around target TSSs
- You need the cell-type-specific features (CRUP-EP/CRUP-PP probabilities, regulatory distance, TPM) — use computeCellTypeFeatures(metaData, replicate, input.free, cores, sequencing, tpmData, pairs) instead
- You intend to train a TargetFinder-style model requiring hundreds of cell-specific genomic datasets — this skill only computes CT-agnostic statistics

## Inputs

- pairs data.frame/GRanges object of enhancer-gene candidates produced by createPairs (all cCRE-ELS within 500 KB of each target gene TSS)
- PrecomputedData.db SQLite database of precomputed Wilcoxon rank-sum tests (CAGE-seq Andersson et al. 2014, DNase hypersensitivity Thurman et al. 2012, DNAse-seq gene expression)
- GENCODE/ENSEMBL gene identifiers for the target genes

## Outputs

- features_generic object: pairs annotated with distance (enhancer midpoint to TSS), cor_CRUP Pearson correlation, and combined_tests (negative log Fisher's combined P-value) columns
- Input to centrePrediction(features_celltype, features_generic) for final classification

## How to apply

After calling createPairs(ids) on a vector of ENSEMBL gene IDs to collect all cCRE-ELS within 500 KB of each gene's TSS, pass the resulting pairs object to computeGenericFeatures(pairs). This attaches three generic features: (1) the genomic distance between the enhancer midpoint and the TSS — the top-ranked feature in CENTRE's importance ranking; (2) cor_CRUP, a precomputed Pearson correlation between target gene expression and enhancer accessibility across 112 tissues (Sheffield et al. 2013); and (3) combined_tests, the negative logarithm of a Fisher's combined probability summarizing four Wilcoxon rank-sum test P-values comparing target transcription between cells where the enhancer is active versus inactive, drawn from PrecomputedData.db (CAGE-seq Andersson 2014, DNase Thurman 2012, and other public datasets). Download PrecomputedData.db via CENTRE::downloadPredcomputedData() or place it in /inst/extdata. Rationale: these CT-agnostic statistics complement the cell-type-specific features derived from H3K27ac/H3K4me1/H3K4me3 ChIP-seq and RNA-seq, and the pre-computation avoids re-running rank-sum tests across large public compendia. Note the article reports generic features alone perform worse than the distance baseline, so they are used in combination with, not instead of, CT-specific features.

## Related tools

- **CENTRE** (Bioconductor R package providing computeGenericFeatures() to attach CT-agnostic features (distance, cor_CRUP, combined_tests) to ET pairs) — https://github.com/slrvv/CENTRE
- **CENTREprecomputed** (ExperimentHub/precomputed package supplying PrecomputedData.db with Wilcoxon rank-sum test statistics underlying the combined_tests Fisher feature) — https://bioconductor.org/packages/release/data/experiment/src/contrib/CENTREprecomputed_1.2.0.tar.gz
- **XGBoost** (Pre-trained extreme gradient boosting classifier that consumes the generic features together with CT-specific features to score ET pairs) — https://github.com/slrvv/CENTRE

## Examples

```
pairs <- createPairs(gene_ids); features_generic <- computeGenericFeatures(pairs)
```

## Evaluation signals

- Every enhancer_id-gene_id row in the output carries all three generic feature columns (distance, cor_CRUP, combined_tests) with no missing values for pairs within 500 KB
- Distance values correspond to enhancer-midpoint-to-TSS distances and never exceed 500,000 bp given the createPairs window
- combined_tests values equal the negative logarithm of a Fisher's combined P-value computed from the precomputed Wilcoxon tests in PrecomputedData.db
- Downstream centrePrediction accepts features_generic alongside features_celltype without column-name mismatches and outputs one labeled probability per pair (interacting when probability > 0.5)

## Limitations

- Generic features are weaker discriminators: on GM12878 RNAPII-ChIAPET, both Fisher's combined probability and Pearson correlation performed worse than the baseline distance method
- ET pair search is restricted to 500 KB; extending this interval would require methodological improvements
- PrecomputedData.db must be downloaded separately (CENTRE::downloadPredcomputedData() or manual download) and placed in /inst/extdata or the generic features cannot be computed

## Evidence

- [methods] computeGenericFeatures computes non-CT-specific features: "The function `computeGenericFeatures(pairs)` computes the features that are not cell type specific"
- [results] 500 KB ET pair search limit: "We however limit the search for ET pairs to a distance of 500 KB which is a commonly accepted threshold"
- [results] Wilcoxon rank-sum test underlying generic feature: "by performing a Wilcoxon rank-sum test between target transcriptional signals between cells where the enhancer is active and those where it is not active"
- [results] Fisher's combined probability summarization: "we summarize these four P-values into a single Fisher's combined probability"
- [readme] Precomputed data download requirement: "CENTRE uses precomputed datasets that the user needs to download either by using
the `CENTRE::downloadPredcomputedData()`"
- [results] Generic features underperform baseline on GM12878: "both generic features perform poorly on distinguishing true and false ET interactions on the GM12878 cell line, being inferior to the baseline distance method"
