---
name: r-scripted-batch-sv-analysis
description: Use when you need to analyze multiple structural variants (e.g. a patient
  cohort or benchmark set) with POSTRE beyond the online single-SV interface, by scripting
  POSTRE_multiSV() in R (>=3.5.0).
when_to_use_negative:
- You only need a single-SV analysis without scripting — the online version at postre-tool.csic.es
  supports Single SV submissions and avoids local installation
- The SVs are implicated in recessive phenotypes — POSTRE is oriented towards prediction
  of SVs causing dominant disorders and its capacity for recessive phenotypes has
  not been specifically evaluated
- SV coordinates are not in hg19 and cannot be lifted — the POSTRE_multiSV() mandatory
  argument requires hg19 coordinates
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: POSTRE_multiSV()
  role: Core R function that sequentially analyzes multiple SVs, taking the 7-column
    SV data frame and the Postre app folder path as mandatory arguments and returning
    per-SV and per-gene prediction tables
  repo: https://github.com/vicsanga/Postre
- name: Postre_wrapper.R
  role: Single source() call to run the latest version of POSTRE in an R console without
    manually downloading the repository
  repo: https://github.com/vicsanga/Postre
- name: R (>=3.5.0)
  role: Runtime environment required to run POSTRE; only R is required, RStudio is
    optional
- name: Shiny
  role: Framework on which the POSTRE graphical interface (Postre_app folder) is built;
    the app folder path must be supplied to POSTRE_multiSV()
  repo: https://shiny.rstudio.com/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:358bfdffdc4f24d3e6a1e93391e28b29430414feae5e2c6b6ea6c2d41b906d7a
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
  - POSTRE_multiSV()
  - Postre_wrapper.R
  - R
  - R (>=3.5.0)
  - STRVCTVRE
  - SVScore
  - Shiny
  - TADA
  - TADeus2
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# r-scripted-batch-sv-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs POSTRE programmatically in R for sequential analysis of multiple structural variants, using the POSTRE_multiSV() function instead of the Shiny GUI. This enables batch pathogenicity prediction of coding and long-range (enhancer-driven) SV pathomechanisms in a cell type/tissue-specific manner.

## When to use

Use when you need to analyze multiple structural variants (e.g. a patient cohort or benchmark set) with POSTRE beyond the online single-SV interface, by scripting POSTRE_multiSV() in R (>=3.5.0).

## When NOT to use

- You only need a single-SV analysis without scripting — the online version at postre-tool.csic.es supports Single SV submissions and avoids local installation
- The SVs are implicated in recessive phenotypes — POSTRE is oriented towards prediction of SVs causing dominant disorders and its capacity for recessive phenotypes has not been specifically evaluated
- SV coordinates are not in hg19 and cannot be lifted — the POSTRE_multiSV() mandatory argument requires hg19 coordinates

## Inputs

- Tab-separated plain text file with one line and 7 tab-separated columns per SV: breakpoint 1 chromosome, breakpoint 1 hg19 coordinates (or comma-separated range), breakpoint 2 chromosome, breakpoint 2 hg19 coordinates, SV type (Inversion, Translocation, Deletion or Duplication), comma-separated phenotype list (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary), unique SV identifier (e.g. Patient1_SV3)
- R data frame with 7 columns containing SV information (coordinates in hg19), e.g. loaded with read.delim as done for Table1_LongRange_SVs.tsv

## Outputs

- pathogenicityPrediction_per_SV: table with a pathogenic prediction for each SV and associated phenotype analyzed
- geneStats_recurrencyAndPathomech: table aggregating pathogenic predictions per gene, phenotype and pathogenic mechanism (coding or long-range)
- Downloadable txt tables when run through the Multiple SV Submission interface

## How to apply

Load the SV cohort as a data frame via read.delim from a tab-separated file in the POSTRE 7-column format (breakpoint 1 chromosome, hg19 coordinates 1, breakpoint 2 chromosome, hg19 coordinates 2, SV type among Inversion/Translocation/Deletion/Duplication, comma-separated phenotypes among head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system or pituitary, unique SV id). For non-base-pair-resolution breakpoints, provide comma-separated coordinate ranges (e.g. 85092268,85092269); for intrachromosomal SVs, breakpoint 1 must have the smaller coordinate. Source Postre_app/POSTRE_multiSV.R (or the latest version via Postre_wrapper.R) and call POSTRE_multiSV(SVs = svs_patients, pathTo_Postre_app_Folder = ...); run the function with no arguments to inspect all mandatory/optional parameters. The function returns two output tables: pathogenicityPrediction_per_SV (a pathogenic prediction per SV and associated phenotype) and geneStats_recurrencyAndPathomech (predictions aggregated per gene, phenotype and pathogenic mechanism — coding or long-range). Re-source the script each time a new POSTRE version is released, as documented in the update history.

## Related tools

- **POSTRE_multiSV()** (Core R function that sequentially analyzes multiple SVs, taking the 7-column SV data frame and the Postre app folder path as mandatory arguments and returning per-SV and per-gene prediction tables) — https://github.com/vicsanga/Postre
- **Postre_wrapper.R** (Single source() call to run the latest version of POSTRE in an R console without manually downloading the repository) — https://github.com/vicsanga/Postre
- **R (>=3.5.0)** (Runtime environment required to run POSTRE; only R is required, RStudio is optional)
- **Shiny** (Framework on which the POSTRE graphical interface (Postre_app folder) is built; the app folder path must be supplied to POSTRE_multiSV()) — https://shiny.rstudio.com/

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); svs <- read.delim("Table1_LongRange_SVs.tsv"); POSTRE_multiSV(SVs = svs, pathTo_Postre_app_Folder = "/PathTo/Postre_app")
```

## Evaluation signals

- Input data frame has exactly 7 columns matching the documented format, with hg19 coordinates and SV types restricted to Inversion, Translocation, Deletion or Duplication
- Phenotype column values are restricted to the documented options: head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system or pituitary
- Both output tables are produced: a per-SV/phenotype prediction table and a per-gene/phenotype/mechanism aggregation table
- Running POSTRE_multiSV() with no arguments returns the documented parameter and output details, confirming correct sourcing of POSTRE_multiSV.R
- Benchmark validation: on the 21-SV positive-control set, POSTRE's gene rankings reproduce the reported 86% (18/21) correct disease-causative gene predictions when compared against TADeus2's 67% (14/21)

## Limitations

- POSTRE uses bulk genomic data (e.g. whole limb buds), so some enhancers such as the ZRS cannot be identified and it fails to predict the SHH duplication case
- Lack of genomic data for the appropriate developmental stages or differentiation time points may affect identification of pathologically relevant genes
- Capacity to identify SVs implicated in recessive phenotypes has not been specifically evaluated; the tool is oriented towards dominant disorders
- The online version supports Single SV submissions only — Multiple SV submissions require local installation
- When using POSTRE as an R function, steps must be repeated each time a new version is released

## Evidence

- [methods] To use it, it requires 2 mandatory arguments. SVs: Data frame with 7 columns containing the SVs information (coordinates have to be in hg19).: [evidence span withheld — anchor status: not_found]
