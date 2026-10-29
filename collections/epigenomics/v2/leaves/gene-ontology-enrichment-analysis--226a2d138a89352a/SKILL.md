---
name: gene-ontology-enrichment-analysis
description: Use when you have per-cluster up-/down-regulated gene lists from one-vs-rest
  SAM differential expression and need GO terms explaining the biology of each cluster.
when_to_use_negative:
- Input gene lists are not from a differential expression step — enrichment of unfiltered,
  genome-wide gene lists yields meaningless term over-representation
- Gene identifiers are not mappable to gene symbols (EnrichR module requires symbols;
  mygene 'scopes' must match the identifier format)
- The goal is de novo pathway discovery beyond GO — a customizable EnrichR database
  could be used instead, but the article only grounds GO enrichment
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3407
tools:
- name: EnrichR
  role: Performs GO enrichment analysis on per-cluster up-/down-regulated DEG lists
  repo: https://github.com/StaafLab/SRIQ
- name: mygene
  role: Maps input gene identifiers (scopes set to input format) to gene symbols required
    by the EnrichR module
- name: SRIQ
  role: Upstream source of cluster-specific DEG lists via SAMDEG one-versus-rest SAM
    analysis
  repo: https://github.com/StaafLab/SRIQ
- name: Python
  role: Runs the SRIQ pipeline's EnrichR enrichment module in the Jupyter analysis
    pipeline
  repo: https://github.com/StaafLab/SRIQ
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:a318734342f9d16a9e1ef1d06b719eeed64032c0cc12cea31b264e35d4f7aff7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/StaafLab/SRIQ
  repo_url: https://github.com/StaafLab/SRIQ
  tools:
  - EnrichR
  - Java
  - Python
  - SRIQ
  - mygene
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gene-ontology-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

EnrichR-based GO enrichment of cluster-specific differentially expressed gene lists, as implemented in the SRIQ pipeline's Python/EnrichR module, to attribute biological programs (e.g., cell cycle, immune response) to unsupervised clusters such as the SRIQ core clusters in TCGA LUAD.

## When to use

Use when you have per-cluster up-/down-regulated gene lists from one-vs-rest SAM differential expression and need GO terms explaining the biology of each cluster.

## When NOT to use

- Input gene lists are not from a differential expression step — enrichment of unfiltered, genome-wide gene lists yields meaningless term over-representation
- Gene identifiers are not mappable to gene symbols (EnrichR module requires symbols; mygene 'scopes' must match the identifier format)
- The goal is de novo pathway discovery beyond GO — a customizable EnrichR database could be used instead, but the article only grounds GO enrichment

## Inputs

- Per-cluster differentially expressed gene lists from SAM one-vers-rest analysis (up- and down-regulated), with SAM q-value and fold-change fields
- Gene identifier format specification (e.g., Ensembl ENSG IDs) for mygene symbol mapping
- GO database selection for EnrichR

## Outputs

- Gene-symbol-mapped DEG lists per cluster
- Per-cluster GO enrichment terms with EnrichR statistics (combined score, p-value) for up- and down-regulated gene sets
- Compiled single table of DEGs and GO enrichment terms across all clusters

## How to apply

After running SAMDEG one-versus-rest (e.g., 'java -jar -Xmx8g SRIQS.jar SAMDEG test.properties false 0.63 6 0 2 newFiltered_35k.txt') on the log2-transformed expression matrix with SRIQ cluster assignments (diameter 0.63, 6 clusters), map each cluster's DEG identifiers to gene symbols using the mygene API with the 'scopes' variable set to the input identifier format (e.g., ENSG), because the EnrichR module assumes gene symbols. Run the EnrichR module of the SRIQ Python pipeline separately on the up-regulated and down-regulated gene lists of each cluster against a GO database. Compile per-cluster DEG lists and enriched GO terms with EnrichR statistics (e.g., combined score, p-value) into a single table covering all 6 clusters.

## Related tools

- **EnrichR** (Performs GO enrichment analysis on per-cluster up-/down-regulated DEG lists) — https://github.com/StaafLab/SRIQ
- **mygene** (Maps input gene identifiers (scopes set to input format) to gene symbols required by the EnrichR module)
- **SRIQ** (Upstream source of cluster-specific DEG lists via SAMDEG one-versus-rest SAM analysis) — https://github.com/StaafLab/SRIQ
- **Python** (Runs the SRIQ pipeline's EnrichR enrichment module in the Jupyter analysis pipeline) — https://github.com/StaafLab/SRIQ

## Examples

```
java -jar -Xmx8g SRIQS.jar SAMDEG test.properties false 0.63 6 0 2 newFiltered_35k.txt  # then run EnrichR on per-cluster symbol-mapped DEG lists
```

## Evaluation signals

- The output table contains DEG rows for each of the 6 clusters with SAM q-value and fold-change fields populated
- Enrichment terms carry EnrichR statistics for both up- and down-regulated gene sets of every cluster
- Every gene submitted to EnrichR is a gene symbol (mygene mapping with 'scopes' set to the input identifier format succeeded)
- Enriched GO terms match known cluster biology — e.g., proliferative cluster 4 shows DNA replication, mitotic spindle and cell cycle terms; cluster 1 shows immune response terms among up-regulated genes

## Limitations

- The EnrichR module assumes gene names are in gene-symbol form, so an identifier-mapping step (mygene) is mandatory and a source of potential ID-loss errors
- Enrichment quality depends on the upstream SAM q-value and fold-change thresholds (e.g., q=0, fold-change=2); overly permissive thresholds inflate gene lists and dilute term specificity
- SRIQ cluster solutions are sensitive to the diameter parameter; enriching clusters from an unstable solution (e.g., diameter 0.63 vs others) affects interpretability

## Evidence

- [results] we performed differential gene expression of the core clusters through a "one versus rest" approach using SAM analysis: [evidence span withheld — anchor status: not_found]
- [results] the highly proliferative core cluster 4 showed up-regulated DNA replication, mitotic spindle and cell cycle GO terms: [evidence span withheld — anchor status: not_found]
