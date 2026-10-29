---
name: structural-variant-breakpoint-parsing-hg19
description: Use when a user provides raw SV calls (two breakpoints, SV type) from
  a patient and they must be converted into the 7-column tab-separated hg19 format
  required by POSTRE before running POSTRE_multiSV().
when_to_use_negative:
- When coordinates are in a genome build other than hg19 — POSTRE requires hg19 and
  coordinates must be lifted/converted first
- When the variant is a single-nucleotide or small indel variant rather than a structural
  variant with two breakpoints (POSTRE handles SVs with coding or long-range pathomechanisms)
- When the phenotype of interest is not among the supported categories (head_neck,
  limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: POSTRE_multiSV()
  role: Consumes the parsed 7-column SV data frame (mandatory argument SVs, hg19 coordinates)
    to run sequential analysis of multiple structural variants
  repo: https://github.com/vicsanga/Postre
- name: POSTRE
  role: Downstream tool whose input format specification defines the 7-column tab-separated,
    hg19, one-line-per-SV structure produced by this skill
  repo: https://github.com/vicsanga/Postre
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:6809621180b0d6c976369554e409eceb3d3e014f210884aa42cad6ec3aa4e8a5
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# structural-variant-breakpoint-parsing-hg19

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse and normalize patient structural variant (SV) records into POSTRE's required plain-text format: one line per SV with 7 tab-separated columns (chromosome and hg19 coordinates for two breakpoints, SV type, comma-separated phenotypes, unique SV id), so POSTRE_multiSV() can compute cell type-specific pathogenicity predictions.

## When to use

Use when a user provides raw SV calls (two breakpoints, SV type) from a patient and they must be converted into the 7-column tab-separated hg19 format required by POSTRE before running POSTRE_multiSV().

## When NOT to use

- When coordinates are in a genome build other than hg19 — POSTRE requires hg19 and coordinates must be lifted/converted first
- When the variant is a single-nucleotide or small indel variant rather than a structural variant with two breakpoints (POSTRE handles SVs with coding or long-range pathomechanisms)
- When the phenotype of interest is not among the supported categories (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary)

## Inputs

- Raw SV calls with two breakpoints per variant (chromosome + genomic position or range for each breakpoint)
- SV type classification (Inversion, Translocation, Deletion, or Duplication)
- Comma-separated list of associated phenotypes (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary)
- Unique SV/patient identifiers

## Outputs

- Plain text file with 1 line and 7 tab-separated columns per SV (e.g. ExampleMultipleSubmission.tsv format) with hg19 coordinates
- Data frame with 7 columns suitable as the mandatory SVs argument of POSTRE_multiSV()

## How to apply

Build one line per SV with exactly 7 tab-separated columns: (1) chromosome of breakpoint 1, (2) hg19 genomic coordinates of breakpoint 1 — when base-pair resolution is unavailable provide a comma-separated range such as 85092268,85092269, (3) chromosome of breakpoint 2, (4) hg19 coordinates of breakpoint 2, (5) SV type restricted to Inversion, Translocation, Deletion or Duplication, (6) comma-separated phenotype list chosen from head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system or pituitary, and (7) a unique SV id such as Patient1_SV3. For strictly intrachromosomal SVs (deletions, inversions, duplications), order breakpoints so breakpoint 1 carries the smaller genomic coordinate and breakpoint 2 the larger; for translocations the ordering does not matter. Each SV must contain only two breakpoints. Write the records as a plain text file with tab separators (e.g. ExampleMultipleSubmission.tsv style) and supply it to POSTRE_multiSV() as a data frame with 7 columns.

## Related tools

- **POSTRE_multiSV()** (Consumes the parsed 7-column SV data frame (mandatory argument SVs, hg19 coordinates) to run sequential analysis of multiple structural variants) — https://github.com/vicsanga/Postre
- **POSTRE** (Downstream tool whose input format specification defines the 7-column tab-separated, hg19, one-line-per-SV structure produced by this skill) — https://github.com/vicsanga/Postre

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); POSTRE_multiSV(SVs, pathTo_Postre_app_Folder)
```

## Evaluation signals

- Each SV line has exactly 7 tab-separated fields and each SV contains only two breakpoints
- For intrachromosomal SVs (deletions, inversions, duplications), breakpoint 1 coordinate is smaller than breakpoint 2; ordering rule relaxed only for translocations
- Column 5 SV type is one of Inversion, Translocation, Deletion, Duplication; column 6 phenotypes are drawn from the seven supported phenotype options
- All coordinates are valid hg19 positions or comma-separated ranges (e.g. 85092268,85092269) and each SV id is unique
- The formatted file loads as a 7-column data frame and POSTRE_multiSV() accepts it without argument errors (running POSTRE_multiSV() without arguments documents expected parameters/outputs)

## Limitations

- Only SV types Inversion, Translocation, Deletion and Duplication are supported; each SV must have exactly two breakpoints
- Coordinates must be in hg19; other genome builds require prior conversion
- Only the seven predefined phenotype categories are accepted, reflecting POSTRE's coverage of limb, craniofacial, cardiac and neurodevelopmental congenital abnormalities plus vision, liver/biliary and pituitary data

## Evidence
