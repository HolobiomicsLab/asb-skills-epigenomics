---
name: time-series-expression-data-input
description: Use when a workflow needs to supply time-series expression data plus
  TF–target interactions and pathway gene sets (KEGG/Reactome/GO) into TSMiner to
  build time-specific TF–pathway regulatory networks.
when_to_use_negative:
- Input data are not time-series (e.g. static or single-condition expression profiles)
  — TSMiner is designed specifically for time-series expression profiles
- No TF–TG interaction information or pathway gene sets are available, since TSMiner
  requires all three input file types
- The goal is single-cell or proteomics integration — the article notes applicability
  to other omics types (e.g. proteomics with PPI) is untested future work
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: TSMiner
  role: Java application that ingests the time-series expression matrix, TF–gene interaction
    file, and KEGG/Reactome/GO annotation files to construct time-specific regulatory
    networks
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime environment for TSMiner (Java 1.5 or later, 1024 MB heap minimum)
    used to launch the tool from the command line
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:d92aeb111c844a2595d85ceb52e63fda2836ebab16dbd2d980663347c6815227
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
  - Java
  - TSMiner
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# time-series-expression-data-input

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Skill for loading and configuring the three input file types required by TSMiner — a time-series expression matrix, a TF–TG interaction file, and gene annotation (KEGG/Reactome pathway) files — to construct time-specific gene regulatory networks from time-series expression profiles.

## When to use

Use when a workflow needs to supply time-series expression data plus TF–target interactions and pathway gene sets (KEGG/Reactome/GO) into TSMiner to build time-specific TF–pathway regulatory networks.

## When NOT to use

- Input data are not time-series (e.g. static or single-condition expression profiles) — TSMiner is designed specifically for time-series expression profiles
- No TF–TG interaction information or pathway gene sets are available, since TSMiner requires all three input file types
- The goal is single-cell or proteomics integration — the article notes applicability to other omics types (e.g. proteomics with PPI) is untested future work

## Inputs

- Time-series gene expression matrix (e.g. mouse LR RNA-seq dataset GSE95135), placed in the 'Time-series data' directory
- TF–gene interaction file (TF–TG pairs, positive and negative correlations), placed in the 'TF-gene file' directory
- Gene annotation files: GO and KEGG/Reactome pathway gene sets, placed in the 'Gene annotation file' directory
- Parameter configuration file (Defaults_for_case_analysis.txt for default LR case-analysis settings)

## Outputs

- Loaded TSMiner model ready to run, producing time-specific TF–pathway regulatory networks
- Saved interactive network map model stored in the 'Saved model' directory

## How to apply

TSMiner integrates three types of input files: a time-series expression matrix (e.g. the mouse liver regeneration RNA-seq dataset GSE95135), a TF–gene interaction file, and gene annotation files containing GO and KEGG/Reactome pathway gene sets. These files are organized in the tool's directories: 'Time-series data' for expression sets, 'TF-gene file' for TF–TG interactions, and 'Gene annotation file' for pathway and GO files. Data input and parameter settings follow TSMiner manual.pdf; for the mouse LR case analysis, default parameters are listed in Defaults_for_case_analysis.txt and the model is launched with 'java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt' followed by pressing 'Run'. Process positively and negatively correlated TF–TG interactions separately (a v1.1.0+ feature) so activation and repression events are modeled independently. The loaded data feed construction of time-specific regulatory networks using TF-encoding genes activated/repressed at specific time points and pathway genes significantly interacting with them.

## Related tools

- **TSMiner** (Java application that ingests the time-series expression matrix, TF–gene interaction file, and KEGG/Reactome/GO annotation files to construct time-specific regulatory networks) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime environment for TSMiner (Java 1.5 or later, 1024 MB heap minimum) used to launch the tool from the command line)

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- All three input file types (time-series expression, TF-gene file, gene annotation) load without error in the TSMiner main interface
- The default LR case analysis (Defaults_for_case_analysis.txt) reproduces the reported mouse liver regeneration run when launched with Case_analysis_of_mouseLR.txt
- With correctly loaded inputs, the LR dataset analysis should yield 389 transcriptional activators and 49 transcriptional repressors
- The resulting KEGG pathway interaction counts should match 109 pathways for activators and 47 for repressors
- Positively and negatively correlated TF–TG interactions are processed as separate runs (TSMiner v1.1.0+ behavior)

## Limitations

- Fewer results are obtained from negative TF–TG interactions than positive ones because known repressors of many genes are limited; future collection of negatively correlated TF–TG interactions may help
- TSMiner requires Java 1.5 or later with at least 1024 MB heap memory
- Applicability to other time-series omics data (e.g. proteomics with PPI information) is not yet demonstrated

## Evidence
