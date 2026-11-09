---
name: single-cell-atac-topic-model-interpretation
description: 'Use when a fitted cisTopic object (e.g. cisTopicObject_pbmc.Rds with
  a selected 30-topic model) needs biological interpretation: assigning functions,
  master regulators, or cell-type signatures to each topic.'
when_to_use_negative:
- cisTopic has not yet been fitted and a model selected — run runModels() over the
  topic grid and selectModel() first (selectModel picks the highest last-iteration
  log likelihood P(D|T))
- The data is not single-cell epigenomic count/binary accessibility data over regulatory
  regions — cisTopic is designed for sparse single-cell epigenomics data such as the
  10X atac_v1_pbmc_5k peaks/fragments
- Working in Python from scratch — cisTopic (R) is deprecated in favor of pycisTopic
  for new analyses
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3169
tools:
- name: cisTopic
  role: R package providing binarizecisTopics, binarizedcisTopicsToCtx, GREAT, topicsRcisTarget
    and getCistromes to interpret fitted topics
  repo: http://github.com/aertslab/cistopic
- name: rGREAT
  role: Performs GO term enrichment on the regions assigned to each binarized topic
    via the cisTopic GREAT() wrapper
- name: RcisTarget
  role: Motif enrichment on topic-enriched regions to derive per-topic TF cistromes
    and master regulators
  repo: https://github.com/aertslab/RcisTarget
- name: AUCell
  role: Used with cisTopic for estimating enrichment of epigenomic signatures within
    cells (signatureCellEnrichment)
  repo: https://github.com/aertslab/AUCell
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_003
  - task_002
  - task_007
  source_papers:
  - doi: 10.1038/s41592-019-0367-1
    title: 'cisTopic: cis-regulatory topic modeling on single-cell ATAC-seq data'
  source_content_hash: sha256:ce92f99cace14c1d1b7c45a297845da9f263d120518d819d69d1b41aac63798e
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - AUCell
  - R
  - RcisTarget
  - cisTopic
  - rGREAT
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 18
---

# single-cell-atac-topic-model-interpretation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Interprets cisTopic LDA topics fitted on single-cell ATAC-seq data (e.g. the 5k PBMC dataset with ~97k regulatory regions) by binarizing topics and annotating their top regions through GREAT GO enrichment, RcisTarget motif enrichment for TF cistromes, and signature enrichment with AUCell-style cell rankings.

## When to use

Use when a fitted cisTopic object (e.g. cisTopicObject_pbmc.Rds with a selected 30-topic model) needs biological interpretation: assigning functions, master regulators, or cell-type signatures to each topic.

## When NOT to use

- cisTopic has not yet been fitted and a model selected — run runModels() over the topic grid and selectModel() first (selectModel picks the highest last-iteration log likelihood P(D|T))
- The data is not single-cell epigenomic count/binary accessibility data over regulatory regions — cisTopic is designed for sparse single-cell epigenomics data such as the 10X atac_v1_pbmc_5k peaks/fragments
- Working in Python from scratch — cisTopic (R) is deprecated in favor of pycisTopic for new analyses

## Inputs

- Precomputed cisTopic object (cisTopicObject_pbmc.Rds) with the selected 30-topic LDA model on the 5,335-cell PBMC dataset (~97k regulatory regions)
- RcisTarget hg19 region-based ranking database (hg19-regions-9species.all_regions.mc8nr.feather)
- Optional cell-type epigenomic signatures, e.g. Corces et al. (2016) bulk hematopoietic ATAC signatures

## Outputs

- great_go_enrichment_per_topic.csv: per-topic enriched GO terms (binomial adjusted p-value <= 0.05, fold enrichment >= 2) from object@binarized.rGREAT
- master_regulators.csv: per-topic ranked TF table (topic, TF, NES, motif, enrichment metrics) from object@binarized.RcisTarget
- Topic-specific cistromes (directly and indirectly annotated, *_extended) and an updated saved cisTopic Rds

## How to apply

First binarize the cisTopics with binarizecisTopics() using the GammaFit method (thrP=0.975) to obtain each topic's top regions. For functional annotation, run the cisTopic GREAT() wrapper over rGREAT with genome='hg19', fold_enrichment=2, geneHits=1, sign=0.05, request_interval=10 (~30 min reference runtime) and extract per-topic enriched GO terms from object@binarized.rGREAT. For regulatory annotation, convert binarized topics to ctx regions with binarizedcisTopicsToCtx (genome='hg19') and run topicsRcisTarget against hg19-regions-9species.all_regions.mc8nr.feather with nesThreshold=3, rocthr=0.005, maxRank=20000, then build cistromes with getCistromes(annotation='Both'). Topics not enriched for any cell-type-specific signature (e.g. Corces et al. 2016 hematopoietic bulk signatures) correspond to promoter topics rather than cell-type-specific enhancers; SPI1 and CEBPB emerge as master regulators of the B cell and monocyte topics.

## Related tools

- **cisTopic** (R package providing binarizecisTopics, binarizedcisTopicsToCtx, GREAT, topicsRcisTarget and getCistromes to interpret fitted topics) — http://github.com/aertslab/cistopic
- **rGREAT** (Performs GO term enrichment on the regions assigned to each binarized topic via the cisTopic GREAT() wrapper)
- **RcisTarget** (Motif enrichment on topic-enriched regions to derive per-topic TF cistromes and master regulators) — https://github.com/aertslab/RcisTarget
- **AUCell** (Used with cisTopic for estimating enrichment of epigenomic signatures within cells (signatureCellEnrichment)) — https://github.com/aertslab/AUCell

## Examples

```
cisTopicObject <- GREAT(cisTopicObject, genome='hg19', fold_enrichment=2, geneHits=1, sign=0.05, request_interval=10)
```

## Evaluation signals

- great_go_enrichment_per_topic.csv contains enriched GO terms (binomial adjusted p-value <= 0.05, fold enrichment >= 2) for each of the 30 topics
- Topics not enriched for any cell-type-specific signature correspond to promoter-enriched topics rather than cell-type-specific enhancers
- master_regulators.csv contains SPI1 and CEBPB among the top-ranked regulators (by NES, high-confidence annotation) of the B cell and monocyte topics respectively
- The saved Rds exists and its per-topic RcisTarget tables (e.g. @binarized.RcisTarget[[1]] and [[3]]) recover the expected master regulators
- Binarized topic region sets derive from GammaFit thresholds with thrP=0.975, and each region may map to more than one ctx region in binarizedcisTopicsToCtx

## Limitations

- cisTopic (R) is deprecated; new analyses should use pycisTopic
- The GREAT wrapper is reference-annotated for genome='hg19' in this workflow with request_interval=10 and ~30 min runtime; other assemblies require the separate mm10/hg38 vignette
- RcisTarget results depend on the database choice (hg19-regions-9species.all_regions.mc8nr.feather, reduced_database=FALSE) and thresholds (nesThreshold=3, rocthr=0.005, maxRank=20000)

## Evidence
