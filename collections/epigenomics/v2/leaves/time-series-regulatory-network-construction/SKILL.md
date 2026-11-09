---
name: time-series-regulatory-network-construction
description: Use when a time-series gene expression dataset (e.g., RNA-seq over a
  biological time course like mouse liver regeneration) must be converted into time-specific
  regulatory networks identifying when TFs and pathways become active.
when_to_use_negative:
- The expression data are not time-series (no temporal dimension), since TSMiner requires
  time-course profiles to assign TF activity to specific time points
- No TF–TG interaction data are available — TSMiner integrates three input types,
  and missing TF-gene interaction information prevents network construction
- The goal is only pathway enrichment of expression clusters without temporal dynamics
  of regulators — clustering tools (hclust, k-means, STEM, WGCNA) cannot evaluate
  temporal dynamics of regulators, but if temporal resolution is irrelevant a simpler
  clustering workflow suffices
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: TSMiner
  role: Core Java tool that constructs the time-specific regulatory networks, processing
    positively and negatively correlated TF–TG interactions separately (v1.1.0 feature);
    v1.2.0 used for the LR case analysis
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime environment for TSMiner; TSMiner's core algorithms and interfaces
    are written in Java and require Java 1.5 or later with a 1024 MB heap minimum
- name: DREM
  role: Comparison method (v. 2.0) run on the same LR RNA-seq data for benchmarking
    sensitivity and accuracy of pathway detection against TSMiner
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:8636cdb08ba9a7cfe6b4204004eb498ff2b10405121f9a75b72a778002220ef2
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
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# time-series-regulatory-network-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs time-specific TF–pathway regulatory networks from time-series expression profiles using TSMiner, which integrates a time-series expression matrix, TF–TG interaction information, and KEGG/Reactome pathway gene sets to identify transcriptional activators and repressors at each time point.

## When to use

Use when a time-series gene expression dataset (e.g., RNA-seq over a biological time course like mouse liver regeneration) must be converted into time-specific regulatory networks identifying when TFs and pathways become active.

## When NOT to use

- The expression data are not time-series (no temporal dimension), since TSMiner requires time-course profiles to assign TF activity to specific time points
- No TF–TG interaction data are available — TSMiner integrates three input types, and missing TF-gene interaction information prevents network construction
- The goal is only pathway enrichment of expression clusters without temporal dynamics of regulators — clustering tools (hclust, k-means, STEM, WGCNA) cannot evaluate temporal dynamics of regulators, but if temporal resolution is irrelevant a simpler clustering workflow suffices

## Inputs

- Time-series expression matrix (e.g., mouse liver regeneration RNA-seq time-course, GEO accession GSE95135)
- TF-gene interaction file (from the 'TF-gene file' directory)
- KEGG/Reactome pathway annotation files and gene ontology files (from the 'Gene annotation file' directory)
- TSMiner configuration file (e.g., Case_analysis_of_mouseLR.txt) with default parameters from Defaults_for_case_analysis.txt

## Outputs

- Time-specific TF-pathway interaction network for transcriptional activators (positive TF–TG interactions), with activation time points
- Time-specific TF-pathway interaction network for transcriptional repressors (negative TF–TG interactions), with repression time points
- Interactive network map with heat maps for exploring time-specific regulatory events

## How to apply

Launch TSMiner v1.2.0 from the command line with a 1024 MB heap (java -mx1024M -jar TSMiner.jar) and load three input types: the time-series expression matrix (e.g., mouse LR RNA-seq data GSE95135), a TF-gene interaction file, and KEGG/Reactome pathway annotation files from the 'Gene annotation file' directory. Configure the run with the defaults in Defaults_for_case_analysis.txt, select the best model, and set 'Maximum number of subpaths' = 3. Run TSMiner separately on positively correlated TF–TG interactions to obtain transcriptional activators and on negatively correlated interactions (the v1.1.0 feature) to obtain transcriptional repressors, recording each TF's activation/repression time point and its significantly interacting KEGG pathways. Explore the resulting time-specific networks with TSMiner's interactive network map and heat maps. In the LR case study this workflow identified 389 activators interacting with 109 KEGG pathways and 49 repressors interacting with 47 KEGG pathways, and TSMiner showed higher sensitivity than DREM (v.2.0), hclust, k-means, STEM and WGCNA across P-value thresholds from 0 to 0.05.

## Related tools

- **TSMiner** (Core Java tool that constructs the time-specific regulatory networks, processing positively and negatively correlated TF–TG interactions separately (v1.1.0 feature); v1.2.0 used for the LR case analysis) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime environment for TSMiner; TSMiner's core algorithms and interfaces are written in Java and require Java 1.5 or later with a 1024 MB heap minimum)
- **DREM** (Comparison method (v. 2.0) run on the same LR RNA-seq data for benchmarking sensitivity and accuracy of pathway detection against TSMiner)

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- Positive-interaction run yields 389 transcriptional activators interacting with 109 KEGG pathways
- Negative-interaction run yields 49 transcriptional repressors interacting with 47 KEGG pathways
- Sensitivity sweep across P-value thresholds 0–0.05 against the 58 known LR-associated pathways ranks TSMiner highest among TSMiner, DREM (v.2.0), hclust, k-means, STEM and WGCNA
- ROC curves against the 78-pathway negative set (drawn from the 499-pathway background) give TSMiner the highest AUC among compared methods
- Network output assigns each TF an explicit activation/repression time point, verifiable against the time-series time points in the input expression matrix

## Limitations

- Fewer results are obtained from negative TF–TG interactions than positive ones because known repressors of many genes are limited; future collection of negatively correlated TF–TG interactions may address this
- DREM-family limitations that motivated TSMiner: dependence on initial parameters without effective initialization, inability to determine when TFs exert influence on targets, and inability to distinguish subnetworks occurring in specific periods
- Although experimental results supported the predictions, precise regulatory mechanisms require more rigorous evaluation
- Requires Java 1.5 or later with at least 1024 MB heap memory; running TSMiner on large time-series datasets may be memory-constrained

## Evidence
