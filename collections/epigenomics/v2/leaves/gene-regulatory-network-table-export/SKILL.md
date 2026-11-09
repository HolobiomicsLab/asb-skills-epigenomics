---
name: gene-regulatory-network-table-export
description: Use when a time-series expression dataset (e.g., mouse liver regeneration
  GSE95135) has been mined with TSMiner and you need machine-readable tables of time-specific
  activators/repressors and their interacting KEGG pathways.
when_to_use_negative:
- The expression data are not time-series, since TSMiner constructs time-specific
  regulatory networks specifically from time-series expression profiles
- You only have clustering or static co-expression analysis goals — clustering methods
  cannot evaluate the temporal dynamics of regulators or pathways, but TSMiner's export
  targets time-specific TF–pathway events
- The input TF–TG interaction file mixes positively and negatively correlated interactions
  without separation, since TSMiner requires them processed separately
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: TSMiner
  role: Primary tool that constructs the time-specific TF–pathway regulatory networks
    and from which the activator and repressor network tables are exported
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime environment; TSMiner core algorithms and interfaces are written in
    Java and launched with a 1024 MB heap minimum
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:e7cc5a200d145095705f7dbbb22f00df9a1cec9ffd215f94d915ff3770bd3fee
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gene-regulatory-network-table-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Export time-specific TF–pathway interaction network tables from TSMiner runs on time-series RNA-seq data, separately for positively correlated (activators) and negatively correlated (repressors) TF–TG interactions. Tables list each transcription factor with its activation/repression time point and the significantly interacting KEGG pathways.

## When to use

Use when a time-series expression dataset (e.g., mouse liver regeneration GSE95135) has been mined with TSMiner and you need machine-readable tables of time-specific activators/repressors and their interacting KEGG pathways.

## When NOT to use

- The expression data are not time-series, since TSMiner constructs time-specific regulatory networks specifically from time-series expression profiles
- You only have clustering or static co-expression analysis goals — clustering methods cannot evaluate the temporal dynamics of regulators or pathways, but TSMiner's export targets time-specific TF–pathway events
- The input TF–TG interaction file mixes positively and negatively correlated interactions without separation, since TSMiner requires them processed separately

## Inputs

- Time-series RNA-seq expression dataset (e.g., GSE95135, mouse liver regeneration after partial hepatectomy)
- TF–gene (TF–TG) interaction file, split into positively and negatively correlated interactions
- KEGG pathway annotation files from the TSMiner 'Gene annotation file' directory
- Default parameter file Defaults_for_case_analysis.txt

## Outputs

- TF–pathway interaction network table for positively correlated interactions: transcriptional activators with activation time points and significantly interacting KEGG pathways (expected: 389 activators, 109 pathways)
- TF–pathway interaction network table for negatively correlated interactions: transcriptional repressors with repression time points and interacting KEGG pathways (expected: 49 repressors, 47 pathways)

## How to apply

Start TSMiner v1.2.0 with 'java -mx1024M -jar TSMiner.jar' (Java 1.5+, 1024 MB heap minimum), then load the time-series expression matrix, the TF-gene interaction file, and the KEGG pathway files from the 'Gene annotation file' and 'TF-gene file' directories, with data input and parameter settings per TSMiner manual.pdf. Configure the run with the defaults in Defaults_for_case_analysis.txt, selecting the best model and setting 'Maximum number of subpaths' = 3. Run TSMiner once on the positively correlated TF–TG interactions to export the activator table with activation time points and interacting KEGG pathways, then run it separately on the negatively correlated interactions (a v1.1.0 feature) to export the repressor table with repression time points. Validation for the mouse LR case: the positive-interaction table must contain 389 activators interacting with 109 KEGG pathways, and the negative-interaction table must contain 49 repressors interacting with 47 KEGG pathways.

## Related tools

- **TSMiner** (Primary tool that constructs the time-specific TF–pathway regulatory networks and from which the activator and repressor network tables are exported) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime environment; TSMiner core algorithms and interfaces are written in Java and launched with a 1024 MB heap minimum)

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- Positive-interaction table row count equals 389 transcriptional activators
- Negative-interaction table row count equals 49 transcriptional repressors
- Positive-interaction table references 109 significantly interacting KEGG pathways; negative-interaction table references 47
- Each exported TF entry is annotated with a specific activation or repression time point during the liver regeneration time course
- Tables are produced from two separate runs (v1.1.0+ feature): one on positively correlated and one on negatively correlated TF–TG interactions

## Limitations

- Fewer results are obtained from negative TF–TG interactions than positive interactions because known repressors of many genes are limited; future collection of negatively correlated TF–TG interactions may help
- TSMiner requires Java 1.5 or later and a minimum of 1024 MB Java heap memory
- Precise regulatory mechanisms require more rigorous evaluation, although experimental results supported the predictions

## Evidence

- [abstract] identified 389 transcriptional activators and 49 transcriptional repressors that were either activated or repressed across the LR process: "identified 389 transcriptional activators and 49 transcriptional repressors that were either activated or repressed across the LR process"
