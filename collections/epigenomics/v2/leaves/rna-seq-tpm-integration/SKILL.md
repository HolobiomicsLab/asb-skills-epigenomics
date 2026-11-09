---
name: rna-seq-tpm-integration
description: Use when computing CENTRE features for enhancer-target pairs and a TPM
  expression table must be attached to each target gene before calling centrePrediction,
  or when checking that RNA-seq inputs match CENTRE's expected schema.
when_to_use_negative:
- When the user already has a complete feature table (features_celltype and features_generic);
  TPM integration has already occurred inside computeCellTypeFeatures()
- When the analysis is training-data curation rather than prediction — TPM > 0 was
  a filter applied to positive BENGI interactions during training, not a runtime feature
  step
- When the cell type of interest lacks RNA-seq data; CENTRE's model expects the expression
  feature and cannot be run with histone ChIP-seq alone
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CENTRE
  role: Bioconductor-style R package whose computeCellTypeFeatures() attaches the
    user-supplied TPM value to each enhancer-target pair
  repo: https://github.com/slrvv/CENTRE
- name: crupR
  role: Normalization of the histone-mark ChIP-seq that runs alongside TPM integration
    within computeCellTypeFeatures()
- name: ExperimentHub
  role: Provides the precomputed HeLa-S3 example TPM table (EH9545, mean TPM over
    replicates)
- name: xgboost
  role: Pre-trained classifier that consumes the TPM feature (ranked position 2 among
    features) with CRUP and distance features
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:fe6d9ff97a6f0f7a77074b56ef54bc0411a658ea3ac3e911bb27fbf6827b9362
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slrvv/CENTRE
  tools:
  - CENTRE
  - ExperimentHub
  - GenomicRanges
  - R
  - centrePrediction
  - computeCellTypeFeatures
  - computeGenericFeatures
  - createPairs
  - crupR
  - xgboost
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# rna-seq-tpm-integration

## Summary

Integrates cell-type-specific RNA-seq TPM values as the expression feature for enhancer-target pair classification in CENTRE, where RNA-seq ranks second among all model features and complements the three histone-mark ChIP-seq inputs.

## When to use

Use when computing CENTRE features for enhancer-target pairs and a TPM expression table must be attached to each target gene before calling centrePrediction, or when checking that RNA-seq inputs match CENTRE's expected schema.

## When NOT to use

- When the user already has a complete feature table (features_celltype and features_generic); TPM integration has already occurred inside computeCellTypeFeatures()
- When the analysis is training-data curation rather than prediction — TPM > 0 was a filter applied to positive BENGI interactions during training, not a runtime feature step
- When the cell type of interest lacks RNA-seq data; CENTRE's model expects the expression feature and cannot be run with histone ChIP-seq alone

## Inputs

- Cell-type-specific RNA-seq TPM data.frame with three columns: ENSEMBL gene IDs, transcript IDs, TPM values for all genes
- pairs data.frame of enhancer-gene pairs (enhancer_id, gene_id) to which TPM values are attached
- metaData data.frame of histone-mark ChIP-seq BAM files (H3K4me1, H3K4me3, H3K27ac) with matched input control

## Outputs

- features_celltype data.frame containing the TPM column per enhancer_id-gene_id pair alongside EP_prob_enh, EP_prob_gene, PP_prob_enh, PP_prob_gene, reg_dist_enh, norm_reg_dist_enh, reg_dist_prom, norm_reg_dist_prom
- TPM value attached per target gene for feeding to centrePrediction()

## How to apply

Prepare a cell-type-specific RNA-seq table with exactly three columns: gene ENSEMBL IDs (gene_id), transcript IDs, and TPM values for all genes. Load the TPM data.frame and pass it as the tpmData argument to computeCellTypeFeatures() alongside the metaData of H3K4me1/H3K4me3/H3K27ac BAM files (with matched input control), so the TPM value of each pair's gene is attached to the cell-type-specific feature table. CENTRE's pre-trained XGBoost classifier then uses this TPM feature together with CRUP-EP/CRUP-PP probabilities and regulatory-distance features; feature ranking places RNA-seq at position 2, behind only genomic distance. For the in-package HeLa-S3 example, the mean TPM over replicates is available via ExperimentHub record EH9545. The interaction threshold applied downstream (probability > 0.5) is independent of TPM preprocessing, so no TPM cutoff is applied at feature-construction time; TPM > 0 was only used to curate positive training interactions.

## Related tools

- **CENTRE** (Bioconductor-style R package whose computeCellTypeFeatures() attaches the user-supplied TPM value to each enhancer-target pair) — https://github.com/slrvv/CENTRE
- **crupR** (Normalization of the histone-mark ChIP-seq that runs alongside TPM integration within computeCellTypeFeatures())
- **ExperimentHub** (Provides the precomputed HeLa-S3 example TPM table (EH9545, mean TPM over replicates))
- **xgboost** (Pre-trained classifier that consumes the TPM feature (ranked position 2 among features) with CRUP and distance features)

## Examples

```
features_celltype <- computeCellTypeFeatures(metaData, replicate = 1, input.free = FALSE, cores = 2, sequencing = 'single', tpmData = tpm, pairs = pairs)
```

## Evaluation signals

- TPM data.frame must contain exactly three columns: gene_id (ENSEMBL), transcript_ids, and TPM for all genes
- The returned cell-type feature data.frame must include a TPM column with one value per enhancer_id-gene_id row, matching the gene of each pair
- In the HeLa-S3 example (chr19 genes ENSG00000105281, enhancers EH38E1958626/EH38E3310851), TPM values derive from ExperimentHub EH9545 mean-over-replicates values
- TPM column values must be non-negative; training data only included positive interactions with TPM > 0
- Downstream centrePrediction() output must assign each pair a probability, with pairs > 0.5 labeled as interacting

## Limitations

- CENTRE requires TPM values for all genes of the cell type of interest — partial expression tables cannot be substituted
- RNA-seq is only one signal: genomic distance ranks first and some generic features (Pearson correlation, Fisher-combined Wilcoxon) performed worse than distance alone on GM12878
- Control ChIP-seq matching the histone-mark experiments is strongly advised but CENTRE can run without it; expression input has no such fallback — it is mandatory

## Evidence

- [readme] Cell-type-specific RNA-seq TPM requirement for CENTRE: [evidence span withheld — anchor status: external_doc]
- [results] RNA-seq ranks second among CENTRE model features: "positions 2 (RNA-seq), 3 (Normalized Regulatory distance enhancers),"
- [results] CENTRE's minimal input requirement including RNA-seq: "our algorithm only requires RNA-seq data as well as ChIP-seq data of the histone modifications H3K27ac, H3K4me1, and H3K4me3 determined for this particular CT"
- [methods] TPM feature attached within cell-type feature computation: "The function `computeCellTypeFeatures()` computes the features that are cell type specific"
