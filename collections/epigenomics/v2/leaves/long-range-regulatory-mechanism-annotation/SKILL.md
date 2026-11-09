---
name: long-range-regulatory-mechanism-annotation
description: Use when a structural variant from a congenital abnormality patient lacks
  a coding explanation and you need to identify the affected gene via a long-range
  regulatory mechanism in a phenotype-matched cell type.
when_to_use_negative:
- The SV is expected to act through a coding mechanism (e.g. truncating deletion within
  a gene body) and long-range regulatory annotation is not the analytical goal
- The relevant cell type or developmental stage lacks enhancer/TAD/expression data
  in POSTRE's supported phenotypes—e.g. bulk whole limb bud data cannot identify the
  ZRS enhancer, so POSTRE fails on the SHH duplication case
- The phenotype under study is recessive; POSTRE is oriented towards dominant disorders
  and its capacity for recessive phenotypes has not been specifically evaluated
edam_operation: http://edamontology.org/operation_3225
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0625
tools:
- name: POSTRE
  role: Predicts pathogenic impact of SVs, identifying causative genes and pathological
    mechanisms (coding and long-range) in a cell-type-specific manner
  repo: https://github.com/vicsanga/Postre
- name: POSTRE_multiSV()
  role: R function for sequential analysis of multiple SVs; takes a 7-column SV data
    frame and returns pathogenicityPrediction_per_SV and geneStats_recurrencyAndPathomech
  repo: https://github.com/vicsanga/Postre
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:05f7fdb50c2038e8c2580576c62458d5cddd45099044e42b615cbd6cf8538f01
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
  - POSTRE_multiSV()
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3225
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 22
---

# long-range-regulatory-mechanism-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates structural variants with long-range (enhancer-driven) pathomechanisms—such as enhancer disconnection and enhancer adoption—by scoring SVs against cell-type-specific enhancer, TAD and gene-expression data using POSTRE, thereby predicting causative genes for non-coding SVs in congenital abnormality patients.

## When to use

Use when a structural variant from a congenital abnormality patient lacks a coding explanation and you need to identify the affected gene via a long-range regulatory mechanism in a phenotype-matched cell type.

## When NOT to use

- The SV is expected to act through a coding mechanism (e.g. truncating deletion within a gene body) and long-range regulatory annotation is not the analytical goal
- The relevant cell type or developmental stage lacks enhancer/TAD/expression data in POSTRE's supported phenotypes—e.g. bulk whole limb bud data cannot identify the ZRS enhancer, so POSTRE fails on the SHH duplication case
- The phenotype under study is recessive; POSTRE is oriented towards dominant disorders and its capacity for recessive phenotypes has not been specifically evaluated

## Inputs

- Tab-separated plain-text SV file with 7 columns: chromosome 1, coordinates 1 (hg19), chromosome 2, coordinates 2, SV type (Inversion/Translocation/Deletion/Duplication), comma-separated phenotypes, SV unique identifier
- hg19 genomic coordinates for SV breakpoints (comma-separated range, e.g. 85092268,85092269, when not base-pair resolution)

## Outputs

- pathogenicityPrediction_per_SV: table with a pathogenic prediction for each SV and associated phenotype, including predicted causative gene, pathomechanism class (coding vs long-range), and cell type
- geneStats_recurrencyAndPathomech: aggregated pathogenic predictions per gene, phenotype and pathogenic mechanism (coding, long-range)
- postre_pathogenicity_predictions.txt: downloadable tab-separated results table

## How to apply

Encode each SV as a 7-column tab-separated row (chromosome 1, coordinates 1, chromosome 2, coordinates 2, SV type from Inversion/Translocation/Deletion/Duplication, comma-separated phenotypes from head_neck/limbs/neurodevelopmental/cardiovascular/vision_eye/liver_biliary_system/pituitary, SV unique identifier) using hg19 coordinates. In R (>=3.5.0), source the Postre_app/POSTRE_multiSV.R script and call POSTRE_multiSV(SVs=svs_patients, pathTo_Postre_app_Folder="<path>/Postre_app/") with default parameters to score each SV against cell-type-specific enhancer catalogs, TAD boundaries, and gene expression data matched to the declared phenotypes. The rationale is that SV pathomechanisms can only be predicted and experimentally validated if cell-type-specific regulatory landscapes (enhancers and TADs) are considered, distinguishing coding effects (gene deletion, truncation) from long-range effects (enhancer disconnection, enhancer adoption). Extract the returned list elements pathogenicityPrediction_per_SV (per-SV/per-phenotype table) and geneStats_recurrencyAndPathomech (aggregated per gene/phenotype/mechanism) and write to a tab-separated txt file. Validate by confirming that the BOFS inversion positive control is predicted to cause loss of TFAP2A expression in neural crest cells via enhancer disconnection.

## Related tools

- **POSTRE** (Predicts pathogenic impact of SVs, identifying causative genes and pathological mechanisms (coding and long-range) in a cell-type-specific manner) — https://github.com/vicsanga/Postre
- **POSTRE_multiSV()** (R function for sequential analysis of multiple SVs; takes a 7-column SV data frame and returns pathogenicityPrediction_per_SV and geneStats_recurrencyAndPathomech) — https://github.com/vicsanga/Postre

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); POSTRE_multiSV(SVs=svs_patients, pathTo_Postre_app_Folder="/PathTo/Postre_app/")
```

## Evaluation signals

- The BOFS inversion positive control is predicted to cause loss of TFAP2A expression in neural crest cells via enhancer disconnection mechanism
- Five out of six experimentally validated long-range SV patients are correctly assigned their causative gene and long-range mechanism (83% sensitivity, 100% specificity at the gene level)
- Reported long-range cases fall into the long-range pathomechanism class rather than coding
- postre_pathogenicity_predictions.txt exists and contains a pathogenic prediction row for every input SV-phenotype pair

## Limitations

- Bulk whole limb bud data cannot identify the ZRS enhancer, so POSTRE fails to predict the SHH duplication case
- Incomplete characterization of enhancers across human cell types limits prediction of long-range pathomechanisms
- Lack of genomic data for appropriate developmental stages or differentiation time points may affect identification of pathologically relevant genes

## Evidence

- [methods] BOFS TFAP2A enhancer disconnection: [evidence span withheld — anchor status: not_found]
- [readme] Cell type specificity rationale: [evidence span withheld — anchor status: not_found]
