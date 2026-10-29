---
name: literature-marker-gene-curation
description: Use when building or extending a cell taxonomy reference database of
  literature-curated marker genes (like CellMatch) to support automatic cluster annotation
  in scRNA-seq pipelines such as scCATCH.
when_to_use_negative:
- When a tissue-specific, literature-curated reference such as CellMatch already covers
  your tissue — extend or use it rather than rebuilding from scratch
- When markers come from a species other than human or mouse without a supported conversion
  path — the base CellMatch is human and mouse only (custom-species support requires
  the scCATCH wiki workflow)
- When the task is de novo marker identification from expression data (e.g., one-against-all
  Wilcoxon testing) rather than curation from published knowledge
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scCATCH
  role: R package whose bundled cellmatch data object defines the reference schema;
    the curated table is supplied to scCATCH for tissue-specific cell type annotation
    via findmarkergene() and findcelltype()
  repo: https://github.com/ZJUFanLab/scCATCH
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:baae7d33cc6bad5db330b2c81b20627a98d62dae249f4dd293f8e6164a4c8b24
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
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# literature-marker-gene-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curate literature-reported, tissue-specific cell marker genes for human and mouse into a unified reference table (e.g., scCATCH's CellMatch) that links each marker gene to cell type, subtype, tissue, species, and supporting reference, enabling evidence-based automatic cell type annotation.

## When to use

Use when building or extending a cell taxonomy reference database of literature-curated marker genes (like CellMatch) to support automatic cluster annotation in scRNA-seq pipelines such as scCATCH.

## When NOT to use

- When a tissue-specific, literature-curated reference such as CellMatch already covers your tissue — extend or use it rather than rebuilding from scratch
- When markers come from a species other than human or mouse without a supported conversion path — the base CellMatch is human and mouse only (custom-species support requires the scCATCH wiki workflow)
- When the task is de novo marker identification from expression data (e.g., one-against-all Wilcoxon testing) rather than curation from published knowledge

## Inputs

- Public marker resource entries from CellMarker, MCA, CancerSEA, and the CD Marker Handbook
- Literature-reported tissue-specific cell markers for human and mouse
- Bundled cellmatch data object from the scCATCH R package (schema reference)

## Outputs

- cellmatch.csv — unified table of marker genes linked to cell type, subtype, tissue, species, and reference
- Custom cellmatch object loadable by scCATCH for annotation

## How to apply

Harvest literature-reported, tissue-specific cell marker entries for human and mouse from public marker resources — CellMarker, MCA (Mouse Cell Atlas), CancerSEA, and the CD Marker Handbook — then normalize them into a unified table linking each marker gene to its cell type, subtype, tissue, species, and supporting reference. Aim for the scale reported for CellMatch: 353 cell types, 686 subtypes, 184 tissue types, 20,792 cell-specific marker genes, and ~2,096 human and mouse references; count distinct entities to verify completeness. Inspect the bundled `cellmatch` data object from the scCATCH R package (installed from CRAN or via devtools::install_github('ZJUFanLab/scCATCH')) to confirm the expected schema, and write the verified table as cellmatch.csv so it can be supplied to scCATCH as a custom cellmatch object. The rationale is that scCATCH's findcelltype() step scores cluster marker genes against this tissue-specific reference, so annotation quality directly depends on reference breadth — a limited reference database can lead to incorrect annotation.

## Related tools

- **scCATCH** (R package whose bundled cellmatch data object defines the reference schema; the curated table is supplied to scCATCH for tissue-specific cell type annotation via findmarkergene() and findcelltype()) — https://github.com/ZJUFanLab/scCATCH

## Examples

```
install.packages('scCATCH'); data(cellmatch); # verify schema, then supply curated table via findmarkergene(obj, species='Human', marker=cellmatch, tissue=...)
```

## Evaluation signals

- Distinct cell type, subtype, and tissue counts in the assembled table match the reported scale (353 cell types, 686 subtypes, 184 tissue types)
- Total cell-specific marker genes and human/mouse references are consistent with the reported 20,792 markers and ~2,096 references
- Table schema matches the bundled cellmatch object expected by scCATCH (gene, cell type, subtype, tissue, species, reference columns)
- Written cellmatch.csv loads successfully as a custom cellmatch input to scCATCH without schema errors
- Downstream findcelltype() annotation accuracy is not degraded versus the bundled database on a benchmark tissue

## Limitations

- The performance of scCATCH majorly depends on the reference database; a limited reference database can lead to incorrect annotation of cell types
- No accession IDs or hosting URLs for CellMatch were given in the article's discussion section; the database is distributed via the scCATCH GitHub data folder
- Applying CellMatch to other methods (SingleR, CellAssign) improves accuracy, but coverage is still restricted to literature-reported human and mouse markers

## Evidence
