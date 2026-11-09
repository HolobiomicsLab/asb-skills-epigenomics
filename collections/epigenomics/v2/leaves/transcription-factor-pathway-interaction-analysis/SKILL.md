---
name: transcription-factor-pathway-interaction-analysis
description: Use when analyzing time-series gene expression profiles (e.g., RNA-seq
  across a biological process such as liver regeneration) to identify which TFs and
  pathways are activated or repressed at specific time points.
when_to_use_negative:
- Input is not a time-series expression profile — clustering methods or static network
  inference are more appropriate when temporal dynamics are not of interest
- TF–TG interaction data or pathway gene-set annotations for the organism of interest
  are unavailable, since TSMiner requires all three input file types
- The goal is a static gene co-expression module detection rather than time-specific
  TF–pathway regulatory events
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: TSMiner
  role: 'Primary tool: integrates time-series expression, TF–TG interactions, and
    GO/KEGG/Reactome pathway files to construct and visualize time-specific regulatory
    networks'
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: 'Runtime environment: TSMiner core algorithms and interfaces are written in
    Java; requires Java 1.5+ and -mx1024M heap'
- name: DREM (v. 2.0)
  role: Comparison baseline for benchmarking sensitivity and accuracy against TSMiner
    on LR RNA-seq data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:31eb522cf054bbaee3fcd53c55584b88b12c8aca136a93bf6ae4b4f9ccf4c63b
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
  - DREM (v. 2.0)
  - Java
  - TSMiner
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# transcription-factor-pathway-interaction-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct time-specific TF–pathway regulatory networks from time-series expression data using TSMiner, which integrates a time-series expression matrix, TF–TG interaction information, and GO/KEGG/Reactome pathway gene sets to identify transcriptional activators and repressors at each time point.

## When to use

Use when analyzing time-series gene expression profiles (e.g., RNA-seq across a biological process such as liver regeneration) to identify which TFs and pathways are activated or repressed at specific time points.

## When NOT to use

- Input is not a time-series expression profile — clustering methods or static network inference are more appropriate when temporal dynamics are not of interest
- TF–TG interaction data or pathway gene-set annotations for the organism of interest are unavailable, since TSMiner requires all three input file types
- The goal is a static gene co-expression module detection rather than time-specific TF–pathway regulatory events

## Inputs

- Time-series gene expression dataset (e.g., mouse liver regeneration RNA-seq, GEO accession GSE95135)
- TF–gene (TF–TG) interaction file (positive and negatively correlated interactions, from 'TF-gene file' directory)
- Gene ontology files and KEGG/Reactome pathway gene-set files (from 'Gene annotation file' directory)
- Parameter file (e.g., Defaults_for_case_analysis.txt)

## Outputs

- Time-specific regulatory network of activated/repressed TFs interacting with pathway gene sets per time point
- Saved model file (e.g., case_analysis_of_mouseLR_model.txt in 'Saved model' directory format)
- Counts of transcriptional activators/repressors and significantly interacting KEGG pathways (e.g., 389 activators, 49 repressors, 109 and 47 KEGG pathways in the LR case study)

## How to apply

Run TSMiner (Java 1.5+, 1024 MB heap) from the TSMiner directory with 'java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt', loading three input file types: a time-series expression dataset (e.g., mouse LR RNA-seq, GSE95135), TF–gene interaction data from the 'TF-gene file' directory, and GO plus KEGG/Reactome pathway files from the 'Gene annotation file' directory. Set parameters per Defaults_for_case_analysis.txt (details in TSMiner manual.pdf) and press 'Run'; TSMiner separately processes positively and negatively correlated TF–TG interactions. Inspect the resulting time-specific regulatory network via the interactive network map visualization, then save the model (e.g., case_analysis_of_mouseLR_model.txt in 'Saved model' format). For evaluation, apply TSMiner alongside DREM (v. 2.0) and clustering tools and compare sensitivity/accuracy in recovering known LR-associated TFs and KEGG pathways (reference sets of 113 TFs and 58 pathways from text-mined MEDLINE abstracts with 'liver regeneration' and 'partial hepatectomy').

## Related tools

- **TSMiner** (Primary tool: integrates time-series expression, TF–TG interactions, and GO/KEGG/Reactome pathway files to construct and visualize time-specific regulatory networks) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime environment: TSMiner core algorithms and interfaces are written in Java; requires Java 1.5+ and -mx1024M heap)
- **DREM (v. 2.0)** (Comparison baseline for benchmarking sensitivity and accuracy against TSMiner on LR RNA-seq data)

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- The run completes without error under the default parameter file (Defaults_for_case_analysis.txt)
- A saved model file (e.g., case_analysis_of_mouseLR_model.txt) exists on disk with non-empty content in the 'Saved model' format
- Sensitivity/accuracy benchmark against the curated reference sets of 113 LR-associated TFs and 58 LR-associated KEGG pathways recovered via MEDLINE text mining plus manual curation
- Output networks separately report activators and repressors, with fewer results expected from negative TF–TG interactions due to limited known repressors
- Interactive network map renders time-specific TF–pathway interactions for each time point of the time-series data

## Limitations

- Fewer results are obtained from negative TF–TG interactions than positive ones because known repressors of many genes are limited; future collection of negatively correlated TF–TG interactions may help
- Although experimental results supported the predictions (e.g., the late-stage LR immune-regulatory pathway cascade), the precise regulatory mechanisms require more rigorous evaluation
- Applicability to other time-series omics data types (e.g., integrating proteomics time-series data with PPI information) is proposed but not yet demonstrated
- Requires Java 1.5 or later and at least 1024 MB of Java heap memory

## Evidence

- [other] Reference sets of known LR TFs and pathways: [evidence span withheld — anchor status: not_found]
