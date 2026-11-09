---
name: time-series-gene-expression-network-inference
description: Use when you have time-series gene expression data plus TF–target interaction
  lists and pathway gene sets, and need to infer which TFs regulate which pathways
  at each time point.
when_to_use_negative:
- Input is a single-condition or steady-state expression matrix with no time-series
  structure — TSMiner requires temporal profiles to assign TF activity to time points.
- TF–target interaction information and pathway gene sets are unavailable — TSMiner
  integrates all three input file types and cannot build TF–pathway networks from
  expression data alone.
- The goal is clustering of expression profiles without regulator/pathway temporal
  dynamics — the article notes clustering methods cannot evaluate temporal dynamics
  of regulators or pathways, but conversely TSMiner is not a pure clustering tool.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: TSMiner
  role: 'Primary tool: integrates time-series expression, TF–TG interactions, and
    pathway gene sets to construct time-specific regulatory networks with interactive
    visualization'
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: DREM
  role: Baseline method (v. 2.0) for comparing sensitivity in detecting known LR-associated
    TFs on the same input data
- name: Java
  role: Runtime environment; TSMiner core algorithms and interfaces are written in
    Java and require 1.5 or later with at least 1024 MB heap
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_008
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:9c6cfe2f8d0f2d91046b5be7303e83c75724d992a3761b55f112e43e2bb56695
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
  - Java
  - TSMiner
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# time-series-gene-expression-network-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct time-specific TF–pathway regulatory networks from time-series expression profiles using TSMiner, which integrates a time-series expression matrix, TF–TG interaction data, and pathway gene sets (GO/KEGG/Reactome) to call TFs activated or repressed at each time point.

## When to use

Use when you have time-series gene expression data plus TF–target interaction lists and pathway gene sets, and need to infer which TFs regulate which pathways at each time point.

## When NOT to use

- Input is a single-condition or steady-state expression matrix with no time-series structure — TSMiner requires temporal profiles to assign TF activity to time points.
- TF–target interaction information and pathway gene sets are unavailable — TSMiner integrates all three input file types and cannot build TF–pathway networks from expression data alone.
- The goal is clustering of expression profiles without regulator/pathway temporal dynamics — the article notes clustering methods cannot evaluate temporal dynamics of regulators or pathways, but conversely TSMiner is not a pure clustering tool.

## Inputs

- Time-series gene expression dataset (e.g. mouse liver regeneration RNA-seq, GSE95135) from the 'Time-series data' directory
- TF–gene (TF–TG) interaction data from the 'TF-gene file' directory
- Gene ontology, KEGG, and Reactome pathway gene sets from the 'Gene annotation file' directory
- Case-analysis configuration file (e.g. Case_analysis_of_mouseLR.txt) and parameter defaults (Defaults_for_case_analysis.txt)

## Outputs

- Time-specific regulatory network model saved in the 'Saved model' directory format (e.g. case_analysis_of_mouseLR_model.txt)
- TF activation/repression calls per time point (permutation-test P-values, BH corrected)
- Significantly interacting TF–pathway pairs per time point (e.g. 389 activators, 49 repressors; 109 and 47 KEGG pathways in the LR case study)
- Interactive network map visualizations

## How to apply

Run TSMiner (Java 1.5+, minimum 1024 MB heap) via 'java -mx1024M -jar TSMiner.jar', loading three input types: the time-series expression dataset from the 'Time-series data' directory, TF–gene interaction data from the 'TF-gene file' directory, and GO plus KEGG/Reactome pathway files from the 'Gene annotation file directory'. Press the 'Run' button with parameters from Defaults_for_case_analysis.txt; TSMiner assigns genes to subpaths of a tree-structure model, identifies TFs significantly regulating each subpath via hypergeometric tests with permutation-test P-values (BH corrected), and classifies TFs as activated or repressed based on the average fold change of their target genes, processing positively and negatively correlated TF–TG interactions separately (v1.1.0 behaviour). Compare sensitivity against baselines such as DREM (v. 2.0) by counting recovered known TFs at P-value cut-offs of 0.01 and 0.05 across 'Maximum number of subpaths' settings (e.g. 3, 4, 5, unlimited). Explore results in the interactive network map viewer and save the model in the 'Saved model' directory format (e.g. case_analysis_of_mouseLR_model.txt).

## Related tools

- **TSMiner** (Primary tool: integrates time-series expression, TF–TG interactions, and pathway gene sets to construct time-specific regulatory networks with interactive visualization) — https://github.com/free1234hm/tsminer-tool.git
- **DREM** (Baseline method (v. 2.0) for comparing sensitivity in detecting known LR-associated TFs on the same input data)
- **Java** (Runtime environment; TSMiner core algorithms and interfaces are written in Java and require 1.5 or later with at least 1024 MB heap)

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- The run completes without error under the default parameter file and yields a saved model file (e.g. case_analysis_of_mouseLR_model.txt) whose existence and non-empty content are verifiable on disk.
- The saved model loads correctly in the TSMiner interactive network map viewer.
- Sensitivity against a curated reference set (e.g. 113 known LR-associated TFs, 58 LR-associated KEGG pathways from text-mined MEDLINE abstracts) can be computed at P-value cut-offs 0.01 and 0.05, with TSMiner expected to recover more known TFs than DREM (e.g. 10 vs 1 unique TFs at P ≤ 0.01).
- Positively and negatively correlated TF–TG interactions are processed separately, and results from negative interactions are expected to be fewer than positive ones due to limited known repressors.
- The ROC/AUC analysis for functional TF groups (e.g. 27 cell-proliferation TFs and 10 growth-suppression TFs at their designated LR periods) yields sensitivity curves over P-value cut-offs spanning 0 to 1.

## Limitations

- Fewer results are obtained from negative TF–TG interactions than from positive interactions because the known repressors of many genes are limited; future collection of negatively correlated TF–TG interactions may help.
- DREM-derived limitations TSMiner addresses (dependence on initial parameters, inability to determine when TFs exert influence, inability to distinguish period-specific subnetworks) suggest care when interpreting baseline DREM comparisons.
- Although experimental results supported the predictions (e.g. the late-stage immune-regulatory cascade), the precise regulatory mechanisms require more rigorous evaluation.
- Applicability to other time-series omics data types (e.g. integrating proteomics time-series data and PPI information) is proposed but not yet demonstrated.
- A minimum of 1024 MB Java heap memory and Java 1.5 or later are required; platform independent otherwise.

## Evidence

- [readme] README case-analysis invocation: [evidence span withheld — anchor status: external_doc]
- [other] Known LR TF reference set: [evidence span withheld — anchor status: not_found]
