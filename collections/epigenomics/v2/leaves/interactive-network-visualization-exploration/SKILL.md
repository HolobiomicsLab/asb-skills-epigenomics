---
name: interactive-network-visualization-exploration
description: Use when a time-series regulatory network has been constructed in TSMiner
  and you need to visually navigate time-specific network maps to identify which TFs
  interact with pathways and quantify overlaps between pathway-connected TF sets.
when_to_use_negative:
- The expression data are not time-series, since TSMiner networks are specifically
  time-point-resolved and clustering-only designs cannot evaluate temporal dynamics
  of regulators or pathways.
- You need repressor (negative TF–TG) interaction results as the primary signal; known
  repressors are limited, so negative-interaction networks are much sparser.
- You require a static whole-network topology analysis rather than identifying which
  TFs interact with specific pathways at specific time points.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: TSMiner
  role: Provides the interactive network map with extensive visualization operations
    used to explore time-specific regulatory networks and extract TF–pathway interactions;
    launched as a Java jar with 1024 MB heap.
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime environment required to launch TSMiner (Java 1.5 or later, minimum
    1024 MB Java heap).
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:4d4c8119ea1d5da73d34a9e38f2a48116d44834687ecf2b136dfe0f99a2c5475
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# interactive-network-visualization-exploration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Explores TSMiner's interactive time-specific TF–pathway network maps to trace regulatory cascades (e.g., the late-stage mouse liver regeneration immune cascade at 36 h, 48 h and 168 h) and record which transcription factors are shared between pathways at each stage.

## When to use

Use when a time-series regulatory network has been constructed in TSMiner and you need to visually navigate time-specific network maps to identify which TFs interact with pathways and quantify overlaps between pathway-connected TF sets.

## When NOT to use

- The expression data are not time-series, since TSMiner networks are specifically time-point-resolved and clustering-only designs cannot evaluate temporal dynamics of regulators or pathways.
- You need repressor (negative TF–TG) interaction results as the primary signal; known repressors are limited, so negative-interaction networks are much sparser.
- You require a static whole-network topology analysis rather than identifying which TFs interact with specific pathways at specific time points.

## Inputs

- TSMiner time-specific regulatory network model built from mouse LR time-series RNA-seq expression data (GSE95135)
- TF–gene interaction data from the 'TF-gene file' directory
- KEGG pathway gene sets from the 'Gene annotation file' directory
- Saved model from the 'Saved model' directory for reloading case analyses

## Outputs

- immune_cascade_summary.tsv with time-point-wise pathway assignments and TF overlap counts
- Saved TSMiner model of the reconstructed case-analysis networks
- Per-stage TF sets and shared-TF counts (e.g., 61 and 38 TFs with 33 overlapping between 36 h pathways)

## How to apply

After launching TSMiner with the mouse LR case analysis (java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt) and pressing 'Run', open the interactive network maps for each time point, using the heatmap mouse-movement response added in v1.1.0 to inspect TF–TG correlations. For each stage of the immune cascade, record the pathway assignments: p53 signalling and apoptosis at 36 h; chemokine signalling, leukocyte transendothelial migration, Fc gamma R-mediated phagocytosis, endocytosis and phagosome at 48 h; cell adhesion molecules and Th1/Th2 cell differentiation at 168 h. For each consecutive stage, extract the TF sets significantly interacting with the stage pathways and compute their overlaps (e.g., 61 and 38 TFs with 33 shared between the first two stages; 7 shared; 17 shared). Write the time-point-wise pathway assignments and TF overlap counts to immune_cascade_summary.tsv and save the reconstructed case-analysis networks as a saved model in the 'Saved model' directory format. Inspect positively and negatively correlated TF–TG interactions separately, since v1.1.0 processes them separately and negative results are sparser because known repressors are limited.

## Related tools

- **TSMiner** (Provides the interactive network map with extensive visualization operations used to explore time-specific regulatory networks and extract TF–pathway interactions; launched as a Java jar with 1024 MB heap.) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime environment required to launch TSMiner (Java 1.5 or later, minimum 1024 MB Java heap).)

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt  # then press 'Run' and explore the network maps interactively
```

## Evaluation signals

- The extracted cascade matches the documented stage structure: p53 signalling and apoptosis at 36 h; immune/migration/phagocytosis pathways at 48 h; cell adhesion molecules and Th1/Th2 differentiation at 168 h.
- TF overlap counts reproduce the reported values: 61 and 38 TFs with 33 shared at the first stage pair, then 7 shared and 17 shared for consecutive stages.
- immune_cascade_summary.tsv contains one row per time point with pathway assignments and overlap counts, and the case-analysis network is saved as a loadable TSMiner saved model.
- Positively and negatively correlated TF–TG interactions were inspected separately, consistent with v1.1.0 behaviour.
- Network elements respond to mouse movement on the heat maps, confirming the interactive visualization layer is functioning.

## Limitations

- Fewer results are obtained from negative TF–TG interactions than positive interactions because the known repressors of many genes are limited.
- Precise regulatory mechanisms require more rigorous evaluation, even though experimental results supported the predictions.
- Requires Java 1.5 or later and a minimum 1024 MB Java heap; large networks may be constrained by heap size.

## Evidence
