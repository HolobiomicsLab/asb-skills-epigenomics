---
name: hg19-motif-ranking-database-usage
description: Use when binarized cisTopic topic regions from a human (hg19) single-cell
  ATAC-seq dataset must be annotated with enriched TF motifs to identify master regulators
  per topic.
when_to_use_negative:
- The scATAC-seq regions are from a non-human genome assembly (e.g. mm10 or hg38)
  — the vignette notes mm10/hg38 support requires a separate motif-enrichment workflow,
  and this feather database is hg19-specific
- You need de novo motif discovery rather than enrichment against a precomputed TF
  motif ranking database
- cisTopic topics have not yet been binarized into top-region sets — run binarizecisTopics
  (GammaFit, thrP=0.975) first
edam_operation: http://edamontology.org/operation_0445
edam_topics:
- http://edamontology.org/topic_0797
tools:
- name: RcisTarget
  role: Performs motif-enrichment analysis on ctx topic regions using the hg19 feather
    ranking database to yield per-topic TF NES rankings
  repo: https://github.com/aertslab/RcisTarget
- name: cisTopic
  role: Provides the binarized topic regions and the topicsRcisTarget wrapper that
    interfaces with RcisTarget
  repo: https://github.com/aertslab/cistopic
- name: R
  role: Runtime environment in which cisTopic and RcisTarget are installed and executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-019-0367-1
    title: 'cisTopic: cis-regulatory topic modeling on single-cell ATAC-seq data'
  source_content_hash: sha256:111b30d48ba5179236b2aa1490d124150d5cecdeb22b51bc8a219e4a25099516
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/aertslab/RcisTarget
  tools:
  - R
  - RcisTarget
  - cisTopic
  edam_operation: http://edamontology.org/operation_0445
  edam_topics:
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# hg19-motif-ranking-database-usage

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Applies the hg19 region-based motif ranking database (hg19-regions-9species.all_regions.mc8nr.feather, mc8nr) through RcisTarget to score transcription-factor motif enrichment in cisTopic binarized topic regions and recover master regulators of cell-type-specific topics.

## When to use

Use when binarized cisTopic topic regions from a human (hg19) single-cell ATAC-seq dataset must be annotated with enriched TF motifs to identify master regulators per topic.

## When NOT to use

- The scATAC-seq regions are from a non-human genome assembly (e.g. mm10 or hg38) — the vignette notes mm10/hg38 support requires a separate motif-enrichment workflow, and this feather database is hg19-specific
- You need de novo motif discovery rather than enrichment against a precomputed TF motif ranking database
- cisTopic topics have not yet been binarized into top-region sets — run binarizecisTopics (GammaFit, thrP=0.975) first

## Inputs

- cisTopic object with binarized topics (e.g. cisTopicObject_pbmc.Rds)
- hg19-regions-9species.all_regions.mc8nr.feather region-based ranking database
- Binarized topic regions (ctx regions) for hg19

## Outputs

- Per-topic motif enrichment tables (cisTopicObject@binarized.RcisTarget slots)
- Ranked master-regulator table (topic, TF, NES, motif, enrichment metrics), e.g. master_regulators.csv

## How to apply

First convert binarized topics (top regions per topic selected via GammaFit thresholding with thrP=0.975) to ctx regions with binarizedcisTopicsToCtx using genome='hg19'. Then run the topicsRcisTarget wrapper against the hg19-regions-9species.all_regions.mc8nr.feather ranking database downloaded from resources.aertslab.org/cistarget/, using reduced_database=FALSE, nesThreshold=3, rocthr=0.005, maxRank=20000 and nCores=5. Extract the per-topic motif enrichment tables (e.g. cisTopicObject@binarized.RcisTarget[[1]] and [[3]]) and rank transcription factors by normalized enrichment score (NES) with the high-confidence annotation. Validate by checking that known cell-type regulators — such as SPI1 for the B cell topic and CEBPB for the monocyte topic in the 10X 5k PBMC dataset — appear among the top-ranked master regulators.

## Related tools

- **RcisTarget** (Performs motif-enrichment analysis on ctx topic regions using the hg19 feather ranking database to yield per-topic TF NES rankings) — https://github.com/aertslab/RcisTarget
- **cisTopic** (Provides the binarized topic regions and the topicsRcisTarget wrapper that interfaces with RcisTarget) — https://github.com/aertslab/cistopic
- **R** (Runtime environment in which cisTopic and RcisTarget are installed and executed)

## Examples

```
cisTopicObject <- topicsRcisTarget(cisTopicObject, genome='hg19', pathToFeather='hg19-regions-9species.all_regions.mc8nr.feather', nesThreshold=3, rocthr=0.005, maxRank=20000, nCores=5)
```

## Evaluation signals

- master_regulators.csv exists in CSV format and contains SPI1 and CEBPB among the top-ranked regulators of the cell-type-specific topics
- SPI1 is recovered for the B cell topic and CEBPB for the monocyte topic, matching the paper's reported finding
- Enrichment output only retains motifs passing nesThreshold=3 and rocthr=0.005, with ranking scans capped at maxRank=20000
- Each topic in cisTopicObject@binarized.RcisTarget contains a motif table with NES and high-confidence annotations populated
- The feather database used is the 9-species, mc8nr hg19 regions database with reduced_database=FALSE (full database)

## Limitations

- cisTopic is deprecated; the README directs users to pycisTopic (installed via SCENIC+) for new analyses
- The current RcisTarget version highlighted here provides region-based databases for human hg19; other assemblies (mm10, hg38) require their own databases/workflow
- Master-regulator recovery depends on the binarization threshold (thrP=0.975) and motif-enrichment cutoffs; promoter topics not enriched for cell-type-specific signatures will not yield specific TFs

## Evidence

- [readme] README confirms RcisTarget feather databases as the download source: [evidence span withheld — anchor status: external_doc]
