---
name: transcription-factor-motif-enrichment-analysis
description: Use when you have binarized topic-enriched regions from a cisTopic model
  on single-cell ATAC-seq data and want to recover topic-specific TF cistromes and
  master regulators such as SPI1 and CEBPB.
when_to_use_negative:
- The input is raw fragments/peaks and no cisTopic topic model has been fit yet —
  run the LDA topic modeling and binarizecisTopics steps first.
- Your regions are from a genome assembly other than what the available RcisTarget
  feather databases cover (cisTopic's current version provides hg19 databases; mm10/hg38
  require separate vignette workflows).
- You need de novo motif discovery rather than recovery of known TF motifs via enrichment
  over a precomputed ranking database.
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: cisTopic
  role: Provides binarizecisTopics, binarizedcisTopicsToCtx, topicsRcisTarget and
    getCistromes to go from topic-enriched regions to topic-specific TF cistromes
    and master regulators
  repo: https://github.com/aertslab/cistopic
- name: RcisTarget
  role: Performs the motif-enrichment scoring (NES, high-confidence annotation) of
    topic regions against the hg19 feather ranking database
  repo: https://github.com/aertslab/RcisTarget
- name: R
  role: Environment in which the cisTopic/RcisTarget workflow is executed
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41592-019-0367-1
    title: 'cisTopic: cis-regulatory topic modeling on single-cell ATAC-seq data'
  source_content_hash: sha256:6637b1f9e474ea7ca225f7b1da78794429060d9d38e6c3c9e53c4f92b514dbff
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/aertslab/cistopic
  tools:
  - R
  - RcisTarget
  - cisTopic
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# transcription-factor-motif-enrichment-analysis

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Identifies master regulator transcription factors of cisTopic topics by running RcisTarget motif-enrichment analysis on the binarized, topic-enriched regions (e.g. PBMC topics), ranking TFs by normalized enrichment score (NES) against an hg19 region-based ranking database.

## When to use

Use when you have binarized topic-enriched regions from a cisTopic model on single-cell ATAC-seq data and want to recover topic-specific TF cistromes and master regulators such as SPI1 and CEBPB.

## When NOT to use

- The input is raw fragments/peaks and no cisTopic topic model has been fit yet — run the LDA topic modeling and binarizecisTopics steps first.
- Your regions are from a genome assembly other than what the available RcisTarget feather databases cover (cisTopic's current version provides hg19 databases; mm10/hg38 require separate vignette workflows).
- You need de novo motif discovery rather than recovery of known TF motifs via enrichment over a precomputed ranking database.

## Inputs

- Precomputed cisTopic object (Rds) with a selected topic model, e.g. cisTopicObject_pbmc.Rds (30-topic model on 5,335 PBMCs with 97k regulatory regions)
- RcisTarget feather ranking database, e.g. hg19-regions-9species.all_regions.mc8nr.feather (hg19, 9 species, mc8nr)
- Binarized topic region sets from binarizecisTopics (GammaFit, thrP=0.975)

## Outputs

- Per-topic motif enrichment tables with TF, NES, motif and enrichment metrics (cisTopicObject@binarized.RcisTarget)
- Ranked master-regulator table per topic, e.g. master_regulators.csv (topic, TF, NES, motif, enrichment metrics)
- Topic-specific cistromes (directly and indirectly annotated, *_extended) via getCistromes
- Updated cisTopic object saved back to Rds

## How to apply

Load the precomputed cisTopic object (e.g. cisTopicObject_pbmc.Rds) and binarize its topics with binarizecisTopics (GammaFit method, thrP=0.975) to select the top regions per topic. Convert these binarized topic regions to the equivalent database region space with binarizedcisTopicsToCtx (genome='hg19'), allowing a region to map to more than one ctx region. Run the topicsRcisTarget wrapper against the hg19-regions-9species.all_regions.mc8nr.feather ranking database with nesThreshold=3, rocthr=0.005, maxRank=20000 and nCores=5 (reduced_database=FALSE) to derive per-topic motif enrichment tables stored in the object's binarized.RcisTarget slot. Rank transcription factors by normalized enrichment score (NES) using the high-confidence motif annotation, then optionally build topic-specific cistromes with getCistromes (annotation='Both', nCores=5) so that *_extended cistromes include both directly and indirectly annotated motifs. Master regulators are the top-NES, high-confidence TFs per cell-type-specific topic — in the 5k PBMC 30-topic model this recovers SPI1 for the B cell topic and CEBPB for the monocyte topic.

## Related tools

- **cisTopic** (Provides binarizecisTopics, binarizedcisTopicsToCtx, topicsRcisTarget and getCistromes to go from topic-enriched regions to topic-specific TF cistromes and master regulators) — https://github.com/aertslab/cistopic
- **RcisTarget** (Performs the motif-enrichment scoring (NES, high-confidence annotation) of topic regions against the hg19 feather ranking database) — https://github.com/aertslab/RcisTarget
- **R** (Environment in which the cisTopic/RcisTarget workflow is executed)

## Examples

```
cisTopicObject <- readRDS('cisTopicObject_pbmc.Rds')
```

## Evaluation signals

- master_regulators.csv exists in CSV format and contains SPI1 and CEBPB among the top-ranked regulators of the cell-type-specific topics, consistent with the paper's reported finding.
- The saved cisTopic object's per-topic RcisTarget tables (e.g. @binarized.RcisTarget[[1]] and [[3]]) recover SPI1 and CEBPB as the master regulators of the B cell and monocyte topics, respectively.
- Motifs pass the enrichment filters used in the wrapper: NES threshold 3, rocthr 0.005, and maxRank 20000 against the mc8nr hg19 database.
- Binarized ctx region conversion succeeded, i.e. every binarized topic region maps to the RcisTarget database region space (genome='hg19').
- Ranked motifs carry a high-confidence annotation before TFs are considered master regulators.

## Limitations

- cisTopic is deprecated in favor of pycisTopic, so this R workflow may not be actively maintained.
- RcisTarget scoring depends on the database scope: only regions present in the hg19 feather database (maxRank 20000) can be scored; regions outside it are excluded via the ctx conversion.
- Master-regulator calls are indirect motif-based inferences; getCistromes with annotation='Both' mixes directly and indirectly annotated motifs in *_extended cistromes.

## Evidence

- [readme] RcisTarget feather databases location: [evidence span withheld — anchor status: external_doc]
