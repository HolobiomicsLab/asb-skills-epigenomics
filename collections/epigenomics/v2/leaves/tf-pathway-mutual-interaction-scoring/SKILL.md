---
name: tf-pathway-mutual-interaction-scoring
description: Use when you have time-series expression profiles plus TF–TG interactions
  and pathway gene sets (KEGG/Reactome) and must decide which pathways significantly
  interact with a time-specific TF.
when_to_use_negative:
- When you only have static (single-timepoint) expression data — TSMiner's MIM scoring
  is designed for time-series expression profiles
- When no TF–TG interaction information is available, since triple-gene blocks require
  known TF–target pairs
- When you need direct TF–TF or protein–protein regulatory events rather than TF-to-pathway
  interactions via target genes
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: TSMiner
  role: Implements triple-gene block construction, MIM calculation, and permutation
    testing to score TF–pathway interactions from time-series data
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime environment required to execute the TSMiner.jar tool (Java 1.5 or
    later, 1024 MB heap)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:d3bd0f861d9b7c7bd37dad281af77132c227eba3df012b1ed4e935e18a6dbe86
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# tf-pathway-mutual-interaction-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores whether a pathway is significantly correlated with a time-specific transcription factor using TSMiner's mutual interaction measure (MIM) computed on TF–TG–PG triple-gene blocks, with permutation-based significance testing to retain genuine TF–pathway regulatory interactions.

## When to use

Use when you have time-series expression profiles plus TF–TG interactions and pathway gene sets (KEGG/Reactome) and must decide which pathways significantly interact with a time-specific TF.

## When NOT to use

- When you only have static (single-timepoint) expression data — TSMiner's MIM scoring is designed for time-series expression profiles
- When no TF–TG interaction information is available, since triple-gene blocks require known TF–target pairs
- When you need direct TF–TF or protein–protein regulatory events rather than TF-to-pathway interactions via target genes

## Inputs

- Time-series gene expression matrix (e.g. mouse LR RNA-seq time-series data from the 'Time-series data' directory)
- TF–gene (TF–TG) interaction file from the 'TF-gene file' directory
- KEGG/Reactome pathway gene sets and gene annotation files from the 'Gene annotation file' directory
- Default parameter file (Defaults_for_case_analysis.txt)

## Outputs

- TF–pathway interaction scores and permutation-based probabilities
- Significant TF–pathway interactions passing score >= 50 and probability >= 50%
- Interactive network map of time-specific TF–pathway regulatory interactions, saved to the 'Saved model' directory

## How to apply

Divide each candidate pathway into genes overlapping the time-specific TF's targets and the remaining pathway genes, then assemble candidate triple-gene blocks of (TF, target gene, pathway gene). Process positively and negatively correlated TF–TG interactions separately, calculating the mutual interaction measure (MIM) for each block and using a permutation procedure to determine blocks with significant MIMs — a significant block indicates the TG is directly regulated by the TF and the PG is indirectly correlated with the TF through the TG. Aggregate block significance into TF-pathway interaction scores and probabilities, applying the permutation test on pathway expression change. Retain TF–pathway interactions passing the default acceptance criteria of score >= 50 and probability >= 50%. For the mouse liver regeneration (LR) case study, run TSMiner with the default parameters from Defaults_for_case_analysis.txt and expect the significant KEGG pathway interactions to reproduce 109 pathways interacting with transcriptional activators and 47 with transcriptional repressors.

## Related tools

- **TSMiner** (Implements triple-gene block construction, MIM calculation, and permutation testing to score TF–pathway interactions from time-series data) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime environment required to execute the TSMiner.jar tool (Java 1.5 or later, 1024 MB heap))

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- The set of significant KEGG pathway interactions reproduces the reported counts for the LR dataset: 109 pathways interacting with transcriptional activators and 47 with transcriptional repressors
- All retained TF–pathway interactions meet the default acceptance criteria (score >= 50 and probability >= 50%)
- Positively and negatively correlated TF–TG interactions are scored in separate processing streams
- Only blocks flagged as significant by the permutation procedure contribute to significant TF–pathway calls
- The saved model in the 'Saved model' directory opens as an interactive network map in TSMiner

## Limitations

- Fewer results are obtained from negative TF–TG interactions than from positive interactions because the known repressors of many genes are limited; future collection of negatively correlated TF–TG interactions may help address this issue
- Although the experimental results supported the predictions, the precise regulatory mechanisms require more rigorous evaluation
- Requires at least 1024 MB of Java heap memory; TSMiner.jar must be launched with e.g. -mx1024M

## Evidence

- [methods] TSMiner expands a time-specific TF to pathways by dividing each pathway into genes overlapping the TF's targets and the remaining pathway genes, building TF–TG–PG triple-gene blocks: [evidence span withheld — anchor status: not_found]
- [methods] Retain TF-pathway interactions passing the default acceptance criteria (score >= 50 and probability >= 50%): [evidence span withheld — anchor status: not_found]
- [readme] from a command line while in the TSMiner directory type the command: java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt: [evidence span withheld — anchor status: external_doc]
