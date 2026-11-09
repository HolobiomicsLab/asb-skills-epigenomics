---
name: gene-pathomechanism-interpretation
description: Use when patient SVs (inversions, translocations, deletions, duplications)
  in hg19 coordinates need candidate causative genes and coding or long-range pathomechanisms
  assigned in a phenotype-specific cellular context.
when_to_use_negative:
- The phenotype of interest is recessive — POSTRE is oriented towards dominant disorders
  and its capacity for recessive phenotypes has not been specifically evaluated
- The pathomechanism depends on an enhancer not detectable in POSTRE's bulk tissue
  data (e.g. the ZRS enhancer within whole limb buds, so SHH duplication cases fail)
- Only a single SV needs quick analysis without installation — the online version
  supports Single SV submissions, but Multiple SV submissions require local installation
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: POSTRE
  role: 'Core tool: cell type/tissue-specific prediction of SV pathogenicity, causative
    genes, and coding/long-range pathomechanisms'
  repo: https://github.com/vicsanga/Postre
- name: POSTRE_multiSV()
  role: R function for sequential analysis of multiple SVs; takes the 7-column SV
    data frame and path to Postre_app folder, returning the two results tables
  repo: https://github.com/vicsanga/Postre
- name: R (>=3.5.0)
  role: Runtime environment required to source POSTRE_multiSV.R and execute the analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:b33ed9cecce1c801ffa8fa3704ded6115442b5a6f5d14af8b9cf4abfb81845a3
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
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# gene-pathomechanism-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interpret structural variant (SV) predictions into concrete causative genes and pathomechanisms (coding gene deletion vs long-range enhancer disconnection/adoption) using POSTRE's cell type-specific outputs, comparing Standard and High-Specificity running modes.

## When to use

Use when patient SVs (inversions, translocations, deletions, duplications) in hg19 coordinates need candidate causative genes and coding or long-range pathomechanisms assigned in a phenotype-specific cellular context.

## When NOT to use

- The phenotype of interest is recessive — POSTRE is oriented towards dominant disorders and its capacity for recessive phenotypes has not been specifically evaluated
- The pathomechanism depends on an enhancer not detectable in POSTRE's bulk tissue data (e.g. the ZRS enhancer within whole limb buds, so SHH duplication cases fail)
- Only a single SV needs quick analysis without installation — the online version supports Single SV submissions, but Multiple SV submissions require local installation

## Inputs

- Plain text (TSV) file of SVs: 1 line and 7 tab-separated columns per SV (chromosome 1, hg19 coordinates 1, chromosome 2, hg19 coordinates 2, SV type, comma-separated phenotype list, unique SV id)
- Patient SV test files from the GitHub Postre testFiles folder (ExampleMultipleSubmission.tsv as format reference)
- Phenotype labels restricted to: head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary

## Outputs

- pathogenicityPrediction_per_SV: table with a pathogenic prediction for each SV and associated phenotype
- geneStats_recurrencyAndPathomech: table aggregating predictions per gene, phenotype, and pathogenic mechanism (coding vs long-range), e.g. LOF/GOF calls on SATB2, MEF2C, FOXG1, SLC2A1
- errors element listing SV ids that failed processing
- Mode comparison: fraction of SVs predicted pathogenic per phenotype under Standard (95%) vs High-Specificity (66%) modes

## How to apply

Format each SV as one tab-separated line with 7 columns (chromosome 1, hg19 coordinates 1, chromosome 2, hg19 coordinates 2, SV type among Inversion/Translocation/Deletion/Duplication, comma-separated phenotypes from head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary, and a unique SV id), using ExampleMultipleSubmission.tsv as reference. In R (>=3.5.0), source Postre_app/POSTRE_multiSV.R, load the manifest with read.delim(sep="\t", header=FALSE), and call POSTRE_multiSV(SVs, pathTo_Postre_app_Folder) first in Standard mode and then in High-Specificity mode. Interpret the two output tables: pathogenicityPrediction_per_SV gives a per-SV/per-phenotype pathogenic call, while geneStats_recurrencyAndPathomech aggregates per gene, phenotype, and mechanism (coding vs long-range), e.g. SATB2, MEF2C, FOXG1, SLC2A1 with LOF/GOF mechanisms; check the errors element for SV ids that failed. Expect ~95% recovery of previously proposed causative genes in Standard mode but ~66% in High-Specificity mode, since the latter requires causative genes to sit in broad polycomb domains, which MEF2C and SLC2A1 lack — so prefer Standard mode when sensitivity matters and High-Specificity when precision matters.

## Related tools

- **POSTRE** (Core tool: cell type/tissue-specific prediction of SV pathogenicity, causative genes, and coding/long-range pathomechanisms) — https://github.com/vicsanga/Postre
- **POSTRE_multiSV()** (R function for sequential analysis of multiple SVs; takes the 7-column SV data frame and path to Postre_app folder, returning the two results tables) — https://github.com/vicsanga/Postre
- **R (>=3.5.0)** (Runtime environment required to source POSTRE_multiSV.R and execute the analysis)

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); SVs <- read.delim("ExampleMultipleSubmission.tsv", sep="\t", header=FALSE); res <- POSTRE_multiSV(SVs, pathTo_Postre_app_Folder)
```

## Evaluation signals

- SV manifest parses as a data frame with exactly 7 columns and all phenotype labels drawn from the allowed set (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary)
- For intronic variants, only those overlapping or disrupting a gene's coding sequence are marked as potentially truncating (updated annotation rule)
- Benchmark reproduction: ~95% of the 41 patient SVs recover the previously proposed causative genes/pathomechanisms in Standard mode; ~66% in High-Specificity mode
- Known sensitivity losses in High-Specificity mode are explained by causative genes lacking broad polycomb domains (MEF2C, SLC2A1)
- Any failed SVs appear as ids in the errors element of the POSTRE_multiSV() output rather than silently missing from results tables

## Limitations

- High-Specificity mode drops sensitivity to 66% because it requires causative genes to have broad polycomb domains, excluding genes like MEF2C and SLC2A1
- Bulk whole limb bud genomic data cannot identify the ZRS enhancer, so POSTRE fails to predict the SHH duplication case
- Incomplete characterization of enhancers across human cell types limits prediction of long-range pathomechanisms; lack of data for appropriate developmental stages can affect identification of pathologically relevant genes
- Enhancer-gene compatibility factors (linear distance, CpG islands, promoter DNA methylation, core promoter elements) are not yet incorporated into POSTRE's scoring
- Each SV must contain only two breakpoints, and coordinates must be in hg19; non-bp-resolution breakpoints are given as comma-separated ranges

## Evidence

- [methods] R function usage: [evidence span withheld — anchor status: not_found]
- [results] benchmark performance: [evidence span withheld — anchor status: not_found]
- [results] High-Specificity limitation: [evidence span withheld — anchor status: not_found]
