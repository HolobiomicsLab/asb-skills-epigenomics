---
name: genomic-region-to-ctx-region-mapping
description: Use when binarized topic or cluster region sets from single-cell ATAC-seq
  (e.g. cisTopic topics) must be converted to the region identifiers of a cistarget/RcisTarget
  database (e.g. hg19-regions-9species) before running motif enrichment.
when_to_use_negative:
- The dataset regions were already defined directly in the cistarget database region
  space, so no coordinate conversion is needed
- You only need topic region scores/rankings within cells (use getRegionsScores or
  signatureCellEnrichment) rather than motif enrichment
- The database is for a different genome assembly than the binarized topics (e.g.
  mm10 or hg38 topics against an hg19 database)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0749
tools:
- name: cisTopic
  role: Provides binarizedcisTopicsToCtx() to convert binarized topic regions to the
    equivalent RcisTarget database (ctx) regions
  repo: https://github.com/aertslab/cisTopic
- name: RcisTarget
  role: Defines the target ctx region space via its feather databases; downstream
    consumer of the mapped regions for motif enrichment
  repo: https://github.com/aertslab/RcisTarget
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-019-0367-1
    title: 'cisTopic: cis-regulatory topic modeling on single-cell ATAC-seq data'
  source_content_hash: sha256:f6f4359435587a109e6c4f6d7d577eaf9b45435dea3cf993dc8bade9102cbbe8
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/aertslab/cisTopic
  tools:
  - R
  - RcisTarget
  - cisTopic
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# Genomic region to cistarget (ctx) region mapping

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Maps binarized cisTopic topic regions onto the region coordinate space of an RcisTarget feather database so that motif enrichment can be computed against database rankings. A single dataset region may map to multiple ctx regions.

## When to use

Use when binarized topic or cluster region sets from single-cell ATAC-seq (e.g. cisTopic topics) must be converted to the region identifiers of a cistarget/RcisTarget database (e.g. hg19-regions-9species) before running motif enrichment.

## When NOT to use

- The dataset regions were already defined directly in the cistarget database region space, so no coordinate conversion is needed
- You only need topic region scores/rankings within cells (use getRegionsScores or signatureCellEnrichment) rather than motif enrichment
- The database is for a different genome assembly than the binarized topics (e.g. mm10 or hg38 topics against an hg19 database)

## Inputs

- cisTopic object with binarized topics (e.g. from binarizecisTopics with GammaFit, thrP=0.975), stored in the object's binarized slot
- Genome assembly identifier matching the cistarget database ('hg19')
- Reference RcisTarget feather database (e.g. hg19-regions-9species.all_regions.mc8nr.feather)

## Outputs

- Binarized ctx regions per topic, coordinate-compatible with the RcisTarget database regions
- cisTopic object updated with the ctx-mapped binarized topics

## How to apply

After binarizing topics with binarizecisTopics (e.g. GammaFit method, thrP=0.975) to obtain top-region sets per topic, run binarizedcisTopicsToCtx with the genome matching the RcisTarget database (genome='hg19' for the hg19-regions-9species.all_regions.mc8nr.feather database). This step is necessary because the dataset's peak regions (e.g. the 97k regulatory regions from 10X 5k PBMCs) do not exactly coincide with the database's region definitions. The mapping is many-to-many: a region can map to more than one ctx region, so downstream enrichment uses the expanded region set. The output binarized ctx regions become the input to topicsRcisTarget (nesThreshold=3, rocthr=0.005, maxRank=20000).

## Related tools

- **cisTopic** (Provides binarizedcisTopicsToCtx() to convert binarized topic regions to the equivalent RcisTarget database (ctx) regions) — https://github.com/aertslab/cisTopic
- **RcisTarget** (Defines the target ctx region space via its feather databases; downstream consumer of the mapped regions for motif enrichment) — https://github.com/aertslab/RcisTarget

## Examples

```
cisTopicObject <- binarizedcisTopicsToCtx(cisTopicObject, genome='hg19')
```

## Evaluation signals

- Each binarized topic yields a non-empty set of ctx regions after mapping (no topic silently maps to zero regions)
- Mapped ctx regions lie on the same genome assembly as the feather database (hg19 coordinates)
- Downstream topicsRcisTarget runs without 'region not found in database' errors, confirming coordinate compatibility
- Per-topic RcisTarget tables recover known master regulators (SPI1 for the B cell topic, CEBPB for the monocyte topic), showing the mapping preserved biologically relevant regions
- The updated cisTopic object can be saved and reloaded (readRDS) with the mapping intact

## Limitations

- The mapping is not one-to-one: a dataset region can map to more than one ctx region, which expands region sets and must be accounted for in downstream enrichment
- cisTopic is deprecated in favor of pycisTopic, so this R-based mapping workflow is legacy
- Region mapping depends on exact overlap between dataset peaks and database region definitions; poorly overlapping peaks may be lost

## Evidence

- [methods] Convert the binarized topic regions to the equivalent RcisTarget database regions with binarizedcisTopicsToCtx (genome='hg19'), allowing a region to map to more than one ctx region.: [evidence span withheld — anchor status: not_found]
- [methods] To do this, we use `RcisTarget` (Aibar et al., 2017). The current version provides databases for human (hg19).: "To do this, we use `RcisTarget` (Aibar et al., 2017). The current version provides databases for human (hg19)."
- [methods] You can find the region-based database at: https://resources.aertslab.org/cistarget/: "You can find the region-based database at: https://resources.aertslab.org/cistarget/"
- [readme] cisTopic is deprecated, use [pycisTopic](https://github.com/aertslab/pycisTopic) instead.: "cisTopic is deprecated, use [pycisTopic](https://github.com/aertslab/pycisTopic) instead."
