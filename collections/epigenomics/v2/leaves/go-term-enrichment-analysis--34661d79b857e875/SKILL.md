---
name: go-term-enrichment-analysis
description: Use when you have a collection of region sets (e.g. binarized cisTopics
  from scATAC-seq) and need per-set GO functional annotation via rGREAT, or when you
  need to identify which topics lack cell-type-signature enrichment.
when_to_use_negative:
- The input is already a gene list rather than a set of genomic regions — standard
  gene-set over-representation tools are more appropriate than GREAT's region-based
  test.
- The regions come from a genome assembly other than the one the GREAT query is configured
  for (the wrapper shown uses genome='hg19'), without adjusting the genome argument.
- The goal is topic model construction or cell-type assignment (runModels/selectModel/signatureCellEnrichment),
  not functional annotation of already-selected region sets.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0089
tools:
- name: cisTopic
  role: R/Bioconductor package providing binarizecisTopics() (GammaFit thresholds,
    thrP=0.975) and the GREAT() wrapper that submits each binarized topic's regions
    to rGREAT and stores results in object@binarized.rGREAT
  repo: http://github.com/aertslab/cistopic
- name: rGREAT
  role: Backend engine performing GREAT Gene Ontology enrichment on the binarized
    region sets (hg19, fold_enrichment=2, geneHits=1, sign=0.05)
- name: R
  role: Language in which cisTopic is implemented and the whole workflow (load Rds,
    binarize, GREAT, write CSV) is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-019-0367-1
    title: 'cisTopic: cis-regulatory topic modeling on single-cell ATAC-seq data'
  source_content_hash: sha256:5bdc53e5fda3ac06f97d16ebe39be4033b0b269e6cbccddc55691d9f5d7090bb
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - R
  - cisTopic
  - rGREAT
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# go-term-enrichment-analysis

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Performs GREAT-based Gene Ontology enrichment on genomic region sets, as applied to cisTopic topics from single-cell ATAC-seq data: each binarized topic (a set of top co-accessible regions) is tested for over-represented GO terms against the hg19 background. In the cisTopic workflow this characterizes the function of each cis-regulatory topic, e.g. distinguishing promoter topics from cell-type-specific enhancer topics.

## When to use

Use when you have a collection of region sets (e.g. binarized cisTopics from scATAC-seq) and need per-set GO functional annotation via rGREAT, or when you need to identify which topics lack cell-type-signature enrichment.

## When NOT to use

- The input is already a gene list rather than a set of genomic regions — standard gene-set over-representation tools are more appropriate than GREAT's region-based test.
- The regions come from a genome assembly other than the one the GREAT query is configured for (the wrapper shown uses genome='hg19'), without adjusting the genome argument.
- The goal is topic model construction or cell-type assignment (runModels/selectModel/signatureCellEnrichment), not functional annotation of already-selected region sets.

## Inputs

- cisTopic object (.Rds, e.g. cisTopicObject_pbmc.Rds) with a selected topic model (30 topics for the 5k PBMC dataset)
- Binarized cisTopics (region sets per topic produced by binarizecisTopics() with thrP=0.975)
- Genome assembly specification (hg19) for GREAT region-to-gene mapping

## Outputs

- great_go_enrichment_per_topic.csv with enriched GO terms (binomial adjusted p-value <= 0.05, fold enrichment >= 2) for each of the 30 topics
- Per-topic enriched GO term tables stored in cisTopicObject@binarized.rGREAT

## How to apply

Load the precomputed cisTopic object (e.g. cisTopicObject_pbmc.Rds containing the selected 30-topic LDA model) and binarize each cisTopic with binarizecisTopics() using GammaFit thresholds (thrP=0.975) so that every topic becomes a discrete set of top regions. Then run the cisTopic GREAT() wrapper over rGREAT with genome='hg19', fold_enrichment=2, geneHits=1, sign=0.05 and request_interval=10; the wrapper submits each binarized region set to GREAT's region-based ontology enrichment, which accounts for the regulatory-domain background rather than a simple gene-list test. Results are stored in object@binarized.rGREAT per topic; export them to great_go_enrichment_per_topic.csv. Interpretation point: topics not enriched for cell-type-specific signatures (e.g. the Corces et al. (2016) bulk hematopoietic ATAC signatures) are promoter topics — their regions map to promoters rather than cell-type-specific enhancers.

## Related tools

- **cisTopic** (R/Bioconductor package providing binarizecisTopics() (GammaFit thresholds, thrP=0.975) and the GREAT() wrapper that submits each binarized topic's regions to rGREAT and stores results in object@binarized.rGREAT) — http://github.com/aertslab/cistopic
- **rGREAT** (Backend engine performing GREAT Gene Ontology enrichment on the binarized region sets (hg19, fold_enrichment=2, geneHits=1, sign=0.05))
- **R** (Language in which cisTopic is implemented and the whole workflow (load Rds, binarize, GREAT, write CSV) is executed)

## Examples

```
cisTopicObject <- GREAT(cisTopicObject, genome='hg19', fold_enrichment=2, geneHits=1, sign=0.05, request_interval=10)
```

## Evaluation signals

- The output table great_go_enrichment_per_topic.csv contains enriched GO terms (binomial adjusted p-value <= 0.05, fold enrichment >= 2) for each of the 30 topics.
- All 30 topics are represented in object@binarized.rGREAT after the GREAT() call — no topic silently failed the web request (request_interval=10 guards rate limits).
- Binarization check: each topic's region set is finite and reproducible given GammaFit thresholds with thrP=0.975.
- Biological invariant: topics not enriched for any cell-type-specific signatures correspond to promoter topics (regions belong to promoters rather than cell-type-specific enhancers).
- Reference runtime ~30 min for the full GREAT run; dramatically shorter times may indicate requests failed or region sets were empty.

## Limitations

- cisTopic is deprecated in favor of pycisTopic; the R wrapper and its GREAT() function may no longer be actively maintained.
- GREAT runs as an online service (request_interval=10), so runs depend on service availability and rate limits; reference runtime is ~30 minutes.
- The enrichment is genome-specific (hg19 in the card, including hg19-regions-9species.all_regions.mc8nr.feather-style databases); other assemblies require different settings (a separate vignette covers mm10 and hg38).
- Enriched GO terms depend on the binarization threshold; with thrP=0.975, only the top regions per topic define each region set.

## Evidence
