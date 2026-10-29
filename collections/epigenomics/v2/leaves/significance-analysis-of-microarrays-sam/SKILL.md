---
name: significance-analysis-of-microarrays-sam
description: Use when you have SRIQ cluster assignments (e.g., a diameter 0.63, 6-cluster
  solution on TCGA LUAD) and a log2-transformed gene expression matrix, and need per-cluster
  one-vs-rest differentially expressed genes.
when_to_use_negative:
- Expression data are raw FPKM counts that have not been off-set (0.1 or values <
  1 set to 1) and log2-transformed, violating SAMDEG's data format requirements
- You need supervised comparison of predefined groups (e.g., tumor vs normal) rather
  than SRIQ cluster one-versus-rest tests
- The dataset is too small to detect small or underrepresented biological subgroups,
  limiting meaningful cluster-level DEG analysis
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: SRIQ
  role: Java pipeline (SRIQS.jar SAMDEG) executing SAM one-versus-rest differential
    expression on extracted clusters
  repo: https://github.com/StaafLab/SRIQ
- name: Java
  role: Runtime for SRIQS.jar / SAMDEG.jar with heap allocation (e.g., -Xmx8g)
- name: Python
  role: Used with the mygene API to map gene identifiers to gene symbols required
    by the EnrichR module
  repo: https://github.com/StaafLab/SRIQ
- name: EnrichR
  role: GO enrichment analysis of per-cluster up- and down-regulated DEG lists against
    a GO database
  repo: https://github.com/StaafLab/SRIQ
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:b5ef91cbd9da190298e027630fa14566a93247ac9a688e48f1811041262871f5
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# significance-analysis-of-microarrays-sam

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run SAM-based one-versus-rest differential gene expression (SAMDEG) on SRIQ core clusters using the SRIQS.jar Java module, calling up- and down-regulated genes per cluster at user-defined q-value and fold-change thresholds from log2-transformed expression data.

## When to use

Use when you have SRIQ cluster assignments (e.g., a diameter 0.63, 6-cluster solution on TCGA LUAD) and a log2-transformed gene expression matrix, and need per-cluster one-vs-rest differentially expressed genes.

## When NOT to use

- Expression data are raw FPKM counts that have not been off-set (0.1 or values < 1 set to 1) and log2-transformed, violating SAMDEG's data format requirements
- You need supervised comparison of predefined groups (e.g., tumor vs normal) rather than SRIQ cluster one-versus-rest tests
- The dataset is too small to detect small or underrepresented biological subgroups, limiting meaningful cluster-level DEG analysis

## Inputs

- Log2-transformed gene expression matrix (tab-separated .txt, e.g., newFiltered_35k.txt)
- SRIQ test.properties configuration file (studyName, studyPath, inFileName, outPath set)
- SRIQ cluster assignments for the selected solution (e.g., diameter 0.63, 6 clusters, spiral=false)
- Gene identifier format for mygene mapping (e.g., Ensembl ENSG IDs)

## Outputs

- Per-cluster SAM differentially expressed gene lists with q-value and fold-change fields (up- and down-regulated)
- EnrichR GO enrichment terms with statistics for up- and down-regulated gene sets per cluster
- Compiled table of per-cluster DEGs and GO enrichment terms

## How to apply

First extract cluster assignments for the chosen SRIQ solution (e.g., EXTRACT with spiral=false, diameter 0.63, 6 clusters) and confirm the expression file is log2-transformed with values off-set by 0.1 or values < 1 set to 1, as required for SAM-analysis. Then run SAMDEG via 'java -jar SRIQS.jar SAMDEG <test.properties> <spiral> <diameter> <n_clusters> <q-value> <fold-change> <gex_file>', e.g., with q-value 0 and fold-change 2, which applies SAM with permutation testing to call one-versus-rest up- and down-regulated genes per core cluster. Downstream, map gene identifiers to gene symbols via the mygene API (set the 'scopes' variable to the input identifier format, e.g., ENSG) because the EnrichR module requires gene symbols, then run EnrichR GO enrichment on each cluster's DEG list and compile DEGs plus enrichment statistics into a single table.

## Related tools

- **SRIQ** (Java pipeline (SRIQS.jar SAMDEG) executing SAM one-versus-rest differential expression on extracted clusters) — https://github.com/StaafLab/SRIQ
- **Java** (Runtime for SRIQS.jar / SAMDEG.jar with heap allocation (e.g., -Xmx8g))
- **Python** (Used with the mygene API to map gene identifiers to gene symbols required by the EnrichR module) — https://github.com/StaafLab/SRIQ
- **EnrichR** (GO enrichment analysis of per-cluster up- and down-regulated DEG lists against a GO database) — https://github.com/StaafLab/SRIQ

## Examples

```
java -jar -Xmx8g SRIQS.jar SAMDEG "F:/test/LUAD_test/test.properties" false 0.63 6 0 2 "F:/test/LUAD_test/newFiltered_35k.txt"
```

## Evaluation signals

- Output table contains DEG rows for each of the clusters in the chosen solution (e.g., all 6 clusters) with SAM q-value and fold-change fields populated
- Enrichment terms carry EnrichR statistics for both up- and down-regulated gene sets
- Input gex file passed to SAMDEG is log2-transformed (values off-set by 0.1 or all values < 1 set to 1) per the README data format requirements
- Gene identifiers in the enrichment input are gene symbols, confirming mygene 'scopes' mapping succeeded
- Reproducibility check: rerunning SAMDEG with identical test.properties, diameter, cluster count, q-value, and fold-change yields the same DEG calls

## Limitations

- SRIQ cluster solutions are sensitive to the cluster diameter parameter, which must be evaluated against stability and occurrence across iterations before DEG analysis
- The combination of a large bag size, high number of permutations, and large datasets (samples and features) can make running SRIQ computation and memory intensive
- The enrichR module assumes gene names are gene symbols, requiring identifier mapping via the mygene API before enrichment

## Evidence

- [results] SAM one-versus-rest: [evidence span withheld — anchor status: not_found]
