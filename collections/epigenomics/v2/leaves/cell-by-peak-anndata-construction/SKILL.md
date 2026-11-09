---
name: cell-by-peak-anndata-construction
description: Use when single-cell calling cards qbed insertions and called peaks (e.g.,
  902 Brd4 peaks from mouse cortex GSE148448) must be converted into a cell-by-peak
  count matrix for integration with Scanpy gene-expression AnnData.
when_to_use_negative:
- Input is already a cell-by-feature count matrix (AnnData/h5ad) rather than raw qbed
  insertion records
- Analysis targets bulk calling cards data rather than single-cell qbed data with
  per-cell barcodes
- Peaks have not yet been called and annotated — call_peaks and annotation must precede
  matrix construction
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0769
tools:
- name: pycallingcards
  role: Provides cc.pp.make_Anndata and cc.pp.filter_peaks to build and filter the
    cell-by-peak AnnData from qbed insertions and called peaks
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Scanpy
  role: Supplies the gene-expression AnnData structure that the peak AnnData is combined
    with; Pycallingcards follows Scanpy's structure for single-cell analysis
  repo: https://scanpy.readthedocs.io/en/stable/
- name: Mudata
  role: Combines the two AnnData objects (peak insertion modality and gene-expression
    modality) into one MuData object for joint analysis
- name: bedtools
  role: Backs cc.pp.annotation so each called peak is annotated with nearest genes
    before AnnData construction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:6e61e6051efd8c24c2537c26d32d1477caf1270c0a6b6472b6ecc5b4570e6b22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - Mudata
  - Scanpy
  - bedtools
  - pycallingcards
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cell-by-peak-anndata-construction

## Summary

Constructs a cell-by-peak AnnData matrix from single-cell calling cards qbed insertions and called Brd4 peaks using pycallingcards.preprocessing.make_Anndata, enabling downstream integration with scRNA-seq expression data in a MuData object for differential peak and peak–gene pairing analyses.

## When to use

Use when single-cell calling cards qbed insertions and called peaks (e.g., 902 Brd4 peaks from mouse cortex GSE148448) must be converted into a cell-by-peak count matrix for integration with Scanpy gene-expression AnnData.

## When NOT to use

- Input is already a cell-by-feature count matrix (AnnData/h5ad) rather than raw qbed insertion records
- Analysis targets bulk calling cards data rather than single-cell qbed data with per-cell barcodes
- Peaks have not yet been called and annotated — call_peaks and annotation must precede matrix construction

## Inputs

- qbed insertion file for single-cell calling cards data (e.g., Brd4 mouse cortex from GEO GSE148448)
- called peak BED file from cc.pp.call_peaks with gene annotations from cc.pp.annotation
- Scanpy gene-expression AnnData with matching cell barcodes (scRNA-seq from the same experiment)

## Outputs

- Cell-by-peak AnnData object (cells × peaks counts) from cc.pp.make_Anndata
- Filtered peak AnnData after cc.pp.filter_peaks
- Combined MuData object containing both insertion-peak and gene-expression modalities

## How to apply

After calling peaks with cc.pp.call_peaks (CCcaller) and annotating each peak with its two nearest genes via cc.pp.annotation (backed by bedtools/pybedtools), build the cell-by-peak AnnData with cc.pp.make_Anndata, which quantizes each cell's qbed insertions falling within called peaks. Optionally apply cc.pp.filter_peaks to the resulting AnnData to remove low-signal peaks. Then combine the peak AnnData with the Scanpy-processed gene-expression AnnData (normalized, Leiden/Louvain-clustered cortical cell types) into a single MuData object so that cell-type-specific binding (cc.tl.rank_peak_groups) and peak–gene pairing (cc.tl.pair_peak_gene_sc) operate on matched cell identifiers. In the reanalysis this yielded ~902 Brd4 peaks over 111,382 insertions in 35,950 cells and recovered the Mef2c and E130114P18Rik binding–expression pairs.

## Related tools

- **pycallingcards** (Provides cc.pp.make_Anndata and cc.pp.filter_peaks to build and filter the cell-by-peak AnnData from qbed insertions and called peaks) — https://github.com/The-Mitra-Lab/pycallingcards
- **Scanpy** (Supplies the gene-expression AnnData structure that the peak AnnData is combined with; Pycallingcards follows Scanpy's structure for single-cell analysis) — https://scanpy.readthedocs.io/en/stable/
- **Mudata** (Combines the two AnnData objects (peak insertion modality and gene-expression modality) into one MuData object for joint analysis)
- **bedtools** (Backs cc.pp.annotation so each called peak is annotated with nearest genes before AnnData construction)

## Examples

```
import pycallingcards as cc; adata_peak = cc.pp.make_Anndata(qbed_data, peak_data, ref); adata_peak = cc.pp.filter_peaks(adata_peak)
```

## Evaluation signals

- The cell-by-peak AnnData has dimensions cells × called peaks (e.g., 35,950 cells × ~902 Brd4 peaks for the mouse cortex reanalysis)
- Cell barcodes in the peak AnnData match those in the gene-expression AnnData before MuData combination
- After cc.pp.filter_peaks, low-count peaks are removed without losing the reported cell-type-specific peaks
- Downstream cc.tl.pair_peak_gene_sc recovers the reported pairs: Mef2c at chr13:83141353-83148478 and E130114P18Rik at chr4:97575305-97588788

## Limitations

- bedtools/pybedtools must be installed for the preceding annotation step; matrix construction depends on annotated peaks
- The peak matrix is only as good as the upstream CCcaller peak call — an incorrect peak set propagates into the AnnData
- Peak-to-cell assignment depends on insertion coordinates falling within called peak intervals; qbed formatting issues will silently produce zero-count cells

## Evidence

- [intro] Preprocessing (pp) helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks."
- [methods] Two Anndata objects can combine into one Mudata object.: "Two Anndata objects can combine into one Mudata object."
- [readme] For single-cell calling cards anaysis, Pycallingcards interacts with Scanpy and the main structure of Pycallingcards also follows the Scanpy.: "For single-cell calling cards anaysis, Pycallingcards interacts with [Scanpy](https://scanpy.readthedocs.io/en/stable/) and the main structure of Pycallingcards also follows the Scanpy."
- [methods] In order to complete the annotation, bedtools (includes pybedtools) also needs to be installed.: "In order to complete the annotation, [bedtools](https://bedtools.readthedocs.io/en/latest/) (includes [pybedtools](https://daler.github.io/pybedtools/)) also needs to be installed."
