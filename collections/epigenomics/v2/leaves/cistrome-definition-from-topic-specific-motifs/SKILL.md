---
name: cistrome-definition-from-topic-specific-motifs
description: Use when a cisTopic object with a selected topic model (e.g. 30 topics
  on 10X 5k PBMCs) is available and per-topic TF cistromes or master regulators must
  be identified from topic-enriched regions.
when_to_use_negative:
- Input regions are already a feature table or gene sets — RcisTarget needs genomic
  regions ranked against a motif-ranking feather database, not precomputed enrichment
  scores.
- The cisTopic object has no selected model yet — first run runModels and selectModel
  before binarizing topics for cistrome construction.
- Working with mm10 or hg38 without the corresponding RcisTarget databases converted
  for your region space — the workflow here is anchored to hg19-regions-9species.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0797
tools:
- name: cisTopic
  role: R/Bioconductor package providing binarizecisTopics, binarizedcisTopicsToCtx,
    topicsRcisTarget and getCistromes to move from topic model to per-topic cistromes
  repo: http://github.com/aertslab/cistopic
- name: RcisTarget
  role: Performs motif enrichment analysis on the binarized topic regions using the
    hg19-regions-9species feather database (nesThreshold=3, maxRank=20000)
  repo: https://github.com/aertslab/RcisTarget
- name: R
  role: Language in which cisTopic and RcisTarget are executed as R/Bioconductor packages
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-019-0367-1
    title: 'cisTopic: cis-regulatory topic modeling on single-cell ATAC-seq data'
  source_content_hash: sha256:8ca191cf51aca09d8509e26d30aba55a158f63fa44c631e5e7b51230567e8f25
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - R
  - RcisTarget
  - cisTopic
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Cistrome definition from topic-specific motifs

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Derive topic-specific transcription-factor cistromes from a cisTopic model by running RcisTarget motif enrichment on the binarized top regions of each topic and annotating the enriched motifs back to regions. This connects Latent-Dirichlet-Allocation topics from single-cell ATAC-seq to candidate master regulators such as SPI1 (B cell topic) and CEBPB (monocyte topic).

## When to use

Use when a cisTopic object with a selected topic model (e.g. 30 topics on 10X 5k PBMCs) is available and per-topic TF cistromes or master regulators must be identified from topic-enriched regions.

## When NOT to use

- Input regions are already a feature table or gene sets — RcisTarget needs genomic regions ranked against a motif-ranking feather database, not precomputed enrichment scores.
- The cisTopic object has no selected model yet — first run runModels and selectModel before binarizing topics for cistrome construction.
- Working with mm10 or hg38 without the corresponding RcisTarget databases converted for your region space — the workflow here is anchored to hg19-regions-9species.

## Inputs

- Precomputed cisTopic object with a selected topic model (cisTopicObject_pbmc.Rds)
- RcisTarget region-based feather database: hg19-regions-9species.all_regions.mc8nr.feather
- Genome assembly designation (hg19)

## Outputs

- Per-topic motif enrichment tables stored in the cisTopicObject's binarized.RcisTarget slot
- Topic-specific cistromes (region sets annotated to enriched motifs), including *_extended versions
- Master-regulator calls per topic (e.g. SPI1 for the B cell topic, CEBPB for the monocyte topic)
- Updated cisTopic object saved to disk as .Rds

## How to apply

Load the precomputed cisTopic object (e.g. cisTopicObject_pbmc.Rds) and binarize topics with binarizecisTopics using the GammaFit method with thrP=0.975 to obtain the top-region set per topic. Convert those regions to the equivalent RcisTarget database regions with binarizedcisTopicsToCtx (genome='hg19'), allowing one region to map to multiple ctx regions. Run topicsRcisTarget against hg19-regions-9species.all_regions.mc8nr.feather with nesThreshold=3, rocthr=0.005, maxRank=20000 and nCores=5 to compute per-topic motif enrichment tables, which are stored in the object's binarized.RcisTarget slot. Finally build topic-specific cistromes with getCistromes using annotation='Both' so that *_extended cistromes include both directly and indirectly annotated motifs; with nCores=5. The rationale is that motifs enriched in a topic's top regions identify the TFs whose binding sites define that cell state's regulatory landscape.

## Related tools

- **cisTopic** (R/Bioconductor package providing binarizecisTopics, binarizedcisTopicsToCtx, topicsRcisTarget and getCistromes to move from topic model to per-topic cistromes) — http://github.com/aertslab/cistopic
- **RcisTarget** (Performs motif enrichment analysis on the binarized topic regions using the hg19-regions-9species feather database (nesThreshold=3, maxRank=20000)) — https://github.com/aertslab/RcisTarget
- **R** (Language in which cisTopic and RcisTarget are executed as R/Bioconductor packages)

## Examples

```
cisTopicObject <- topicsRcisTarget(cisTopicObject, database='hg19-regions-9species.all_regions.mc8nr.feather', nesThreshold=3, rocthr=0.005, maxRank=20000, nCores=5); cisTopicObject <- getCistromes(cisTopicObject, annotation='Both', nCores=5)
```

## Evaluation signals

- The saved .Rds file exists and, when reloaded, contains populated per-topic RcisTarget tables in the binarized.RcisTarget slot.
- The motif enrichment tables for the B cell and monocyte topics recover SPI1 and CEBPB as master regulators, respectively.
- Each topic in the object yields its own enrichment table and cistrome after getCistromes with annotation='Both'.
- binarizedcisTopicsToCtx reports a mapping in which a region may map to more than one ctx region — a null or one-to-one-only mapping indicates a region-space mismatch.
- Interactive per-topic motif enrichment tables (e.g. Topics 1 and 3) can be inspected without errors.

## Limitations

- cisTopic (R) is deprecated in favor of pycisTopic, so long-term maintenance of this workflow is limited.
- Only regions that can be mapped to the RcisTarget ctx region space (hg19-regions-9species) contribute to the enrichment; unmappable regions are lost.
- Results depend on the binarization threshold (GammaFit, thrP=0.975) and enrichment cutoffs (nesThreshold=3, rocthr=0.005, maxRank=20000); changing them changes cistrome membership.

## Evidence

- [other] Applying RcisTarget motif enrichment to topic regions yields topic-specific TF cistromes, revealing SPI1 and CEBPB as master regulators of the B cell and monocyte topics, respectively.: [evidence span withheld — anchor status: not_found]
- [methods] To do this, we use `RcisTarget` (Aibar et al., 2017). The current version provides databases for human (hg19).: "To do this, we use `RcisTarget` (Aibar et al., 2017). The current version provides databases for human (hg19)."
- [readme] RcisTarget feather databases are available at https://resources.aertslab.org/cistarget/: [evidence span withheld — anchor status: external_doc]
- [intro] cisTopic is an R-package to simultaneously identify cell states and cis-regulatory topics from single cell epigenomics data.: "cisTopic is an R-package to simultaneously identify cell states and cis-regulatory topics from single cell epigenomics data."
