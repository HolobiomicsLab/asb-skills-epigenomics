---
name: tf-overlap-set-comparison
description: Use when a time-specific TF–pathway interaction network has been constructed
  (e.g., by TSMiner from time-series expression data) and you need to measure how
  many TFs are shared between consecutive-stage pathways to assemble a regulatory
  cascade.
when_to_use_negative:
- Expression data are not time-series (TSMiner constructs time-specific networks and
  cannot be applied to single-condition or static interaction data).
- The goal is TF–target binding prediction rather than quantifying overlap between
  already-predicted TF–pathway associations.
- Negatively correlated TF–TG interactions dominate the question of interest, since
  known repressors are limited and fewer results are obtained from negative interactions.
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: TSMiner
  role: Constructs the time-specific regulatory networks whose interacting TF sets
    are compared for overlaps across cascade stages; run with the mouse LR case-analysis
    defaults
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime environment used to launch TSMiner (Java 1.5 or later, minimum 1024
    MB Java heap)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:9511529ba72a2eeb960ec95c8fe53cebf2c2ba3570dfa688017a7e56c9b29982
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
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# tf-overlap-set-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compare the sets of transcription factors (TFs) significantly interacting with each pathway in a time-specific regulatory cascade to quantify shared regulators and infer which pathways belong to the same regulatory cascade, as demonstrated for the late-stage liver regeneration (LR) immune cascade built with TSMiner.

## When to use

Use when a time-specific TF–pathway interaction network has been constructed (e.g., by TSMiner from time-series expression data) and you need to measure how many TFs are shared between consecutive-stage pathways to assemble a regulatory cascade.

## When NOT to use

- Expression data are not time-series (TSMiner constructs time-specific networks and cannot be applied to single-condition or static interaction data).
- The goal is TF–target binding prediction rather than quantifying overlap between already-predicted TF–pathway associations.
- Negatively correlated TF–TG interactions dominate the question of interest, since known repressors are limited and fewer results are obtained from negative interactions.

## Inputs

- Time-specific TF–pathway interaction networks from TSMiner (mouse LR case analysis model)
- Time-series LR RNA-seq expression dataset (GSE95135) as loaded via the 'Time-series data' directory
- TF–gene interaction data from the package's 'TF-gene file' directory
- KEGG pathway gene sets from the package's 'Gene annotation file' directory

## Outputs

- Per-stage TF sets significantly interacting with each cascade pathway (e.g., 61 TFs for p53 signalling, 38 for apoptosis)
- Pairwise TF overlap counts between consecutive cascade stages (33, 7, 17 shared TFs)
- immune_cascade_summary.tsv with time-point-wise pathway assignments and TF overlap counts
- Saved TSMiner case-analysis network model

## How to apply

First construct the time-specific regulatory networks with TSMiner by launching it with the mouse LR case analysis settings (java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt) and loading the LR time-series RNA-seq data, the TF–gene interaction file and the KEGG pathway gene sets. Extract, per cascade stage, the TF sets significantly interacting with each pathway: p53 signalling and apoptosis at 36 h; chemokine signalling, leukocyte transendothelial migration, Fc gamma R-mediated phagocytosis, endocytosis and phagosome at 48 h; cell adhesion molecules and Th1/Th2 cell differentiation at 168 h. For each consecutive stage pair, compute the overlap size between the interacting TF sets, recording e.g. 61 and 38 TFs with 33 shared between the first two stages, then 7 and 17 shared for later stage pairs. Large overlaps between pathways at the same time point indicate that pathways in the same regulatory cascade share common regulators; write the time-point-wise pathway assignments and TF overlap counts to a summary file such as immune_cascade_summary.tsv. Note that positively and negatively correlated TF–TG interactions are processed separately (v1.1.0 behaviour), so overlap counts should be reported consistently per interaction sign.

## Related tools

- **TSMiner** (Constructs the time-specific regulatory networks whose interacting TF sets are compared for overlaps across cascade stages; run with the mouse LR case-analysis defaults) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime environment used to launch TSMiner (Java 1.5 or later, minimum 1024 MB Java heap))

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- The first two immune-cascade stages (p53 signalling and apoptosis, 36 h) yield 61 and 38 interacting TFs with 33 overlapping TFs, matching the published cascade.
- Consecutive-stage overlap counts for the full cascade reproduce 33, 7 and 17 shared TFs across the 36 h → 48 h → 168 h stages.
- TF sets are separated by interaction sign because TSMiner (v1.1.0+) processes positively and negatively correlated TF–TG interactions separately.
- immune_cascade_summary.tsv contains one row per time point with pathway assignments and overlap counts, and no pathway is assigned to the wrong time point (e.g., chemokine signalling must appear at 48 h, not 36 h).
- The reconstructed case-analysis network can be saved and reloaded as a TSMiner saved model.

## Limitations

- Fewer results are obtained from negative TF–TG interactions than from positive interactions because the known repressors of many genes are limited; future collection of negatively correlated TF–TG interactions may help.
- All experimental results support the predictions, but the precise regulatory mechanisms require more rigorous evaluation.
- TSMiner requires at least 1024 MB Java heap memory and Java 1.5 or later, and is distributed as a Java GUI jar rather than a scriptable pipeline.

## Evidence

- [results] The two apoptosis-related pathways significantly interacted with 61 and 38 TFs, with 33 overlapping TFs, indicating that pathways involved in the same regulatory cascade share common regulators.: [evidence span withheld — anchor status: not_found]
