---
name: kegg-pathway-annotation-mapping
description: Use when a time-series gene expression analysis needs pathway-level interpretation
  of TF–target networks, and KEGG/Reactome/GO gene-set files must be joined to genes
  and TFs to identify pathway–regulator interactions.
when_to_use_negative:
- The input is a single-condition expression dataset with no time-series structure,
  since TSMiner and this mapping target time-specific regulatory events
- Pathway-level interpretation is not needed and only gene-level TF–target dynamics
  are of interest
- No curated pathway gene-set files (KEGG/Reactome/GO) are available for the organism
  under study
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0621
tools:
- name: TSMiner
  role: Consumes the KEGG/Reactome/GO pathway gene-set files as one of its three input
    types to build time-specific TF–pathway interaction networks
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime for TSMiner (Java 1.5 or later, minimum 1024 MB heap) used to execute
    the pathway-mapping case analysis
provenance:
  source_task_ids:
  - task_008
  - task_007
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:34118ea41bd97d70ddffc22e0924196a4c2fb5bed0c5a2f9bcad08a88b29492d
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# kegg-pathway-annotation-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps genes from a time-series expression dataset onto KEGG (or Reactome/GO) pathway gene sets supplied as annotation files, so that TF–pathway interactions and time-specific pathway activity can be extracted from regulatory network tools such as TSMiner.

## When to use

Use when a time-series gene expression analysis needs pathway-level interpretation of TF–target networks, and KEGG/Reactome/GO gene-set files must be joined to genes and TFs to identify pathway–regulator interactions.

## When NOT to use

- The input is a single-condition expression dataset with no time-series structure, since TSMiner and this mapping target time-specific regulatory events
- Pathway-level interpretation is not needed and only gene-level TF–target dynamics are of interest
- No curated pathway gene-set files (KEGG/Reactome/GO) are available for the organism under study

## Inputs

- KEGG pathway gene-set files from the 'Gene annotation file' directory (KEGG/Reactome/GO annotation files)
- Time-series gene expression matrix (e.g. mouse LR RNA-seq dataset GSE95135) from the 'Time-series data' directory
- TF–gene interaction file from the 'TF-gene file' directory

## Outputs

- Time-specific TF–pathway interaction network tables (activators: 109 KEGG pathways; repressors: 47 KEGG pathways)
- Time-point-wise pathway assignments with TF overlap counts (e.g. immune_cascade_summary.tsv)
- Benchmark table of pathway detection sensitivity/AUC per method (e.g. tsminer_pathway_benchmark_results.csv)

## How to apply

Load the KEGG/Reactome pathway gene-set files from the tool's 'Gene annotation file' directory (e.g. TSMiner's packaged annotation files) and supply them, together with the time-series expression matrix and the TF–gene interaction file, as the three input types required by TSMiner, following the data input rules in TSMiner manual.pdf. TSMiner uses these gene sets to evaluate which biological pathways show significant mutual interactions with TFs that are activated or repressed at each time point, producing time-specific TF–pathway interaction networks. For the mouse liver regeneration (GSE95135) case analysis, run with default parameters from Defaults_for_case_analysis.txt and maximum number of subpaths = 3, processing positively and negatively correlated TF–TG interactions separately (v1.1.0 behaviour). Validate the mapping by checking that the positive-interaction table contains 109 interacting KEGG pathways and the negative-interaction table contains 47, and, when benchmarking against known biology, use the 58 known LR-associated pathways as the positive set and a 78-pathway negative set drawn from the 499-pathway background.

## Related tools

- **TSMiner** (Consumes the KEGG/Reactome/GO pathway gene-set files as one of its three input types to build time-specific TF–pathway interaction networks) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime for TSMiner (Java 1.5 or later, minimum 1024 MB heap) used to execute the pathway-mapping case analysis)

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- Positive TF–TG interaction run yields 389 transcriptional activators interacting with exactly 109 KEGG pathways
- Negative TF–TG interaction run yields 49 transcriptional repressors interacting with exactly 47 KEGG pathways
- Late-stage immune cascade reproduces the reported overlaps: 61 and 38 TFs with 33 shared between p53 signalling and apoptosis, then 7 shared and 17 shared between consecutive cascade stages
- Benchmark against the 58 known LR-associated pathways (negative set: 78 pathways from the 499-pathway background) reproduces TSMiner ranking highest in sensitivity and AUC versus hclust, k-means, STEM and WGCNA
- Output TSVs contain time-point assignments consistent with the cascade: p53 signalling and apoptosis at 36 h, immune/chemokine pathways at 48 h, cell adhesion molecules and Th1/Th2 differentiation at 168 h

## Limitations

- Pathway interpretation is restricted to the provided KEGG/Reactome/GO gene sets; negatively correlated TF–TG interaction data are still limited, yielding fewer repressor–pathway results (47 vs 109 pathways)
- Precise regulatory mechanisms inferred from pathway–TF interactions require more rigorous experimental evaluation
- Requires Java 1.5 or later and a minimum 1024 MB Java heap; TSMiner is platform independent but the case analysis depends on packaged example files

## Evidence
