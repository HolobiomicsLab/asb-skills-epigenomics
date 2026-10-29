---
name: kmer-frequency-computation
description: Use when you have a set of labeled genomic regions (e.g. HOT vs COLD
  from ChIP-seq analysis) and need sequence-derived features — k-mer frequencies,
  GC content, CpG O/E ratio — for a classifier or feature-importance analysis.
when_to_use_negative:
- Input is already a numeric feature table — recomputing k-mer frequencies would be
  redundant; proceed directly to model fitting.
- The goal is TF motif analysis of HOT regions — the paper notes HOT regions lack
  canonical motifs for the TFs thought to be bound there, so k-mer composition features
  are not a substitute for motif testing.
- Regions are not defined from ChIP-seq binding data (e.g. random or expression-derived
  intervals) — the HOT/COLD discrimination framing from this paper would not apply.
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3474
tools:
- name: glmnet
  role: Fits the elastic net classifiers (one per species, 10-fold cross-validated)
    on the k-mer/GC/CpG feature matrix to discriminate HOT from COLD regions and yield
    feature importances and AUCs
- name: HOT-region-main.R
  role: Main analysis script in the repository used for the HOT region analysis, including
    the feature computation and modeling workflow
  repo: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:cd3013e506111552f50bdc68c5436512c306b674e9d1c8ca9137a62797cfd6bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
  tools:
  - HOT-region-main.R
  - glmnet
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# kmer-frequency-computation

## Summary

Computes k-mer frequency vectors (k = 2, 3, 4) plus GC content, GC skew, and CpG observed/expected ratio per genomic region, producing the feature matrix used to discriminate HOT from COLD regions with elastic net classifiers (glmnet).

## When to use

Use when you have a set of labeled genomic regions (e.g. HOT vs COLD from ChIP-seq analysis) and need sequence-derived features — k-mer frequencies, GC content, CpG O/E ratio — for a classifier or feature-importance analysis.

## When NOT to use

- Input is already a numeric feature table — recomputing k-mer frequencies would be redundant; proceed directly to model fitting.
- The goal is TF motif analysis of HOT regions — the paper notes HOT regions lack canonical motifs for the TFs thought to be bound there, so k-mer composition features are not a substitute for motif testing.
- Regions are not defined from ChIP-seq binding data (e.g. random or expression-derived intervals) — the HOT/COLD discrimination framing from this paper would not apply.

## Inputs

- Set of HOT region genomic coordinates (from upstream HOT-region calling over ENCODE/modENCODE ChIP-seq data)
- Set of COLD (non-HOT) region genomic coordinates
- Reference genome sequences for the regions of interest

## Outputs

- k-mer frequency vectors (k = 2, 3, 4) per region
- Per-region GC content, GC skew, and CpG observed/expected (O/E) ratio
- Labeled feature matrix (HOT/COLD class labels plus sequence features)
- Feature-importance table of top-ranked elastic net coefficients and cross-validated AUCs

## How to apply

Extract the genomic sequences corresponding to each region set (HOT and COLD regions produced by the upstream peak-analysis step), then compute k-mer frequency vectors for k = 2, 3, and 4 over each region. Per region, additionally calculate GC content, GC skew, and the CpG observed/expected (O/E) ratio, and assemble these into a feature matrix with HOT/COLD class labels. The rationale comes from the paper's finding that GC/CpG-rich k-mers and CpG O/E ratio are the most discriminatory features of HOT regions across species, so the CpG-related features should be present in every model matrix. Feed the matrix into species-specific elastic net classifiers (glmnet with 10-fold cross-validation) and extract top-ranked coefficients into a feature-importance table. Validate by checking that cross-validated AUCs fall in the reported 0.82–0.94 range and that CpG/GC-rich k-mers with the CpG O/E ratio rank among the top features.

## Related tools

- **glmnet** (Fits the elastic net classifiers (one per species, 10-fold cross-validated) on the k-mer/GC/CpG feature matrix to discriminate HOT from COLD regions and yield feature importances and AUCs)
- **HOT-region-main.R** (Main analysis script in the repository used for the HOT region analysis, including the feature computation and modeling workflow) — https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions

## Examples

```
Rscript Scripts/HOT-region-main.R
```

## Evaluation signals

- Cross-validation AUCs of the four species-specific elastic net classifiers fall within the reported 0.82–0.94 range
- CpG/GC-rich k-mers and the CpG O/E ratio rank among the top features in each species' model coefficient table
- Feature matrix rows match the number of HOT + COLD regions with complete k = 2, 3, 4 frequency vectors and no missing GC/CpG O/E values
- Per-region CpG O/E ratio values are positive and consistent with the region's GC content and length

## Limitations

- The paper argues HOT regions are likely ChIP-seq artifacts ('hyper-ChIPable' regions), so GC/CpG composition features may reflect artificial ChIP-seq enrichment rather than true TF binding biology.
- HOT regions lack canonical motifs for the TFs thought to be bound there, so k-mer features cannot explain motifless binding mechanistically.
- Validation of KO ChIP-seq results is complicated by technical confounders such as low read numbers, low library complexity, or unsuccessful IP.

## Evidence

- [abstract] HOT regions have discriminatory sequence characteristics and genomic features including GC/CpG-rich k-mers, R-loops, and G-quadruplex DNA: "We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures"
- [intro] Cross-species sequence feature comparison motivation: "To date, there has been no cross-species comparison of HOT regions in terms of sequence features. The sequence features that are shared across species can provide a mechanistic insight into HOT"
- [methods] Workflow: k-mer feature computation and glmnet modeling: [evidence span withheld — anchor status: not_found]
- [results] Validation of model performance range: [evidence span withheld — anchor status: not_found]
- [readme] Repository script grounding: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
