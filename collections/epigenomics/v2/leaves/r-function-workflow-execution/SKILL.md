---
name: r-function-workflow-execution
description: Use when many SVs must be scored for pathogenicity in a scripted, parameterized
  R (>=3.5.0) pipeline rather than through the Shiny Multiple SV Submission GUI, e.g.
  batch benchmarking of hg19 SV sets.
when_to_use_negative:
- Only a single SV needs analysis — the online version at postre-tool.csic.es supports
  Single SV submissions without local installation
- SV coordinates are not in hg19, or phenotypes fall outside the supported categories
  (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system,
  pituitary)
- The goal is recessive-disorder SV interpretation — POSTRE is oriented towards dominant
  disorders and its capacity for recessive phenotypes has not been specifically evaluated
edam_operation: http://edamontology.org/operation_3958
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3175
tools:
- name: POSTRE_multiSV()
  role: Batch R function sourced from Postre_app/POSTRE_multiSV.R and invoked with
    the 7-column SV data frame and Postre_app folder path to score each SV against
    cell-type-specific enhancer, TAD and expression data
  repo: https://github.com/vicsanga/Postre
- name: POSTRE
  role: Shiny-based tool whose Postre_app folder supplies the mandatory pathTo_Postre_app_Folder
    argument and whose testFiles folder provides example SV inputs
  repo: https://github.com/vicsanga/Postre
- name: R
  role: Runtime (version >=3.5.0) required to source POSTRE_multiSV.R and execute
    POSTRE_multiSV() locally
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:af5dbc71e3b9bfd9310f60dafa43342032f6f82b43ee5bc5cbee4c8f07c6656d
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
  edam_operation: http://edamontology.org/operation_3958
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# r-function-workflow-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs POSTRE's batch structural-variant pathogenicity predictor as an R function: source Postre_app/POSTRE_multiSV.R in R (>=3.5.0) and call POSTRE_multiSV() on a 7-column hg19 SV data frame to return per-SV and per-gene pathogenicity tables covering coding and long-range pathomechanisms.

## When to use

Use when many SVs must be scored for pathogenicity in a scripted, parameterized R (>=3.5.0) pipeline rather than through the Shiny Multiple SV Submission GUI, e.g. batch benchmarking of hg19 SV sets.

## When NOT to use

- Only a single SV needs analysis — the online version at postre-tool.csic.es supports Single SV submissions without local installation
- SV coordinates are not in hg19, or phenotypes fall outside the supported categories (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary)
- The goal is recessive-disorder SV interpretation — POSTRE is oriented towards dominant disorders and its capacity for recessive phenotypes has not been specifically evaluated

## Inputs

- Plain-text, tab-separated SV file (1 line and 7 columns per SV): chromosome 1, coordinates 1 (hg19), chromosome 2, coordinates 2, SV type (Inversion/Translocation/Deletion/Duplication), comma-separated phenotype list, unique SV id — e.g. testFiles/Table1_LongRange_SVs.tsv or ExampleMultipleSubmission.tsv
- R data frame with 7 columns containing SVs in hg19 coordinates (the mandatory SVs argument)
- pathTo_Postre_app_Folder: local path to the downloaded Postre_app folder holding cell-type-specific enhancer, TAD and expression data

## Outputs

- pathogenicityPrediction_per_SV ('Results per SV and phenotype') table with a pathogenicity prediction for each SV-phenotype association analyzed
- geneStats_recurrencyAndPathomech ('Results per gene and phenotype') table aggregating predictions per gene, phenotype and pathogenic mechanism (coding, long-range)
- errors list element reporting failed submissions
- Downloadable txt tables, e.g. postre_pathogenicity_predictions.txt

## How to apply

Download the Postre repository and, in R (>=3.5.0), load the predictor with source("/PathTo/Postre_app/POSTRE_multiSV.R") (or source the Postre_wrapper.R URL to always fetch the latest version). Read the SV file with read.delim(sep="\t", stringsAsFactors=FALSE, header=FALSE) so each row carries the seven required columns: chromosome 1, coordinates 1 (hg19, comma-separated ranges such as 85092268,85092269 when base-pair resolution is unavailable), chromosome 2, coordinates 2, SV type (Inversion, Translocation, Deletion or Duplication), comma-separated phenotypes from head_neck/limbs/neurodevelopmental/cardiovascular/vision_eye/liver_biliary_system/pituitary, and a unique SV id. Call POSTRE_multiSV(SVs=<df>, pathTo_Postre_app_Folder="<path>/Postre_app/") with default parameters; running the function without arguments prints parameter and output documentation. Extract the returned list elements pathogenicityPrediction_per_SV ('Results per SV and phenotype'), geneStats_recurrencyAndPathomech ('Results per gene and phenotype') and errors, writing the per-SV table to a tab-separated txt such as postre_pathogenicity_predictions.txt. Re-source the script whenever a new POSTRE version is released, and validate by confirming a labelled prediction exists for every SV-phenotype pair and that positive controls (e.g. the BOFS inversion causing loss of TFAP2A expression in neural crest cells via enhancer disconnection) are recovered.

## Related tools

- **POSTRE_multiSV()** (Batch R function sourced from Postre_app/POSTRE_multiSV.R and invoked with the 7-column SV data frame and Postre_app folder path to score each SV against cell-type-specific enhancer, TAD and expression data) — https://github.com/vicsanga/Postre
- **POSTRE** (Shiny-based tool whose Postre_app folder supplies the mandatory pathTo_Postre_app_Folder argument and whose testFiles folder provides example SV inputs) — https://github.com/vicsanga/Postre
- **R** (Runtime (version >=3.5.0) required to source POSTRE_multiSV.R and execute POSTRE_multiSV() locally)

## Examples

```
source('/PathTo/Postre_app/POSTRE_multiSV.R'); pred <- POSTRE_multiSV(SVs=svs, pathTo_Postre_app_Folder='/PathTo/Postre_app/')
```

## Evaluation signals

- The returned list contains the three elements pathogenicityPrediction_per_SV, geneStats_recurrencyAndPathomech and errors
- postre_pathogenicity_predictions.txt exists and holds a labelled pathogenicity prediction for every input SV-phenotype pair
- Positive-control SVs (Table 1) are classified as pathogenic with their reported causative genes and coding/long-range mechanisms, e.g. the BOFS inversion predicted to cause TFAP2A loss in neural crest cells via enhancer disconnection
- Benchmarking against healthy-individual gnomAD SVs yields mostly non-pathogenic calls (96.8% Standard / 99.2% High Specificity for the 10,000-SV random set; 89.5%/98.3% for the 500 largest SVs)
- Running POSTRE_multiSV() without arguments returns the parameter and output documentation, confirming the script was sourced correctly

## Limitations

- Bulk whole limb bud data cannot identify the ZRS enhancer, so POSTRE fails to predict the SHH duplication case and SHH appears as an inactive gene
- POSTRE is oriented towards the prediction of SVs causing dominant disorders; its capacity to identify SVs implicated in recessive phenotypes has not been specifically evaluated
- Incomplete enhancer characterization across human cell types and lack of genomic data for appropriate developmental stages can limit long-range pathomechanism prediction; the online version only supports Single SV submissions, so Multiple SV submissions require local installation

## Evidence

- [methods] Mandatory arguments: [evidence span withheld — anchor status: not_found]
