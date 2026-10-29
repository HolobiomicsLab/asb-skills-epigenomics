---
name: cell-type-specific-differential-peak-analysis
description: Use when single-cell CC/qbed insertion data (e.g., Brd4 mouse cortex,
  GSE148448) must be converted to a cell-by-peak AnnData and differential TF binding
  peaks per cell type must be linked to nearby gene expression.
when_to_use_negative:
- Input is already a cell-by-peak count matrix — cc.pp.call_peaks and cc.pp.make_Anndata
  operate on qbed insertion records, not feature tables
- The goal is bulk (non-single-cell) calling cards peak calling without cell-type
  stratification — a simpler CCcaller/MACCs bulk workflow suffices
- The task is motif discovery or ChIP-seq comparison only; use pycallingcards' Tools
  (tl) motif calling or plotting functions instead of the differential peak–gene pairing
  pipeline
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0203
tools:
- name: pycallingcards
  role: core module providing reading (rd), preprocessing (pp), tools (tl) and plotting
    (pl) for qbed-to-peak, annotation, AnnData construction, differential peaks and
    peak–gene pairing
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: CCcaller
  role: peak caller used via cc.pp.call_peaks to pinpoint TF binding sites from single-cell
    CC insertions
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Scanpy
  role: clusters the matched scRNA-seq cells to define cortical cell types; pycallingcards
    follows Scanpy structure for single-cell CC analysis
  repo: https://scanpy.readthedocs.io/en/stable/
- name: bedtools
  role: with pybedtools, backs cc.pp.annotation to assign nearest genes to called
    peaks
  repo: https://bedtools.readthedocs.io/en/latest/
- name: Mudata
  role: combines the peak AnnData and gene-expression AnnData into one MuData object
    for joint analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:6b4683d398e327fd34c74109ec140adf4f59ecbab1ac5713d7f2c4d5faf55b6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - CCcaller
  - Mudata
  - Scanpy
  - bedtools
  - pycallingcards
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cell-type-specific-differential-peak-analysis

## Summary

A Pycallingcards-based workflow for detecting cell-type-specific transcription factor binding peaks from single-cell calling cards (CC) data and pairing differential peaks with nearby gene expression to generate binding–expression hypotheses.

## When to use

Use when single-cell CC/qbed insertion data (e.g., Brd4 mouse cortex, GSE148448) must be converted to a cell-by-peak AnnData and differential TF binding peaks per cell type must be linked to nearby gene expression.

## When NOT to use

- Input is already a cell-by-peak count matrix — cc.pp.call_peaks and cc.pp.make_Anndata operate on qbed insertion records, not feature tables
- The goal is bulk (non-single-cell) calling cards peak calling without cell-type stratification — a simpler CCcaller/MACCs bulk workflow suffices
- The task is motif discovery or ChIP-seq comparison only; use pycallingcards' Tools (tl) motif calling or plotting functions instead of the differential peak–gene pairing pipeline

## Inputs

- single-cell CC qbed insertion file (e.g., mouse cortex Brd4 qbed from GEO GSE148448; 111,382 insertions across 35,950 cells)
- matched scRNA-seq expression AnnData for the same cells
- genome annotation for peak-to-gene annotation (via bedtools/pybedtools)

## Outputs

- called TF binding peak set (e.g., ~902 Brd4 peaks in the mouse cortex reanalysis)
- annotated peak table with two nearest genes per peak
- cell-by-peak AnnData, optionally combined with expression AnnData into a MuData object
- differential peak ranking per cell type (cc.tl.rank_peak_groups)
- peak–gene pairing table from cc.tl.pair_peak_gene_sc linking cell-type-specific peaks to nearby gene expression

## How to apply

Load the single-cell qbed insertion data with pycallingcards.reading.read_qbed (e.g., mouse cortex Brd4 data from GEO accession GSE148448) and the matched scRNA-seq data, clustering cells with Scanpy (standard normalization and Leiden/Louvain) to define cell types. Call TF binding peaks with cc.pp.call_peaks (CCcaller), annotate each peak with its two nearest genes via cc.pp.annotation (backed by bedtools/pybedtools), and build a cell-by-peak matrix with cc.pp.make_Anndata; optionally filter peaks with cc.pp.filter_peaks. Combine the peak AnnData with the gene-expression AnnData into one MuData object. Run differential peak analysis per cell type with cc.tl.rank_peak_groups, then pair cell-type-specific peaks with nearby gene expression using cc.tl.pair_peak_gene_sc to surface candidate binding–expression relationships.

## Related tools

- **pycallingcards** (core module providing reading (rd), preprocessing (pp), tools (tl) and plotting (pl) for qbed-to-peak, annotation, AnnData construction, differential peaks and peak–gene pairing) — https://github.com/The-Mitra-Lab/pycallingcards
- **CCcaller** (peak caller used via cc.pp.call_peaks to pinpoint TF binding sites from single-cell CC insertions) — https://github.com/The-Mitra-Lab/pycallingcards
- **Scanpy** (clusters the matched scRNA-seq cells to define cortical cell types; pycallingcards follows Scanpy structure for single-cell CC analysis) — https://scanpy.readthedocs.io/en/stable/
- **bedtools** (with pybedtools, backs cc.pp.annotation to assign nearest genes to called peaks) — https://bedtools.readthedocs.io/en/latest/
- **Mudata** (combines the peak AnnData and gene-expression AnnData into one MuData object for joint analysis)

## Examples

```
import pycallingcards as cc; qbed = cc.rd.read_qbed('GSE148448_brd4.qbed'); peaks = cc.pp.call_peaks(qbed, method='CCcaller'); peak_adata = cc.pp.make_Anndata(qbed, peaks); cc.tl.rank_peak_groups(peak_adata, 'cell_type'); cc.tl.pair_peak_gene_sc(mudata)
```

## Evaluation signals

- Called peak set should reproduce the reported ~902 peaks for the Brd4 mouse cortex scCC reanalysis
- The peak–gene pairing table should recover Mef2c paired with peak chr13:83141353-83148478 (highest binding in excitatory neurons)
- The pairing table should recover E130114P18Rik paired with peak chr4:97575305-97588788, with strongest binding in astrocytes and neuroblasts
- Each annotated peak should carry its two nearest genes from the bedtools/pybedtools annotation step
- Cell types with strongest binding at a differential peak should show correspondingly higher expression of the paired nearby gene

## Limitations

- Reproducibility of the exact peak count depends on matching the reported parameters; the article targets the reported set of 902 peaks for the mouse cortex data
- bedtools (including pybedtools) must be installed for annotation to complete — annotation will fail without it
- Peak–gene pairing yields potential hypotheses for TF binding–gene expression relationships, not confirmed regulatory mechanisms

## Evidence

- [intro] For single-cell calling cards anaysis, Pycallingcards interacts with Scanpy and the main structure of Pycallingcards also follows the Scanpy.: "For single-cell calling cards anaysis, Pycallingcards interacts with [Scanpy](https://scanpy.readthedocs.io/en/stable/) and the main structure of Pycallingcards also follows the Scanpy."
- [intro] Preprocessing (pp) helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks."
- [methods] Differential peak analysis is then followed and pair analysis is completed to find out potential hypothesis: "Differential peak analysis is then followed and pair analysis is completed to find out potential hypothesis"
- [methods] Two Anndata objects can combine into one Mudata object.: "Two Anndata objects can combine into one Mudata object."
- [methods] In order to complete the annotation, bedtools (includes pybedtools) also needs to be installed.: "In order to complete the annotation, [bedtools](https://bedtools.readthedocs.io/en/latest/) (includes [pybedtools](https://daler.github.io/pybedtools/)) also needs to be installed."
- [abstract] This analysis revealed novel cell-type-specific binding sites and potential sex-linked TF regulators: "This analysis revealed novel cell-type-specific binding sites and potential sex-linked TF regulators, furthering our understanding of TF binding and gene expression relationships."
