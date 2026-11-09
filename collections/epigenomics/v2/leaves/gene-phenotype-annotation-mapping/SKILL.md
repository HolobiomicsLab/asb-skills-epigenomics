---
name: gene-phenotype-annotation-mapping
description: Use when a candidate gene must be scored for its association with a structural-variant-associated
  phenotype group (head_neck, limbs, neurodevelopmental, cardiovascular) using HPO/MPO
  gene-phenotype annotations.
when_to_use_negative:
- When the phenotype of interest falls outside the phenotype groups POSTRE supports
  (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system,
  pituitary) — the module only scores those predefined groups.
- When the goal is to predict SV pathogenic mechanisms (coding vs long-range) — POSTRE's
  cell type-specific enhancer/TAD analysis handles that, not this annotation-mapping
  step.
- When experimental validation of a gene-phenotype link is needed — this skill only
  aggregates curated ontology annotations, not functional evidence.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: POSTRE
  role: Software whose genePhenoScore module is reconstructed; consumes the gene-phenotype
    score table during SV pathogenicity prediction
  repo: https://github.com/vicsanga/Postre
- name: R
  role: Implementation language for parsing the OBO DAGs and annotation files and
    writing the tab-separated genePhenoScore.tsv output
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:9151e1a5e440db46f56371b37e9ea6a57fe6f2cb2d962a12e6252d4e0c2f1ab3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/vicsanga/Postre
  repo_url: https://github.com/vicsanga/Postre
  tools:
  - POSTRE
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gene-phenotype-annotation-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs POSTRE's genePhenoScore module by parsing HPO (hp.obo, genes_to_phenotype.txt) and MPO (mp.obo, HMD_HumanPhenotype.rpt) resources to link human genes to the four POSTRE reference phenotype groups (head_neck, limbs, neurodevelopmental, cardiovascular) and score each gene-phenotype pair as 1 (human HPO), 0.5 (mouse ortholog MPO only, Standard Running Mode), or 0.

## When to use

Use when a candidate gene must be scored for its association with a structural-variant-associated phenotype group (head_neck, limbs, neurodevelopmental, cardiovascular) using HPO/MPO gene-phenotype annotations.

## When NOT to use

- When the phenotype of interest falls outside the phenotype groups POSTRE supports (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary) — the module only scores those predefined groups.
- When the goal is to predict SV pathogenic mechanisms (coding vs long-range) — POSTRE's cell type-specific enhancer/TAD analysis handles that, not this annotation-mapping step.
- When experimental validation of a gene-phenotype link is needed — this skill only aggregates curated ontology annotations, not functional evidence.

## Inputs

- hp.obo (Human Phenotype Ontology DAG)
- genes_to_phenotype.txt (HPO gene-to-phenotype annotations)
- mp.obo (Mammalian Phenotype Ontology DAG)
- HMD_HumanPhenotype.rpt (MGD human-mouse ortholog phenotype mapping)

## Outputs

- genePhenoScore.tsv — tab-separated table with one row per gene and phenotype group, scores in {0, 0.5, 1}
- Per-group HPO/MPO descendant term sets (term closure under each root term)

## How to apply

Download hp.obo and genes_to_phenotype.txt from the HPO website, and mp.obo (OBO Foundry) and HMD_HumanPhenotype.rpt (MGD) for mouse orthology. Parse the four POSTRE reference phenotype groups and traverse the hp.obo/mp.obo DAGs to collect every HPO/MPO term nested under each group's root term, so genes annotated to any descendant term inherit the group association. Assign each gene 1 if it is linked to the phenotype group via human HPO annotations, 0.5 if it is linked only through its mouse ortholog in the HMD_HumanPhenotype.rpt human-mouse mapping (Standard Running Mode only), and 0 if there is no association. Aggregate per-gene scores into a genePhenoScore table with one row per gene and phenotype group, written as a tab-separated plain text file (genePhenoScore.tsv) consistent with POSTRE's tab-separated file conventions. When POSTRE then evaluates a candidate gene against an SV-associated phenotype, the genePhenoscore of 1 signals human association, 0.5 mouse-only association, and 0 no association.

## Related tools

- **POSTRE** (Software whose genePhenoScore module is reconstructed; consumes the gene-phenotype score table during SV pathogenicity prediction) — https://github.com/vicsanga/Postre
- **R** (Implementation language for parsing the OBO DAGs and annotation files and writing the tab-separated genePhenoScore.tsv output)

## Examples

```
source("https://raw.githubusercontent.com/vicsanga/Postre/main/Postre_wrapper.R")
```

## Evaluation signals

- Every genePhenoScore value is in {0, 0.5, 1}: 1 only for genes annotated in genes_to_phenotype.txt, 0.5 only for mouse-ortholog-only links via HMD_HumanPhenotype.rpt, 0 otherwise.
- The output table has exactly one row per gene per phenotype group and is tab-separated plain text (genePhenoScore.tsv), matching POSTRE's file conventions.
- The set of terms linked to each phenotype group equals the full descendant closure of the group's root HPO/MPO term in hp.obo/mp.obo.
- A gene annotated to a highly nested HPO term still receives the group association, confirming DAG traversal rather than root-term-only matching.
- In POSTRE's Standard Running Mode, a candidate gene associated with the same phenotype only in mice is assigned genePhenoscore 0.5, not 0 or 1.

## Limitations

- The 0.5 (mouse ortholog) score is applied only in POSTRE's Standard Running Mode; in other modes mouse-only associations may not be credited.
- POSTRE is oriented towards prediction of SVs causing dominant disorders and its capacity to identify SVs implicated in recessive phenotypes has not been specifically evaluated, so phenotype-group annotations may not generalize to recessive presentations.
- Incomplete characterization of disease-relevant biology (e.g. enhancers across human cell types) limits downstream pathomechanism prediction even when gene-phenotype annotation is correct.

## Evidence
