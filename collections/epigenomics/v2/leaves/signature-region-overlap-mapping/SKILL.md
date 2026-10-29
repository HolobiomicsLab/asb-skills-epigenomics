---
name: signature-region-overlap-mapping
description: Use when curated epigenomic/bulk ATAC signature region sets must be intersected
  with the peaks of a cisTopic object and converted into per-cell AUC enrichment scores
  (e.g. PBMC marker gene sets on 10X 5k PBMC data).
when_to_use_negative:
- The topic model has not been selected yet — the predictive distribution and cell-specific
  region rankings required by signatureCellEnrichment do not exist (run runModels/selectModel
  first).
- Signature sets lack genomic region coordinates that can be intersected with dataset
  peaks (default 40% overlap will discard non-mappable signatures).
- Input is gene-level expression rather than scATAC-seq regions — the article's workflow
  maps epigenomic signatures to regions of the topic model, not transcriptomic signatures.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: cisTopic
  role: provides getSignaturesRegions (minOverlap=0.4 intersection/mapping) and signatureCellEnrichment
    on the cisTopic object
  repo: https://github.com/aertslab/cisTopic
- name: AUCell
  role: builds cell-specific region rankings from the predictive distribution and
    computes per-cell signature AUC enrichment
  repo: https://github.com/aertslab/AUCell
- name: Rtsne
  role: used via runtSNE (method='Probability', seed=123, pca=FALSE) to visualize
    per-signature AUC scores across cells
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-019-0367-1
    title: 'cisTopic: cis-regulatory topic modeling on single-cell ATAC-seq data'
  source_content_hash: sha256:8f3bb85a21f64776b2199132d9141a1a41286b28acf53dfe6883cfb0099e6fe3
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/aertslab/cisTopic
  tools:
  - AUCell
  - R
  - Rtsne
  - cisTopic
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# signature-region-overlap-mapping

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Maps curated epigenomic signature regions (e.g. Corces et al. 2016 bulk hematopoietic ATAC signatures) onto the regions of a cisTopic scATAC-seq object by genomic overlap, then scores per-cell signature enrichment with AUCell. This is how cell-type signatures are projected onto a selected topic model in cisTopic.

## When to use

Use when curated epigenomic/bulk ATAC signature region sets must be intersected with the peaks of a cisTopic object and converted into per-cell AUC enrichment scores (e.g. PBMC marker gene sets on 10X 5k PBMC data).

## When NOT to use

- The topic model has not been selected yet — the predictive distribution and cell-specific region rankings required by signatureCellEnrichment do not exist (run runModels/selectModel first).
- Signature sets lack genomic region coordinates that can be intersected with dataset peaks (default 40% overlap will discard non-mappable signatures).
- Input is gene-level expression rather than scATAC-seq regions — the article's workflow maps epigenomic signatures to regions of the topic model, not transcriptomic signatures.

## Inputs

- cisTopic object with a selected topic model (Rds, e.g. cisTopicObject_pbmc.Rds)
- cell-by-region predictive distribution matrix from predictiveDistribution()
- curated signature region sets in genomic coordinates (e.g. Corces et al. 2016 bulk hematopoietic ATAC signatures, PBMC marker sets)
- cell-specific region rankings built with AUCell_buildRankings

## Outputs

- signature-to-dataset-region mapping restricted to >=40% (minOverlap=0.4) overlaps
- cell-by-signature AUC score matrix in the cisTopic cell data slot, exportable as signature_cell_enrichment.csv
- t-SNE feature plots colored by per-signature AUC (signature_enrichment_tsne.png)

## How to apply

Start from a cisTopic object with a selected model (e.g. the 30-topic LDA model in cisTopicObject_pbmc.Rds) and compute the predictive distribution as the cell-by-region likelihood matrix. Build cell-specific region rankings with AUCell_buildRankings on that predictive distribution, since cisTopic uses AUCell for estimating the enrichment of epigenomic signatures within cells. Intersect and map the external signature regions to dataset regions with getSignaturesRegions, keeping overlaps of at least 40% (minOverlap=0.4). Run signatureCellEnrichment with the mapped signatures and the cell-specific region rankings (selected.signatures='all', aucMaxRank=0.3*nrow(aucellRankings)) to obtain per-cell AUC values, stored in the object's cell data slot. Export the cell-by-signature AUC matrix (e.g. signature_cell_enrichment.csv) and optionally overlay AUCs on a cell t-SNE (runtSNE with method='Probability') via plotFeatures.

## Related tools

- **cisTopic** (provides getSignaturesRegions (minOverlap=0.4 intersection/mapping) and signatureCellEnrichment on the cisTopic object) — https://github.com/aertslab/cisTopic
- **AUCell** (builds cell-specific region rankings from the predictive distribution and computes per-cell signature AUC enrichment) — https://github.com/aertslab/AUCell
- **Rtsne** (used via runtSNE (method='Probability', seed=123, pca=FALSE) to visualize per-signature AUC scores across cells)

## Examples

```
signatureCellEnrichment(cisTopicObject, aucellRankings, selected.signatures='all', aucMaxRank=0.3*nrow(aucellRankings))
```

## Evaluation signals

- Every retained signature maps to dataset regions with at least 40% overlap (minOverlap=0.4 in getSignaturesRegions).
- The exported CSV contains one AUC column per curated signature (e.g. CD4 T cell, monocyte, B cell, NK cell) for all 5,335 PBMC cells.
- aucMaxRank is set to 0.3*nrow(aucellRankings), a fixed fraction of the ranked regions per cell.
- plotFeatures t-SNE overlay shows spatially coherent, cell-type-specific enrichment of each signature's AUC values.
- Topics/regions not enriched for any cell-type-specific signature should correspond to promoter topics, a sanity pattern noted in the article.

## Limitations

- cisTopic (R) is deprecated in favor of pycisTopic, so this R-based overlap/enrichment workflow may need porting for new projects.
- Only a minimum-overlap default of 40% is documented; signatures with weaker overlap to the dataset peaks are dropped from the enrichment step.
- The current RcisTarget region databases cited in the article provide hg19 only, constraining genome compatibility of the mapped signatures.

## Evidence
