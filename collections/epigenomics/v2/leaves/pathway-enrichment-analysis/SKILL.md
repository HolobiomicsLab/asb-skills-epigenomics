---
name: pathway-enrichment-analysis
description: Use when comparing a network- or clustering-based method's ability to
  recover known biological pathways from time-series expression data, requiring sensitivity/AUC
  benchmarking against curated pathway sets.
when_to_use_negative:
- The input lacks pathway annotation (no KEGG/Reactome gene sets), so enrichment against
  the 499-pathway background cannot be defined
- The expression data are not time-series and no time-specific regulatory network
  or dynamic module detection is intended
- The goal is exploratory visualization of TF-pathway links rather than quantitative
  benchmarking against a curated positive/negative pathway set
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3407
tools:
- name: TSMiner
  role: 'Primary method: constructs time-specific regulatory networks and outputs
    predicted TF-pathway interactions for activators and repressors from time-series
    expression data'
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: DREM (v. 2.0)
  role: 'Comparator: run on the same LR RNA-seq data followed by KEGG/Reactome pathway
    enrichment to obtain comparison pathway predictions'
- name: Java
  role: Runtime environment for TSMiner (Java 1.5 or later, 1024 MB heap)
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:8c406aa11836989d68bea6c080ed01fd3919df1b668fc291d7b08498fcaaa1ef
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
  - TSMiner
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# pathway-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark-style workflow for detecting known pathway associations (here, 58 known LR-associated KEGG pathways) by combining tool-derived pathway predictions with KEGG/Reactome enrichment analysis, then scoring sensitivity and AUC against curated positive and negative pathway sets.

## When to use

Use when comparing a network- or clustering-based method's ability to recover known biological pathways from time-series expression data, requiring sensitivity/AUC benchmarking against curated pathway sets.

## When NOT to use

- The input lacks pathway annotation (no KEGG/Reactome gene sets), so enrichment against the 499-pathway background cannot be defined
- The expression data are not time-series and no time-specific regulatory network or dynamic module detection is intended
- The goal is exploratory visualization of TF-pathway links rather than quantitative benchmarking against a curated positive/negative pathway set

## Inputs

- Mouse liver regeneration (LR) RNA-seq time-series expression dataset (GSE95135)
- TF-gene interaction files (positively and negatively correlated interactions processed separately)
- KEGG/Reactome pathway annotation files from the 'Gene annotation file' directory
- Case analysis configuration file Case_analysis_of_mouseLR.txt and default parameters from Defaults_for_case_analysis.txt

## Outputs

- Predicted TF-pathway interactions split by transcriptional activator and repressor groups
- Sensitivity values at P-value thresholds 0–0.05 per method against 58 known LR-associated pathways
- ROC curves and AUC per method against the 78-pathway negative set
- Consolidated benchmark table (e.g. tsminer_pathway_benchmark_results.csv)

## How to apply

Run the candidate method (e.g. TSMiner v1.2.0 via 'java -mx1024M -jar TSMiner.jar' on the mouse liver regeneration case analysis configuration Case_analysis_of_mouseLR.txt) and the comparator methods (DREM v2.0, hclust, k-means, STEM, WGCNA) on the same time-series expression data, extracting predicted TF-pathway interactions for transcriptional activator and repressor groups. For the comparator clustering methods, follow each prediction with KEGG/Reactome pathway enrichment analysis on the resulting gene modules. Compute sensitivity at P-value thresholds swept from 0 to 0.05 against the 58 known LR-associated pathways, tally uniquely identified pathways per method, and build ROC curves with AUC against a 78-pathway negative set drawn from the 499-pathway KEGG background. Consolidate all metrics per method into a results table and accept the analysis if TSMiner ranks highest in sensitivity and accuracy, as reported in the paper.

## Related tools

- **TSMiner** (Primary method: constructs time-specific regulatory networks and outputs predicted TF-pathway interactions for activators and repressors from time-series expression data) — https://github.com/free1234hm/tsminer-tool.git
- **DREM (v. 2.0)** (Comparator: run on the same LR RNA-seq data followed by KEGG/Reactome pathway enrichment to obtain comparison pathway predictions)
- **Java** (Runtime environment for TSMiner (Java 1.5 or later, 1024 MB heap))

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- TSMiner's sensitivity curve remains consistently and significantly above hclust, k-means, STEM and WGCNA across P-value thresholds 0 to 0.05
- TSMiner achieves the highest AUC against the 78-pathway negative set drawn from the 499-pathway background
- TSMiner's KEGG predictions include 109 activator-interacting and 47 repressor-interacting pathways, consistent with the paper's reported counts
- The benchmark table reproduces TSMiner ranking highest in both sensitivity and accuracy among all compared methods
- Saved TSMiner model (saved_model.txt) loads in the TSMiner interactive network map viewer

## Limitations

- Fewer results are obtained from negative TF–TG interactions than from positive interactions because known repressors of many genes are limited; future collection of negatively correlated TF–TG interactions may help address this issue
- Clustering methods cannot evaluate the temporal dynamics of regulators or pathways, limiting fair dynamic comparison
- The benchmark's negative pathway set (78 pathways from a 499-pathway background) constrains AUC interpretation to KEGG pathways only

## Evidence
