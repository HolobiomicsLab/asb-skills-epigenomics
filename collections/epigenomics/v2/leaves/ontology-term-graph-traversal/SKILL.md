---
name: ontology-term-graph-traversal
description: Use when a gene set must be linked to broad phenotype categories via
  ontology annotations, requiring expansion of root terms (e.g. HPO/MPO) into all
  nested child terms before matching gene annotation files.
when_to_use_negative:
- When the phenotype of interest is not represented by POSTRE's reference groups (head_neck,
  limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary)
  — root-term traversal will not cover phenotypes outside these DAG branches.
- When reproducible mode is required and mouse-derived (MPO/orthology) evidence must
  be excluded, since the 0.5 mouse-annotation score applies in Standard Running Mode
  only.
edam_operation: http://edamontology.org/operation_3559
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3325
tools:
- name: POSTRE
  role: 'Downstream consumer: the genePhenoScore table built by ontology traversal
    feeds POSTRE''s per-gene and per-SV pathogenicity predictions for the associated
    phenotypes'
  repo: https://github.com/vicsanga/Postre
- name: R (>=3.5.0)
  role: Implementation language for parsing OBO DAGs, joining annotation files, and
    writing the genePhenoScore.tsv output
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:17da8bfeb5cf945bc12b146488d6120ed5869aeb9fadbf91e7f9d0464a9d3c6c
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
  - R (>=3.5.0)
  edam_operation: http://edamontology.org/operation_3559
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3325
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# ontology-term-graph-traversal

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Traverses OBO ontology DAGs (hp.obo, mp.obo) to collect all descendant terms under reference phenotype root terms, so that human (HPO) and mouse (MPO) gene-phenotype annotations can be mapped onto a small set of phenotype groups (head_neck, limbs, neurodevelopmental, cardiovascular) as used in POSTRE's genePhenoScore.

## When to use

Use when a gene set must be linked to broad phenotype categories via ontology annotations, requiring expansion of root terms (e.g. HPO/MPO) into all nested child terms before matching gene annotation files.

## When NOT to use

- When the phenotype of interest is not represented by POSTRE's reference groups (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary) — root-term traversal will not cover phenotypes outside these DAG branches.
- When reproducible mode is required and mouse-derived (MPO/orthology) evidence must be excluded, since the 0.5 mouse-annotation score applies in Standard Running Mode only.

## Inputs

- hp.obo (Human Phenotype Ontology DAG, from the HPO website)
- genes_to_phenotype.txt (HPO human gene-phenotype annotation file)
- mp.obo (Mammalian Phenotype Ontology DAG, from OBO Foundry)
- HMD_HumanPhenotype.rpt (human-mouse orthology phenotype mapping from MGD)

## Outputs

- genePhenoScore.tsv — a tab-separated table with one row per gene and phenotype group (head_neck, limbs, neurodevelopmental, cardiovascular), with scores of 1 (human HPO association), 0.5 (mouse-only MPO association, Standard Running Mode) or 0 (no association)

## How to apply

Download hp.obo and genes_to_phenotype.txt from the HPO website, and mp.obo from OBO Foundry plus HMD_HumanPhenotype.rpt from MGD. Parse the reference phenotype groups (head_neck, limbs, neurodevelopmental, cardiovascular) and traverse the hp.obo/mp.obo DAGs to collect every HPO/MPO term descending from each group's root term. Assign each gene a score of 1 if it is linked to the group via a human HPO annotation, 0.5 if it is linked only through its mouse ortholog via the HMD_HumanPhenotype.rpt mapping (Standard Running Mode only), and 0 if there is no association. Aggregate per-gene scores into a gene-phenotype table and write it as a tab-separated plain text file (genePhenoScore.tsv), consistent with POSTRE's tab-separated file conventions.

## Related tools

- **POSTRE** (Downstream consumer: the genePhenoScore table built by ontology traversal feeds POSTRE's per-gene and per-SV pathogenicity predictions for the associated phenotypes) — https://github.com/vicsanga/Postre
- **R (>=3.5.0)** (Implementation language for parsing OBO DAGs, joining annotation files, and writing the genePhenoScore.tsv output)

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); POSTRE_multiSV(SVs, pathTo_Postre_app_Folder)
```

## Evaluation signals

- Every HPO/MPO term collected for a group is verifiably a descendant of that group's root term in the hp.obo/mp.obo DAG (no terms leaking in from sibling branches).
- Gene scores are restricted to the values 1, 0.5 and 0; 0.5 occurs only for genes linked via HMD_HumanPhenotype.rpt mouse ortholog annotations and never when a direct human HPO link exists.
- The output genePhenoScore.tsv is tab-separated with one row per gene per phenotype group and no missing phenotype-group columns.
- Genes annotated with an HPO term that is nested under a reference group's root are correctly scored 1 for that group (spot-check against genes_to_phenotype.txt).

## Limitations

- Mouse-derived scores (0.5) are only used in POSTRE's Standard Running Mode, not in reproducible mode.
- The traversal is anchored to POSTRE's fixed reference phenotype groups; phenotypes outside these roots are not captured.
- Annotation quality depends on the completeness of HPO/MPO resources; missing or stale genes_to_phenotype.txt / HMD_HumanPhenotype.rpt files will produce incomplete gene-phenotype links.

## Evidence

- [other] Gene-phenotype links are built from HPO terms (hp.obo, genes_to_phenotype.txt) and MPO terms (mp.obo, HMD_HumanPhenotype.rpt), where genes associated with any HPO/MPO term nested under a reference HPO/MPO for a phenotype group are linked to that group.: [evidence span withheld — anchor status: not_found]
- [other] POSTRE assigns a candidate gene a genePhenoscore of 1 if the gene is associated with the same phenotype in humans, 0.5 if it is associated with the same phenotype in mice but not in humans (Standard Running Mode only), and 0 otherwise.: [evidence span withheld — anchor status: not_found]
- [other] Aggregate the per-gene scores into a genePhenoScore table (one row per gene and phenotype group) and write it as a tab-separated plain text file, genePhenoScore.tsv, consistent with POSTRE's tab-separated file conventions.: [evidence span withheld — anchor status: not_found]
