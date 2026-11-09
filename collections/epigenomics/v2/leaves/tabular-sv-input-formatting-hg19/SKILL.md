---
name: tabular-sv-input-formatting-hg19
description: Use when SV calls (VCF, BEDPE, or paper tables) must be converted into
  POSTRE's 7-column tab-separated hg19 format before running POSTRE_multiSV() or the
  Shiny Multiple SV Submission.
when_to_use_negative:
- Coordinates are on a non-hg19 assembly (e.g. hg38) and cannot be liftover — POSTRE
  requires hg19
- The SV has more than two breakpoints or a type outside Inversion/Translocation/Deletion/Duplication
  — the format requires exactly two breakpoints per SV
- Only a single SV needs analysis via the online version at postre-tool.csic.es, which
  supports Single SV submissions only (Multiple SV submissions require local installation)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: POSTRE
  role: Target tool whose 7-column tab-separated hg19 SV schema this formatting skill
    produces input for
  repo: https://github.com/vicsanga/Postre
- name: POSTRE_multiSV()
  role: R function consuming the formatted 7-column data frame (SVs argument, hg19
    coordinates) for batch pathogenicity prediction
  repo: https://github.com/vicsanga/Postre
- name: R (>=3.5.0)
  role: Runtime used to read the TSV with read.delim and call POSTRE_multiSV()
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:5ca1367a604efbd569167eb344da93f02f8dd9b8c03a9b67dc7d14a5805ae5ae
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
  - POSTRE_multiSV()
  - R
  - R (>=3.5.0)
  - STRVCTVRE
  - SVScore
  - TADA
  - TADeus2
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# tabular-sv-input-formatting-hg19

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Formats structural variant calls into POSTRE's 7-column tab-separated schema (two breakpoints, hg19 coordinates, SV type, phenotypes, SV id) so they can be loaded with read.delim in R and passed to POSTRE_multiSV() for cell type-specific pathogenicity prediction.

## When to use

Use when SV calls (VCF, BEDPE, or paper tables) must be converted into POSTRE's 7-column tab-separated hg19 format before running POSTRE_multiSV() or the Shiny Multiple SV Submission.

## When NOT to use

- Coordinates are on a non-hg19 assembly (e.g. hg38) and cannot be liftover — POSTRE requires hg19
- The SV has more than two breakpoints or a type outside Inversion/Translocation/Deletion/Duplication — the format requires exactly two breakpoints per SV
- Only a single SV needs analysis via the online version at postre-tool.csic.es, which supports Single SV submissions only (Multiple SV submissions require local installation)

## Inputs

- SV call set (e.g. paper Table1_LongRange_SVs.tsv, testFiles/ExampleMultipleSubmission.tsv, or patient VCF/BEDPE records)
- hg19 genomic coordinates for both breakpoints (base-pair resolution or comma-separated ranges)
- SV type from {Inversion, Translocation, Deletion, Duplication}
- Comma-separated phenotype categories from {head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary}
- Unique SV identifier per variant (e.g. Patient1_SV3)

## Outputs

- Plain text / TSV file with 1 line and 7 tab-separated columns per SV
- R data frame with 7 columns (SVs argument) loadable via read.delim and accepted by POSTRE_multiSV()

## How to apply

Build a plain text file or R data frame with one line per SV and exactly 7 tab-separated columns: chromosome of breakpoint 1, hg19 coordinate of breakpoint 1, chromosome of breakpoint 2, hg19 coordinate of breakpoint 2, SV type, comma-separated phenotype list, and a unique SV identifier. When breakpoints lack base-pair resolution, supply a comma-separated range such as 85092268,85092269. For intrachromosomal SVs (deletions, inversions, duplications) order breakpoint 1 at the smaller coordinate and breakpoint 2 at the larger; for translocations order does not matter. Restrict SV type to Inversion, Translocation, Deletion or Duplication, and phenotypes to head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system or pituitary (e.g. head_neck,neurodevelopmental,cardiovascular). Load the file in R with read.delim (as done for Table1_LongRange_SVs.tsv) and pass the resulting data frame as the SVs argument to POSTRE_multiSV(), which requires hg19 coordinates.

## Related tools

- **POSTRE** (Target tool whose 7-column tab-separated hg19 SV schema this formatting skill produces input for) — https://github.com/vicsanga/Postre
- **POSTRE_multiSV()** (R function consuming the formatted 7-column data frame (SVs argument, hg19 coordinates) for batch pathogenicity prediction) — https://github.com/vicsanga/Postre
- **R (>=3.5.0)** (Runtime used to read the TSV with read.delim and call POSTRE_multiSV())

## Examples

```
svs_patients <- read.delim("Table1_LongRange_SVs.tsv", header=FALSE); source("Postre_app/POSTRE_multiSV.R"); POSTRE_multiSV(SVs = svs_patients, pathTo_Postre_app_Folder = "Postre_app/")
```

## Evaluation signals

- Every SV row parses to exactly 7 tab-separated fields with no missing columns
- All coordinates are hg19; non-base-pair breakpoints use the comma-separated range form (e.g. 85092268,85092269)
- SV type is one of Inversion, Translocation, Deletion, Duplication; phenotypes are drawn only from the seven supported categories
- Intrachromosomal SVs have breakpoint 1 at the smaller coordinate and breakpoint 2 at the larger
- read.delim loads the file and POSTRE_multiSV(SVs = svs_patients, pathTo_Postre_app_Folder = ...) runs without schema errors, returning pathogenicityPrediction_per_SV and geneStats_recurrencyAndPathomech

## Limitations

- Each SV must contain only two breakpoints; complex variants cannot be represented directly
- Phenotype categories are fixed to the seven supported tissue/system groups, limiting submission to POSTRE's covered congenital-disorder domains (limb, craniofacial, cardiac, neurodevelopmental, etc.)
- Coordinates must be in hg19; no assembly conversion is performed by the format itself

## Evidence
