---
name: regulatory-distance-feature-computation
description: Use when building cell-type-specific features for enhancer-target (ET)
  pairs from H3K27ac/H3K4me1/H3K4me3 ChIP-seq, to quantify regulatory density in the
  enhancer-promoter interval rather than relying on genomic distance alone.
when_to_use_negative:
- The input is already a computed feature table with RD columns present — recomputation
  via CRUP is unnecessary and expensive.
- ET pairs extend beyond the 500 KB threshold, which CENTRE does not support; extending
  the interval requires methodological improvements.
- No cell-type-specific histone ChIP-seq for H3K27ac, H3K4me1, H3K4me3 is available
  — RD features cannot be computed cell-type-specifically.
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0602
tools:
- name: CENTRE
  role: R/Bioconductor framework whose computeCellTypeFeatures() function computes
    the RD features for ET pairs
  repo: https://github.com/slrvv/CENTRE
- name: CRUP-EP
  role: Outputs enhancer probability for every 100-bp genomic bin from the normalized
    histone ChIP-seq; bins with probability > 0.5 count toward reg_dist_enh
- name: CRUP-PP
  role: Derived from the complementary of CRUP-EP's second classifier; promoter probabilities
    > 0.5 count toward reg_dist_prom
- name: crupR
  role: Normalizes the three histone-mark ChIP-seq experiments (e.g. cores = 2) into
    genome-browser tracks before CRUP-EP/PP application
- name: GenomicRanges
  role: findOverlaps() overlaps CRUP probabilities with the 500-bp-extended enhancer
    and TSS regions
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:777d1f4d0dd03ea2286658fd366c91d508f9f31f411b3691898954fa85721855
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
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# regulatory-distance-feature-computation

## Summary

Computes CENTRE's regulatory-density (RD) cell-type-specific features by applying CRUP-EP and CRUP-PP to the genomic window between an enhancer and its target, quantifying the fraction of 100-bp bins classified as active enhancers or promoters. RD captures intervening regulatory activity, which carries cell-type-specific signal beyond genomic distance alone.

## When to use

Use when building cell-type-specific features for enhancer-target (ET) pairs from H3K27ac/H3K4me1/H3K4me3 ChIP-seq, to quantify regulatory density in the enhancer-promoter interval rather than relying on genomic distance alone.

## When NOT to use

- The input is already a computed feature table with RD columns present — recomputation via CRUP is unnecessary and expensive.
- ET pairs extend beyond the 500 KB threshold, which CENTRE does not support; extending the interval requires methodological improvements.
- No cell-type-specific histone ChIP-seq for H3K27ac, H3K4me1, H3K4me3 is available — RD features cannot be computed cell-type-specifically.

## Inputs

- BAM files of H3K4me1, H3K4me3, H3K27ac ChIP-seq for the cell type of interest (plus matched input control)
- Enhancer (cCRE-ELS) and target gene (GENCODE TSS) region coordinates in hg38
- Enhancer-target pairs dataframe with gene GENCODE IDs and cCRE-ELS IDs (pairs within 500 KB)

## Outputs

- Per-pair RD features: reg_dist_enh, norm_reg_dist_enh, reg_dist_prom, norm_reg_dist_prom (counts/fractions of 100-bp bins with CRUP-EP or CRUP-PP probability > 0.5 between enhancer and target)
- Feature table row appended to the cell-type feature dataframe (e.g. celltype_features.csv) consumed by centrePrediction()

## How to apply

First normalize the three histone-mark ChIP-seq experiments (BAM format, with matched input control advised) for the cell type of interest using crupR. Apply CRUP-EP to obtain enhancer probability for every 100-bp genomic bin, and derive CRUP-PP promoter probabilities from the complementary of CRUP-EP's second classifier. Restrict to the window between the two regulatory elements (ET pairs limited to 500 KB) and compute reg_dist_enh, reg_dist_prom as counts of 100-bp bins with CRUP-EP or CRUP-PP probability > 0.5, plus normalized variants (norm_reg_dist_enh, norm_reg_dist_prom) dividing by the total number of bins. In the CENTRE R package these features are produced by computeCellTypeFeatures(), which also overlaps CRUP probabilities with the 500-bp-extended cCRE-ELS enhancer and GENCODE TSS regions via GenomicRanges findOverlaps. Rationale: RD reflects the fraction of intervening regions classified as enhancers or promoters and is a top-ranked cell-type-specific feature in CENTRE's feature ranking.

## Related tools

- **CENTRE** (R/Bioconductor framework whose computeCellTypeFeatures() function computes the RD features for ET pairs) — https://github.com/slrvv/CENTRE
- **CRUP-EP** (Outputs enhancer probability for every 100-bp genomic bin from the normalized histone ChIP-seq; bins with probability > 0.5 count toward reg_dist_enh)
- **CRUP-PP** (Derived from the complementary of CRUP-EP's second classifier; promoter probabilities > 0.5 count toward reg_dist_prom)
- **crupR** (Normalizes the three histone-mark ChIP-seq experiments (e.g. cores = 2) into genome-browser tracks before CRUP-EP/PP application)
- **GenomicRanges** (findOverlaps() overlaps CRUP probabilities with the 500-bp-extended enhancer and TSS regions)

## Examples

```
features_celltype <- CENTRE::computeCellTypeFeatures(pairs, metaData, cores = 2)
```

## Evaluation signals

- The returned feature dataframe contains exactly the columns EP_prob_enh, EP_prob_gene, PP_prob_enh, PP_prob_gene, reg_dist_enh, norm_reg_dist_enh, reg_dist_prom, norm_reg_dist_prom, TPM.
- Normalized RD features fall in [0, 1] since they are fractions of total 100-bp bins between the pair.
- On the HeLa-S3 chr19 example, features are computed for the two example pairs (gene ENSG00000105281 with enhancers EH38E1958626 / EH38E3310851).
- Ablation check: an XGBoost model including the four RD features should achieve higher average precision (AUPRC) than an RD-ablated model on chromosome-split outer CV (e.g. GM12878 RNAPII-ChIAPET), matching the paper's feature ranking where normalized regulatory distance enhancers ranks among the top cell-type-specific features (position 3).
- RD stratification: ET pairs interacting in one cell type but not another should show smaller intervening regulatory activity in the interacting cell type (probability > 0.5 labeling).

## Limitations

- Interactions are restricted to within 500 KB; extending the interval requires future methodological improvements.
- RD features depend on ChIP-seq data quality; a matched input control is strongly advised though CENTRE can run without it.
- The fraction of intervening regulatory activity differs across cell types, so RD values are not comparable across cell types without cell-type-specific re-computation.

## Evidence

- [results] We apply CRUP-EP and CRUP-PP to the window between the two regulatory elements and extract the fraction of regions classified as enhancers or promoters: "We apply CRUP-EP and CRUP-PP to the window between the two regulatory elements and extract the fraction of regions classified as enhancers or promoters"
- [methods] CRUP-EP outputs the enhancer probability for every 100-bp genomic bin: "CRUP-EP outputs the enhancer probability for every 100-bp genomic bin"
- [results] Genomic distance (position 1), Wilcoxon tests combined (position 6), and CRUP correlation (position 9) are generic features, while positions 2 (RNA-seq), 3 (Normalized Regulatory distance enhancers),: "Genomic distance (position 1), Wilcoxon tests combined (position 6), and CRUP correlation (position 9) are generic features, while positions 2 (RNA-seq), 3 (Normalized Regulatory distance enhancers),"
- [readme] Additionally, a Control ChIP-seq experiment to match the HM ChIP-seq is strongly advised but CENTRE can also run without it.: [evidence span withheld — anchor status: external_doc]
