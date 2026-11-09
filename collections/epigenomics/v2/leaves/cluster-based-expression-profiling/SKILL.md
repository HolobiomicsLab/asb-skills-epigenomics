---
name: cluster-based-expression-profiling
description: Use when you have a completed SRIQ cluster solution with core-cluster
  assignments plus a log2-transformed expression matrix (e.g., newFiltered_35k.txt)
  and need per-cluster DEGs and GO terms to interpret cluster biology.
when_to_use_negative:
- The expression matrix is raw (not offset by 0.1 / values <1 set to 1, log2-transformed)
  — SAM-analysis requires these preprocessing steps first
- Gene identifiers are not gene symbols and cannot be mapped (EnrichR requires symbols
  via the mygene API)
- You need to detect small or underrepresented subgroups in a small dataset — SRIQ
  cannot circumvent this general limitation
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: SRIQ
  role: Provides the SRIQS.jar/SAMDEG entry point used to run one-versus-rest SAM
    differential expression on the selected cluster solution
  repo: https://github.com/StaafLab/SRIQ
- name: EnrichR
  role: Gene enrichment analysis of per-cluster DEG lists against customizable GO
    databases; requires gene symbols
  repo: https://github.com/StaafLab/SRIQ
- name: Java
  role: Runtime for SRIQS.jar / SAMDEG.jar commands (e.g., java -jar -Xmx8g)
- name: Python
  role: Jupyter analysis_pipeline for enrichment and gene mapping via the mygene API
    (set 'scopes' to input identifier format)
  repo: https://github.com/StaafLab/SRIQ
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:d72efa9f61b1d16c54a003ea77033045a65cd83d3e48e43f5fe4a71b875cefb3
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cluster-based-expression-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Characterizes clusters from an SRIQ solution (e.g., the TCGA LUAD diameter-0.63, 6-cluster solution) by one-versus-rest SAM differential expression per core cluster and EnrichR GO enrichment of the resulting up-/down-regulated gene lists, revealing cluster-specific transcriptional programs such as cell cycle or immune response.

## When to use

Use when you have a completed SRIQ cluster solution with core-cluster assignments plus a log2-transformed expression matrix (e.g., newFiltered_35k.txt) and need per-cluster DEGs and GO terms to interpret cluster biology.

## When NOT to use

- The expression matrix is raw (not offset by 0.1 / values <1 set to 1, log2-transformed) — SAM-analysis requires these preprocessing steps first
- Gene identifiers are not gene symbols and cannot be mapped (EnrichR requires symbols via the mygene API)
- You need to detect small or underrepresented subgroups in a small dataset — SRIQ cannot circumvent this general limitation

## Inputs

- Log2-transformed, offset/median-centered gene expression matrix (tab-separated .txt, e.g., newFiltered_35k.txt with Gene column and TCGA sample columns)
- Cluster assignments extracted from the selected SRIQ solution (spiral flag, diameter, number of clusters, e.g., false 0.63 6)
- test.properties file configured with studyName, studyPath, inFileName, outPath
- Per-cluster up-/down-regulated DEG gene-symbol lists (for EnrichR)

## Outputs

- Per-cluster SAM DEG tables with up- and down-regulated genes including SAM q-value and fold-change fields
- Per-cluster EnrichR GO enrichment terms with EnrichR statistics for up- and down-regulated gene sets
- Compiled single table of per-cluster DEGs and GO enrichment terms

## How to apply

Load the log2-transformed expression matrix together with cluster assignments extracted from the chosen SRIQ solution (e.g., diameter 0.63, 6 clusters, spiral=false), then run SAMDEG in one-versus-rest mode per cluster with q-value and fold-change thresholds (the paper's example uses q-value 0 and fold-change 2). Map gene identifiers to gene symbols via the mygene API (set the 'scopes' variable to your identifier format) because the EnrichR module requires gene symbols. Run EnrichR on each cluster's up- and down-regulated DEG lists against a GO database and compile DEG statistics (SAM q-value, fold-change) and EnrichR enrichment terms into a single table. Correct application is confirmed when DEG rows exist for all 6 clusters with q-value and fold-change populated and enrichment terms carry EnrichR statistics.

## Related tools

- **SRIQ** (Provides the SRIQS.jar/SAMDEG entry point used to run one-versus-rest SAM differential expression on the selected cluster solution) — https://github.com/StaafLab/SRIQ
- **EnrichR** (Gene enrichment analysis of per-cluster DEG lists against customizable GO databases; requires gene symbols) — https://github.com/StaafLab/SRIQ
- **Java** (Runtime for SRIQS.jar / SAMDEG.jar commands (e.g., java -jar -Xmx8g))
- **Python** (Jupyter analysis_pipeline for enrichment and gene mapping via the mygene API (set 'scopes' to input identifier format)) — https://github.com/StaafLab/SRIQ

## Examples

```
java -jar -Xmx8g SRIQS.jar SAMDEG test.properties false 0.63 6 0 2 newFiltered_35k.txt
```

## Evaluation signals

- Output table contains DEG rows for each of the 6 clusters with SAM q-value and fold-change fields populated
- Enrichment terms carry EnrichR statistics for both up- and down-regulated gene sets
- SAMDEG was run on the same solution parameters used for EXTRACT (e.g., false 0.63 6) so DEGs correspond to the intended clusters
- All DEG lists supplied to EnrichR use gene symbols after mygene mapping (scopes set to the input identifier format)
- Biology check: in the published LUAD solution, the proliferative core cluster 4 shows DNA replication, mitotic spindle and cell cycle GO terms up-regulated, while cluster 1 shows immune response terms among up-regulated genes

## Limitations

- Cluster solutions are sensitive to the diameter parameter, which must be considered in the context of stability and occurrence of a solution across iterations
- The combination of large bag size, high number of permutations, and large datasets makes SRIQ computation and memory intensive (e.g., -Xmx8g)
- The enrichR module assumes gene names are in the form of gene symbols; identifier mapping via mygene is a required intermediate step

## Evidence

- [methods] SAMDEG command structure: "java -jar <path-to/SRIQS.jar> SAMDEG <path-to/test.properties> <spiral (true or false)> <diameter> <no. of clusters> <q-value> <fold-change> <log2_transformed_gex_file>"
- [results] One-versus-rest SAM design: [evidence span withheld — anchor status: not_found]
- [readme] EnrichR symbol requirement: "The enrichR module assumes that gene names are in the form of gene symbols. I have implemented [mygene](https://mygene.info) api, set the variable 'scopes' as the format of your gene names."
- [readme] Pipeline feature set: "* Differential gene expression with SAM * Gene enrichment analysis with EnrichR against customizable databases"
