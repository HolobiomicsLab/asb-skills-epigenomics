---
name: region-to-gene-annotation
description: Use when you have topic-defined region sets (e.g. from binarizecisTopics
  on a 30-topic PBMC scATAC-seq model) and need per-topic GO term enrichment to functionally
  characterize topics, such as distinguishing promoter topics from cell-type-specific
  enhancer topics.
when_to_use_negative:
- When the goal is motif/transcription-factor enrichment rather than GO term annotation
  — RcisTarget with region-based feather databases (e.g. hg19-regions-9species.all_regions.mc8nr.feather)
  is the appropriate skill
- When region-to-gene assignment must be TSS-distance-based rather than regulatory-domain-based,
  since GREAT assigns genes via basal regulatory domains and may not match nearest-gene
  annotations
- When using non-hg19 assemblies without checking support — the article's vignette
  for mm10/hg38 is separate, and the default GREAT workflow shown uses hg19
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3125
tools:
- name: cisTopic
  role: R/Bioconductor package providing the GREAT() wrapper that runs rGREAT on binarized
    topic region sets and stores per-topic GO enrichment in @binarized.rGREAT
  repo: http://github.com/aertslab/cistopic
- name: rGREAT
  role: R client for the GREAT analysis server; performs GO term enrichment on the
    topic region sets with genome='hg19', fold_enrichment=2, geneHits=1, sign=0.05
- name: R
  role: Language in which cisTopic and the enrichment workflow are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-019-0367-1
    title: 'cisTopic: cis-regulatory topic modeling on single-cell ATAC-seq data'
  source_content_hash: sha256:26a4200a106972b0603fe738224c2e7db98f7155d041e0711e068263125abeab
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - R
  - cisTopic
  - rGREAT
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3125
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# region-to-gene-annotation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Annotates cisTopic-derived genomic regions with functional gene terms by running GREAT (via rGREAT) on the binarized region sets of each topic, using the hg19 genome, to assign biological function (e.g. promoter vs. cell-type-specific enhancer character) to topics.

## When to use

Use when you have topic-defined region sets (e.g. from binarizecisTopics on a 30-topic PBMC scATAC-seq model) and need per-topic GO term enrichment to functionally characterize topics, such as distinguishing promoter topics from cell-type-specific enhancer topics.

## When NOT to use

- When the goal is motif/transcription-factor enrichment rather than GO term annotation — RcisTarget with region-based feather databases (e.g. hg19-regions-9species.all_regions.mc8nr.feather) is the appropriate skill
- When region-to-gene assignment must be TSS-distance-based rather than regulatory-domain-based, since GREAT assigns genes via basal regulatory domains and may not match nearest-gene annotations
- When using non-hg19 assemblies without checking support — the article's vignette for mm10/hg38 is separate, and the default GREAT workflow shown uses hg19

## Inputs

- cisTopic object RDS file (cisTopicObject_pbmc.Rds) with a selected 30-topic LDA model
- Binarized topic region sets produced by binarizecisTopics() with GammaFit thresholds (thrP=0.975)
- Genome assembly specification (hg19) for the GREAT/rGREAT analysis

## Outputs

- Per-topic enriched GO terms stored in cisTopicObject@binarized.rGREAT
- CSV table of enriched GO terms per topic (great_go_enrichment_per_topic.csv), filtered at binomial adjusted p-value <= 0.05 and fold enrichment >= 2

## How to apply

Load the precomputed cisTopic object (cisTopicObject_pbmc.Rds) containing the selected 30-topic model, then binarize the cisTopics with binarizecisTopics() using GammaFit thresholds (thrP=0.975) so each topic becomes a discrete set of top regions. Run the cisTopic GREAT() wrapper, which calls rGREAT with genome='hg19', fold_enrichment=2, geneHits=1, sign=0.05, and request_interval=10 (reference runtime ~30 min). GREAT's rationale is to assign regulatory regions to genes based on the basal regulatory domain rather than distance-to-TSS, which suits enhancer sets lacking direct gene assignments. Extract the per-topic enriched GO terms stored in object@binarized.rGREAT and export them as a CSV table (great_go_enrichment_per_topic.csv). Interpretation follows the paper: topics whose regions are not enriched for cell-type-specific signatures (from signatureCellEnrichment) are promoter topics, i.e. their regions belong to promoters rather than cell-type-specific enhancers.

## Related tools

- **cisTopic** (R/Bioconductor package providing the GREAT() wrapper that runs rGREAT on binarized topic region sets and stores per-topic GO enrichment in @binarized.rGREAT) — http://github.com/aertslab/cistopic
- **rGREAT** (R client for the GREAT analysis server; performs GO term enrichment on the topic region sets with genome='hg19', fold_enrichment=2, geneHits=1, sign=0.05)
- **R** (Language in which cisTopic and the enrichment workflow are executed)

## Examples

```
cisTopicObject <- GREAT(cisTopicObject, genome='hg19', fold_enrichment=2, geneHits=1, sign=0.05, request_interval=10)
```

## Evaluation signals

- The output CSV contains enriched GO terms for each of the 30 topics, with every reported term satisfying binomial adjusted p-value <= 0.05 and fold enrichment >= 2
- Binarization yields exactly one discrete region set per topic (30 sets), each derived with thrP=0.975 GammaFit thresholds
- Topics previously identified as not enriched for any cell-type-specific signature (via signatureCellEnrichment) correspond to promoter-enriched topics in the GO results
- The @binarized.rGREAT slot of the cisTopic object is populated with rGREAT results for all binarized topics after the ~30 min run completes
- Genome mismatch errors or empty enrichment results indicate the hg19 setting does not match the region coordinates in the cisTopic object

## Limitations

- cisTopic is deprecated in favor of pycisTopic, so this R-based GREAT wrapper workflow is legacy; new analyses should use pycisTopic
- rGREAT queries an external web server, so results depend on request_interval pacing (request_interval=10) and service availability; the reference runtime is ~30 min
- The default GREAT call is genome-specific (hg19); running with mm10 or hg38 requires the dedicated vignette workflow ('Running GREAT and motif enrichment with the mm10 and hg38 genome assemblies')

## Evidence
