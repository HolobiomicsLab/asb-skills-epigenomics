---
name: topic-region-binarization
description: Use when a fitted cisTopic object (e.g. the 30-topic 10X 5k PBMC model)
  must be reduced to per-topic region sets before running rGREAT GREAT() or topicsRcisTarget
  motif enrichment on hg19.
when_to_use_negative:
- The goal is cell-level analysis (e.g. cell state annotation or signatureCellEnrichment)
  rather than region-set-level enrichment, since binarization discards cell-topic
  and graded region-topic information
- No model has been selected yet — binarizecisTopics() requires a cisTopic object
  after runModels() and selectModel(), not raw fragment/peak inputs
- Working with the deprecated R cisTopic package for new analyses — the README states
  cisTopic is deprecated in favor of pycisTopic
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0749
tools:
- name: cisTopic
  role: Provides getRegionsScores() and binarizecisTopics() (GammaFit, thrP=0.975)
    to turn topic-region proportion scores into per-topic top-region sets
  repo: https://github.com/aertslab/cisTopic
- name: RcisTarget
  role: Consumes the binarized topic regions (converted with binarizedcisTopicsToCtx
    to hg19 feather-database regions) for per-topic motif enrichment
  repo: https://github.com/aertslab/RcisTarget
- name: rGREAT
  role: Consumes the binarized per-topic region sets for GREAT GO enrichment (cisTopic
    GREAT() wrapper, genome='hg19')
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41592-019-0367-1
    title: 'cisTopic: cis-regulatory topic modeling on single-cell ATAC-seq data'
  source_content_hash: sha256:9bf23d29de8905687b5d69340c8aaaa83b05ecc8b03d56c9e1297145f0d5bbd2
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/aertslab/cisTopic
  tools:
  - R
  - RcisTarget
  - cisTopic
  - rGREAT
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# topic-region-binarization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Converts the continuous region-topic proportion scores of a cisTopic model into discrete per-topic sets of top regions (binarized cisTopics) so that downstream region-set analyses such as GREAT GO enrichment and RcisTarget motif enrichment can be run on each topic.

## When to use

Use when a fitted cisTopic object (e.g. the 30-topic 10X 5k PBMC model) must be reduced to per-topic region sets before running rGREAT GREAT() or topicsRcisTarget motif enrichment on hg19.

## When NOT to use

- The goal is cell-level analysis (e.g. cell state annotation or signatureCellEnrichment) rather than region-set-level enrichment, since binarization discards cell-topic and graded region-topic information
- No model has been selected yet — binarizecisTopics() requires a cisTopic object after runModels() and selectModel(), not raw fragment/peak inputs
- Working with the deprecated R cisTopic package for new analyses — the README states cisTopic is deprecated in favor of pycisTopic

## Inputs

- cisTopic object with a selected topic model (e.g. cisTopicObject_pbmc.Rds, 30 topics on the 10X 5k PBMC dataset)
- Region-topic proportion scores (produced internally by getRegionsScores())

## Outputs

- Binarized cisTopics: a per-topic list of top region sets stored in the object's binarized.cisTopics slot
- ctx-mapped binarized topic regions (via binarizedcisTopicsToCtx, genome='hg19') for RcisTarget input

## How to apply

First compute region-specific topic assignment proportions with getRegionsScores(), which scores each region based on the proportion of its assignments to each topic. Then apply binarizecisTopics() using the GammaFit method with thrP=0.975 to select the top regions per topic, yielding the binarized.cisTopics slot. When preparing regions for RcisTarget, additionally convert the binarized topic regions to the feather-database region space with binarizedcisTopicsToCtx (genome='hg19'), noting that one region may map to more than one ctx region. The resulting per-topic region sets are the direct inputs to GREAT() (genome='hg19', fold_enrichment=2, geneHits=1, sign=0.05) and topicsRcisTarget (nesThreshold=3, rocthr=0.005, maxRank=20000).

## Related tools

- **cisTopic** (Provides getRegionsScores() and binarizecisTopics() (GammaFit, thrP=0.975) to turn topic-region proportion scores into per-topic top-region sets) — https://github.com/aertslab/cisTopic
- **RcisTarget** (Consumes the binarized topic regions (converted with binarizedcisTopicsToCtx to hg19 feather-database regions) for per-topic motif enrichment) — https://github.com/aertslab/RcisTarget
- **rGREAT** (Consumes the binarized per-topic region sets for GREAT GO enrichment (cisTopic GREAT() wrapper, genome='hg19'))

## Examples

```
cisTopicObject <- binarizecisTopics(cisTopicObject, method='GammaFit', thrP=0.975, plot=FALSE)
```

## Evaluation signals

- After binarizecisTopics() with GammaFit thrP=0.975, each of the 30 topics in binarized.cisTopics contains a non-empty set of top regions
- Downstream GREAT() on the binarized topics yields enriched GO terms (binomial adjusted p-value <= 0.05, fold enrichment >= 2) for each of the 30 topics in object@binarized.rGREAT
- Downstream topicsRcisTarget on the ctx-converted topics recovers SPI1 and CEBPB as master regulators of the B cell and monocyte topics
- Topics that are NOT enriched for cell-type-specific signatures should correspond to promoter topics, i.e. their binarized regions overlap promoters rather than cell-type-specific enhancers
- binarizedcisTopicsToCtx may map one region to more than one ctx region — check the ctx region sets are at least as large as the input binarized sets

## Limitations

- Binarization is threshold-dependent: the GammaFit thrP=0.975 cutoff defines which regions count as 'top' for each topic, and different thresholds change downstream enrichment results
- cisTopic (R) is deprecated in favor of pycisTopic, so this workflow reflects the legacy R/Bioconductor version
- The rGREAT-based GREAT() wrapper queries the GREAT web service (request_interval=10, reference runtime ~30 min for 30 topics), so it is slow and network-dependent
- Region-to-ctx mapping for RcisTarget is many-to-one, so binarized topic region sets are approximate in the database region space

## Evidence
