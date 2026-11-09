---
name: tabular-scoring-table-construction
description: Use when building a gene-to-phenotype scoring lookup from HPO (hp.obo,
  genes_to_phenotype.txt) and MPO (mp.obo, HMD_HumanPhenotype.rpt) resources, where
  each gene needs a tiered score per phenotype group.
when_to_use_negative:
- The SV already has a pathogenicity prediction per gene/phenotype/mechanism from
  POSTRE_multiSV() — this skill builds the gene-phenotype scoring resource, not the
  per-SV or per-gene results tables.
- The analysis concerns recessive phenotypes — POSTRE is oriented towards dominant
  disorders and 'its capacity to identify SVs implicated in recessive phenotypes has
  not been specifically evaluated'.
- Only mouse phenotype data exists and the human-only (0 score for mouse-only genes)
  mode is desired — the 0.5 tier applies in Standard Running Mode only.
edam_topics:
- http://edamontology.org/topic_0166
tools:
- name: POSTRE
  role: Defines the genePhenoScore tiers (1/0.5/0), the four-to-seven reference phenotype
    groups, and the tab-separated output convention the scoring table must follow
  repo: https://github.com/vicsanga/Postre
- name: R
  role: Implementation language for parsing the obo DAGs and annotation files and
    writing genePhenoScore.tsv (POSTRE requires R >=3.5.0)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:39ca4b88babf61a02394bd6142f6c85ebe6dcb0753bbf0fd42bf87bea4db9f88
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0166
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# tabular-scoring-table-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs per-gene numeric scoring tables (e.g. POSTRE's genePhenoScore) by aggregating ontology-based gene-phenotype associations into tiered scores (1 human, 0.5 mouse-only, 0 none) and emitting them as tab-separated plain text files. It mirrors POSTRE's convention of one row per gene and phenotype group so candidate genes can be scored against SV-associated phenotypes.

## When to use

Use when building a gene-to-phenotype scoring lookup from HPO (hp.obo, genes_to_phenotype.txt) and MPO (mp.obo, HMD_HumanPhenotype.rpt) resources, where each gene needs a tiered score per phenotype group.

## When NOT to use

- The SV already has a pathogenicity prediction per gene/phenotype/mechanism from POSTRE_multiSV() — this skill builds the gene-phenotype scoring resource, not the per-SV or per-gene results tables.
- The analysis concerns recessive phenotypes — POSTRE is oriented towards dominant disorders and 'its capacity to identify SVs implicated in recessive phenotypes has not been specifically evaluated'.
- Only mouse phenotype data exists and the human-only (0 score for mouse-only genes) mode is desired — the 0.5 tier applies in Standard Running Mode only.

## Inputs

- hp.obo (Human Phenotype Ontology DAG)
- genes_to_phenotype.txt (human gene-to-HPO-term annotations)
- mp.obo (Mammalian Phenotype Ontology DAG)
- HMD_HumanPhenotype.rpt (human-mouse ortholog phenotype mapping from MGD)

## Outputs

- genePhenoScore.tsv: tab-separated plain text table with one row per gene and phenotype group, numeric scores in {0, 0.5, 1}

## How to apply

Parse the four POSTRE reference phenotype groups (head_neck, limbs, neurodevelopmental, cardiovascular — later extended with vision_eye, liver_biliary_system, pituitary) and traverse the hp.obo and mp.obo DAGs to collect all HPO/MPO terms descending from each group's root term. Link a gene to a group if it is annotated to any HPO term nested under that group in genes_to_phenotype.txt (score 1) or only via its mouse ortholog in HMD_HumanPhenotype.rpt (score 0.5, Standard Running Mode only); genes with no association get 0. Aggregate the per-gene scores into a table with one row per gene and phenotype group. Write the result as a tab-separated plain text file (genePhenoScore.tsv), consistent with POSTRE's convention that 'The data must be stored in a plain text file with column values separated by tabulations'.

## Related tools

- **POSTRE** (Defines the genePhenoScore tiers (1/0.5/0), the four-to-seven reference phenotype groups, and the tab-separated output convention the scoring table must follow) — https://github.com/vicsanga/Postre
- **R** (Implementation language for parsing the obo DAGs and annotation files and writing genePhenoScore.tsv (POSTRE requires R >=3.5.0))

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); POSTRE_multiSV()
```

## Evaluation signals

- Every emitted score is exactly one of {0, 0.5, 1}; no other numeric values appear in genePhenoScore.tsv
- Each gene linked to any HPO/MPO term descending from a phenotype-group root term appears with a non-zero score for that group; DAG traversal (not just direct annotation) determines membership
- Genes associated only via mouse orthologs (HMD_HumanPhenotype.rpt) receive 0.5, while genes with human HPO evidence (genes_to_phenotype.txt) receive 1 for the same group
- Output is a tab-separated plain text file with one row per gene-phenotype group pair, loadable by POSTRE's pipeline
- Running POSTRE_multiSV() without arguments documents the expected parameters and outputs, which can be cross-checked against the constructed table

## Limitations

- POSTRE is oriented towards dominant disorders; recessive phenotype SVs have not been specifically evaluated
- Lack of genomic data for appropriate developmental stages may affect identification of pathologically relevant genes, independent of the gene-phenotype scores
- The genePhenoScore covers only fixed reference phenotype groups (head_neck, limbs, neurodevelopmental, cardiovascular, plus vision_eye, liver_biliary_system, pituitary); phenotypes outside these groups cannot be scored
- The mouse-only 0.5 tier applies in Standard Running Mode only

## Evidence

- [full_text] POSTRE assigns a candidate gene a genePhenoscore of 1 if the gene is associated with the same phenotype in humans, 0.5 if it is associated with the same phenotype in mice: [evidence span withheld — anchor status: not_found]
