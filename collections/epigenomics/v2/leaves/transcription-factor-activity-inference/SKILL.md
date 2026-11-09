---
name: transcription-factor-activity-inference
description: Use when you have a time-series expression matrix plus TF–TG interaction
  data and need to identify which TFs significantly regulate each subpath of a tree-structure
  model and when they are activated or repressed.
when_to_use_negative:
- Input is a single-condition or static (non-time-series) expression profile — TSMiner
  is built specifically for time-series expression data
- No TF–gene interaction data are available for the organism, since the hypergeometric/permutation
  scoring depends on TF-TG interaction information
- The goal is clustering expression profiles rather than inferring time-specific regulator
  activity — clustering methods cannot evaluate the temporal dynamics of regulators
  or pathways
edam_operation: http://edamontology.org/operation_0575
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: TSMiner
  role: Runs the hypergeometric + permutation TF prediction module, assigns TFs to
    subpaths of the tree-structure model, and classifies activation/repression time
    points
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime environment required to execute TSMiner (core algorithms and interfaces
    written in Java, 1.5 or later, 1024 MB heap)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:4db3d4e52cfdb72cade49cd21f4609bec68aa8ac6c68cf02f4d8c094c0362e65
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
  edam_operation: http://edamontology.org/operation_0575
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# transcription-factor-activity-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

TSMiner's TF prediction module assigns transcription factors to time-specific subpaths of a tree-structure model using a hypergeometric test followed by a permutation test (Benjamini–Hochberg corrected), then classifies each TF as activated or repressed at specific time points based on the average fold change of its target genes.

## When to use

Use when you have a time-series expression matrix plus TF–TG interaction data and need to identify which TFs significantly regulate each subpath of a tree-structure model and when they are activated or repressed.

## When NOT to use

- Input is a single-condition or static (non-time-series) expression profile — TSMiner is built specifically for time-series expression data
- No TF–gene interaction data are available for the organism, since the hypergeometric/permutation scoring depends on TF-TG interaction information
- The goal is clustering expression profiles rather than inferring time-specific regulator activity — clustering methods cannot evaluate the temporal dynamics of regulators or pathways

## Inputs

- Time-series gene expression matrix (e.g., mouse LR RNA-seq time-course data, GSE95135)
- TF–gene (TF-TG) interaction file covering positive and negative (activator/repressor) interactions
- Gene annotation file with Gene ontology and KEGG/Reactome pathway gene sets
- Configuration file (e.g., Case_analysis_of_mouseLR.txt) with parameter settings from Defaults_for_case_analysis.txt

## Outputs

- Time-specific regulatory network model (saved_model.txt) linking TFs to tree-structure subpaths and pathways
- Per-TF permutation-test P-values (BH corrected) mapping TFs to specific activation/repression time points
- Interactive network map viewable in the TSMiner visualization interface

## How to apply

Launch TSMiner (Java 1.5+, 1024 MB heap: 'java -mx1024M -jar TSMiner.jar') and load the time-series expression data (e.g., the mouse LR RNA-seq dataset), TF-gene interaction files, and Gene ontology/KEGG/Reactome pathway annotation files with parameters from TSMiner manual.pdf. TSMiner first assigns genes to subpaths of the tree-structure model, then scores each TF–subpath combination with a hypergeometric test and confirms significance with a permutation test using BH-corrected P-values. Positively and negatively correlated TF-TG interactions are processed separately (a feature of version v1.1.0+). A significant TF is then classified as activated or repressed at each time point according to the average fold change of its target genes, yielding a time-specific TF–pathway regulatory network. Save the resulting model as saved_model.txt and confirm it loads in the TSMiner interactive network map viewer.

## Related tools

- **TSMiner** (Runs the hypergeometric + permutation TF prediction module, assigns TFs to subpaths of the tree-structure model, and classifies activation/repression time points) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime environment required to execute TSMiner (core algorithms and interfaces written in Java, 1.5 or later, 1024 MB heap))

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- A TF is assigned to time points only when its permutation-test P-values pass Benjamini–Hochberg correction — check that reported TFs carry corrected P-values
- Activation vs. repression labels must be consistent with the average fold change of the TF's target genes (positive fold change → activated; negative → repressed)
- Positively and negatively correlated TF-TG interactions must be processed separately, yielding distinct activator and repressor results (LR application: 389 activators vs. 49 repressors)
- The saved model (saved_model.txt) must load correctly in the TSMiner interactive network map viewer
- Sensitivity/accuracy sanity check: recovered TFs and pathways should overlap with known LR-associated regulators when applied to the mouse LR benchmark dataset

## Limitations

- Fewer results are obtained from negative TF–TG interactions than positive ones because known repressors of many genes are limited; future collection of negatively correlated TF–TG interactions may help
- DREM-lineage limitations apply to initialization: dependence on initial parameters without effective initialization motivated TSMiner's redesign
- Requires Java 1.5 or later and at least 1024 MB of heap memory; a 'Download link:' placeholder in the README means the tool must be fetched from the GitHub repository

## Evidence

- [methods] In the TF prediction stage, a TF is assigned to time points upon receiving significant permutation-test P-values (BH corrected) and is subsequently classified as activated or repressed based on the average fold change of its target genes.: [evidence span withheld — anchor status: not_found]
- [readme] Otherwise from a command line while in the TSMiner directory type the command: java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt, then press the 'Run' button.: [evidence span withheld — anchor status: external_doc]
