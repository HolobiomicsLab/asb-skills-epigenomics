---
name: permutation-significance-testing
description: 'Use when a TF–pathway interaction score from time-series expression
  data needs an empirical significance call: TSMiner derives each block''s probability
  via permutation testing and accepts interactions with score >= 50 and probability
  >= 50%.'
when_to_use_negative:
- Input is a static (non-time-series) expression matrix — TSMiner requires time-series
  profiles to evaluate temporal dynamics of regulators or pathways.
- 'Dense repressor-pathway calls are needed: known negatively correlated TF–TG interactions
  are limited, so results from negative interactions will be sparse.'
- 'Direct TF-to-pathway-gene regulation is required: a significant block certifies
  the TG as directly regulated and the PG only indirectly correlated with the TF through
  the TG.'
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: TSMiner
  role: Executes MIM scoring of TF–TG–PG triple-gene blocks and the permutation procedure
    that assigns TF-pathway interaction scores and probabilities, applying the default
    score >= 50 / probability >= 50% acceptance filters.
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime required to launch TSMiner.jar (core algorithms and interfaces written
    in Java) with a minimum 1024 MB heap for the permutation workflow.
  repo: http://www.java.com
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:3613945d0200a9f4abe6065c61fc63f73ad58512f3d67fa3e6f0a579d4b9b1c5
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# permutation-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Permutation-based empirical significance testing of TF–pathway interactions in TSMiner: TF–TG–PG triple-gene blocks are scored with the mutual interaction measure (MIM) and compared against a permutation-derived null, so only TF–pathway pairs exceeding chance (score >= 50, probability >= 50%) are accepted.

## When to use

Use when a TF–pathway interaction score from time-series expression data needs an empirical significance call: TSMiner derives each block's probability via permutation testing and accepts interactions with score >= 50 and probability >= 50%.

## When NOT to use

- Input is a static (non-time-series) expression matrix — TSMiner requires time-series profiles to evaluate temporal dynamics of regulators or pathways.
- Dense repressor-pathway calls are needed: known negatively correlated TF–TG interactions are limited, so results from negative interactions will be sparse.
- Direct TF-to-pathway-gene regulation is required: a significant block certifies the TG as directly regulated and the PG only indirectly correlated with the TF through the TG.

## Inputs

- Time-series gene expression profile (mouse LR RNA-seq, GSE95135) from the 'Time-series data' directory
- TF–gene (TF–TG) interaction file from the 'TF-gene file' directory
- KEGG/Reactome pathway gene sets from the 'Gene annotation file' directory
- Candidate TF–pathway pairs partitioned into TG (TF targets) and PG (remaining pathway genes)

## Outputs

- MIM scores for each TF–TG–PG triple-gene block with permutation-derived significance probabilities
- TF-pathway interaction scores and probabilities for each TF–pathway pair
- Set of accepted significant TF–pathway interactions (score >= 50, probability >= 50%) and the saved interactive network model

## How to apply

Load the time-series expression matrix (e.g. the mouse LR RNA-seq dataset GSE95135), the TF–gene interaction file, and the KEGG/Reactome pathway gene sets from the 'Gene annotation file' directory into TSMiner, running under Java with at least 1024 MB heap. For each candidate pathway, partition its genes into target genes (TG) overlapping the time-specific TF's targets and the remaining pathway genes (PG), then assemble TF–TG–PG triple-gene blocks, processing positively and negatively correlated TF–TG interactions separately. Compute the mutual interaction measure (MIM) for each block, where a significant block indicates the TG is directly regulated by the TF and the PG is indirectly correlated with the TF through the TG. Run the permutation procedure on pathway expression change to build an empirical null and assign each TF–pathway pair an interaction score and probability. Retain TF–pathway interactions passing the default acceptance criteria (score >= 50 and probability >= 50%), then validate the LR case against the reported 109 activator-interacting and 47 repressor-interacting KEGG pathways before saving the model.

## Related tools

- **TSMiner** (Executes MIM scoring of TF–TG–PG triple-gene blocks and the permutation procedure that assigns TF-pathway interaction scores and probabilities, applying the default score >= 50 / probability >= 50% acceptance filters.) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime required to launch TSMiner.jar (core algorithms and interfaces written in Java) with a minimum 1024 MB heap for the permutation workflow.) — http://www.java.com

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- On the mouse LR default case, significant KEGG pathway interactions should reproduce the reported counts: 109 pathways interacting with transcriptional activators and 47 with transcriptional repressors.
- Every retained TF–pathway interaction passes the default acceptance criteria of score >= 50 and probability >= 50%.
- Positively and negatively correlated TF–TG interactions were scored in separate runs (TSMiner v1.1.0+ behavior), and each called block exceeds the MIM threshold set by the permutation procedure.
- Probabilities are interpretable as percentages from the permutation null (probability >= 50%), not as p-value tails.
- The run completes under java -mx1024M -jar TSMiner.jar and the resulting model is saved in the 'Saved model' directory.

## Limitations

- Fewer results are obtained from negative TF–TG interactions than from positive interactions because the known repressors of many genes are limited; future collection of negatively correlated TF–TG interactions may help.
- A significant triple-gene block indicates the pathway gene is indirectly correlated with the TF through the target gene, and the precise regulatory mechanisms require more rigorous evaluation despite experimental support.
- Requires Java 1.5 or later and a minimum 1024 MB Java heap; testing is bounded by the KEGG/Reactome and GO gene-annotation files shipped in the 'Gene annotation file' directory.

## Evidence
