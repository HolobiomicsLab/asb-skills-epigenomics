---
name: healthy-cohort-sv-specificity-benchmarking
description: 'Use when validating the specificity of an SV pathogenicity classifier:
  SV sets from healthy individuals (e.g. gnomAD) should yield predominantly non-pathogenic
  calls, so a low non-pathogenic rate flags poor Type-1 error control.'
when_to_use_negative:
- Evaluating sensitivity on known pathogenic patient SVs — this skill measures Type-1
  error control on healthy-cohort SVs, not disease-causative SV recovery (e.g. the
  BOFS TFAP2A inversion case is assessed separately).
- SVs in coordinates other than hg19 — the required input format mandates hg19 genomic
  coordinates, so liftover is needed first or results are invalid.
- SVs implicating phenotypes outside POSTRE's supported categories (limb, craniofacial,
  cardiac, neurodevelopmental, vision_eye, liver_biliary_system, pituitary) — the
  tool cannot provide a meaningful cell type-specific prediction for other contexts.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3673
tools:
- name: POSTRE
  role: SV pathogenicity prediction tool being benchmarked; run in Standard and High-Specificity
    modes on healthy-cohort SV sets
  repo: https://github.com/vicsanga/Postre
- name: POSTRE_multiSV()
  role: R-function batch mode for the Multiple SV analysis; called with the SVs data
    frame and pathTo_Postre_app_Folder
  repo: https://github.com/vicsanga/Postre
- name: Shiny
  role: Framework of the Multiple SV Submission running mode (local Shiny app) used
    as the second running mode
  repo: https://shiny.rstudio.com/
- name: R (>=3.5.0)
  role: Runtime environment required to source POSTRE_multiSV.R and execute POSTRE_multiSV()
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:5069b0cf117add62b9cdf4568629b0038b9fecde82259507cfe08a7c6ebd0df3
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# healthy-cohort-sv-specificity-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a structural-variant pathogenicity predictor's Type-1 error control by running SVs from healthy individuals (gnomAD) through the tool and measuring the percentage of SV-phenotype associations called non-pathogenic under each running mode.

## When to use

Use when validating the specificity of an SV pathogenicity classifier: SV sets from healthy individuals (e.g. gnomAD) should yield predominantly non-pathogenic calls, so a low non-pathogenic rate flags poor Type-1 error control.

## When NOT to use

- Evaluating sensitivity on known pathogenic patient SVs — this skill measures Type-1 error control on healthy-cohort SVs, not disease-causative SV recovery (e.g. the BOFS TFAP2A inversion case is assessed separately).
- SVs in coordinates other than hg19 — the required input format mandates hg19 genomic coordinates, so liftover is needed first or results are invalid.
- SVs implicating phenotypes outside POSTRE's supported categories (limb, craniofacial, cardiac, neurodevelopmental, vision_eye, liver_biliary_system, pituitary) — the tool cannot provide a meaningful cell type-specific prediction for other contexts.

## Inputs

- Healthy-individual (gnomAD) SV sets from the POSTRE GitHub testFiles folder, including a random 10,000-SV set and the 500 largest SVs
- Plain-text SV files with 1 line and 7 tab-separated columns per SV (chromosome 1, hg19 coordinates 1, chromosome 2, hg19 coordinates 2, SV type, comma-separated phenotype list, unique SV id)
- Phenotype labels restricted to: head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary

## Outputs

- 'Results per SV and phenotype' table (downloadable txt) with a labelled pathogenicity prediction for each SV-phenotype association
- Non-pathogenic call percentage per SV set per running mode (Standard and High Specificity)
- Benchmark table comparing reproduced percentages to reported values (96.8%/99.2% random set; 89.5%/98.3% 500 largest SVs)

## How to apply

Obtain the healthy-individual (gnomAD) SV sets from the POSTRE GitHub testFiles folder, including the 500 largest SVs, and verify each entry follows the 7-column tab-separated hg19 format (chromosome 1, coordinates 1, chromosome 2, coordinates 2, SV type, phenotype list, unique SV id), where SV type is one of Inversion, Translocation, Deletion or Duplication and phenotypes are comma-separated from head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system or pituitary. Run the batch analysis through the R-function mode by sourcing Postre_app/POSTRE_multiSV.R in R (>=3.5.0) and calling POSTRE_multiSV() with the SVs data frame and pathTo_Postre_app_Folder, using default parameters. Run the same SV sets through the second running mode, the Shiny-based Multiple SV Submission, which returns downloadable txt tables. From each run, extract the 'Results per SV and phenotype' table and tally pathogenic vs non-pathogenic calls for every SV-phenotype association. Compute the non-pathogenic percentage per SV set and running mode, and compare against the reported benchmarks: 96.8% (Standard mode) and 99.2% (High Specificity mode) for the random 10,000-SV set, and 89.5%/98.3% for the 500 largest SVs, which the paper interprets as good Type-1 error control.

## Related tools

- **POSTRE** (SV pathogenicity prediction tool being benchmarked; run in Standard and High-Specificity modes on healthy-cohort SV sets) — https://github.com/vicsanga/Postre
- **POSTRE_multiSV()** (R-function batch mode for the Multiple SV analysis; called with the SVs data frame and pathTo_Postre_app_Folder) — https://github.com/vicsanga/Postre
- **Shiny** (Framework of the Multiple SV Submission running mode (local Shiny app) used as the second running mode) — https://shiny.rstudio.com/
- **R (>=3.5.0)** (Runtime environment required to source POSTRE_multiSV.R and execute POSTRE_multiSV())

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); POSTRE_multiSV(SVs, pathTo_Postre_app_Folder)
```

## Evaluation signals

- The reproduced non-pathogenic percentage under Standard mode matches the reported 96.8% for the random 10,000-SV healthy set (and 99.2% under High Specificity mode).
- The 500-largest-SV healthy set reproduces the reported 89.5% (Standard) and 98.3% (High Specificity) non-pathogenic rates.
- Every SV-phenotype association analyzed has a labelled pathogenicity prediction in the 'Results per SV and phenotype' output table (no missing calls).
- Input files pass the 7-column, tab-separated, hg19-format validation (with SV types from Inversion/Translocation/Deletion/Duplication and recognized phenotype labels) before running.
- The High Specificity mode yields a higher non-pathogenic percentage than the Standard mode across all healthy SV sets, as in the reported results.

## Limitations

- Bulk whole-tissue genomic data may miss cell-type-specific enhancers, which can affect predictions (e.g. POSTRE fails on the SHH ZRS duplication case because bulk limb bud data cannot identify the ZRS enhancer).
- Prediction of long-range pathomechanisms is limited by the still incomplete characterization of enhancers across different human cell types.
- POSTRE is oriented towards dominant disorders; its capacity to identify SVs implicated in recessive phenotypes has not been specifically evaluated.
- Machine-learning classifiers trained on databases biased towards coding deleterious effects risk overfitting if the training data does not faithfully recapitulate the general population.

## Evidence

- [results] Reported benchmark percentages for healthy-cohort SVs: [evidence span withheld — anchor status: not_found]
- [readme] Availability of test files including healthy-cohort sets: [evidence span withheld — anchor status: not_found]
