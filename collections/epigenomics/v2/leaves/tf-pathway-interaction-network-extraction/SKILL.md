---
name: tf-pathway-interaction-network-extraction
description: Use when time-series gene expression data (e.g. mouse LR RNA-seq across
  post-PH time points) must be converted into time-specific TF–pathway regulatory
  networks to identify which TFs regulate pathways at each stage.
when_to_use_negative:
- Expression data are not time-series — TSMiner captures dynamic TF–pathway events
  specific to time-course profiles; static clustering or single-condition differential
  analyses are the wrong fit.
- The goal is cluster/module detection alone — clustering methods cannot evaluate
  the temporal dynamics of regulators or pathways, which is exactly what this skill
  provides instead.
- Only negatively correlated TF–TG interactions are required and no repressor interaction
  data are available — results from negative interactions will be sparse because known
  repressors of many genes are limited.
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: TSMiner
  role: Primary tool that constructs time-specific TF–pathway regulatory networks
    from time-series expression profiles, TF–TG interactions, and pathway gene sets,
    with interactive network map visualization
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime environment; TSMiner's core algorithms and interfaces are written
    in Java and it is launched with a minimum 1024 MB heap
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:e668cfdf62f5d1ad98caf0ad0ac660767d57a551e9b0f8f37e02e0878f3c2384
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# tf-pathway-interaction-network-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract time-specific transcription factor (TF)–pathway interaction networks from time-series RNA-seq expression profiles using TSMiner, integrating TF–gene interaction data and KEGG/Reactome pathway gene sets to reconstruct regulatory cascades such as the late-stage mouse liver regeneration immune cascade.

## When to use

Use when time-series gene expression data (e.g. mouse LR RNA-seq across post-PH time points) must be converted into time-specific TF–pathway regulatory networks to identify which TFs regulate pathways at each stage.

## When NOT to use

- Expression data are not time-series — TSMiner captures dynamic TF–pathway events specific to time-course profiles; static clustering or single-condition differential analyses are the wrong fit.
- The goal is cluster/module detection alone — clustering methods cannot evaluate the temporal dynamics of regulators or pathways, which is exactly what this skill provides instead.
- Only negatively correlated TF–TG interactions are required and no repressor interaction data are available — results from negative interactions will be sparse because known repressors of many genes are limited.

## Inputs

- Time-series RNA-seq expression dataset (e.g. mouse liver regeneration GSE95135) from the 'Time-series data' directory
- TF–gene (TF–TG) interaction data from the 'TF-gene file' directory
- Pathway gene sets (KEGG/Reactome/Gene Ontology files) from the 'Gene annotation file' directory
- Case analysis settings file (e.g. Case_analysis_of_mouseLR.txt with Defaults_for_case_analysis.txt)

## Outputs

- Time-specific TF–pathway regulatory networks (activated and repressed) per time point
- Per-stage TF sets significantly interacting with stage pathways and pairwise TF overlap counts (e.g. 61 and 38 TFs with 33 shared)
- Interactive network maps and a saved model of the reconstructed case-analysis networks
- Summary table (e.g. immune_cascade_summary.tsv) of time-point-wise pathway assignments and TF overlap counts

## How to apply

Launch TSMiner from the tsminer-tool directory with 'java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt', which loads the mouse LR case analysis with the default settings in Defaults_for_case_analysis.txt (requires Java 1.5+ and at least 1024 MB heap). Load three inputs per TSMiner manual.pdf: the time-series expression matrix (e.g. GSE95135 LR RNA-seq), the TF–gene interaction file, and the pathway gene annotation file (KEGG/Reactome from the 'Gene annotation file' directory). Press 'Run' to construct time-specific regulatory networks, with positively and negatively correlated TF–TG interactions processed separately as of TSMiner v1.1.0. Explore the interactive network maps to extract cascades stage by stage (e.g. p53 signalling and apoptosis at 36 h; chemokine signalling, leukocyte transendothelial migration, Fc gamma R-mediated phagocytosis, endocytosis and phagosome at 48 h; cell adhesion molecules and Th1/Th2 differentiation at 168 h). For each stage, record the TF sets significantly interacting with the stage pathways and their pairwise overlaps (61 and 38 TFs with 33 shared between the first two apoptosis-related stages; 7 and 17 shared at later transitions), writing the time-point-wise assignments and TF overlap counts to a summary file and saving the reconstructed networks as a saved model.

## Related tools

- **TSMiner** (Primary tool that constructs time-specific TF–pathway regulatory networks from time-series expression profiles, TF–TG interactions, and pathway gene sets, with interactive network map visualization) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime environment; TSMiner's core algorithms and interfaces are written in Java and it is launched with a minimum 1024 MB heap)

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- The two late-stage apoptosis-related pathways (p53 signalling and apoptosis) yield 61 and 38 significantly interacting TFs with 33 overlapping TFs, matching the published cascade counts.
- Subsequent cascade stages reproduce the expected smaller overlaps (7 shared TFs between 48 h immune pathways; 17 shared TFs between 168 h pathways).
- Networks are time-point-specific: TFs are reported as activated or repressed at a specific time, and pathways are assigned to their correct cascade stages (36 h, 48 h, 168 h).
- Positively and negatively correlated TF–TG interactions are processed separately (v1.1.0 behaviour), with fewer repressor results expected due to limited known repressors.
- Reconstructed networks are saved as a saved model in the 'Saved model' directory and the cascade summary is written to a TSV with time-point-wise pathway assignments.

## Limitations

- Fewer results are obtained from negative TF–TG interactions than positive interactions because known repressors of many genes are limited; negatively correlated TF–TG interaction data still need to be collected.
- Although experimental results strongly supported the predictions, the precise regulatory mechanisms require more rigorous evaluation.
- Applicability to other time-series omics data (e.g. integrating proteomics time-series data with PPI information) is not yet demonstrated and is left as future research.
- TSMiner requires Java 1.5 or later and a minimum 1024 MB Java heap; the case-analysis workflow depends on the settings file Defaults_for_case_analysis.txt.

## Evidence
