---
name: enhancer-disconnection-analysis
description: Use when an hg19 structural variant (deletion, inversion, translocation
  or duplication) lies outside a gene's coding sequence and a long-range, enhancer-driven
  pathomechanism is suspected for a congenital phenotype (limb, craniofacial, cardiac,
  neurodevelopmental, vision_eye, liver_biliary_system.
when_to_use_negative:
- The SV's suspected mechanism is coding (truncation/deletion of the coding sequence)
  rather than long-range enhancer regulation — although POSTRE reports coding events,
  enhancer-disconnection interpretation applies only when breakpoints spare the gene
  body.
- The disease-relevant cell type or developmental stage is not represented in POSTRE's
  genomic data (e.g. the SHH/ZRS case fails because POSTRE uses bulk whole limb bud
  data in which the ZRS enhancer cannot be identified).
- The phenotype is recessive — POSTRE is oriented towards dominant disorders and its
  capacity for recessive phenotypes has not been specifically evaluated.
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: POSTRE
  role: Shiny-based tool whose POSTRE_multiSV() R function scores SV breakpoints against
    cell-type-specific enhancer, TAD and expression data to detect enhancer disconnection
  repo: https://github.com/vicsanga/Postre
- name: R (>=3.5.0)
  role: Runtime environment in which POSTRE_multiSV.R is sourced and executed
- name: XiaoTaoWang/domaincaller
  role: TAD-calling implementation (Directionality Index based) referenced in the
    article's context of TAD definition used for long-range pathomechanism prediction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:e51f860d553df73d01def53eecf12bc4a16a9753e532e13047537e74a9af1b75
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
  - R (>=3.5.0)
  - Shiny
  - XiaoTaoWang/domaincaller
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# enhancer-disconnection-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts SV-driven enhancer disconnection — loss of gene expression caused by breakpoints separating a gene from its cell-type-specific enhancers — using POSTRE's cell-type-specific enhancer, TAD and expression data (hg19).

## When to use

Use when an hg19 structural variant (deletion, inversion, translocation or duplication) lies outside a gene's coding sequence and a long-range, enhancer-driven pathomechanism is suspected for a congenital phenotype (limb, craniofacial, cardiac, neurodevelopmental, vision_eye, liver_biliary_system, pituitary).

## When NOT to use

- The SV's suspected mechanism is coding (truncation/deletion of the coding sequence) rather than long-range enhancer regulation — although POSTRE reports coding events, enhancer-disconnection interpretation applies only when breakpoints spare the gene body.
- The disease-relevant cell type or developmental stage is not represented in POSTRE's genomic data (e.g. the SHH/ZRS case fails because POSTRE uses bulk whole limb bud data in which the ZRS enhancer cannot be identified).
- The phenotype is recessive — POSTRE is oriented towards dominant disorders and its capacity for recessive phenotypes has not been specifically evaluated.

## Inputs

- Structural variant with two breakpoints in hg19 coordinates (base-pair or comma-separated range, e.g. 85092268,85092269)
- 7-column tab-separated SV file (chr1, coord1, chr2, coord2, SV type: Inversion/Translocation/Deletion/Duplication, phenotype list, SV id)
- Phenotype labels restricted to: head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary

## Outputs

- pathogenicityPrediction_per_SV table (Results per SV and phenotype) with predicted causative gene, pathogenicity class and long-range vs coding mechanism
- geneStats_recurrencyAndPathomech table (Results per gene and phenotype) aggregating predictions per gene, phenotype and pathogenic mechanism
- Downloadable txt table (e.g. postre_pathogenicity_predictions.txt)
- errors list element reporting per-SV processing failures

## How to apply

Format the SV as a 7-column tab-separated record (chromosome 1, coordinates 1, chromosome 2, coordinates 2, SV type, comma-separated phenotypes from head_neck/limbs/neurodevelopmental/cardiovascular/vision_eye/liver_biliary_system/pituitary, SV id) in hg19 coordinates. In R (>=3.5.0) source Postre_app/POSTRE_multiSV.R and call POSTRE_multiSV(SVs=..., pathTo_Postre_app_Folder=...), which scores each SV against cell-type-specific enhancer, TAD and expression data so that a breakpoint disconnecting an enhancer from its target gene within the same TAD is flagged as enhancer disconnection. For a single SV the online version at postre-tool.csic.es suffices; multiple SVs require local installation. Inspect the returned pathogenicityPrediction_per_SV list element to confirm the mechanism is classified as 'long-range' rather than 'coding', and geneStats_recurrencyAndPathomech for the per-gene aggregation. Benchmark the call against validated examples such as the BOFS inversion, where POSTRE predicts loss of TFAP2A expression in neural crest cells through enhancer disconnection.

## Related tools

- **POSTRE** (Shiny-based tool whose POSTRE_multiSV() R function scores SV breakpoints against cell-type-specific enhancer, TAD and expression data to detect enhancer disconnection) — https://github.com/vicsanga/Postre
- **R (>=3.5.0)** (Runtime environment in which POSTRE_multiSV.R is sourced and executed)
- **XiaoTaoWang/domaincaller** (TAD-calling implementation (Directionality Index based) referenced in the article's context of TAD definition used for long-range pathomechanism prediction)

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); svs <- read.delim("testFiles/Table1_LongRange_SVs.tsv", sep="\t", stringsAsFactors=FALSE, header=FALSE); res <- POSTRE_multiSV(SVs=svs, pathTo_Postre_app_Folder="/PathTo/Postre_app/")
```

## Evaluation signals

- pathogenicityPrediction_per_SV contains a prediction row for every input SV-phenotype pair with mechanism class 'long-range', not 'coding'
- Positive-control recovery: the BOFS syndrome inversion is predicted to cause loss of TFAP2A expression in neural crest cells via enhancer disconnection
- Benchmark: 5 of 6 experimentally validated long-range SV patients recovered with the correct single causative gene (83% sensitivity, 100% specificity at the gene level)
- geneStats_recurrencyAndPathomech aggregates the prediction per gene, phenotype and pathogenic mechanism without contradictory coding/long-range calls
- errors list element is empty for successfully processed SVs

## Limitations

- Enhancers are incompletely characterized across human cell types, limiting prediction of long-range pathomechanisms.
- Bulk genomic data (e.g. whole limb bud) may miss specific enhancers: POSTRE cannot identify the ZRS enhancer and therefore fails to predict the SHH duplication case.
- Lack of genomic data for the appropriate developmental stages or differentiation time points may affect identification of pathologically relevant genes.
- Factors such as linear distance, CpG islands, promoter DNA methylation and core promoter elements influencing enhancer-gene compatibility within TADs are not yet incorporated into POSTRE's scoring.

## Evidence

- [methods] For this patient, with BOFS syndrome carrying an inversion (Laugsch et al., 2019), POSTRE successfully predicts the loss of TFAP2A expression in neural crest cells through an enhancer disconnection: [evidence span withheld — anchor status: not_found]
- [results] For five out of six positive-control patients, POSTRE successfully predicted the single causative gene affected by each SV and the implicated long-range pathomechanism, achieving 83% sensitivity and 100% specificity at the gene level.: [evidence span withheld — anchor status: not_found]
