---
name: cell-taxonomy-reference-database-construction
description: Use when a cluster-based cell annotation tool (e.g. scCATCH) needs a
  tissue-specific reference of marker genes linked to cell types, subtypes, tissues,
  and species, or when the bundled cellmatch data must be rebuilt or extended.
when_to_use_negative:
- Input is already an annotated single-cell dataset needing annotation — use scCATCH's
  findmarkergene()/findcelltype() workflow instead, supplying this database as reference
- Markers come from a species other than human or mouse without normalization support
  — the reconstructed database as described covers human and mouse references only
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scCATCH
  role: R package whose bundled cellmatch data defines the target schema; the reconstructed
    table is supplied as a custom cellmatch object for findcelltype() annotation
  repo: https://github.com/ZJUFanLab/scCATCH
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:f5b41f6c12cc00258c30eaa73e3cbcfee19e6b328b97e9628048a7c458884bdd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZJUFanLab/scCATCH
  repo_url: https://github.com/ZJUFanLab/scCATCH
  tools:
  - scCATCH
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cell-taxonomy-reference-database-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct a tissue-specific cell taxonomy reference database (CellMatch) by aggregating literature-reported, tissue-specific cell marker entries for human and mouse from public marker resources (CellMarker, MCA, CancerSEA, CD Marker Handbook) into a unified marker-to-cell-type table usable by scCATCH.

## When to use

Use when a cluster-based cell annotation tool (e.g. scCATCH) needs a tissue-specific reference of marker genes linked to cell types, subtypes, tissues, and species, or when the bundled cellmatch data must be rebuilt or extended.

## When NOT to use

- Input is already an annotated single-cell dataset needing annotation — use scCATCH's findmarkergene()/findcelltype() workflow instead, supplying this database as reference
- Markers come from a species other than human or mouse without normalization support — the reconstructed database as described covers human and mouse references only

## Inputs

- Literature-reported tissue-specific cell marker entries from CellMarker, MCA (Mouse Cell Atlas), CancerSEA, and the CD Marker Handbook
- scCATCH bundled cellmatch data object (schema reference)
- Species scope: human and mouse

## Outputs

- cellmatch.csv — unified table linking marker genes to cell type, subtype, tissue, species, and supporting reference
- Custom cellmatch object loadable by scCATCH for findcelltype() annotation

## How to apply

Install scCATCH from CRAN or via devtools::install_github('ZJUFanLab/scCATCH') and inspect the bundled cellmatch object to confirm the expected schema (marker gene, cell type, subtype, tissue, species, reference). Harvest literature-reported, tissue-specific cell marker entries for human and mouse from CellMarker, MCA, CancerSEA, and the CD Marker Handbook. Normalize the harvested entries into one unified table linking each marker gene to its cell type, subtype, tissue, species (human or mouse), and supporting reference. Count distinct cell types, subtypes, tissue types, and references and compare against the reported scale of 353 cell types, 686 subtypes, 184 tissue types, and ~2,096 references. Write the verified table to cellmatch.csv so it can be supplied to scCATCH as a custom cellmatch object.

## Related tools

- **scCATCH** (R package whose bundled cellmatch data defines the target schema; the reconstructed table is supplied as a custom cellmatch object for findcelltype() annotation) — https://github.com/ZJUFanLab/scCATCH

## Examples

```
install.packages('scCATCH'); data(cellmatch); obj <- findcelltype(obj, cellmatch = custom_cellmatch)
```

## Evaluation signals

- Distinct cell type count in the assembled table matches the reported panel of 353 cell types
- Distinct subtype count matches 686 subtypes
- Distinct tissue type count matches 184 tissue types
- Reference count is consistent with the reported 2,096 human and mouse references
- Table schema matches the bundled cellmatch object (marker gene, cell type, subtype, tissue, species, reference columns) and loads via scCATCH's custom-cellmatch path

## Limitations

- scCATCH performance majorly depends on the reference database; a limited reference database can lead to incorrect annotation of cell types
- No accession IDs or hosting URLs for the CellMatch database are given in the article discussion section, so provenance must be traced via the GitHub data directory
- Coverage is restricted to human and mouse references in the reported database

## Evidence

- [methods] CellMatch includes a panel of 353 cell types and related 686 subtypes associated with 184 tissue types, and 2,096 references of human and mouse.: "CellMatch includes a panel of 353 cell types and related 686 subtypes associated with 184 tissue types, and 2,096 references of human and mouse."
- [intro] Construction of tissue-specific cell taxonomy reference databases (CellMatch) with tissue-specific cell markers reported in the literature from humans or mice.: "Construction of tissue-specific cell taxonomy reference databases (CellMatch) with tissue-specific cell markers reported in the literature from humans or mice."
- [readme] Allow users to use custom `cellmatch`: "__Allow users to [use custom `cellmatch`](https://github.com/ZJUFanLab/scCATCH/wiki/Use-custom-cellmatch)__"
- [methods] devtools::install_github('ZJUFanLab/scCATCH'): "devtools::install_github('ZJUFanLab/scCATCH')"
- [discussion] The performance of scCATCH majorly depends on the reference database. The limited reference database potentially led to incorrect annotation of cell types via scCATCH: "The performance of scCATCH majorly depends on the reference database. The limited reference database potentially led to incorrect annotation of cell types via scCATCH"
