---
name: benchmark-sensitivity-evaluation
description: Use when a new network/pathway-detection method must be validated against
  established clustering tools on a time-series dataset for which a curated set of
  known associated pathways exists.
when_to_use_negative:
- No gold-standard set of known pathway associations exists for the tissue/process
  under study, so sensitivity cannot be scored
- The input is a static (single-time-point) expression dataset, not a time series
  — TSMiner and the time-specific-network comparison are not applicable
- The aim is exploratory pathway discovery rather than method benchmarking; a direct
  enrichment analysis suffices
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: TSMiner
  role: Candidate method under evaluation; constructs time-specific TF-pathway regulatory
    networks and outputs predicted TF-pathway interactions for activator and repressor
    groups
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: DREM (v. 2.0)
  role: Comparator method run on the same LR RNA-seq data followed by pathway enrichment
    to produce comparison pathway predictions
- name: hclust
  role: Clustering comparator; pathway predictions obtained via KEGG/Reactome enrichment
    of clusters
- name: k-means
  role: Clustering comparator; pathway predictions obtained via KEGG/Reactome enrichment
    of clusters
- name: STEM
  role: Clustering comparator; pathway predictions obtained via KEGG/Reactome enrichment
    of clusters
- name: WGCNA
  role: Clustering comparator; pathway predictions obtained via KEGG/Reactome enrichment
    of modules
- name: Java
  role: Runtime for TSMiner; launch with a 1024 MB heap (java -mx1024M -jar TSMiner.jar)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:f16929ed1e8c1150464b47069d17766c04041a38d27f472e0e445e2f786e5d0f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/free1234hm/tsminer-tool
  repo_url: https://github.com/free1234hm/tsminer-tool.git
  tools:
  - DREM
  - DREM (v. 2.0)
  - Java
  - STEM
  - TSMiner
  - WGCNA
  - hclust
  - k-means
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# benchmark-sensitivity-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a time-series regulatory-network method (e.g. TSMiner) against comparison methods (DREM, hclust, k-means, STEM, WGCNA) by computing sensitivity and AUC for detecting known LR-associated KEGG/Reactome pathways at P-value thresholds sweeping 0–0.05.

## When to use

Use when a new network/pathway-detection method must be validated against established clustering tools on a time-series dataset for which a curated set of known associated pathways exists.

## When NOT to use

- No gold-standard set of known pathway associations exists for the tissue/process under study, so sensitivity cannot be scored
- The input is a static (single-time-point) expression dataset, not a time series — TSMiner and the time-specific-network comparison are not applicable
- The aim is exploratory pathway discovery rather than method benchmarking; a direct enrichment analysis suffices

## Inputs

- Time-series RNA-seq expression matrix (mouse liver regeneration, GEO GSE95135)
- TSMiner case-analysis configuration file Case_analysis_of_mouseLR.txt and Defaults_for_case_analysis.txt
- TF-gene interaction files (positively and negatively correlated interactions)
- KEGG/Reactome pathway annotation files from the TSMiner 'Gene annotation file' directory
- Curated positive set of 58 known LR-associated pathways
- Negative set of 78 pathways drawn from a 499-pathway background

## Outputs

- Per-method sensitivity values across P-value thresholds 0–0.05
- ROC curves and AUC per method
- Counts of uniquely identified pathways per method
- Consolidated benchmark table (tsminer_pathway_benchmark_results.csv)

## How to apply

Run the candidate method (TSMiner v1.2.0 under Java with a 1024 MB heap, loading Case_analysis_of_mouseLR.txt with defaults from Defaults_for_case_analysis.txt) and each comparator (DREM v2.0, hclust, k-means, STEM, WGCNA) on the same mouse LR RNA-seq time-series (GSE95135) with the same TF-gene interaction and KEGG/Reactome annotation files. Extract predicted TF-pathway interactions (activator and repressor groups separately for TSMiner) and, for clustering methods, perform downstream KEGG/Reactome enrichment to obtain comparable pathway predictions. Compute sensitivity at P-value thresholds sweeping 0 to 0.05 against the 58 known LR-associated pathways, tally uniquely identified pathways per method, and build ROC curves with AUC against a 78-pathway negative set drawn from the 499-pathway background. Consolidate per-method sensitivity, unique counts and AUC into a results table and accept the benchmark only if the ranking (TSMiner highest in sensitivity and accuracy) reproduces the published ordering.

## Related tools

- **TSMiner** (Candidate method under evaluation; constructs time-specific TF-pathway regulatory networks and outputs predicted TF-pathway interactions for activator and repressor groups) — https://github.com/free1234hm/tsminer-tool.git
- **DREM (v. 2.0)** (Comparator method run on the same LR RNA-seq data followed by pathway enrichment to produce comparison pathway predictions)
- **hclust** (Clustering comparator; pathway predictions obtained via KEGG/Reactome enrichment of clusters)
- **k-means** (Clustering comparator; pathway predictions obtained via KEGG/Reactome enrichment of clusters)
- **STEM** (Clustering comparator; pathway predictions obtained via KEGG/Reactome enrichment of clusters)
- **WGCNA** (Clustering comparator; pathway predictions obtained via KEGG/Reactome enrichment of modules)
- **Java** (Runtime for TSMiner; launch with a 1024 MB heap (java -mx1024M -jar TSMiner.jar))

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- The sensitivity curve across P-value thresholds 0–0.05 reproduces TSMiner ranking highest among TSMiner, DREM, hclust, k-means, STEM and WGCNA
- AUC against the 78-pathway negative set (from the 499-pathway background) ranks TSMiner highest in accuracy
- Sensitivity is computed against exactly the 58 known LR-associated pathways
- TSMiner results separate positively and negatively correlated TF-TG interactions into distinct activator/repressor outputs (109 and 47 KEGG pathways in the published run)
- The consolidated table tsminer_pathway_benchmark_results.csv contains one row per method with sensitivity, unique-pathway count and AUC columns

## Limitations

- Fewer results are obtained from negative TF–TG interactions than from positive interactions because known repressors of many genes are limited
- Clustering methods cannot evaluate the temporal dynamics of regulators or pathways, so the comparison is inherently asymmetric in what each method can represent
- Precision/regulatory mechanism validity requires more rigorous experimental evaluation beyond the benchmark metrics

## Evidence

- [intro] By applying TSMiner, DREM (v. 2.0) and four widely used clustering tools for LR RNA-seq data, we demonstrated the highest sensitivity and accuracy of TSMiner in detecting known LR-associated TFs and: "By applying TSMiner, DREM (v. 2.0) and four widely used clustering tools for LR RNA-seq data, we demonstrated the highest sensitivity and accuracy of TSMiner in detecting known LR-associated TFs and"
- [other] Run TSMiner on the time-series expression data, TF-gene interaction files and KEGG/Reactome pathway annotation files: [evidence span withheld — anchor status: not_found]
- [readme] java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt: [evidence span withheld — anchor status: external_doc]
- [intro] However, clustering methods cannot evaluate the temporal dynamics of regulators or pathways.: "However, clustering methods cannot evaluate the temporal dynamics of regulators or pathways."
- [discussion] fewer results were obtained from the negative TF–TG interactions than from the positive interactions because the known repressors of many genes were limited.: "fewer results were obtained from the negative TF–TG interactions than from the positive interactions because the known repressors of many genes were limited."
