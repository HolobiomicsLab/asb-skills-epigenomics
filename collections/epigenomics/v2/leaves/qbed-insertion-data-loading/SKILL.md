---
name: qbed-insertion-data-loading
description: Use when the input is raw Calling Cards sequencing output as qbed/ccf
  insertion files (e.g. from GEO accessions like GSE148448) or a built-in pycallingcards
  dataset, and peaks must be called.
when_to_use_negative:
- Input is already a called-peaks file or annotated peak BED — the Reading module's
  peak I/O functions handle those directly, no qbed loading needed.
- Data are bulk ChIP-seq reads rather than Calling Cards transposon insertions; qbed
  loading assumes insertion-coordinate records.
- Downstream goal is differential peak or motif analysis on an existing Anndata object
  that already contains insertion counts.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0749
tools:
- name: pycallingcards
  role: Python module whose Reading (rd) module reads/saves qbed and peak data and
    whose datasets module provides built-in Calling Cards datasets
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Scanpy
  role: Interop target for single-cell Calling Cards data; Pycallingcards follows
    Scanpy's structure and Anndata object model
  repo: https://scanpy.readthedocs.io/en/stable/
- name: bedtools
  role: Required dependency (including pybedtools) for the annotation step that follows
    qbed loading
  repo: https://bedtools.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:b2a9ef0f17b19b30fa927f565f7ca65f1265e8fdc36c78ead9859b189ddb1c58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - MACCs
  - Pycallingcards
  - Python
  - Scanpy
  - bedtools
  - pycallingcards
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# qbed-insertion-data-loading

## Summary

Load Calling Cards transposon insertion data in qbed (or ccf) format into pycallingcards dataframes/objects so downstream peak calling (CCcaller, MACCs), annotation, and Anndata construction can proceed. This is the entry point of the Pycallingcards preprocessing pipeline.

## When to use

Use when the input is raw Calling Cards sequencing output as qbed/ccf insertion files (e.g. from GEO accessions like GSE148448) or a built-in pycallingcards dataset, and peaks must be called.

## When NOT to use

- Input is already a called-peaks file or annotated peak BED — the Reading module's peak I/O functions handle those directly, no qbed loading needed.
- Data are bulk ChIP-seq reads rather than Calling Cards transposon insertions; qbed loading assumes insertion-coordinate records.
- Downstream goal is differential peak or motif analysis on an existing Anndata object that already contains insertion counts.

## Inputs

- qbed insertion file (bed-like format with chromosome, start, end per insertion)
- ccf Calling Cards fragment file
- built-in pycallingcards datasets (e.g. mousecortex_data, mouse_brd4_data, SP1_K562HCT116_data, SP1_Cre_data)
- GEO accession data (GSE148448, GSE156821, GSE128493, GSE134427)

## Outputs

- pandas-style qbed dataframe of insertions consumed by pycallingcards preprocessing (pp) functions
- optional saved qbed/peak files via the Reading (rd) module's save functions
- Anndata/Mudata objects for single-cell Calling Cards analysis (after downstream pp steps)

## How to apply

Read the qbed/ccf insertion file with pycallingcards' Reading (rd) module, which provides functions to read and save qbed and peak data; alternatively pull a built-in dataset via the datasets module (e.g. datasets.mousecortex_data, datasets.mouse_brd4_data, datasets.SP1_K562HCT116_data, datasets.SP1_Cre_data, or download from GEO accessions GSE148448, GSE156821, GSE128493, GSE134427). The loaded insertion dataframe has one row per transposon insertion with chromosome coordinates, ready for preprocessing.clean_qbed and preprocessing.call_peaks. For single-cell Calling Cards data, structure the result to interoperate with Scanpy-style Anndata objects (two Anndata objects can be combined into one Mudata object). Rationale: qbed loading is the first step before CCcaller greedy peak calling or MACCs window-scan calling can traverse insertions per chromosome.

## Related tools

- **pycallingcards** (Python module whose Reading (rd) module reads/saves qbed and peak data and whose datasets module provides built-in Calling Cards datasets) — https://github.com/The-Mitra-Lab/pycallingcards
- **Scanpy** (Interop target for single-cell Calling Cards data; Pycallingcards follows Scanpy's structure and Anndata object model) — https://scanpy.readthedocs.io/en/stable/
- **bedtools** (Required dependency (including pybedtools) for the annotation step that follows qbed loading) — https://bedtools.readthedocs.io/en/latest/

## Examples

```
import pycallingcards as cc; qbed = cc.datasets.mouse_brd4_data()
```

## Evaluation signals

- Loaded qbed dataframe contains one row per insertion with chromosome coordinate fields consistent with qbed format.
- Built-in dataset loaders (e.g. datasets.mouse_brd4_data) return the expected published dataset from GSE148448 and related accessions without errors.
- The loaded dataframe can be passed directly to preprocessing.clean_qbed and preprocessing.call_peaks without schema errors.
- For single-cell data, insertion counts can be assembled into an Anndata object (or combined into Mudata) compatible with Scanpy.

## Limitations

- Annotation downstream of loading requires bedtools/pybedtools to be installed separately.
- The four built-in datasets cover only the published mouse cortex, glioblastoma, and SP1/BRD4 K562/HCT116 experiments; other data must be fetched from GEO manually.

## Evidence

- [intro] **Reading (rd)** includes several functions to read and save qbed and peak data.: "Reading (rd) includes several functions to read and save qbed and peak data."
- [methods] it can be downloaded from [GEO](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE148448).: "it can be downloaded from [GEO](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE148448)."
- [readme] Pycallingcards is composed of five different part: datasets, reading (rd), preprocessing (pp), tools (tl) and plotting (pl).: "Pycallingcards is composed of five different part: datasets, reading (rd), preprocessing (pp), tools (tl) and plotting (pl)."
- [methods] For single-cell calling cards anaysis, Pycallingcards interacts with [Scanpy](https://scanpy.readthedocs.io/en/stable/): "For single-cell calling cards anaysis, Pycallingcards interacts with [Scanpy](https://scanpy.readthedocs.io/en/stable/)"
