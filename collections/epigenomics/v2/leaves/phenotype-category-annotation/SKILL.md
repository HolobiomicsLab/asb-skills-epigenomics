---
name: phenotype-category-annotation
description: Use when preparing SV input for POSTRE and each variant must be tagged
  with one or more disease-relevant phenotype categories from the seven supported
  options to drive tissue-specific enhancer/TAD analysis.
when_to_use_negative:
- The patient's phenotype does not map to any of the seven supported categories (head_neck,
  limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary)
- The SV is implicated in a recessive phenotype, since POSTRE is oriented towards
  dominant disorders and its capacity for recessive phenotypes has not been specifically
  evaluated
- The input is already a POSTRE results table (e.g. geneStats_recurrencyAndPathomech)
  rather than raw SV breakpoints needing annotation
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: POSTRE
  role: Consumes the phenotype-annotated SV manifest to perform cell type/tissue-specific
    pathogenicity prediction of coding and long-range mechanisms
  repo: https://github.com/vicsanga/Postre
- name: POSTRE_multiSV()
  role: R function that ingests the 7-column annotated SV data frame and returns per-SV/phenotype
    and per-gene/phenotype/mechanism result tables
  repo: https://github.com/vicsanga/Postre
- name: R (>=3.5.0)
  role: Runtime required to load the annotated manifest with read.delim and execute
    POSTRE_multiSV()
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:6c3c049731e282e7b4c1a66e671bcd35e785295668d8c0d1198a325e784e8a45
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# phenotype-category-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates each structural variant (SV) with a comma-separated list of POSTRE-supported phenotype categories (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary) in column 6 of a tab-separated SV manifest, so POSTRE can run cell type/tissue-specific pathogenicity predictions.

## When to use

Use when preparing SV input for POSTRE and each variant must be tagged with one or more disease-relevant phenotype categories from the seven supported options to drive tissue-specific enhancer/TAD analysis.

## When NOT to use

- The patient's phenotype does not map to any of the seven supported categories (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary)
- The SV is implicated in a recessive phenotype, since POSTRE is oriented towards dominant disorders and its capacity for recessive phenotypes has not been specifically evaluated
- The input is already a POSTRE results table (e.g. geneStats_recurrencyAndPathomech) rather than raw SV breakpoints needing annotation

## Inputs

- Plain text/tab-separated SV manifest with 7 columns per SV (chromosome 1, hg19 coordinates 1, chromosome 2, hg19 coordinates 2, SV type, phenotype list, unique SV id)
- Patient clinical phenotype information mapped to the seven POSTRE categories
- ExampleMultipleSubmission.tsv format reference from the Postre testFiles folder

## Outputs

- Annotated SV file with column 6 populated by comma-separated phenotype category lists
- Downstream: pathogenicityPrediction_per_SV table (pathogenic prediction per SV and phenotype)
- Downstream: geneStats_recurrencyAndPathomech table (aggregated predictions per gene, phenotype and mechanism: coding vs long-range)

## How to apply

Store SV information in a plain text file with 7 tab-separated columns; column 6 holds the comma-separated phenotype list drawn strictly from head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system or pituitary (e.g. head_neck,neurodevelopmental,cardiovascular). The phenotype categories determine which cell type/tissue-specific regulatory landscapes (enhancers and TADs) POSTRE consults, so annotation should reflect the patient's clinically reported abnormalities (hg19 coordinates in columns 2 and 4, comma-separated ranges when breakpoint resolution is not base pair). For one-chromosome SVs (deletions, inversions, duplications) order breakpoints by coordinate before annotating; for translocations order does not matter. After annotation, load the manifest in R with read.delim(sep="\t", header=FALSE) and pass it to POSTRE_multiSV() in Standard or High-Specificity mode; per-phenotype predictions appear in pathogenicityPrediction_per_SV and geneStats_recurrencyAndPathomech.

## Related tools

- **POSTRE** (Consumes the phenotype-annotated SV manifest to perform cell type/tissue-specific pathogenicity prediction of coding and long-range mechanisms) — https://github.com/vicsanga/Postre
- **POSTRE_multiSV()** (R function that ingests the 7-column annotated SV data frame and returns per-SV/phenotype and per-gene/phenotype/mechanism result tables) — https://github.com/vicsanga/Postre
- **R (>=3.5.0)** (Runtime required to load the annotated manifest with read.delim and execute POSTRE_multiSV())

## Examples

```
chr8	85092268	chr8	85249635	Inversion	head_neck,neurodevelopmental	Patient1_SV1  # column 6 = phenotype annotation; then: SVs <- read.delim("SVs.tsv", sep="\t", header=FALSE)
```

## Evaluation signals

- Every SV row has exactly 7 tab-separated columns and column 6 contains only values from the allowed set: head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary
- Coordinates in columns 2 and 4 are hg19; non-base-pair-resolution breakpoints are given as comma-separated ranges (e.g. 85092268,85092269)
- Column 5 is one of Inversion, Translocation, Deletion or Duplication; for one-chromosome SVs breakpoint 1 has the smaller coordinate
- Column 7 holds a unique SV identifier (e.g. Patient1_SV3) with no duplicates across the manifest
- Reproducing the 41-patient benchmark yields ~95% success in Standard mode and ~66% in High-Specificity mode, with causative genes such as SATB2, MEF2C, FOXG1 and SLC2A1 recoverable from the per-gene table

## Limitations

- Bulk whole limb bud data cannot identify the ZRS enhancer, so POSTRE fails to predict the SHH duplication case even with correct limb phenotype annotation
- Lack of genomic data for the appropriate developmental stages or differentiation time points may affect identification of pathologically relevant genes
- The online version (postre-tool.csic.es) supports only Single SV submissions; multiple annotated SVs must be run via the local R installation

## Evidence
