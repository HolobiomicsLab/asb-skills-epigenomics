---
name: tabular-sv-manifest-construction
description: Use when preparing patient structural variant (SV) breakpoints (hg19)
  for input into POSTRE or POSTRE_multiSV(), and the raw variant list must be converted
  into the tool's 7-column tab-separated manifest format with phenotypes and unique
  SV ids.
when_to_use_negative:
- The variants are SNVs/indels or have only one breakpoint — POSTRE requires each
  structural variant to contain exactly two breakpoints
- Coordinates are in a genome build other than hg19 — POSTRE requires hg19 genomic
  coordinates
- The phenotype of interest is not covered by POSTRE's seven supported phenotype categories,
  or the aim is recessive-disorder analysis, which POSTRE has not been evaluated for
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: POSTRE
  role: Target tool that consumes the 7-column SV manifest for pathogenicity prediction
    of coding and long-range pathomechanisms
  repo: https://github.com/vicsanga/Postre
- name: POSTRE_multiSV()
  role: R function called on the manifest (SVs data frame with 7 columns, hg19 coordinates)
    for sequential analysis of multiple SVs
  repo: https://github.com/vicsanga/Postre
- name: R (>=3.5.0)
  role: Runtime environment in which the manifest is loaded with read.delim and passed
    to POSTRE_multiSV()
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:7af06251d1348a1d1e61a7d631ea5744e5961deefca595fc4ddd908f002167de
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# tabular-sv-manifest-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a plain-text, tab-separated manifest of structural variants (7 columns per SV) in the exact format required by POSTRE's Single/Multiple SV Submission, so that SVs can be analyzed for coding or long-range pathomechanisms in a cell type-specific manner.

## When to use

Use when preparing patient structural variant (SV) breakpoints (hg19) for input into POSTRE or POSTRE_multiSV(), and the raw variant list must be converted into the tool's 7-column tab-separated manifest format with phenotypes and unique SV ids.

## When NOT to use

- The variants are SNVs/indels or have only one breakpoint — POSTRE requires each structural variant to contain exactly two breakpoints
- Coordinates are in a genome build other than hg19 — POSTRE requires hg19 genomic coordinates
- The phenotype of interest is not covered by POSTRE's seven supported phenotype categories, or the aim is recessive-disorder analysis, which POSTRE has not been evaluated for

## Inputs

- Patient SV breakpoint coordinates in hg19 (base-pair resolution or comma-separated ranges when lower resolution)
- SV type classification (Inversion, Translocation, Deletion or Duplication)
- Phenotype terms from the POSTRE vocabulary (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary)
- Unique SV/patient identifiers
- Format reference file: ExampleMultipleSubmission.tsv from the Postre testFiles folder

## Outputs

- Plain text tab-separated manifest with 1 line and 7 columns per SV
- R data frame (e.g. from read.delim with sep='\t', header=FALSE) suitable as the SVs argument to POSTRE_multiSV()

## How to apply

Build one line per SV with exactly 7 tab-separated columns: chromosome 1, hg19 coordinates 1, chromosome 2, hg19 coordinates 2, SV type (Inversion, Translocation, Deletion or Duplication), a comma-separated phenotype list drawn from head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system or pituitary, and a unique SV identifier (e.g. Patient1_SV3). For intra-chromosomal SVs (deletions, inversions, duplications) order breakpoints so that breakpoint 1 has the smaller coordinate and breakpoint 2 the larger; for translocations ordering does not matter. When breakpoints lack base-pair resolution, provide a comma-separated coordinate range such as 85092268,85092269. Store the manifest as a plain text file with tabulations separating column values (TSV, no header expected when read via read.delim with header=FALSE), using the ExampleMultipleSubmission.tsv file in the Postre testFiles folder as the format reference before running POSTRE_multiSV(SVs, pathTo_Postre_app_Folder).

## Related tools

- **POSTRE** (Target tool that consumes the 7-column SV manifest for pathogenicity prediction of coding and long-range pathomechanisms) — https://github.com/vicsanga/Postre
- **POSTRE_multiSV()** (R function called on the manifest (SVs data frame with 7 columns, hg19 coordinates) for sequential analysis of multiple SVs) — https://github.com/vicsanga/Postre
- **R (>=3.5.0)** (Runtime environment in which the manifest is loaded with read.delim and passed to POSTRE_multiSV())

## Examples

```
SVs <- read.delim("patient_SVs.tsv", sep="\t", header=FALSE); source("/PathTo/Postre_app/POSTRE_multiSV.R"); POSTRE_multiSV(SVs, "/PathTo/Postre_app")
```

## Evaluation signals

- Every data line splits into exactly 7 tab-separated fields with no header row when read via read.delim(sep='\t', header=FALSE)
- Column 5 values are restricted to Inversion, Translocation, Deletion or Duplication; column 6 values come only from the seven allowed phenotype terms
- For deletions, inversions and duplications, breakpoint 1 coordinates are strictly smaller than breakpoint 2 coordinates on the same chromosome
- All coordinates are hg19 and low-resolution breakpoints use comma-separated ranges (e.g. 85092268,85092269)
- Each SV id in column 7 is unique, and POSTRE_multiSV() runs without format errors, returning both the per-SV/per-phenotype and per-gene/per-phenotype result tables

## Limitations

- Only seven phenotype categories are supported (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary); SVs outside these disease areas cannot be annotated
- Bulk tissue genomic data can miss fine-scale enhancers — e.g. POSTRE fails on the SHH ZRS duplication because whole limb bud data cannot identify the ZRS enhancer
- The High-Specificity running mode requires causative genes to have broad polycomb domains, so genes like MEF2C and SLC2A1 drop sensitivity (66% vs 95% in Standard mode)
- POSTRE is oriented towards dominant disorders; its capacity for recessive phenotypes has not been specifically evaluated

## Evidence
