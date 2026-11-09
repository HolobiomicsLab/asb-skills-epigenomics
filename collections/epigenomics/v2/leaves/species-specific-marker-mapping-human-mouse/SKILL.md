---
name: species-specific-marker-mapping-human-mouse
description: Use when a single-cell cluster annotation or marker-lookup task requires
  a curated, tissue-specific marker reference restricted to human or mouse, or when
  rebuilding/extending a CellMatch-style cellmatch table.
when_to_use_negative:
- The analysis uses species other than human or mouse without first building a custom
  cellmatch (default reference covers only human and mouse)
- The input is raw scRNA-seq data requiring clustering — this skill covers the reference
  database, not marker gene identification (findmarkergene handles that)
- A limited or tissue-mismatched reference would be the only available option, since
  a limited reference database can lead to incorrect annotation
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: scCATCH
  role: R package whose bundled cellmatch data defines the target schema; accepts
    custom cellmatch tables for species-specific marker matching in findmarkergene()
    and findcelltype()
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Upstream source of scRNA-seq data and cluster identities; scCATCH object can
    be created directly from a Seurat object
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:cbb4dc36efa13105f1b9808aab08622d6359fcc4f68bed77501860a265089e0a
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
  - Seurat
  - scCATCH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# species-specific-marker-mapping-human-mouse

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct and apply a species-annotated (human/mouse) tissue-specific cell marker reference database, following CellMatch for scCATCH: aggregate literature-reported markers into a unified table linking each gene to cell type, subtype, tissue, species, and reference.

## When to use

Use when a single-cell cluster annotation or marker-lookup task requires a curated, tissue-specific marker reference restricted to human or mouse, or when rebuilding/extending a CellMatch-style cellmatch table.

## When NOT to use

- The analysis uses species other than human or mouse without first building a custom cellmatch (default reference covers only human and mouse)
- The input is raw scRNA-seq data requiring clustering — this skill covers the reference database, not marker gene identification (findmarkergene handles that)
- A limited or tissue-mismatched reference would be the only available option, since a limited reference database can lead to incorrect annotation

## Inputs

- Literature-reported tissue-specific cell marker entries from CellMarker, MCA, CancerSEA, and the CD Marker Handbook
- Species label (human or mouse) per marker entry
- Bundled scCATCH cellmatch data object (schema reference)
- cellmatch.csv custom reference table

## Outputs

- Unified marker-gene table linking gene to cell type, subtype, tissue, species, and supporting reference
- cellmatch.csv compatible with scCATCH as a custom cellmatch object
- Counts of distinct cell types, subtypes, tissue types, and references for verification

## How to apply

Harvest literature-reported tissue-specific cell markers for human and mouse from public resources (CellMarker, MCA, CancerSEA, and the CD Marker Handbook), then normalize entries into a unified table linking each marker gene to its cell type, subtype, tissue, species, and supporting reference. Verify the assembled table against the reported scale — 353 cell types, 686 subtypes, 184 tissue types, and 2,096 human and mouse references — before use. Install scCATCH from CRAN or via devtools::install_github('ZJUFanLab/scCATCH') and inspect its bundled cellmatch data object to confirm the expected schema, then write the verified table to cellmatch.csv in a format acceptable as a custom cellmatch object supplied to findmarkergene()/findcelltype(). Species (human or mouse) must be recorded per entry so findmarkergene(obj, species, marker, tissue, cancer) can match cluster markers against the correct species-specific markers.

## Related tools

- **scCATCH** (R package whose bundled cellmatch data defines the target schema; accepts custom cellmatch tables for species-specific marker matching in findmarkergene() and findcelltype()) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Upstream source of scRNA-seq data and cluster identities; scCATCH object can be created directly from a Seurat object)

## Examples

```
install.packages('scCATCH'); data(cellmatch); write.csv(cellmatch, 'cellmatch.csv')  # rebuild custom reference, then: obj <- findmarkergene(obj, species = 'Human', marker = 'cellmarker', tissue = 'BoneMarrow', cancer = 'cancer')
```

## Evaluation signals

- Counts of distinct cell types, subtypes, tissue types, and references in the assembled table match the reported scale (353 cell types, 686 subtypes, 184 tissue types, ~2,096 references)
- Every row in cellmatch.csv carries a valid species label limited to human or mouse
- The custom cellmatch loads into scCATCH without schema errors and is accepted by findmarkergene(obj, species, marker, tissue, cancer)
- Marker-to-cell-type assignments are traceable to a supporting literature reference for each entry
- Source resources (CellMarker, MCA, CancerSEA, CD Marker Handbook) are all represented in the aggregated table

## Limitations

- The performance of scCATCH depends on the reference database; a limited reference database can lead to incorrect annotation of cell types
- The default reference covers only human and mouse; other species require the custom cellmatch workflow
- Markers are literature-reported and inherited from the source resources, so coverage reflects those databases' biases

## Evidence
