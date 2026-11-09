---
name: immune-response-cascade-reconstruction
description: Use when time-series RNA-seq expression data plus TF–gene interactions
  and pathway gene sets are available and the goal is to identify TFs regulating pathway
  cascades at specific time points.
when_to_use_negative:
- The expression data are not time-series (no time-course sampling); clustering methods
  cannot evaluate temporal dynamics of regulators or pathways, and TSMiner's time-specific
  construction is unnecessary.
- Only differentially expressed gene lists are available without TF–TG interaction
  data or pathway gene sets — TSMiner integrates all three input file types.
- The goal is static co-expression module detection rather than identifying which
  TFs interact with pathways at specific time points.
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0204
tools:
- name: TSMiner
  role: Constructs time-specific TF–pathway regulatory networks from the LR time-series
    expression data and provides interactive network map visualization for extracting
    the immune cascade stages
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime environment required to launch TSMiner (Java 1.5 or later, minimum
    1024 MB heap via -mx1024M)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:693c80720cb0b3ed030bdcdc49ec67a77a5279780c8fd41defdcf8baa30452e4
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
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# immune-response-cascade-reconstruction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs time-specific TF–pathway regulatory cascades across late-stage mouse liver regeneration (LR) using TSMiner, quantifying shared transcription factor regulators between consecutive immune-related KEGG pathway stages (e.g. 61 and 38 TFs with 33 overlapping for p53 signalling and apoptosis at 36 h).

## When to use

Use when time-series RNA-seq expression data plus TF–gene interactions and pathway gene sets are available and the goal is to identify TFs regulating pathway cascades at specific time points.

## When NOT to use

- The expression data are not time-series (no time-course sampling); clustering methods cannot evaluate temporal dynamics of regulators or pathways, and TSMiner's time-specific construction is unnecessary.
- Only differentially expressed gene lists are available without TF–TG interaction data or pathway gene sets — TSMiner integrates all three input file types.
- The goal is static co-expression module detection rather than identifying which TFs interact with pathways at specific time points.

## Inputs

- Time-series RNA-seq expression matrix of mouse liver regeneration after partial hepatectomy (GEO accession GSE95135)
- TF–gene (TF–TG) interaction file from the package's 'TF-gene file' directory
- KEGG pathway gene sets from the package's 'Gene annotation file' directory
- Case analysis settings file (Defaults_for_case_analysis.txt / Case_analysis_of_mouseLR.txt)

## Outputs

- Time-specific TF–pathway regulatory networks for each LR time point (activated and repressed)
- Immune cascade summary table (immune_cascade_summary.tsv) with time-point pathway assignments and TF overlap counts (61/38 TFs, 33 shared; 7 shared; 17 shared)
- Saved TSMiner model of the reconstructed case-analysis networks

## How to apply

Launch TSMiner from the tsminer-tool directory with 'java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt' (requires Java 1.5+ and at least 1024 MB heap), loading the mouse LR case analysis settings from Defaults_for_case_analysis.txt. Supply the three TSMiner input files: time-series LR RNA-seq expression data (GSE95135), TF–gene interaction data, and KEGG pathway gene sets, following the data input rules in TSMiner manual.pdf. Press 'Run' to build time-specific regulatory networks; with v1.1.0+ behaviour, positively and negatively correlated TF–TG interactions are processed separately. Explore the interactive network maps and extract the immune cascade: p53 signalling and apoptosis at 36 h; chemokine signalling, leukocyte transendothelial migration, Fc gamma R-mediated phagocytosis, endocytosis and phagosome at 48 h; cell adhesion molecules and Th1/Th2 cell differentiation at 168 h. For each consecutive stage, record TF sets sharing associations with the stage pathways and their overlaps (61 and 38 TFs with 33 shared; then 7 shared; then 17 shared), writing results to immune_cascade_summary.tsv and saving the reconstructed case-analysis networks as a saved model.

## Related tools

- **TSMiner** (Constructs time-specific TF–pathway regulatory networks from the LR time-series expression data and provides interactive network map visualization for extracting the immune cascade stages) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime environment required to launch TSMiner (Java 1.5 or later, minimum 1024 MB heap via -mx1024M))

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- The p53 signalling pathway and apoptosis stages at 36 h interact with 61 and 38 TFs respectively, with 33 TFs shared between them — deviations suggest input or parameter errors.
- The 48 h stage (chemokine signalling, leukocyte transendothelial migration, Fc gamma R-mediated phagocytosis, endocytosis, phagosome) shares 7 TFs with the preceding stage, and the 168 h stage (cell adhesion molecules, Th1/Th2 cell differentiation) shares 17.
- Positively and negatively correlated TF–TG interactions are processed as separate result sets (v1.1.0+); negative-interaction results should be fewer due to limited known repressors.
- The saved model file is written to the 'Saved model' directory and the summary is exported as immune_cascade_summary.tsv with one row per time point and pathway assignment.
- Networks should include the expected activated (389 TFs) and repressed (49 TFs) transcription factors across the LR process, and pathways significantly interacting with activators/repressors (109 and 47 KEGG pathways).

## Limitations

- Fewer results are obtained from negative TF–TG interactions than positive interactions because known repressors of many genes are limited; negatively correlated TF–TG interaction data still need to be collected.
- Although experimental results supported the predictions, the precise regulatory mechanisms require more rigorous evaluation.
- Applicability to other time-series omics data (e.g. integrating proteomics time-series data with PPI information) is not yet demonstrated and is left as future research.
- TSMiner requires Java 1.5 or later and a minimum of 1024 MB Java heap memory; results depend on the DREM-licensed framework (GPL v3.0) and its initialization.

## Evidence

- [readme] Otherwise from a command line while in the TSMiner directory type the command: java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt, then press the 'Run' button.: [evidence span withheld — anchor status: external_doc]
