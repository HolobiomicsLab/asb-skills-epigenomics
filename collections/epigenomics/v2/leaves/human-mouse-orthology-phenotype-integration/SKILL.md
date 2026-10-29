---
name: human-mouse-orthology-phenotype-integration
description: Use when a candidate gene list must be linked to a congenital-abnormality
  phenotype group and scored for human or mouse gene-phenotype support, e.g. to build
  POSTRE's genePhenoScore table from hp.obo, genes_to_phenotype.txt, mp.obo and HMD_HumanPhenotype.rpt.
when_to_use_negative:
- The phenotype of interest is not covered by POSTRE's reference groups — only head_neck,
  limbs, neurodevelopmental, cardiovascular (and in later versions vision_eye, liver_biliary_system,
  pituitary) are supported SV input phenotypes
- The goal is SV structural-impact annotation itself (coding vs long-range mechanisms);
  this skill only supplies the phenotype-support score, not enhancer/TAD analysis
- No network access to HPO, OBO Foundry or MGD resources, since all four source files
  (hp.obo, genes_to_phenotype.txt, mp.obo, HMD_HumanPhenotype.rpt) must be downloaded
  first
edam_operation: http://edamontology.org/operation_0226
tools:
- name: POSTRE
  role: Consumes the genePhenoScore to weight candidate genes by phenotype support;
    phenotype labels in POSTRE submissions must match the reference groups built by
    this skill
  repo: https://github.com/vicsanga/Postre
- name: R
  role: Environment in which the genePhenoScore module is reconstructed and POSTRE
    is run (R version >=3.5.0 required)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:b0689393c9bed39f447f9b41d88164aa9ca4034c799fc7792a83082eefdb1f1b
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# human-mouse-orthology-phenotype-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates Human Phenotype Ontology (HPO) gene annotations and mouse ortholog Mammalian Phenotype Ontology (MPO) annotations to score candidate genes against POSTRE's reference phenotype groups (head_neck, limbs, neurodevelopmental, cardiovascular), producing the per-gene genePhenoScore used in SV pathogenicity prediction.

## When to use

Use when a candidate gene list must be linked to a congenital-abnormality phenotype group and scored for human or mouse gene-phenotype support, e.g. to build POSTRE's genePhenoScore table from hp.obo, genes_to_phenotype.txt, mp.obo and HMD_HumanPhenotype.rpt.

## When NOT to use

- The phenotype of interest is not covered by POSTRE's reference groups — only head_neck, limbs, neurodevelopmental, cardiovascular (and in later versions vision_eye, liver_biliary_system, pituitary) are supported SV input phenotypes
- The goal is SV structural-impact annotation itself (coding vs long-range mechanisms); this skill only supplies the phenotype-support score, not enhancer/TAD analysis
- No network access to HPO, OBO Foundry or MGD resources, since all four source files (hp.obo, genes_to_phenotype.txt, mp.obo, HMD_HumanPhenotype.rpt) must be downloaded first

## Inputs

- hp.obo (Human Phenotype Ontology DAG, from HPO website)
- genes_to_phenotype.txt (HPO gene-to-phenotype annotation file)
- mp.obo (Mammalian Phenotype Ontology DAG, from OBO Foundry)
- HMD_HumanPhenotype.rpt (human-mouse orthology phenotype mapping from MGD)

## Outputs

- genePhenoScore.tsv (tab-separated table, one row per gene and phenotype group, with scores of 1, 0.5 or 0)
- Per-phenotype-group gene sets (human-evidence genes vs mouse-ortholog-only genes)

## How to apply

Download hp.obo and genes_to_phenotype.txt from the HPO website and mp.obo from OBO Foundry plus the human-mouse orthology phenotype mapping HMD_HumanPhenotype.rpt from MGD. Parse the four POSTRE reference phenotype groups (head_neck, limbs, neurodevelopmental, cardiovascular) and traverse the hp.obo/mp.obo DAGs to collect every HPO/MPO term nested under each group's root term. Assign a gene a score of 1 for a phenotype group if it is annotated with any descending HPO term in genes_to_phenotype.txt (human evidence), 0.5 if it is linked only through its mouse ortholog's MPO annotations in HMD_HumanPhenotype.rpt (Standard Running Mode only), and 0 if neither applies. Aggregate the per-gene, per-group scores into a tab-separated genePhenoScore.tsv, matching POSTRE's tab-separated file conventions and its accepted phenotype labels (head_neck, limbs, neurodevelopmental, cardiovascular, plus vision_eye, liver_biliary_system, pituitary in later versions).

## Related tools

- **POSTRE** (Consumes the genePhenoScore to weight candidate genes by phenotype support; phenotype labels in POSTRE submissions must match the reference groups built by this skill) — https://github.com/vicsanga/Postre
- **R** (Environment in which the genePhenoScore module is reconstructed and POSTRE is run (R version >=3.5.0 required))

## Examples

```
Rscript -e 'hpo <- read.delim("genes_to_phenotype.txt"); ortho <- read.delim("HMD_HumanPhenotype.rpt")'
```

## Evaluation signals

- Every gene score in the output is exactly 1, 0.5 or 0 — any other value indicates a scoring error
- Output is a tab-separated file (genePhenoScore.tsv) with one row per gene-phenotype-group pair, covering all four reference groups
- A gene linked to a group only via HMD_HumanPhenotype.rpt mouse orthology receives 0.5, never 1, in Standard Running Mode
- Every gene linked via genes_to_phenotype.txt carries an HPO term that is a descendant of the group's root term in hp.obo (verify by DAG traversal)
- Running POSTRE_multiSV() with the regenerated genePhenoScore reproduces phenotype-compatible pathogenicity calls on the test files in the repository's testFiles folder

## Limitations

- Mouse-only (MPO) evidence yields 0.5 and is counted only in POSTRE's Standard Running Mode
- A gene is scored 0 even if phenotype relevance exists outside HPO/MPO annotations captured by these files; the four reference groups are limited to limb, craniofacial, cardiac and neurodevelopmental congenital abnormalities in the described version
- POSTRE's capacity to identify SVs implicated in recessive phenotypes has not been specifically evaluated, so phenotype scoring is oriented towards dominant disorders

## Evidence

- [methods] Gene-phenotype links are built from HPO and MPO gene-phenotype annotation files: [evidence span withheld — anchor status: not_found]
- [methods] Score assignment rules: [evidence span withheld — anchor status: not_found]
- [methods] Ontology DAG traversal step: [evidence span withheld — anchor status: not_found]
