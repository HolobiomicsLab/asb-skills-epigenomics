---
name: genomic-interval-overlap-annotation
description: Use when per-region feature values (e.g. enhancer/promoter probabilities
  from bin-level tracks) must be transferred onto annotated genomic intervals (cCRE-ELS,
  TSSs) as input features for downstream ET-pair classification.
when_to_use_negative:
- You need cell-type-agnostic features (genomic distance, Wilcoxon rank-sum / Fisher's
  combined P-values) — use computeGenericFeatures instead
- The enhancer-target distance exceeds 500 KB, outside CENTRE's supported search interval
- You only have precomputed feature tables and need classification only — use centrePrediction
  on existing feature vectors
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
tools:
- name: GenomicRanges
  role: Provides the findOverlaps function used to intersect 100-bp-bin CRUP probabilities
    with extended enhancer and TSS regions
  repo: https://github.com/slrvv/CENTRE
- name: CRUP-EP
  role: Produces per-bin enhancer probabilities from H3K4me1/H3K4me3/H3K27ac ChIP-seq
    that are then overlapped onto regions
  repo: https://github.com/slrvv/CENTRE
- name: CRUP-PP
  role: Derived from the complementary of CRUP-EP's second classifier; provides promoter
    probabilities overlapped onto the same regions
  repo: https://github.com/slrvv/CENTRE
- name: crupR
  role: Normalizes the three histone-mark ChIP-seq experiments (cores = 2) into genome-browser
    tracks before bin-level probability computation
  repo: https://github.com/slrvv/CENTRE
- name: ExperimentHub
  role: Serves the in-package RNA-seq TPM table (EH9545, mean TPM over replicates)
    joined onto the ET pairs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:267c3c6d46d1ad33c3fad32fdeabcfe4b3040b188fe484cfd8de951e76f6e594
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slrvv/CENTRE
  tools:
  - BiocStyle
  - Bioconductor
  - CENTRE
  - CRUP-EP
  - CRUP-PP
  - EpiCompare
  - ExperimentHub
  - GenomicRanges
  - R
  - crupR
  - knitr
  - rmarkdown
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- genomic-interval-overlap-analysis
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Genomic interval overlap annotation

## Summary

Overlaps per-bin regulatory probabilities (CRUP-EP enhancer and CRUP-PP promoter scores over 100-bp genomic bins) onto cCRE-ELS enhancer and GENCODE TSS regions using GenomicRanges findOverlaps, to attach cell-type-specific enhancer/promoter activity scores to enhancer-target (ET) pairs in CENTRE.

## When to use

Use when per-region feature values (e.g. enhancer/promoter probabilities from bin-level tracks) must be transferred onto annotated genomic intervals (cCRE-ELS, TSSs) as input features for downstream ET-pair classification.

## When NOT to use

- You need cell-type-agnostic features (genomic distance, Wilcoxon rank-sum / Fisher's combined P-values) — use computeGenericFeatures instead
- The enhancer-target distance exceeds 500 KB, outside CENTRE's supported search interval
- You only have precomputed feature tables and need classification only — use centrePrediction on existing feature vectors

## Inputs

- Normalized histone-mark ChIP-seq genome-browser tracks (H3K4me1, H3K4me3, H3K27ac BAMs with matched input control, e.g. ENCSR000APW, ENCSR340WQU, ENCSR000AOC, ENCSR000AOB)
- CRUP-EP enhancer probabilities per 100-bp genomic bin
- CRUP-PP promoter probabilities per 100-bp genomic bin
- cCRE-LS enhancer regions and GENCODE TSS target regions extended to 500 bp as GRanges objects
- ET pairs data.frame (e.g. gene ENSG00000105281 with enhancers EH38E1958626 / EH38E3310851)

## Outputs

- data.frame with per-pair columns EP_prob_enh, EP_prob_gene, PP_prob_enh, PP_prob_gene (five scores each from replicate-aware CRUP runs)
- regulatory-distance features reg_dist_enh, norm_reg_dist_enh, reg_dist_prom, norm_reg_dist_prom (counts and normalized fractions of 100-bp bins with probability > 0.5 between pair members)
- celltype_features.csv containing the above plus the target gene's RNA-seq TPM value

## How to apply

First harmonize interval definitions: extend cCRE-ELS enhancer regions and GENCODE TSS target regions to 500 bp and restrict everything to the chromosome under analysis (e.g. chr19 in the HeLa-S3 example). Then apply CRUP-EP to normalized H3K4me1/H3K4me3/H3K27ac ChIP-seq (crupR normalization, matched input control) to obtain enhancer probabilities for every 100-bp genomic bin, and derive CRUP-PP promoter probabilities from the complementary of CRUP-EP's second classifier. Use the findOverlaps function from the GenomicRanges library to overlap bin-level probabilities with the extended regions, yielding per-pair scores (EP_prob_enh, EP_prob_gene, PP_prob_enh, PP_prob_gene). For regulatory-distance features, count (and normalize as fractions of) 100-bp bins with CRUP-EP > 0.5 or CRUP-PP > 0.5 between each enhancer-target pair.

## Related tools

- **GenomicRanges** (Provides the findOverlaps function used to intersect 100-bp-bin CRUP probabilities with extended enhancer and TSS regions) — https://github.com/slrvv/CENTRE
- **CRUP-EP** (Produces per-bin enhancer probabilities from H3K4me1/H3K4me3/H3K27ac ChIP-seq that are then overlapped onto regions) — https://github.com/slrvv/CENTRE
- **CRUP-PP** (Derived from the complementary of CRUP-EP's second classifier; provides promoter probabilities overlapped onto the same regions) — https://github.com/slrvv/CENTRE
- **crupR** (Normalizes the three histone-mark ChIP-seq experiments (cores = 2) into genome-browser tracks before bin-level probability computation) — https://github.com/slrvv/CENTRE
- **ExperimentHub** (Serves the in-package RNA-seq TPM table (EH9545, mean TPM over replicates) joined onto the ET pairs)

## Examples

```
features_celltype <- computeCellTypeFeatures(pairs, metaData = HeLaS3_metaData, RNAseqTPM = tpm_EH9545, cores = 2)
```

## Evaluation signals

- Returned data.frame written to celltype_features.csv contains exactly the expected columns: EP_prob_enh, EP_prob_gene, PP_prob_enh, PP_prob_gene, reg_dist_enh, norm_reg_dist_enh, reg_dist_prom, norm_reg_dist_prom, TPM
- All probabilities are in [0, 1]; regulatory-distance counts are non-negative integers with normalized fractions in [0, 1]
- Features are populated for the two example chr19 pairs (ENSG00000105281 with EH38E1958626 / EH38E3310851) when run on the HeLa-S3 in-package example
- Each ET pair has five EP_prob scores (one per CRUP run/replicate configuration) for both enhancer and gene regions
- Bins counted as regulatory distance are strictly those with CRUP-EP > 0.5 or CRUP-PP > 0.5 within the window between pair members

## Limitations

- CENTRE restricts ET-pair search and feature computation to interactions within 500 KB; extending this interval requires future methodological improvements
- Matched input control ChIP-seq is strongly advised; the pipeline can run without it but normalization quality suffers
- Including more histone marks could improve cCRE and regulatory-distance annotation but comes at the cost of extra experiments, making practical application harder

## Evidence

- [methods] Overlap uplifted regions with ENCODE Registry of cCREs: "Then we overlapped the uplifted regions with the latest version 2 of ENCODE Registry of cCREs on hg38"
- [methods] findOverlaps used for region-probability intersection: "[findOverlaps function from GenomicRanges library (Lawrence et al. 2013)]"
- [methods] CRUP-EP per-bin probabilities: "we applied the CRUP-EP algorithm. CRUP-EP outputs the enhancer probability for every 100-bp genomic bin"
- [results] Regulatory distance from fraction of enhancer/promoter bins between pair members: "We apply CRUP-EP and CRUP-PP to the window between the two regulatory elements and extract the fraction of regions classified as enhancers or promoters"
- [readme] Required input data: three histone-mark ChIP-seq and RNA-seq for the cell type of interest: [evidence span withheld — anchor status: external_doc]
