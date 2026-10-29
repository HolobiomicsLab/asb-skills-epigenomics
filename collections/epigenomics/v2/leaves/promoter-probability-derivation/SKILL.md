---
name: promoter-probability-derivation
description: Use when computing CENTRE cell-type-specific features for enhancer-target
  pairs and promoter probabilities (PP_prob_enh, PP_prob_gene) are needed from normalized
  histone-mark ChIP-seq tracks of the cell type of interest.
when_to_use_negative:
- You only need cell-type agnostic features (Wilcoxon tests, Fisher's combined probability)
  — use computeGenericFeatures instead
- No histone-mark ChIP-seq (H3K27ac, H3K4me1, H3K4me3) for the cell type of interest
  is available, since CRUP-EP/CRUP-PP require these three marks
- Input is already a finished CENTRE feature table from centrePrediction-ready output
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: CRUP-EP
  role: applied to normalized histone ChIP-seq; its second classifier's complementary
    output yields the CRUP-PP promoter probability per 100-bp bin
- name: CRUP-PP
  role: promoter probability score overlapped with enhancer and TSS regions via findOverlaps
- name: crupR
  role: R package used to normalize the H3K4me1, H3K4me3, H3K27ac ChIP-seq experiments
    before CRUP-EP/CRUP-PP (cores = 2 in the HeLa-S3 example)
  repo: https://github.com/slrvv/CENTRE
- name: GenomicRanges
  role: provides findOverlaps to map per-bin promoter probabilities onto the extended
    enhancer/TSS regions
- name: CENTRE
  role: Bioconductor/R package whose computeCellTypeFeatures() wraps the promoter-probability
    derivation and writes celltype_features.csv
  repo: https://github.com/slrvv/CENTRE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:6ae1e417974798ea60d88aad099d000e343b1f34024855d8d9b0feb1b0250a19
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# promoter-probability-derivation

## Summary

Derives cell-type-specific promoter probabilities (CRUP-PP) from H3K27ac, H3K4me1, and H3K4me3 ChIP-seq data by taking the complementary output of CRUP-EP's second classifier, then overlapping per-100-bp-bin probabilities with enhancer and TSS regions to score ET pairs as CENTRE cell-type features.

## When to use

Use when computing CENTRE cell-type-specific features for enhancer-target pairs and promoter probabilities (PP_prob_enh, PP_prob_gene) are needed from normalized histone-mark ChIP-seq tracks of the cell type of interest.

## When NOT to use

- You only need cell-type agnostic features (Wilcoxon tests, Fisher's combined probability) — use computeGenericFeatures instead
- No histone-mark ChIP-seq (H3K27ac, H3K4me1, H3K4me3) for the cell type of interest is available, since CRUP-EP/CRUP-PP require these three marks
- Input is already a finished CENTRE feature table from centrePrediction-ready output

## Inputs

- BAM files of CT-specific H3K4me1, H3K4me3, H3K27ac ChIP-seq (plus matched input control, e.g. ENCSR000AOB)
- pairs data.frame of enhancer (cCRE-ELS, e.g. EH38E1958626) / target gene (GENCODE/ENSEMBL ID, e.g. ENSG00000105281) pairs
- cCRE-LS enhancer regions and GENCODE TSS regions (extended to 500 bp) as GRanges
- metaData data.frame with ENCODE accession IDs, replicate, input.free, sequencing mode, and target chromosome (e.g. chr19)

## Outputs

- PP_prob_enh and PP_prob_gene promoter probability scores per ET pair
- reg_dist_prom and norm_reg_dist_prom: counts and normalized fractions of 100-bp bins with CRUP-PP > 0.5 between each ET pair
- celltype_features.csv data.frame with EP/PP probabilities, regulatory-distance features, and TPM columns

## How to apply

First normalize the three histone-mark ChIP-seq experiments (H3K4me1, H3K4me3, H3K27ac plus matched input control, e.g. ENCSR000APW/ENCSR340WQU/ENCSR000AOC/ENCSR000AOB for HeLa-S3) with crupR. CRUP-EP is applied over the region between each ET pair, outputting enhancer probability for every 100-bp genomic bin; because CRUP-EP internally contains a promoter classifier, the CRUP-PP promoter probability is extracted as the complementary of CRUP-EP's second classifier. Extend cCRE-ELS enhancer regions and GENCODE TSS target regions to 500 bp, then use GenomicRanges findOverlaps to assign PP probabilities to the regions, yielding PP_prob_enh and PP_prob_gene scores per pair. Regions with CRUP-PP > 0.5 between the pair are additionally counted as promoter-bin regulatory-distance features (reg_dist_prom, norm_reg_dist_prom). Validation for the HeLa-S3 chr19 example: the returned data.frame written to celltype_features.csv must contain PP_prob_enh, PP_prob_gene, reg_dist_prom, norm_reg_dist_prom columns alongside the EP and TPM columns for the two example pairs (gene ENSG00000105281 with enhancers EH38E1958626 / EH38E3310851).

## Related tools

- **CRUP-EP** (applied to normalized histone ChIP-seq; its second classifier's complementary output yields the CRUP-PP promoter probability per 100-bp bin)
- **CRUP-PP** (promoter probability score overlapped with enhancer and TSS regions via findOverlaps)
- **crupR** (R package used to normalize the H3K4me1, H3K4me3, H3K27ac ChIP-seq experiments before CRUP-EP/CRUP-PP (cores = 2 in the HeLa-S3 example)) — https://github.com/slrvv/CENTRE
- **GenomicRanges** (provides findOverlaps to map per-bin promoter probabilities onto the extended enhancer/TSS regions)
- **CENTRE** (Bioconductor/R package whose computeCellTypeFeatures() wraps the promoter-probability derivation and writes celltype_features.csv) — https://github.com/slrvv/CENTRE

## Examples

```
features_celltype <- computeCellTypeFeatures(pairs, metaData, RNAseqTPM, cores = 2)
```

## Evaluation signals

- Returned data.frame contains the expected columns EP_prob_enh, EP_prob_gene, PP_prob_enh, PP_prob_gene, reg_dist_enh, norm_reg_dist_enh, reg_dist_prom, norm_reg_dist_prom, TPM
- Output for the HeLa-S3 example is written to celltype_features.csv with rows for the two chr19 pairs (ENSG00000105281 with EH38E1958626 / EH38E3310851)
- Promoter probabilities fall in [0, 1] per 100-bp bin, with bins scored active at CRUP-PP > 0.5 contributing to reg_dist_prom
- PP probabilities derive consistently from CRUP-EP (complementary of the second classifier), so PP and EP probabilities on the same bins are coherent

## Limitations

- Only three histone marks are used; the article notes inclusion of more HMs could improve cCRE/RD annotation 'but this comes to the cost of extra experiments and makes practical application of our method harder'
- A matched input control ChIP-seq experiment is 'strongly advised but CENTRE can also run without it', implying reduced reliability without control
- CRUP-EP outputs probabilities only for 100-bp genomic bins, so promoter calls are bin-resolution, not base-pair resolution

## Evidence

- [results] Based on the way CRUP works, we could also extract probabilities that a region constitutes an active promoter, namely CRUP-PP promoter probability: "Based on the way CRUP works, we could also extract probabilities that a region constitutes an active promoter, namely CRUP-PP promoter probability"
- [results] CRUP-EP applied to window between ET pairs: "We apply CRUP-EP and CRUP-PP to the window between the two regulatory elements and extract the fraction of regions classified as enhancers or promoters"
- [methods] CRUP-EP outputs enhancer probability for every 100-bp bin: "we applied the CRUP-EP algorithm. CRUP-EP outputs the enhancer probability for every 100-bp genomic bin"
- [methods] findOverlaps from GenomicRanges: "[findOverlaps function from GenomicRanges library (Lawrence et al. 2013)]"
- [readme] README: BAM input requirement for the three histone marks: [evidence span withheld — anchor status: external_doc]
