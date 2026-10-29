---
name: structural-variant-pathogenicity-prediction
description: Use when you need to classify non-coding or coding SVs (inversions, translocations,
  deletions, duplications in hg19) as pathogenic vs non-pathogenic for congenital-abnormality
  phenotypes and recover the causative gene and long-range pathomechanism.
when_to_use_negative:
- SVs implicated in recessive phenotypes — POSTRE is oriented towards dominant disorders
  and has not been specifically evaluated for recessive disease
- Cases whose critical enhancer requires developmental-stage-specific data POSTRE
  lacks (e.g. the ZRS enhancer is not identifiable from bulk whole limb bud data,
  so the SHH duplication case fails)
- Phenotypes outside the supported categories (head_neck, limbs, neurodevelopmental,
  cardiovascular, vision_eye, liver_biliary_system, pituitary)
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: POSTRE (Prediction Of STRuctural variant Effects)
  role: 'Core tool: scores SVs against cell-type-specific enhancer, TAD and expression
    data to predict pathogenicity, causative gene and pathomechanism'
  repo: https://github.com/vicsanga/Postre
- name: POSTRE_multiSV()
  role: R function mode for batch analysis of multiple SVs; takes SVs data frame and
    pathTo_Postre_app_Folder as mandatory arguments
  repo: https://github.com/vicsanga/Postre
- name: Shiny
  role: Framework underlying the POSTRE GUI, supporting Single SV and Multiple SV
    Submission analyses with downloadable txt tables
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:76a409c9dd1a6228512c43df333c13d88188ba65863d61b9d7fd10283b54c124
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
  - CADD-SV
  - POSTRE
  - POSTRE (Prediction Of STRuctural variant Effects)
  - POSTRE_multiSV()
  - R
  - STRVCTVRE
  - SVScore
  - Shiny
  - TADA
  - TADeus2
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# structural-variant-pathogenicity-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict the pathogenicity of human structural variants (SVs) and identify their disease-causative genes and pathomechanisms (coding or long-range, e.g. enhancer disconnection, enhancer adoption) in a cell type/tissue-specific manner using POSTRE, which scores SVs against cell-type-specific enhancer, TAD and expression data.

## When to use

Use when you need to classify non-coding or coding SVs (inversions, translocations, deletions, duplications in hg19) as pathogenic vs non-pathogenic for congenital-abnormality phenotypes and recover the causative gene and long-range pathomechanism.

## When NOT to use

- SVs implicated in recessive phenotypes — POSTRE is oriented towards dominant disorders and has not been specifically evaluated for recessive disease
- Cases whose critical enhancer requires developmental-stage-specific data POSTRE lacks (e.g. the ZRS enhancer is not identifiable from bulk whole limb bud data, so the SHH duplication case fails)
- Phenotypes outside the supported categories (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary)

## Inputs

- Plain-text tab-separated SV file with 7 columns: chromosome 1, hg19 coordinates 1 (comma-separated range if not bp-resolution, e.g. 85092268,85092269), chromosome 2, hg19 coordinates 2, SV type (Inversion/Translocation/Deletion/Duplication), comma-separated phenotype list, unique SV id
- SV data frame in R with 7 columns (hg19 coordinates), e.g. read from testFiles/Table1_LongRange_SVs.tsv or ExampleMultipleSubmission.tsv
- Phenotype category list restricted to: head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary

## Outputs

- pathogenicityPrediction_per_SV: table with one pathogenicity prediction per SV-phenotype pair (downloadable as txt)
- geneStats_recurrencyAndPathomech: aggregation of pathogenic predictions per gene, phenotype and pathogenic mechanism (coding vs long-range)
- errors: list of SV ids that failed processing

## How to apply

Format each SV as one tab-separated line of 7 columns (chromosome 1, hg19 coordinates 1, chromosome 2, hg19 coordinates 2, SV type (Inversion/Translocation/Deletion/Duplication), comma-separated phenotype list from head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary, and a unique SV id). In R (>=3.5.0), source Postre_app/POSTRE_multiSV.R (or the Postre_wrapper.R from GitHub) and call POSTRE_multiSV(SVs, pathTo_Postre_app_Folder), optionally in Standard or High-Specificity mode — note that High-Specificity requires causative genes to have broad polycomb domains, so genes like MEF2C and SLC2A1 are missed. The function scores each SV against cell-type-specific enhancer, TAD and expression data and returns three list elements: pathogenicityPrediction_per_SV, geneStats_recurrencyAndPathomech and errors; results can also be obtained via the Shiny GUI (Multiple SV Submission, downloadable txt tables) or the online Single SV version at postre-tool.csic.es. Inspect the two output tables — 'Results per SV and phenotype' (one pathogenic prediction per SV-phenotype pair) and 'Results per gene and phenotype' (aggregation per gene, phenotype and coding/long-range mechanism).

## Related tools

- **POSTRE (Prediction Of STRuctural variant Effects)** (Core tool: scores SVs against cell-type-specific enhancer, TAD and expression data to predict pathogenicity, causative gene and pathomechanism) — https://github.com/vicsanga/Postre
- **POSTRE_multiSV()** (R function mode for batch analysis of multiple SVs; takes SVs data frame and pathTo_Postre_app_Folder as mandatory arguments) — https://github.com/vicsanga/Postre
- **Shiny** (Framework underlying the POSTRE GUI, supporting Single SV and Multiple SV Submission analyses with downloadable txt tables)

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); svs <- read.delim("testFiles/Table1_LongRange_SVs.tsv", sep="\t", stringsAsFactors=FALSE, header=FALSE); res <- POSTRE_multiSV(SVs=svs, pathTo_Postre_app_Folder="/PathTo/Postre_app/")
```

## Evaluation signals

- Positive-control check: POSTRE should predict loss of TFAP2A expression in neural crest cells via enhancer disconnection for the BOFS inversion patient (Laugsch et al., 2019)
- Sensitivity/specificity benchmark: 83% sensitivity and 100% specificity at gene level on six experimentally validated positive-control SVs; 5/6 causative genes recovered
- Table 2 benchmark: 95% success rate (Standard mode) vs 66% (High-Specificity mode) on the 41-patient cohort, with reported causative genes such as SATB2, MEF2C, FOXG1 and SLC2A1 present in the per-gene table
- Specificity benchmark: 96.8% of gnomAD healthy-individual SV-phenotype associations labelled non-pathogenic in Standard mode (99.2% High-Specificity; 89.5%/98.3% for the 500 largest SVs)
- Output schema check: every input SV-phenotype pair has a labelled prediction in the 'Results per SV and phenotype' table and failures are captured in the errors element

## Limitations

- High-Specificity mode requires causative genes to have broad polycomb domains, dropping sensitivity to 66% (misses MEF2C, SLC2A1)
- Incomplete characterization of enhancers across human cell types limits prediction of long-range pathomechanisms; lack of data for the appropriate developmental stages can affect identification of pathologically relevant genes
- Bulk whole limb bud data cannot identify the ZRS enhancer, so POSTRE fails to predict the SHH duplication case
- Coordinates must be hg19; only two-breakpoint SVs of type Inversion, Translocation, Deletion or Duplication are supported

## Evidence

- [methods] 7-column hg19 input format for POSTRE_multiSV: [evidence span withheld — anchor status: not_found]
- [methods] BOFS/TFAP2A enhancer disconnection positive control: [evidence span withheld — anchor status: not_found]
