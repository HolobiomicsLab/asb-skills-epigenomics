---
name: gc-content-and-skew-calculation
description: Use when you have genomic region sets (e.g. HOT vs COLD intervals) in
  FASTA or BED+reference form and need sequence-composition features (GC content,
  GC skew, CpG O/E, k-mer frequencies) for classification.
when_to_use_negative:
- Input is already a numeric feature table or a precomputed feature matrix — this
  skill computes sequence-composition features from raw genomic regions, which would
  be redundant.
- The goal is functional or motif-level characterization of HOT regions (e.g. canonical
  TF motif enrichment, R-loop or G-quadruplex annotation) rather than sequence-composition
  discrimination features.
- Regions lack an available reference genome or extracted sequences, so GC/k-mer/CpG
  O/E statistics cannot be computed.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: glmnet
  role: 'Downstream consumer of the GC/k-mer/CpG O/E feature matrix: trains elastic
    net classifiers with 10-fold cross-validation to discriminate HOT from COLD regions'
  repo: BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:1a8cc88ffd0e283ef6480f3efe31c92439ce081207ac6cd2f8e4c3dffe4d083e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
  tools:
  - glmnet
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# GC content and skew calculation

## Summary

Computes per-region GC content, GC skew, CpG observed/expected ratio, and k-mer frequency vectors (k = 2, 3, 4) over HOT and COLD genomic regions, producing the feature matrix used to discriminate HOT regions and identify GC/CpG-rich k-mers as top predictive features.

## When to use

Use when you have genomic region sets (e.g. HOT vs COLD intervals) in FASTA or BED+reference form and need sequence-composition features (GC content, GC skew, CpG O/E, k-mer frequencies) for classification.

## When NOT to use

- Input is already a numeric feature table or a precomputed feature matrix — this skill computes sequence-composition features from raw genomic regions, which would be redundant.
- The goal is functional or motif-level characterization of HOT regions (e.g. canonical TF motif enrichment, R-loop or G-quadruplex annotation) rather than sequence-composition discrimination features.
- Regions lack an available reference genome or extracted sequences, so GC/k-mer/CpG O/E statistics cannot be computed.

## Inputs

- HOT and COLD region sets (genomic intervals from the upstream task)
- Corresponding genomic sequences for each region (extracted from reference genome)

## Outputs

- Per-region k-mer frequency vectors for k = 2, 3, 4
- Per-region GC content, GC skew, and CpG observed/expected (O/E) ratio values
- Assembled feature matrix with HOT/COLD class labels for downstream glmnet classification

## How to apply

Load the HOT and COLD region sets produced by the upstream peak-calling step and extract the corresponding genomic sequences. For each region, compute k-mer frequency vectors for k = 2, 3, and 4, plus GC content, GC skew, and the CpG observed/expected (O/E) ratio, then assemble these into a feature matrix with HOT/COLD class labels. These features feed species-specific elastic net classifiers trained in glmnet with 10-fold cross-validation, which is the rationale for including sequence-composition metrics rather than raw sequence. Decision point: verify that CpG/GC-rich k-mers together with the CpG O/E ratio rank among the top-ranked coefficients in each fitted model, since the paper reports these as the most important predictive features across all species.

## Related tools

- **glmnet** (Downstream consumer of the GC/k-mer/CpG O/E feature matrix: trains elastic net classifiers with 10-fold cross-validation to discriminate HOT from COLD regions) — BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions

## Examples

```
Rscript Scripts/HOT-region-main.R
```

## Evaluation signals

- Cross-validation AUC of each species-specific elastic net classifier falls within the reported 0.82–0.94 range.
- CpG/GC-rich k-mers together with the CpG observed/expected ratio rank among the top-ranked coefficients in each fitted model's feature-importance table.
- Feature matrix contains one row per HOT/COLD region with k-mer frequencies (k = 2, 3, 4), GC content, GC skew, and CpG O/E ratio columns plus class labels.
- Classifier count matches the study design: four species-specific elastic net models, each tuned with 10-fold cross-validation.

## Limitations

- The study found HOT regions lack canonical motifs for TFs thought to be bound there, so sequence-composition features discriminate HOT regions but do not explain canonical TF binding.
- The paper argues HOT regions are likely ChIP-seq artifacts ('hyper-ChIPable' regions); GC/CpG-rich features may reflect this artificial enrichment rather than true regulatory biology.
- AUC is cross-validated within species; the paper notes no cross-species comparison of HOT region sequence features existed previously, so feature transferability across species is not established by this workflow.

## Evidence

- [results] Species-specific ML: [evidence span withheld — anchor status: not_found]
- [abstract] Discriminatory sequence features: "We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures"
- [intro] Cross-species gap: "To date, there has been no cross-species comparison of HOT regions in terms of sequence features."
- [readme] Analysis scripts: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
