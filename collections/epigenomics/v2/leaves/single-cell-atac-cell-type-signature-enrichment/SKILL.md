---
name: single-cell-atac-cell-type-signature-enrichment
description: Use when a selected cisTopic LDA model exists for a scATAC-seq dataset
  and you must quantify per-cell enrichment of external epigenomic signatures (marker
  gene sets or bulk ATAC profiles) to assign or visualize cell-type identity.
when_to_use_negative:
- The dataset is already a cell-by-feature expression or score matrix and no topic
  model exists — AUCell rankings here depend on cisTopic's predictive distribution,
  not raw counts
- Signature regions are in a different genome assembly and cannot be mapped to dataset
  regions with at least 40% overlap
- The goal is motif/transcription-factor inference rather than per-cell signature
  scoring (use RcisTarget-based workflow instead)
tools:
- name: cisTopic
  role: Provides predictiveDistribution(), getSignaturesRegions() and signatureCellEnrichment()
    used to rank regions per cell and score signatures
  repo: https://github.com/aertslab/cistopic
- name: AUCell
  role: Builds cell-specific region rankings (AUCell_buildRankings) and computes per-cell
    AUC of signature regions
  repo: https://github.com/aertslab/AUCell
- name: Rtsne
  role: Used by runtSNE to project cell-topic probability distributions for overlaying
    signature AUC values
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-019-0367-1
    title: 'cisTopic: cis-regulatory topic modeling on single-cell ATAC-seq data'
  source_content_hash: sha256:4450e9de1fd30a662fd989edd137cdef065994719b5f95aa6a5ac3a9db39dd28
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/aertslab/cistopic
  tools:
  - AUCell
  - R
  - Rtsne
  - cisTopic
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Single-cell ATAC-seq cell-type signature enrichment

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

cisTopic-based scoring of curated cell-type signatures (e.g. PBMC marker sets, Corces et al. bulk hematopoietic ATAC signatures) on a single-cell ATAC-seq topic model: cell-specific region rankings are built from the model's predictive distribution and converted to per-cell AUC values with AUCell via signatureCellEnrichment.

## When to use

Use when a selected cisTopic LDA model exists for a scATAC-seq dataset and you must quantify per-cell enrichment of external epigenomic signatures (marker gene sets or bulk ATAC profiles) to assign or visualize cell-type identity.

## When NOT to use

- The dataset is already a cell-by-feature expression or score matrix and no topic model exists — AUCell rankings here depend on cisTopic's predictive distribution, not raw counts
- Signature regions are in a different genome assembly and cannot be mapped to dataset regions with at least 40% overlap
- The goal is motif/transcription-factor inference rather than per-cell signature scoring (use RcisTarget-based workflow instead)

## Inputs

- Precomputed cisTopic object with a selected topic model (cisTopicObject_pbmc.Rds, 30-topic LDA on 5,335 PBMCs)
- Curated cell-type signature region sets (PBMC marker gene sets / Corces et al. 2016 bulk hematopoietic ATAC signatures)
- Dataset peak regions (atac_v1_pbmc_5k_peaks.bed) against which signature regions are mapped

## Outputs

- Cell-by-signature AUC score matrix (one AUC column per curated signature, e.g. CD4 T cell, monocyte, B cell, NK cell, for all 5,335 cells), exported as signature_cell_enrichment.csv
- t-SNE feature plots colored by per-signature AUC values (signature_enrichment_tsne.png)

## How to apply

Load the precomputed cisTopic object holding the selected topic model (e.g. cisTopicObject_pbmc.Rds with 30 topics) and compute its predictive distribution, i.e. the cell-by-region likelihood matrix, with predictiveDistribution(). Feed this matrix to AUCell_buildRankings to obtain cell-specific region rankings. Intersect and map the curated signature regions to dataset regions with getSignaturesRegions using a minimum overlap of 0.4 (40%). Run signatureCellEnrichment with selected.signatures='all' and aucMaxRank=0.3*nrow(aucellRankings), which uses the rankings to compute an AUC per signature per cell; the AUC scores land in the cell data slot. For visualization, run t-SNE on the cell-topic probability distributions (runtSNE with seed=123, pca=FALSE, method='Probability') and overlay each signature's AUC with plotFeatures.

## Related tools

- **cisTopic** (Provides predictiveDistribution(), getSignaturesRegions() and signatureCellEnrichment() used to rank regions per cell and score signatures) — https://github.com/aertslab/cistopic
- **AUCell** (Builds cell-specific region rankings (AUCell_buildRankings) and computes per-cell AUC of signature regions) — https://github.com/aertslab/AUCell
- **Rtsne** (Used by runtSNE to project cell-topic probability distributions for overlaying signature AUC values)

## Examples

```
cisTopicObject <- signatureCellEnrichment(cisTopicObject, selected.signatures='all', aucMaxRank=0.3*nrow(aucellRankings))
```

## Evaluation signals

- The exported signature_cell_enrichment.csv contains exactly one AUC column per curated signature (e.g. CD4 T cell, monocyte, B cell, NK cell) for all 5,335 PBMC cells
- Signature-to-dataset region mapping succeeded with minOverlap=0.4 (>=40% overlap of signature regions with dataset regions)
- aucMaxRank was set to 0.3*nrow(aucellRankings) when calling signatureCellEnrichment
- The t-SNE overlay (seed=123, pca=FALSE, method='Probability') shows spatially coherent, cell-type-specific enrichment — each signature's AUC is concentrated on a distinct island of cells
- Signatures scoring near zero across cells correspond to promoter (non-cell-type-specific) topics, consistent with the article's observation

## Limitations

- cisTopic is deprecated in favor of pycisTopic; the R workflow shown here reflects the legacy R package
- Promoter topics are not enriched for cell-type-specific signatures, so signatures may fail to separate cell types if they map mostly to promoter regions
- Accurate signature scoring depends on having first selected a good topic model (e.g. 30 topics by highest log likelihood P(D|T))

## Evidence
