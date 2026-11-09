---
name: cell-type-specific-genomic-data-integration
description: Use when scoring structural variants (inversions, translocations, deletions,
  duplications) with hg19 breakpoints against phenotypes like head_neck, limbs, neurodevelopmental,
  cardiovascular, vision_eye, liver_biliary_system or pituitary to identify causative
  genes and coding vs long-range.
when_to_use_negative:
- The SV is suspected to cause a recessive disorder — POSTRE is oriented towards dominant
  disorders and its capacity for recessive phenotypes has not been evaluated
- The relevant cell type, developmental stage or differentiation time point lacks
  genomic data (e.g. bulk whole limb bud data cannot identify the ZRS enhancer, so
  the SHH duplication case fails)
- A single SV needs only a quick online check — the online version at postre-tool.csic.es
  supports only Single SV submissions, not Multiple SV submissions
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0654
tools:
- name: POSTRE
  role: Shiny-based tool that performs the cell type/tissue-specific integration of
    enhancers, TADs and expression to predict SV pathogenicity, causative genes and
    mechanisms
  repo: https://github.com/vicsanga/Postre
- name: POSTRE_multiSV()
  role: R function to run POSTRE programmatically on a data frame of multiple SVs
    with pathTo_Postre_app_Folder argument
  repo: https://github.com/vicsanga/Postre
- name: R (>=3.5.0)
  role: Runtime environment required to source POSTRE_multiSV.R and execute POSTRE_multiSV()
- name: Shiny
  role: Framework POSTRE is built with, providing the Single/Multiple SV submission
    GUI as an alternative to the R function
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:9e30f7d6a4cc7587dccf496c2dc33ab7cb8f8b1f7042b02141b0e96d2a42c40b
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
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cell-type-specific-genomic-data-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates cell type/tissue-specific enhancer, TAD and gene expression data to predict structural variant (SV) pathogenicity, causative genes, and long-range pathomechanisms (e.g. enhancer disconnection, enhancer adoption) as implemented in POSTRE.

## When to use

Use when scoring structural variants (inversions, translocations, deletions, duplications) with hg19 breakpoints against phenotypes like head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system or pituitary to identify causative genes and coding vs long-range mechanisms.

## When NOT to use

- The SV is suspected to cause a recessive disorder — POSTRE is oriented towards dominant disorders and its capacity for recessive phenotypes has not been evaluated
- The relevant cell type, developmental stage or differentiation time point lacks genomic data (e.g. bulk whole limb bud data cannot identify the ZRS enhancer, so the SHH duplication case fails)
- A single SV needs only a quick online check — the online version at postre-tool.csic.es supports only Single SV submissions, not Multiple SV submissions

## Inputs

- SV data frame or plain-text TSV with 7 columns: chr1, coordinates1 (hg19), chr2, coordinates2, SV type (Inversion/Translocation/Deletion/Duplication), comma-separated phenotypes (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary), SV id
- Example/test files such as testFiles/Table1_LongRange_SVs.tsv and testFiles/ExampleMultipleSubmission.tsv
- Postre_app folder containing cell-type-specific enhancer, TAD and expression data

## Outputs

- pathogenicityPrediction_per_SV: table with a pathogenic prediction for each SV and associated phenotype (downloadable as txt)
- geneStats_recurrencyAndPathomech: pathogenic predictions aggregated per gene, phenotype and pathogenic mechanism (coding, long-range)
- errors list from POSTRE_multiSV() output

## How to apply

Represent each SV as a 7-column, tab-separated record: chromosome and hg19 coordinates for both breakpoints (comma-separated ranges when resolution is lower than base pair), SV type (Inversion, Translocation, Deletion or Duplication), comma-separated phenotype list, and a unique SV id. In R (>=3.5.0), source the Postre_app/POSTRE_multiSV.R script from the downloaded repository and call POSTRE_multiSV() with the SVs data frame and pathTo_Postre_app_Folder so each SV is scored against cell-type-specific enhancer, TAD and expression data for the selected phenotypes. The rationale is that exact pathomechanisms can only be predicted if cell type-specific regulatory landscapes (enhancers and TADs) are considered, rather than generic coding-effect classifiers. Interpret the returned list elements: pathogenicityPrediction_per_SV (one pathogenic prediction per SV/phenotype) and geneStats_recurrencyAndPathomech (aggregated per gene, phenotype and mechanism: coding or long-range). Validate against known positive controls, e.g. the BOFS inversion should yield loss of TFAP2A expression in neural crest cells via enhancer disconnection.

## Related tools

- **POSTRE** (Shiny-based tool that performs the cell type/tissue-specific integration of enhancers, TADs and expression to predict SV pathogenicity, causative genes and mechanisms) — https://github.com/vicsanga/Postre
- **POSTRE_multiSV()** (R function to run POSTRE programmatically on a data frame of multiple SVs with pathTo_Postre_app_Folder argument) — https://github.com/vicsanga/Postre
- **R (>=3.5.0)** (Runtime environment required to source POSTRE_multiSV.R and execute POSTRE_multiSV())
- **Shiny** (Framework POSTRE is built with, providing the Single/Multiple SV submission GUI as an alternative to the R function)

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); svs <- read.delim("testFiles/Table1_LongRange_SVs.tsv", sep="\t", stringsAsFactors=FALSE, header=FALSE); res <- POSTRE_multiSV(SVs=svs, pathTo_Postre_app_Folder="/PathTo/Postre_app/")
```

## Evaluation signals

- Positive-control check: for five out of six experimentally validated long-range SV patients (Table 1), POSTRE predicts the single causative gene and long-range pathomechanism, giving 83% sensitivity and 100% specificity at the gene level
- BOFS syndrome inversion (Laugsch et al., 2019) is predicted to cause loss of TFAP2A expression in neural crest cells via enhancer disconnection
- Every input SV-phenotype pair receives a pathogenic prediction row in the Results per SV and phenotype table
- Reported long-range cases are classified under the long-range pathomechanism class rather than coding
- Calling POSTRE_multiSV() with no arguments returns parameter and output details, confirming the function is correctly sourced

## Limitations

- Incomplete characterization of enhancers across human cell types limits prediction of long-range pathomechanisms
- Bulk whole limb bud data cannot identify the ZRS enhancer, so POSTRE fails to predict the SHH duplication case
- Lack of genomic data for appropriate developmental stages or differentiation time points may affect identification of pathologically relevant genes
- POSTRE is oriented towards dominant disorders; its capacity for recessive phenotypes has not been specifically evaluated
- Additional enhancer-gene compatibility factors (linear distance, CpG islands, promoter DNA methylation, core promoter elements) are not yet incorporated into the scoring

## Evidence

- [abstract] By considering disease-relevant cellular contexts, POSTRE identifies SVs with either coding or long-range pathological consequences with high specificity and sensitivity.: "By considering disease-relevant cellular contexts, POSTRE identifies SVs with either coding or long-range pathological consequences with high specificity and sensitivity."
- [intro] the exact pathomechanisms can only be predicted and experimentally validated if cell type specific regulatory landscapes (i.e. enhancers and TADs) are considered: "the exact pathomechanisms can only be predicted and experimentally validated if cell type specific regulatory landscapes (i.e. enhancers and TADs) are considered"
- [methods] SVs: Data frame with 7 columns containing the SVs information (coordinates have to be in hg19).: "SVs: Data frame with 7 columns containing the SVs information (coordinates have to be in hg19)."
- [methods] For this patient, with BOFS syndrome carrying an inversion (Laugsch et al., 2019), POSTRE successfully predicts the loss of TFAP2A expression in neural crest cells through an enhancer disconnection: [evidence span withheld — anchor status: not_found]
- [readme] The first one (Results per SV and phenotype) is a table with a pathogenic prediction for each of the SVs and associated phenotypes analyzed.: "The first one (Results per SV and phenotype) is a table with a pathogenic prediction for each of the SVs and associated phenotypes analyzed."
- [discussion] since POSTRE uses bulk genomic data generated in whole limb buds (125,126), the ZRS enhancer can not be identified and SHH appears as an inactive gene: "since POSTRE uses bulk genomic data generated in whole limb buds (125,126), the ZRS enhancer can not be identified and SHH appears as an inactive gene"
