---
name: predictive-distribution-region-ranking
description: Use when a fitted cisTopic LDA model (e.g. a 30-topic model on 5,335
  PBMC cells) exists and per-cell enrichment of curated epigenomic signatures is needed,
  requiring cell-specific region rankings as input.
when_to_use_negative:
- Input is raw scATAC-seq data (fragments.tsv.gz / peaks.bed) without a fitted topic
  model — build and select a model with runModels/selectModel first
- You need topic-level or region-level scores rather than per-cell signature enrichment
  — use getRegionsScores or binarizecisTopics instead
- cisTopic is deprecated for new work — the README directs users to pycisTopic for
  the equivalent Python workflow
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3295
tools:
- name: cisTopic
  role: R package providing predictiveDistribution() and signatureCellEnrichment()
    to derive cell-specific region rankings and signature AUC scores from the selected
    topic model
  repo: https://github.com/aertslab/cisTopic
- name: AUCell
  role: AUCell_buildRankings consumes the predictive distribution matrix to produce
    the cell-specific region rankings used for signature enrichment
  repo: https://github.com/aertslab/AUCell
- name: R
  role: Language in which cisTopic and AUCell are run as an R/Bioconductor workflow
- name: Rtsne
  role: Used (via runtSNE with target='cell', method='Probability') to project cell-topic
    distributions for visualizing signature AUC overlays
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-019-0367-1
    title: 'cisTopic: cis-regulatory topic modeling on single-cell ATAC-seq data'
  source_content_hash: sha256:b9e73e36abf402b999e6c04a10279c4ff0444c21d04cefd61aa6670d2bcbdb47
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/aertslab/cisTopic
  tools:
  - AUCell
  - R
  - Rtsne
  - cisTopic
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# predictive-distribution-region-ranking

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Derives cell-specific region rankings from a selected cisTopic topic model by computing the predictive distribution (cell-by-region likelihood matrix) and feeding it into AUCell_buildRankings, so that curated signatures (e.g. PBMC marker gene sets) can later be scored per cell via signatureCellEnrichment.

## When to use

Use when a fitted cisTopic LDA model (e.g. a 30-topic model on 5,335 PBMC cells) exists and per-cell enrichment of curated epigenomic signatures is needed, requiring cell-specific region rankings as input.

## When NOT to use

- Input is raw scATAC-seq data (fragments.tsv.gz / peaks.bed) without a fitted topic model — build and select a model with runModels/selectModel first
- You need topic-level or region-level scores rather than per-cell signature enrichment — use getRegionsScores or binarizecisTopics instead
- cisTopic is deprecated for new work — the README directs users to pycisTopic for the equivalent Python workflow

## Inputs

- cisTopic R object with a selected topic model (e.g. cisTopicObject_pbmc.Rds, 30-topic LDA on 5,335 PBMC cells over 97k regulatory regions)
- Predictive distribution matrix (cell-by-region likelihood) from predictiveDistribution()
- Curated signature region sets (e.g. PBMC marker gene sets / Corces et al. 2016 bulk hematopoietic ATAC signatures)

## Outputs

- Cell-specific region rankings (AUCell rankings object built from the predictive distribution)
- Cell-by-signature AUC score matrix in the cisTopic object's cell data slot (exportable as signature_cell_enrichment.csv)

## How to apply

Load the precomputed cisTopic object (e.g. cisTopicObject_pbmc.Rds) after the model has been selected by selectModel, which picks the model with the highest log likelihood P(D|T). Compute the predictive distribution with cisTopic's predictiveDistribution(), yielding a cell-by-region likelihood matrix rather than using raw accessibility counts. Pass that matrix to AUCell_buildRankings to build cell-specific region rankings; these rankings are the input AUCell requires for signature scoring. Subsequently map curated signature regions to dataset regions with getSignaturesRegions (default at least 40% overlap, e.g. minOverlap=0.4 for Corces et al. (2016) bulk hematopoietic ATAC signatures) and run signatureCellEnrichment with aucMaxRank set to a generous fraction of rankings (e.g. 0.3*nrow(aucellRankings)) to obtain per-cell AUC values stored in the cell data slot.

## Related tools

- **cisTopic** (R package providing predictiveDistribution() and signatureCellEnrichment() to derive cell-specific region rankings and signature AUC scores from the selected topic model) — https://github.com/aertslab/cisTopic
- **AUCell** (AUCell_buildRankings consumes the predictive distribution matrix to produce the cell-specific region rankings used for signature enrichment) — https://github.com/aertslab/AUCell
- **R** (Language in which cisTopic and AUCell are run as an R/Bioconductor workflow)
- **Rtsne** (Used (via runtSNE with target='cell', method='Probability') to project cell-topic distributions for visualizing signature AUC overlays)

## Examples

```
cisTopicObject <- readRDS('cisTopicObject_pbmc.Rds'); predMat <- predictiveDistribution(cisTopicObject); aucellRankings <- AUCell_buildRankings(predMat)
```

## Evaluation signals

- The AUCell rankings object is built from the predictive distribution matrix, not from the raw binary accessibility matrix
- Downstream signatureCellEnrichment output contains one AUC column per curated signature (e.g. CD4 T cell, monocyte, B cell, NK cell) for all 5,335 PBMC cells
- t-SNE overlay of signature AUC values shows spatially coherent, cell-type-specific enrichment for each signature
- Signature regions were mapped with getSignaturesRegions using at least 40% overlap (minOverlap=0.4), consistent with the paper's default
- aucMaxRank was set to a large fraction of the rankings (0.3*nrow(aucellRankings)) so sparse signatures are not missed

## Limitations

- cisTopic itself is deprecated in favor of pycisTopic, so this R-only workflow must be reimplemented for maintained software
- The predictive distribution depends on the selected model quality; a poorly selected model (not the highest log likelihood P(D|T)) will distort rankings
- Signature mapping is limited by overlap between curated signature regions and dataset regions; signatures with less than the 40% overlap threshold may not transfer

## Evidence
