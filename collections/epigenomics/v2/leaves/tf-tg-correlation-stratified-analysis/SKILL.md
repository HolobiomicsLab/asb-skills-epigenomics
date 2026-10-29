---
name: tf-tg-correlation-stratified-analysis
description: Use when a time-series expression analysis must distinguish TFs that
  activate targets (positive TF–TG correlation) from TFs that repress targets (negative
  TF–TG correlation), producing separate activator and repressor networks per time
  point.
when_to_use_negative:
- The TF–TG interaction file lacks correlation-sign annotations (only undirected or
  unsigned interactions are available), so no positive/negative split can be made.
- The analysis goal is a single merged regulatory network regardless of activation/repression
  direction rather than separate time-specific activator and repressor networks.
- Input is not a time-series expression profile (e.g., static bulk or single-timepoint
  data), since TSMiner constructs time-specific networks from time-series expression
  profiles.
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: TSMiner v1.2.0
  role: Constructs the time-specific TF–pathway interaction networks; run once per
    correlation sign to yield activators (positive interactions) and repressors (negative
    interactions)
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime environment in which TSMiner is launched from the command line with
    a 1024 MB heap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:ffff6b208f961b24a9a71a843e2e5f52ec6f2cb62e4506b1b9b81bb87a60865a
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
  - TSMiner v1.2.0
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# tf-tg-correlation-stratified-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A TSMiner analysis strategy (introduced in v1.1.0) that processes positively and negatively correlated TF–TG interactions as separate input runs, so that time-specific transcriptional activators and transcriptional repressors are identified in distinct TF–pathway interaction networks rather than mixed in a single network.

## When to use

Use when a time-series expression analysis must distinguish TFs that activate targets (positive TF–TG correlation) from TFs that repress targets (negative TF–TG correlation), producing separate activator and repressor networks per time point.

## When NOT to use

- The TF–TG interaction file lacks correlation-sign annotations (only undirected or unsigned interactions are available), so no positive/negative split can be made.
- The analysis goal is a single merged regulatory network regardless of activation/repression direction rather than separate time-specific activator and repressor networks.
- Input is not a time-series expression profile (e.g., static bulk or single-timepoint data), since TSMiner constructs time-specific networks from time-series expression profiles.

## Inputs

- Time-series RNA-seq expression matrix (mouse liver regeneration dataset, GEO accession GSE95135)
- TF-gene interaction file with correlation signs, split into positively and negatively correlated subsets (from the 'TF-gene file' directory)
- KEGG pathway gene-set files (from the 'Gene annotation file' directory)
- TSMiner parameter file Defaults_for_case_analysis.txt (best model selected, Maximum number of subpaths = 3)

## Outputs

- TF–pathway interaction network table for positive TF–TG interactions: 389 transcriptional activators with activation time points and 109 interacting KEGG pathways
- TF–pathway interaction network table for negative TF–TG interactions: 49 transcriptional repressors with repression time points and 47 interacting KEGG pathways

## How to apply

Launch TSMiner v1.2.0 with 'java -mx1024M -jar TSMiner.jar' (Java 1.5+, 1024 MB heap minimum) and load the mouse liver regeneration time-series RNA-seq dataset (GSE95135), the TF-gene interaction file, and KEGG pathway files from the 'Gene annotation file' and 'TF-gene file' directories. Configure the run with the defaults in Defaults_for_case_analysis.txt, selecting the best model and 'Maximum number of subpaths' = 3. Run TSMiner once on the positively correlated TF–TG interactions to obtain transcriptional activators with activation time points and interacting KEGG pathways, then run it separately on the negatively correlated interactions (the v1.1.0 feature) to obtain transcriptional repressors with repression time points. Export both TF–pathway interaction network tables and compare their counts against the published benchmark. The rationale is that activators and repressors act through opposite regulatory signs, so merging signs would obscure the time-specific direction of regulation.

## Related tools

- **TSMiner v1.2.0** (Constructs the time-specific TF–pathway interaction networks; run once per correlation sign to yield activators (positive interactions) and repressors (negative interactions)) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime environment in which TSMiner is launched from the command line with a 1024 MB heap)

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- Positive-interaction run yields exactly 389 transcriptional activators and 109 interacting KEGG pathways.
- Negative-interaction run yields exactly 49 transcriptional repressors and 47 interacting KEGG pathways.
- Activators and repressors appear in separate output tables, each with per-time-point activation or repression assignments across the LR process.
- The run used TSMiner v1.1.0 or later (the version that introduced separate processing of positive and negative TF–TG interactions) with the Defaults_for_case_analysis.txt settings.
- The negative-interaction results are expected to be sparser than the positive-interaction results because known repressors of many genes are limited.

## Limitations

- Fewer results are obtained from negative TF–TG interactions than from positive interactions because known repressors of many genes are limited; future collection of negatively correlated TF–TG interactions may address this.
- Separate sign processing requires TSMiner v1.1.0 or later; earlier v1.0.0 releases cannot split interactions by correlation sign.
- TSMiner requires Java 1.5 or later with at least 1024 MB of Java heap memory; the case-analysis command runs on the bundled mouse LR example configuration.

## Evidence
