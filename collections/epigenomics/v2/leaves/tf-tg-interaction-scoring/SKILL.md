---
name: tf-tg-interaction-scoring
description: Use when building time-specific TF-pathway regulatory networks from time-series
  RNA-seq (e.g. mouse liver regeneration data) plus a TF-gene interaction file and
  KEGG/Reactome pathway gene sets, and you need significant TF-pathway interaction
  scores.
when_to_use_negative:
- Input is a static (non-time-series) expression matrix — TSMiner is designed for
  time-series expression profiles and cannot evaluate temporal dynamics of regulators
  or pathways from clustering-style inputs.
- No TF-gene interaction information is available for the organism/tissue — the MIM
  triple-gene block scoring depends on known TF–TG interactions.
- The goal is pathway enrichment on a single gene list — clustering methods with standard
  enrichment analysis (e.g. hclust, k-means, STEM, WGCNA) are the comparison approaches,
  not this MIM-based scoring.
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: TSMiner
  role: Primary tool that assembles TF–TG–PG triple-gene blocks, computes the MIM,
    runs the permutation significance procedure, and outputs the time-specific TF-pathway
    regulatory network (v1.2.0)
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime environment required to launch TSMiner with a 1024 MB heap (Java 1.5
    or later)
- name: DREM
  role: Comparison method (v. 2.0) whose outputs are used to benchmark the sensitivity
    and accuracy of the TF-pathway interaction scoring against clustering methods
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:7f176b643b05fce68a989321163d9fc1a193acd3219c85351983120f15770bb7
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
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# tf-tg-interaction-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scoring TF–pathway and TF–TG regulatory interactions from time-series expression data using TSMiner's MIM-based triple-gene block (TF–TG–PG) construction with permutation-based significance testing, to retain only significant interactions for time-specific regulatory networks.

## When to use

Use when building time-specific TF-pathway regulatory networks from time-series RNA-seq (e.g. mouse liver regeneration data) plus a TF-gene interaction file and KEGG/Reactome pathway gene sets, and you need significant TF-pathway interaction scores.

## When NOT to use

- Input is a static (non-time-series) expression matrix — TSMiner is designed for time-series expression profiles and cannot evaluate temporal dynamics of regulators or pathways from clustering-style inputs.
- No TF-gene interaction information is available for the organism/tissue — the MIM triple-gene block scoring depends on known TF–TG interactions.
- The goal is pathway enrichment on a single gene list — clustering methods with standard enrichment analysis (e.g. hclust, k-means, STEM, WGCNA) are the comparison approaches, not this MIM-based scoring.

## Inputs

- Time-series gene expression matrix (e.g. mouse LR RNA-seq time-series data from the 'Time-series data' directory)
- TF-gene interaction file (from the 'TF-gene file' directory)
- KEGG/Reactome pathway gene sets (from the 'Gene annotation file' directory)
- Case analysis configuration file (e.g. Case_analysis_of_mouseLR.txt)

## Outputs

- TF-pathway interaction scores and permutation-based probabilities for each TF-pathway pair
- Significant TF-pathway interactions (score >= 50, probability >= 50%) separated into transcriptional activator and repressor groups
- Interactive network map of time-specific TF-pathway regulatory interactions, saved to the 'Saved model' directory

## How to apply

Load the time-series expression matrix, TF-gene interaction file, and KEGG/Reactome pathway gene sets into TSMiner, processing positively and negatively correlated TF-TG interactions separately. For each candidate pathway, divide its genes into those overlapping the TF's targets (TG) and the remaining pathway genes (PG), and assemble TF–TG–PG triple-gene blocks. Compute the mutual interaction measure (MIM) for each block and apply a permutation procedure to determine blocks with significant MIMs — a significant block indicates the TG is directly regulated by the TF and the PG is indirectly correlated with the TF through the TG. Retain TF-pathway interactions passing the default acceptance criteria (score >= 50 and probability >= 50%), applying the permutation test on pathway expression change, then generate the interactive network map of time-specific TF-pathway regulatory interactions.

## Related tools

- **TSMiner** (Primary tool that assembles TF–TG–PG triple-gene blocks, computes the MIM, runs the permutation significance procedure, and outputs the time-specific TF-pathway regulatory network (v1.2.0)) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime environment required to launch TSMiner with a 1024 MB heap (Java 1.5 or later))
- **DREM** (Comparison method (v. 2.0) whose outputs are used to benchmark the sensitivity and accuracy of the TF-pathway interaction scoring against clustering methods)

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- The set of significant KEGG pathway interactions for the mouse LR dataset reproduces the reported counts: 109 pathways significantly interacting with transcriptional activators and 47 with transcriptional repressors.
- All retained TF-pathway interactions pass the default acceptance criteria: interaction score >= 50 and probability >= 50%.
- Positively and negatively correlated TF-TG interactions are processed separately, producing separate activator and repressor network outputs.
- Sensitivity against the 58 known LR-associated pathways at P-value thresholds 0–0.05 and AUC against the 78-pathway negative set should show TSMiner ranking highest among compared methods (DREM, hclust, k-means, STEM, WGCNA).
- The saved model is present in the 'Saved model' directory and the network map opens with interactive visualization.

## Limitations

- Fewer results are obtained from negative TF–TG interactions than from positive interactions because known repressors of many genes are limited; negatively correlated TF–TG interaction data still need to be collected.
- Although experimental results supported the predictions, the precise regulatory mechanisms require more rigorous evaluation.
- TSMiner's applicability to other time-series omics data types (e.g. proteomics with PPI information) is not yet demonstrated and is left as future research.
- A minimum of 1024 MB Java heap memory is required, which may constrain large datasets.

## Evidence

- [results] TSMiner expands a time-specific TF to pathways by dividing each pathway into genes overlapping the TF's targets and the remaining pathway genes, building TF–TG–PG triple-gene blocks: [evidence span withheld — anchor status: not_found]
- [results] a significant block indicates the TG is directly regulated by the TF and the PG is indirectly correlated with the TF through the TG: [evidence span withheld — anchor status: not_found]
- [results] Retain TF-pathway interactions passing the default acceptance criteria (score >= 50 and probability >= 50%): [evidence span withheld — anchor status: not_found]
