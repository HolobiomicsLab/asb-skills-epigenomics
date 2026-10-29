---
name: phenotype-group-term-propagation
description: Use when a candidate gene must be scored for association with an SV-associated
  phenotype group using HPO (hp.obo, genes_to_phenotype.txt) and MPO (mp.obo, HMD_HumanPhenotype.rpt)
  annotations, as in POSTRE's genePhenoScore module.
when_to_use_negative:
- A precomputed genePhenoScore.tsv already ships with the POSTRE installation and
  no updated HPO/MPO releases need to be incorporated
- The phenotypes of interest fall outside the POSTRE reference phenotype groups (e.g.
  vision_eye, liver_biliary_system, pituitary added via cell-type data, not via this
  HPO/MPO module)
- The goal is SV pathogenicity prediction itself (use POSTRE_multiSV()) rather than
  building the gene-phenotype scoring layer that feeds it
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: POSTRE
  role: The genePhenoScore module being reconstructed belongs to POSTRE; the output
    genePhenoScore.tsv must match POSTRE's tab-separated file conventions for downstream
    use by POSTRE_multiSV()
  repo: https://github.com/vicsanga/Postre
- name: R (>=3.5.0)
  role: Implementation language for parsing the OBO DAGs and annotation files and
    writing genePhenoScore.tsv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:09524c904ceefb037cc4c427af5fb4df21745410984bcd852404de9d6ebde9f8
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# phenotype-group-term-propagation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs POSTRE's genePhenoScore module by propagating HPO/MPO ontology terms from four reference phenotype-group roots (head_neck, limbs, neurodevelopmental, cardiovascular) to build gene-phenotype links, assigning each gene 1 (human HPO), 0.5 (mouse MPO via ortholog) or 0 per group.

## When to use

Use when a candidate gene must be scored for association with an SV-associated phenotype group using HPO (hp.obo, genes_to_phenotype.txt) and MPO (mp.obo, HMD_HumanPhenotype.rpt) annotations, as in POSTRE's genePhenoScore module.

## When NOT to use

- A precomputed genePhenoScore.tsv already ships with the POSTRE installation and no updated HPO/MPO releases need to be incorporated
- The phenotypes of interest fall outside the POSTRE reference phenotype groups (e.g. vision_eye, liver_biliary_system, pituitary added via cell-type data, not via this HPO/MPO module)
- The goal is SV pathogenicity prediction itself (use POSTRE_multiSV()) rather than building the gene-phenotype scoring layer that feeds it

## Inputs

- hp.obo (Human Phenotype Ontology DAG from the HPO website)
- genes_to_phenotype.txt (human gene-to-HPO-term annotations from the HPO website)
- mp.obo (Mammalian Phenotype Ontology DAG from OBO Foundry)
- HMD_HumanPhenotype.rpt (human-mouse orthology phenotype mapping from MGD)

## Outputs

- genePhenoScore.tsv: tab-separated table with one row per gene and phenotype group, containing scores of 1 (human HPO association), 0.5 (mouse-only association, Standard Running Mode) or 0 (no association)

## How to apply

Parse the four POSTRE reference phenotype groups (head_neck, limbs, neurodevelopmental, cardiovascular) and traverse the hp.obo and mp.obo DAGs to collect every HPO/MPO term descending from each group's root term. A gene is linked to a phenotype group if it is annotated with any HPO term nested under that group's root (from genes_to_phenotype.txt), yielding a score of 1; if it is linked only through its mouse ortholog's MPO annotation (via HMD_HumanPhenotype.rpt), it receives 0.5 in POSTRE's Standard Running Mode; otherwise 0. Aggregate the per-gene scores into one row per gene and phenotype group and write genePhenoScore.tsv as a tab-separated plain text file, matching POSTRE's tab-separated file conventions so it can be consumed downstream by POSTRE_multiSV().

## Related tools

- **POSTRE** (The genePhenoScore module being reconstructed belongs to POSTRE; the output genePhenoScore.tsv must match POSTRE's tab-separated file conventions for downstream use by POSTRE_multiSV()) — https://github.com/vicsanga/Postre
- **R (>=3.5.0)** (Implementation language for parsing the OBO DAGs and annotation files and writing genePhenoScore.tsv)

## Evaluation signals

- Every HPO/MPO term descending from each group's root term appears in the group's nested-term set (DAG traversal completeness check)
- Any gene annotated with a term nested under a group root is linked to that group and receives score 1; genes linked only via HMD_HumanPhenotype.rpt mouse orthologs receive exactly 0.5; all others receive 0
- Output is a tab-separated plain text file genePhenoScore.tsv with one row per gene and phenotype group, consistent with POSTRE's tab-separated file conventions
- The four phenotype groups present are exactly head_neck, limbs, neurodevelopmental and cardiovascular
- Re-running the module on unchanged hp.obo, genes_to_phenotype.txt, mp.obo and HMD_HumanPhenotype.rpt reproduces an identical genePhenoScore table

## Limitations

- The 0.5 mouse-only score applies in POSTRE's Standard Running Mode only; other modes are not defined in the provided context
- Gene-phenotype links depend on the currency and completeness of HPO/MPO annotations and the HMD human-mouse orthology mapping; genes absent from these files are scored 0 regardless of true relevance

## Evidence

- [full_text] Scoring scheme for gene-phenotype association: [evidence span withheld — anchor status: not_found]
- [full_text] Term-nesting rule for group membership: [evidence span withheld — anchor status: not_found]
- [full_text] Input resources for the module: [evidence span withheld — anchor status: not_found]
