---
name: enhancer-probability-prediction
description: Use when you have cell-type-specific BAM files for H3K27ac, H3K4me1 and
  H3K4me3 (plus matched input control) and RNA-seq TPM values, and need CT-specific
  features (EP/PP probabilities, regulatory distances, TPM) for ET pairs.
when_to_use_negative:
- The input is already a feature table (generic or CT-specific features computed)
  — proceed instead to centrePrediction()
- No cell-type-specific ChIP-seq for H3K27ac/H3K4me1/H3K4me3 is available for the
  CT of interest — CT-specific features cannot be computed
- You only need cell-type-agnostic statistics (Fisher's combined probability, Pearson
  correlation, genomic distance) — use computeGenericFeatures(pairs) instead
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0749
tools:
- name: CENTRE
  role: Bioconductor/R package whose computeCellTypeFeatures() function orchestrates
    the CT-specific feature computation
  repo: https://github.com/slrvv/CENTRE
- name: CRUP-EP
  role: Outputs enhancer probability for every 100-bp genomic bin; overlapped with
    extended cCRE-ELS and TSS regions to produce EP_prob_enh/EP_prob_gene and used
    for regulatory-distance features
- name: CRUP-PP
  role: Extracted from the complementary of CRUP-EP's second classifier; gives promoter
    probabilities overlapped with the same regions to yield PP_prob_enh/PP_prob_gene
- name: crupR
  role: Normalizes the three histone-mark ChIP-seq experiments to produce genome-browser
    tracks prior to probability computation
- name: GenomicRanges
  role: findOverlaps is used to overlap 100-bp-bin probabilities with the extended
    enhancer/TSS regions
- name: ExperimentHub
  role: Source of the example RNA-seq TPM table (EH9545, mean TPM over replicates)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:f05f1e80aaf09a99546ca009afab7d318cfcac98cbfd97bde44fd55626904e05
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slrvv/CENTRE
  tools:
  - CENTRE
  - CRUP-EP
  - CRUP-PP
  - ExperimentHub
  - GenomicRanges
  - R
  - crupR
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# enhancer-probability-prediction

## Summary

Computes cell-type-specific enhancer and promoter probability features for enhancer-target (ET) pairs by applying CRUP-EP and CRUP-PP to H3K4me1, H3K4me3 and H3K27ac ChIP-seq (via crupR), and attaching cell-type RNA-seq TPM values, as implemented in CENTRE's computeCellTypeFeatures().

## When to use

Use when you have cell-type-specific BAM files for H3K27ac, H3K4me1 and H3K4me3 (plus matched input control) and RNA-seq TPM values, and need CT-specific features (EP/PP probabilities, regulatory distances, TPM) for ET pairs.

## When NOT to use

- The input is already a feature table (generic or CT-specific features computed) — proceed instead to centrePrediction()
- No cell-type-specific ChIP-seq for H3K27ac/H3K4me1/H3K4me3 is available for the CT of interest — CT-specific features cannot be computed
- You only need cell-type-agnostic statistics (Fisher's combined probability, Pearson correlation, genomic distance) — use computeGenericFeatures(pairs) instead

## Inputs

- BAM files for H3K4me1, H3K4me3 and H3K27ac ChIP-seq for the cell type of interest (e.g. HeLa-S3 ENCSR000APW, ENCSR340WQU, ENCSR000AOC) with matched input control (ENCSR000AOB)
- metaData data.frame specifying BAM paths, replicate, input.free, sequencing (e.g. 'single') and chromosome (e.g. 'chr19')
- Cell-type-specific RNA-seq TPM table (e.g. ExperimentHub EH9545, mean TPM over replicates) with ENSEMBL ID, transcript ID and TPM columns
- pairs data.frame of GENCODE gene IDs and cCRE-ELS enhancer IDs (e.g. ENSG00000105281 with EH38E1958626 / EH38E3310851)

## Outputs

- data.frame (written to celltype_features.csv) with columns EP_prob_enh, EP_prob_gene, PP_prob_enh, PP_prob_gene, reg_dist_enh, norm_reg_dist_enh, reg_dist_prom, norm_reg_dist_prom and TPM for each ET pair
- Normalized genome-browser tracks over the target chromosome from crupR

## How to apply

Normalize the three histone-mark ChIP-seq experiments with crupR (e.g. cores = 2), using the metaData data.frame with replicate, input.free and sequencing settings. Extend cCRE-ELS enhancer regions and GENCODE TSS target regions to 500 bp, run CRUP-EP to obtain enhancer probabilities per 100-bp bin, and use GenomicRanges findOverlaps to attach five EP_prob_enh and five EP_prob_gene scores per pair; derive CRUP-PP promoter probabilities from the complementary of CRUP-EP's second classifier to get PP_prob_enh and PP_prob_gene. Compute reg_dist_enh, norm_reg_dist_enh, reg_dist_prom and norm_reg_dist_prom as counts (and normalized fractions) of 100-bp bins with CRUP-EP > 0.5 or CRUP-PP > 0.5 between each ET pair, and append the RNA-seq TPM value for the pair's gene. The rationale is that CT-specific epigenetic activity of the enhancer, the promoter and the window between them is the most informative feature block for the downstream XGBoost classifier, complementing generic (cell-type-agnostic) statistics.

## Related tools

- **CENTRE** (Bioconductor/R package whose computeCellTypeFeatures() function orchestrates the CT-specific feature computation) — https://github.com/slrvv/CENTRE
- **CRUP-EP** (Outputs enhancer probability for every 100-bp genomic bin; overlapped with extended cCRE-ELS and TSS regions to produce EP_prob_enh/EP_prob_gene and used for regulatory-distance features)
- **CRUP-PP** (Extracted from the complementary of CRUP-EP's second classifier; gives promoter probabilities overlapped with the same regions to yield PP_prob_enh/PP_prob_gene)
- **crupR** (Normalizes the three histone-mark ChIP-seq experiments to produce genome-browser tracks prior to probability computation)
- **GenomicRanges** (findOverlaps is used to overlap 100-bp-bin probabilities with the extended enhancer/TSS regions)
- **ExperimentHub** (Source of the example RNA-seq TPM table (EH9545, mean TPM over replicates))

## Examples

```
features_celltype <- CENTRE::computeCellTypeFeatures(pairs, metaData, RNAseqTPM)
```

## Evaluation signals

- Returned data.frame contains exactly the expected columns: EP_prob_enh, EP_prob_gene, PP_prob_enh, PP_prob_gene, reg_dist_enh, norm_reg_dist_enh, reg_dist_prom, norm_reg_dist_prom, TPM
- Result is written to celltype_features.csv for the two example chr19 pairs (ENSG00000105281 with EH38E1958626 / EH38E3310851)
- Each ET pair carries five EP_prob_enh and five EP_prob_gene scores (one per replicate/track) from CRUP-EP
- regulatory-distance features are counts of 100-bp bins with CRUP-EP > 0.5 or CRUP-PP > 0.5 between the pair, with norm_* being the corresponding fractions (0–1 range)
- TPM values match the RNA-seq table for the gene of each pair

## Limitations

- CENTRE requires ChIP-seq data for exactly three histone modifications (H3K27ac, H3K4me1, H3K4me3) for the CT of interest; inclusion of more HMs would improve cCRE/RD annotation but requires extra experiments
- A matched input control ChIP-seq experiment is strongly advised; CENTRE can run without it (input.free = TRUE) but at reduced quality
- CT-specific features complement but do not replace generic features; genomic distance still ranks first among feature importances, so CT features alone are insufficient

## Evidence

- [results] The CT-specific features are computed by applying the CRUP-EP and CRUP-PP functions to the histone mark ChIP-seq data, yielding probabilities that cCRE-ELS and TSS regions are active: [evidence span withheld — anchor status: not_found]
- [methods] CRUP-EP outputs the enhancer probability for every 100-bp genomic bin: "we applied the CRUP-EP algorithm. CRUP-EP outputs the enhancer probability for every 100-bp genomic bin"
- [results] We apply CRUP-EP and CRUP-PP to the window between the two regulatory elements and extract the fraction of regions classified as enhancers or promoters: "We apply CRUP-EP and CRUP-PP to the window between the two regulatory elements and extract the fraction of regions classified as enhancers or promoters"
- [results] our algorithm only requires RNA-seq data as well as ChIP-seq data of the histone modifications H3K27ac, H3K4me1, and H3K4me3 determined for this particular CT: "our algorithm only requires RNA-seq data as well as ChIP-seq data of the histone modifications H3K27ac, H3K4me1, and H3K4me3"
- [readme] Cell-type specific histone ChIP-seq in BAM format for H3K27ac, H3K4me3 and H3K4me1. Additionally, a Control ChIP-seq experiment to match the HM ChIP-seq is strongly advised: [evidence span withheld — anchor status: external_doc]
