---
name: pathway-geneset-mapping
description: Use when you have a time-series expression matrix, known TF–gene interactions,
  and pathway gene sets (KEGG/Reactome/GO), and you need to determine which pathways
  are significantly regulated by TFs activated or repressed at specific time points.
when_to_use_negative:
- You only have static (single-timepoint) expression data — TSMiner requires time-series
  expression profiles to capture time-specific TF-pathway events
- You lack a TF-gene interaction file — the pathway expansion relies on known TF-TG
  interactions to build triple-gene blocks
- Your goal is to detect TF-target edges directly without pathway-level context —
  DREM-style tree modeling may be more appropriate for TF-TG splits alone
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: TSMiner
  role: Constructs time-specific regulatory networks by integrating the time-series
    expression matrix, TF-TG interaction file, and KEGG/Reactome pathway gene sets;
    scores TF-pathway pairs with MIM and permutation tests and renders the interactive
    network map
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime environment required to launch TSMiner.jar (Java 1.5 or later, 1024
    MB heap)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:6f3ba8ca010dadec30088a1541a392b53ce8f6cbb355b740af8caccc18c164ca
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
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# pathway-geneset-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps time-series TF–target gene interactions onto curated pathway gene sets (KEGG/Reactome/Gene Ontology) to identify which biological pathways show significant mutual interactions with time-specific transcription factors, using TF–TG–PG triple-gene blocks scored by mutual information measure with permutation-based significance testing in TSMiner.

## When to use

Use when you have a time-series expression matrix, known TF–gene interactions, and pathway gene sets (KEGG/Reactome/GO), and you need to determine which pathways are significantly regulated by TFs activated or repressed at specific time points.

## When NOT to use

- You only have static (single-timepoint) expression data — TSMiner requires time-series expression profiles to capture time-specific TF-pathway events
- You lack a TF-gene interaction file — the pathway expansion relies on known TF-TG interactions to build triple-gene blocks
- Your goal is to detect TF-target edges directly without pathway-level context — DREM-style tree modeling may be more appropriate for TF-TG splits alone

## Inputs

- Time-series gene expression matrix (e.g., mouse LR RNA-seq time-course, GEO GSE95135)
- TF-gene interaction file (known TF-TG regulatory pairs, from 'TF-gene file' directory)
- KEGG/Reactome pathway gene sets and Gene Ontology files (from 'Gene annotation file' directory)

## Outputs

- TF-pathway interaction scores and permutation-based probabilities
- Set of significant TF-pathway interactions (score >= 50, probability >= 50%)
- Interactive network map of time-specific TF-pathway regulatory interactions
- Saved TSMiner model file (in 'Saved model' directory)

## How to apply

Load the time-series expression data, the TF-gene interaction file, and the KEGG/Reactome pathway gene sets from the 'Gene annotation file' directory into TSMiner. For each candidate pathway, divide its genes into those overlapping the TF's target genes (TG) and the remaining pathway genes (PG), then assemble TF–TG–PG triple-gene blocks, processing positively and negatively correlated TF-TG interactions separately. Score each TF-pathway pair using the mutual interaction measure (MIM) computed over the triple-gene blocks, with a permutation procedure to assign probabilities that a significant MIM block reflects direct TF regulation of the TG and indirect correlation of the PG through the TG. Retain TF-pathway interactions passing the default acceptance criteria of score >= 50 and probability >= 50%, then generate the interactive network map of time-specific TF-pathway regulatory interactions and save the model. A significant triple-gene block indicates the target gene is directly regulated by the TF and the pathway gene is indirectly correlated with the TF through the target gene.

## Related tools

- **TSMiner** (Constructs time-specific regulatory networks by integrating the time-series expression matrix, TF-TG interaction file, and KEGG/Reactome pathway gene sets; scores TF-pathway pairs with MIM and permutation tests and renders the interactive network map) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime environment required to launch TSMiner.jar (Java 1.5 or later, 1024 MB heap))

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- Validation against the reported LR dataset counts: 109 KEGG pathways significantly interacting with transcriptional activators and 47 with transcriptional repressors
- All retained TF-pathway interactions satisfy the default acceptance thresholds (score >= 50 and probability >= 50%)
- Identified activator/repressor TF counts reproduce the reported 389 transcriptional activators and 49 transcriptional repressors for the LR process
- Negatively correlated TF-TG interactions yield fewer significant pathways than positively correlated ones (expected, given limited known repressors)
- Resulting model is saved to the 'Saved model' directory and the network map loads with interactive visualization intact

## Limitations

- Fewer results are obtained from negative TF-TG interactions than positive ones because known repressors of many genes are limited; future collection of negatively correlated TF-TG interactions may address this
- Although experimental results strongly support the predictions, the precise regulatory mechanisms require more rigorous evaluation
- Applicability to other time-series omics data (e.g., integrating proteomics time-series data and PPI information) is not yet demonstrated

## Evidence

- [methods] dividing each pathway into genes overlapping the TF's targets and remaining pathway genes, building TF–TG–PG triple-gene blocks: [evidence span withheld — anchor status: not_found]
- [methods] Retain TF-pathway interactions passing the default acceptance criteria (score >= 50 and probability >= 50%): [evidence span withheld — anchor status: not_found]
