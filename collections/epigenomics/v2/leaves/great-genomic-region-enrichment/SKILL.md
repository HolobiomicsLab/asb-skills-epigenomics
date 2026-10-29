---
name: great-genomic-region-enrichment
description: Use when a set of genomic regions (e.g. binarized cisTopics from single-cell
  ATAC-seq on hg19) needs GO functional annotation, or when interpreting which topics
  correspond to promoters vs cell-type-specific enhancers.
when_to_use_negative:
- The input is already a gene list or feature table rather than a set of genomic regions
  — GREAT specifically tests cis-regulatory region enrichment against regulatory domains
- cisTopics have not been binarized (no top-region sets defined), since GREAT() is
  run on the binarized topics in @binarized.rGREAT
- Working outside the cisTopic framework entirely (e.g. arbitrary peak sets with different
  annotation needs); note also that cisTopic itself is deprecated in favor of pycisTopic
edam_operation: http://edamontology.org/operation_3753
tools:
- name: cisTopic
  role: Provides binarizecisTopics() to derive top-region sets per topic and a GREAT()
    wrapper that submits them to rGREAT and stores results in @binarized.rGREAT
  repo: http://github.com/aertslab/cisTopic
- name: rGREAT
  role: R client performing the GREAT GO enrichment test on the binarized topic region
    sets against hg19 regulatory domains
- name: R
  role: Implementation language for the cisTopic/RGREAT enrichment workflow and CSV
    export
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-019-0367-1
    title: 'cisTopic: cis-regulatory topic modeling on single-cell ATAC-seq data'
  source_content_hash: sha256:fbbcd2cffb337548f40b09140800be9c0e8e096424ef6c10226279377e459f1e
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - R
  - cisTopic
  - rGREAT
  edam_operation: http://edamontology.org/operation_3753
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# great-genomic-region-enrichment

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Per-topic functional annotation of cisTopic regions via GREAT GO enrichment (through the rGREAT R package), which tests whether topic regions are enriched near GO genes accounting for regulatory-domain background. In the 5k PBMC case it reveals that topics lacking cell-type-specific signature enrichment are promoter topics.

## When to use

Use when a set of genomic regions (e.g. binarized cisTopics from single-cell ATAC-seq on hg19) needs GO functional annotation, or when interpreting which topics correspond to promoters vs cell-type-specific enhancers.

## When NOT to use

- The input is already a gene list or feature table rather than a set of genomic regions — GREAT specifically tests cis-regulatory region enrichment against regulatory domains
- cisTopics have not been binarized (no top-region sets defined), since GREAT() is run on the binarized topics in @binarized.rGREAT
- Working outside the cisTopic framework entirely (e.g. arbitrary peak sets with different annotation needs); note also that cisTopic itself is deprecated in favor of pycisTopic

## Inputs

- cisTopic object (.Rds) with a selected topic model, e.g. cisTopicObject_pbmc.Rds (30 topics, 10X 5k PBMC regions)
- Binarized topics (top regions per topic) obtained via binarizecisTopics() with GammaFit thresholds (thrP=0.975)
- Genome assembly specification for rGREAT (hg19 for the PBMC 10X workflow)

## Outputs

- Per-topic enriched GO terms stored in cisTopicObject@binarized.rGREAT (binomial adjusted p-value <= 0.05, fold enrichment >= 2)
- CSV table of GREAT GO enrichment per topic, e.g. great_go_enrichment_per_topic.csv
- Functional interpretation: promoter topics vs cell-type-specific enhancer topics

## How to apply

Load the precomputed cisTopic object (e.g. cisTopicObject_pbmc.Rds with the selected 30-topic model) and binarize each cisTopic with binarizecisTopics() using GammaFit thresholds (thrP=0.975) so that every topic is reduced to a set of top regions. Then run the cisTopic GREAT() wrapper around rGREAT with genome='hg19', fold_enrichment=2, geneHits=1, sign=0.05, and request_interval=10 (reference runtime ~30 min), which submits each topic's region set to GREAT and tests GO term enrichment against regulatory domains. The enrichment results are stored in the object's @binarized.rGREAT slot; extract the per-topic GO terms (binomial adjusted p-value <= 0.05, fold enrichment >= 2) and write them to a CSV table such as great_go_enrichment_per_topic.csv. Interpret topics together with signature enrichment results: regions in topics not enriched for any cell-type-specific signatures (e.g. the Corces et al. 2016 bulk hematopoietic ATAC signatures) are promoter topics rather than cell-type-specific enhancers.

## Related tools

- **cisTopic** (Provides binarizecisTopics() to derive top-region sets per topic and a GREAT() wrapper that submits them to rGREAT and stores results in @binarized.rGREAT) — http://github.com/aertslab/cisTopic
- **rGREAT** (R client performing the GREAT GO enrichment test on the binarized topic region sets against hg19 regulatory domains)
- **R** (Implementation language for the cisTopic/RGREAT enrichment workflow and CSV export)

## Examples

```
cisTopicObject <- GREAT(cisTopicObject, genome='hg19', fold_enrichment=2, geneHits=1, sign=0.05, request_interval=10)
```

## Evaluation signals

- The output table contains enriched GO terms for each of the 30 topics, filtered at binomial adjusted p-value <= 0.05 and fold enrichment >= 2
- The cisTopic object's @binarized.rGREAT slot is populated after the GREAT() call (reference runtime ~30 min at request_interval=10)
- Topics previously found not enriched for any cell-type-specific signatures (Corces et al. 2016 hematopoietic signatures) show promoter-associated GO terms rather than enhancer/cell-type terms
- Each binarized topic yields a non-empty region set from binarizecisTopics() with thrP=0.975 before submission
- The exported great_go_enrichment_per_topic.csv has one entry set per topic id with GO term IDs and enrichment statistics

## Limitations

- cisTopic is deprecated; use pycisTopic instead
- The wrapper is demonstrated with genome='hg19'; running GREAT with other assemblies (mm10, hg38) is covered by a separate cisTopic v3 vignette and may require adjusted settings
- GREAT results depend on the binarization thresholds: changing thrP (0.975) changes the region sets submitted and hence the enrichment outcomes
- rGREAT submits jobs to a web service, hence the request_interval=10 parameter and the ~30 min runtime; network failures can interrupt the analysis

## Evidence

- [readme] README documents GREAT vignettes for other genome assemblies: [evidence span withheld — anchor status: external_doc]
